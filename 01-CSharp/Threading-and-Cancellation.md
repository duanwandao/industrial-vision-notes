# C# 线程、线程池、WinForms UI 封送与协作式取消

> **P0｜待学习 / 待验证｜建立于 2026-11-06；更新至 2026-11-14**
>
> 面向 .NET 8 / C# 12 WinForms 工业上位机。本文是概念与模拟代码笔记；没有连接真实相机或 PLC。真实 SDK 的回调线程、采集停止、缓冲区所有权和释放规则必须以具体厂商/版本文档为准。

## 1. 先建立正确的心智模型

| 概念 | 是什么 | 不要误解成 |
|---|---|---|
| Thread（线程） | 操作系统调度执行的一条路径；托管线程由 .NET 管理 | 一个业务任务 |
| Task | 一项操作及其完成状态、结果或异常的抽象 | 一个专属线程 |
| ThreadPool（线程池） | .NET 管理并复用的一组工作线程及其调度机制 | 每个 Task 都有一个固定线程 |
| async | 允许方法使用 await；方法会先同步执行，遇到未完成的 await 才挂起后续 | 自动创建后台线程 |
| await | 等待可等待操作完成；操作未完成时让当前方法暂时返回 | 阻塞当前线程 |
| CancellationToken | 取消通知/请求的只读令牌 | 强制杀线程的命令 |

Task 可以由线程池执行（例如 Task.Run），也可以代表不需要专门占用线程等待的异步 I/O、Task.Delay，或已经完成的结果（Task.FromResult）。因此 Task 与 Thread 不是一一对应关系。

## 2. WinForms UI 线程与消息循环

### 2.1 消息循环做什么

Windows 会向窗口所属线程投递窗口消息，例如鼠标按下、键盘输入、窗口重绘、大小变化等。WinForms 的 Application.Run(form) 启动窗体消息循环：UI 线程持续获取并分发消息，框架再调用对应控件/窗体的事件处理逻辑。

常见消息示例：

| 消息 | 含义 |
|---|---|
| WM_PAINT | 窗口需要绘制 |
| WM_MOUSEMOVE | 鼠标移动 |
| WM_LBUTTONDOWN | 鼠标左键按下 |
| WM_KEYDOWN | 键盘按键按下 |
| WM_SIZE | 窗口大小改变 |
| WM_CLOSE | 请求关闭窗口 |

消息循环不是“不断创建线程”，而是 UI 线程持续处理自己的消息队列。

### 2.2 HWND、Control.Handle 与窗口过程

WinForms 的 Control 是 .NET 托管对象；HWND（窗口句柄）是 Windows 原生窗口对象的标识。二者相关，但不是同一个对象。

- Control.Handle 的类型是 IntPtr，表示对应的原生窗口句柄。
- 控件对象可以先被构造出来，底层 HWND 则可能延迟创建。
- IsHandleCreated 只检查句柄是否已创建；读取 Handle 属性本身可能触发句柄创建，因此不要用读取 Handle 的方式来“单纯检查”句柄状态。
- HandleCreated / HandleDestroyed 可观察句柄创建和销毁；某些操作可能导致句柄重建，不能把某个 HWND 值当作控件整个生命周期内永远有效的标识。
- 不要为了调用 Invoke / BeginInvoke 而从工作线程主动读取 Handle 来触发创建。应先在 UI 线程完成窗体/控件初始化，再启动后台工作。

标准 WinForms 启动代码中的 Application.Run(form) 在调用它的当前线程上运行消息循环，不会额外创建一个 UI 线程。通常 Main 所在的主线程就是 UI 线程。

窗口过程（Window Procedure）是窗口接收并处理消息的入口。WinForms 可以通过重写 WndProc 观察或处理窗口消息：

~~~csharp
protected override void WndProc(ref Message m)
{
    const int WM_SIZE = 0x0005;

    if (m.Msg == WM_SIZE)
    {
        Console.WriteLine("窗口尺寸发生变化");
    }

    base.WndProc(ref m);
}
~~~

- m.Msg 是消息编号；Message 还包含 WParam、LParam 等消息参数。
- base.WndProc(ref m) 让 WinForms 基类继续执行默认处理。除非明确要拦截并接管某条消息，否则不要随意省略。
- GetMessage / TranslateMessage / DispatchMessage 是理解 Win32 消息循环的经典模型；WinForms 已经封装了消息循环，普通业务窗体不需要自己编写它。
- 常见输入和窗口消息会排队等待线程处理；同步发送的消息也可能直接进入窗口过程，因此不要把“所有消息都先进同一个队列”当成绝对规则。

### 2.3 为什么 UI 线程不能做长时间同步工作

按钮事件通常由 UI 线程处理。如果事件处理器直接执行 Thread.Sleep、耗时计算、同步网络调用或长时间阻塞式设备调用，UI 线程就不能及时处理后续消息，表现为窗口卡顿、无法重绘或“未响应”。

~~~csharp
private void btnStart_Click(object? sender, EventArgs e)
{
    Thread.Sleep(5000); // 如果在 UI 线程执行，界面会被阻塞约 5 秒
}
~~~

UI 线程被阻塞，不等于整个进程的所有线程都停止；后台线程仍可能工作，只是 UI 消息处理停滞。

## 3. InvokeRequired、Invoke 与 BeginInvoke

WinForms 控件通常由创建其窗口句柄的 UI 线程拥有。后台线程不能随意直接读写控件。

### 3.1 InvokeRequired：判断是否需要封送

~~~csharp
if (lblStatus.InvokeRequired)
{
    // 当前调用线程不是该控件句柄所属线程，需要封送
}
~~~

- 返回 true：通常表示需要切换到控件所属线程。
- 返回 false：当前线程不需要封送。

重要边界：如果控件及其父级句柄尚未创建，InvokeRequired 可能返回 false，即使当前代码并不在 UI 线程。因此要确保控件句柄在 UI 线程创建完成后，再从后台线程访问封送 API；不能把 InvokeRequired=false 无条件理解为“当前一定是 UI 线程”。

### 3.2 BeginInvoke：异步安排 UI 操作

~~~csharp
lblStatus.BeginInvoke(new Action(() =>
{
    lblStatus.Text = "图像采集完成";
}));
~~~

它把委托安排到拥有该控件句柄的线程执行，通常是 UI 线程。调用方提交后就可以继续，不等待委托执行完成。

BeginInvoke：
- 不创建新线程；
- 不会把当前后台线程“变成”UI 线程；
- 不会在调用它的后台线程直接执行委托；
- 依赖控件句柄和 WinForms 的封送/消息处理机制；
- 提交后委托可能稍后才执行，调用方继续执行与 UI 委托完成之间没有同步等待关系。

Action 是 .NET 内置委托类型，表示无参数、无返回值的方法。Lambda 表达式提供要执行的代码；new Action(...) 将 Lambda 转成 BeginInvoke 所需的委托。

### 3.3 Invoke：同步安排 UI 操作

~~~csharp
lblStatus.Invoke(new Action(() =>
{
    lblStatus.Text = "图像采集完成";
}));

Console.WriteLine("UI 委托执行完后，调用线程才继续");
~~~

Invoke 会等待 UI 线程执行委托完成后再返回。

| API | 执行位置 | 调用方是否等待委托完成 |
|---|---|---|
| InvokeRequired | 只做判断 | 不适用 |
| Invoke | 控件所属线程 | 是 |
| BeginInvoke | 控件所属线程 | 否，提交后立即返回 |

Invoke 可能导致死锁：例如 UI 线程用 Wait/Result 等待后台线程，而后台线程又调用 Invoke 等待 UI 线程；双方互相等待。BeginInvoke 不让后台调用方同步等待 UI 委托，但仍需考虑窗体关闭、句柄销毁和排队委托的生命周期。

BeginInvoke 提交成功不代表 UI 委托一定已经执行。若委托内部可能抛异常，应在委托内部处理；调用 BeginInvoke 外层的 try/catch 不一定能捕获稍后在 UI 线程执行委托时发生的异常。

### 3.4 可复用的 UI 封送辅助方法

~~~csharp
private void Ui(Action action)
{
    if (IsDisposed || !IsHandleCreated)
        return;

    try
    {
        if (InvokeRequired)
            BeginInvoke(action);
        else
            action();
    }
    catch (ObjectDisposedException)
    {
        // 窗体/控件在调度期间已释放
    }
    catch (InvalidOperationException)
    {
        // 句柄状态变化等导致无法调度
    }
}
~~~

这是教学用的简化封装，不是完整的窗体生命周期方案。检查状态与 BeginInvoke 之间仍可能发生关闭竞争；异步排队的委托也可能在窗体开始关闭后才到达。真实项目应在关闭时先请求停止并等待后台工作结束，再释放窗体/相机资源。不要仅靠吞掉异常来掩盖持续发生的生命周期错误。

### 3.5 BeginInvoke 与 Task.Run 不一样

- Task.Run：把工作交给线程池执行，常用于 CPU 密集型工作或包装必须放到后台执行的同步阻塞工作。
- BeginInvoke：把 UI 操作交给控件所属线程执行。
- BeginInvoke 不负责图像处理；Task.Run 也不会自动让后台线程有权修改控件。

.NET 9 起 WinForms 增加 Control.InvokeAsync；本仓库当前 .NET 8 项目应使用 Invoke/BeginInvoke 等 .NET 8 可用 API，不能直接复制 .NET 9 专属 API。

## 4. async / await、Task.Delay 与 Thread.Sleep

### 4.1 async 不会自动创建线程

~~~csharp
private async void btnTest_Click(object? sender, EventArgs e)
{
    Console.WriteLine(Environment.CurrentManagedThreadId); // UI 线程

    await Task.Run(() =>
    {
        Console.WriteLine(Environment.CurrentManagedThreadId); // 通常是线程池工作线程
        Thread.Sleep(1000); // 阻塞当前工作线程
    });

    Console.WriteLine(Environment.CurrentManagedThreadId); // WinForms UI 上下文中通常回到 UI 线程
}
~~~

WinForms UI 事件处理器中的 await 默认捕获当前 SynchronizationContext。若 await 的 Task 尚未完成，方法会挂起并把执行权交还消息循环；Task 完成后，后续通常被安排回捕获的 UI 上下文。因此 await 后通常可以更新控件。

这是有条件的：没有捕获 UI 上下文、使用 ConfigureAwait(false)、在没有 UI SynchronizationContext 的位置启动等情况，不能假设后续一定回 UI 线程。若 await 的任务已经完成，后续也可能直接同步继续执行。

### 4.2 Thread.Sleep 与 Task.Delay

~~~csharp
Thread.Sleep(500);       // 阻塞当前线程 500ms
await Task.Delay(500);   // 异步等待；不需要专门占用线程在这里睡眠
~~~

Task.Delay 通过计时/任务完成机制等待时间，不需要一个工作线程整个等待期都被占住；到期后继续执行后续逻辑。Thread.Sleep 则占用并阻塞当前线程。Task.Delay(ms, token) 还能响应取消请求。

### 4.3 CPU 密集型与 I/O 密集型

- CPU 密集型图像处理：可评估放到 Task.Run/线程池，避免占用 UI 线程；还要控制并发数量和图像队列长度。
- 原生异步 I/O：优先直接 await 对应的 ReadAsync、WriteAsync 等 API，不要为了“异步”再包一层 Task.Run。
- 同步阻塞式设备 SDK：是否使用 Task.Run、如何停止，必须看 SDK 的线程安全和停止 API；Task.Run 不能自动中断 SDK 内部阻塞调用。

## 5. ThreadPool（线程池）

ThreadPool 是 .NET Runtime 管理的一组可复用工作线程及其调度机制。把工作提交给线程池后，Runtime 会根据队列和负载安排工作线程执行；任务完成后，线程可以继续处理其他工作。

~~~csharp
Task task = Task.Run(() =>
{
    ProcessImage();
});
~~~

Task.Run 默认把委托提交给线程池调度。不要认为每调用一次 Task.Run 就创建一个专属新线程，也不要认为一个 Task 永远绑定同一个线程。异步委托 await 后恢复执行时，可能由不同的线程池线程继续。

线程池的价值是减少频繁创建/销毁线程的成本并统一调度，但线程池不是无限资源：
- Thread.Sleep 或长时间同步阻塞会占住工作线程；
- 无限制地为每帧图像 Task.Run 可能造成排队、延迟和内存增长；
- 采集 30 FPS、处理只能达到 20 FPS 时，必须设计有界队列、丢帧/背压策略或降低采集速率，而不是无限堆积任务；
- 真实相机 SDK 可能自带采集回调线程，不要假定每帧都应额外创建一个线程或 Task。

## 6. Lambda local、闭包与集合表达式

### 6.1 Lambda local 是什么

Visual Studio 调试器提示中的 “lambda local” 是调试器对变量位置的描述，不是 C# 关键字或特殊变量类型。它表示这个局部变量声明在 Lambda 表达式内部：

~~~csharp
Task.Run(() =>
{
    string name = "Tom"; // Lambda 内部的局部变量，可被调试器标为 lambda local
    Console.WriteLine(name);
});
~~~

如果 Lambda 使用外部方法中的局部变量，则会捕获该变量，形成闭包（Closure）：

~~~csharp
int count = 0;

Action increment = () =>
{
    count++; // 捕获外部局部变量 count
};
~~~

Lambda local（Lambda 内声明的局部变量）与 captured local（被 Lambda 捕获的外部局部变量）不是一回事。闭包可能延长被捕获变量的生命周期；多个线程共同访问被捕获变量时仍需考虑线程安全。

### 6.2 数组初始化与 C# 12 集合表达式

~~~csharp
string[] names =
{
    "Tom",
    "Jack",
    "Lucy"
};

string[] names2 = ["2222", "33333"]; // C# 12 集合表达式，需要目标类型
~~~

第一种是传统数组初始化器；第二种是 C# 12 集合表达式。它们在此处都创建 string[]。集合表达式不是 AddRange，也不是对象初始化器。

~~~csharp
cmbMode.Items.AddRange(new object[]
{
    "安全：Invoke 封送回 UI 线程",
    "危险：后台线程直接改控件"
});
~~~

Items 是 ComboBox 的项目集合；AddRange 批量添加数组中的项目。object[] 可以接收 string，因为 string 派生自 object。

## 7. 线程安全：Race Condition、Interlocked、lock 与 volatile

### 7.1 竞态条件与丢失更新

~~~csharp
private int _errorCount = 0;

// 两个线程都执行
_errorCount++;
~~~

++ 不是一个可依赖的原子读改写操作。可将其概念性拆成“读取旧值 → 加一 → 写回”。两个线程可能同时读到 0，分别算出 1，再分别写回 1，最终得到 1，而期望是 2。这是竞态条件（Race Condition），此例也叫 Lost Update（更新丢失）。

### 7.2 Interlocked：简单原子操作

~~~csharp
int newValue = Interlocked.Increment(ref _errorCount);
~~~

Interlocked.Increment 原子地将整数加一，并返回增加后的值。常见 API：
- Increment / Decrement：原子加一/减一；
- Add：原子增加指定值；
- Exchange：原子交换；
- CompareExchange：比较并按条件交换。

ref 让 Interlocked 操作调用方的原始变量，而不是普通值副本。Interlocked 适合单个变量的简单原子操作，不会自动保护多个字段组成的整体状态，也不会让 UI 控件线程安全。

### 7.3 lock：保护一段临界区

~~~csharp
private readonly object _gate = new();
private int _errorCount;

private void AddError()
{
    lock (_gate)
    {
        _errorCount++;
        Console.WriteLine(_errorCount);
    }
}
~~~

同一时刻，只有成功取得同一个锁对象的线程能进入该临界区。所有访问共享状态的代码路径必须遵守同一把锁，否则互斥不完整。不要随意 lock(this)、字符串或公开对象；优先使用 private readonly object。不要在 lock 内执行长时间 I/O、await 或调用可能回调其他线程的未知代码。

### 7.4 volatile：可见性与内存访问顺序，不是原子递增

~~~csharp
private volatile bool _stopRequested;

private void Worker()
{
    while (!_stopRequested)
    {
        DoOneStep();
    }
}

private void RequestStop()
{
    _stopRequested = true;
}
~~~

volatile 修饰字段，给该字段的读写提供特定的可见性和内存顺序语义（volatile 写具有 release 语义，volatile 读具有 acquire 语义）。它适用于简单共享状态标志等有限场景。

但要注意：
- volatile 不会把 _counter++ 变成原子操作；
- volatile 不会自动保护多个变量之间的不变量；
- volatile 不是 lock 或 Interlocked 的替代品；
- 它修饰字段，不是局部变量或属性；可用类型也受 C# 语言规则限制；
- 不要把它简单描述成“每次都直接从物理 RAM 读取”或“保证任何情况下瞬间看到最新值”。

对任务取消，通常优先使用 CancellationToken；对计数器使用 Interlocked；对一组需要整体一致的状态使用 lock 或其他同步原语。


### 7.5 SemaphoreSlim：控制并发许可，支持异步等待

~~~csharp
private readonly SemaphoreSlim _gate = new(1, 1);

private async Task RunTransactionAsync(CancellationToken token)
{
    await _gate.WaitAsync(token);
    try
    {
        // 临界区：例如一次完整的设备命令—响应事务
        await DoAsyncWork();
    }
    finally
    {
        _gate.Release();
    }
}
~~~

- `new SemaphoreSlim(1, 1)`：初始许可数 1、最大许可数 1；同一时刻只允许一个任务持有许可。
- `WaitAsync`：异步等待许可；等待时不会像同步 `Wait()` 那样阻塞当前线程。
- `Release`：归还许可。成功获取后必须确保只释放一次，通常放在 `finally`。
- 如果 `WaitAsync(token)` 在获取许可前被取消，它会抛出取消异常，后面的 try/finally 尚未进入，因此不应 Release。
- `SemaphoreSlim(3, 3)` 最多允许三个任务同时进入；它不仅能做互斥，也能做并发限流。
- `lock` 适合短小同步临界区，不能在 lock 块内 await；SemaphoreSlim 可跨 await 保护异步事务。
- 信号量只约束使用同一个实例并遵守约定的代码路径；它不自动提供 FIFO 顺序、事务 ID、响应分帧或迟到响应恢复。

串口示例与帧边界见 [SerialPort 与 SemaphoreSlim](SerialPort-and-SemaphoreSlim.md)。同一个串口应有唯一读取入口；SemaphoreSlim 保护命令—响应事务，但不能替代协议解析。

## 8. CancellationTokenSource、CancellationToken 与协作式取消

### 8.1 两个对象的职责

- CancellationTokenSource（CTS）：取消请求的发起者/管理对象，可调用 Cancel()。
- CancellationToken：传递给工作代码的令牌，工作代码通过它观察取消请求。
- token.IsCancellationRequested：查询是否已请求取消。
- token.ThrowIfCancellationRequested()：若已请求取消，则抛出 OperationCanceledException。
- Task.Delay(delay, token) 等支持 Token 的 API 可以响应取消。

Cancel() 只发出取消请求，不会强制杀死线程。后台任务必须定期检查 Token，或调用支持 Token 的 API，在安全边界退出；这叫协作式取消（Cooperative Cancellation）。

### 8.2 Console 最小示例

~~~csharp
using var cts = new CancellationTokenSource();
CancellationToken token = cts.Token;

Task worker = Task.Run(async () =>
{
    int count = 0;

    while (true)
    {
        token.ThrowIfCancellationRequested();

        count++;
        Console.WriteLine("工作中：" + count);

        await Task.Delay(500, token);
    }
}, token);

await Task.Delay(3000);
cts.Cancel();

try
{
    await worker;
}
catch (OperationCanceledException) when (token.IsCancellationRequested)
{
    Console.WriteLine("任务按请求取消");
}
~~~

这里的 Task.Run 第二个 token 参数主要让尚未开始执行的任务在取消后可以不启动。若委托已经开始运行，传入 Task.Run 的 Token 不会自动中断委托；循环内部仍要检查 Token，或把 Token 传给可取消 API。

Task.Delay(500, token) 在等待期间收到取消请求时会提前结束并以 OperationCanceledException 报告取消。若使用 Task.Delay(500) 不传 Token，取消请求不会中断这次延迟，循环只能等延迟结束后再检查。

### 8.3 CTS 生命周期

- 创建 CTS；
- 取出 Token 并传给所有相关工作；
- 调用 Cancel 发出请求；
- await/Join 等方式确认使用它的任务已经结束；
- 最后 Dispose CTS。

Dispose 不等于 Cancel。不要在后台任务仍可能使用 CTS/Token 注册资源时就提前 Dispose。using/using var 可以管理释放，但必须保证作用域退出前已经等待相关任务结束。

### 8.4 WinForms 开始/停止模拟

以下是单任务教学示例，假设 Ui(Action) 是前文的 UI 封送辅助方法，且窗体句柄已在 UI 线程创建。它模拟循环采集，不代表任何真实相机 SDK：

~~~csharp
private CancellationTokenSource? _captureCts;
private Task? _captureTask;

private async void btnStart_Click(object? sender, EventArgs e)
{
    if (_captureTask is { IsCompleted: false })
        return;

    var cts = new CancellationTokenSource();
    _captureCts = cts;
    CancellationToken token = cts.Token;

    Task task = Task.Run(async () =>
    {
        int frame = 0;

        while (true)
        {
            token.ThrowIfCancellationRequested();

            int currentFrame = ++frame; // frame 只由这个工作循环访问
            Ui(() => lblCounter.Text = currentFrame.ToString());

            await Task.Delay(150, token);
        }
    }, token);

    _captureTask = task;

    try
    {
        await task;
    }
    catch (OperationCanceledException) when (token.IsCancellationRequested)
    {
        // 预期的停止路径
    }
    catch (Exception ex)
    {
        lblStatus.Text = "采集失败：" + ex.Message; // await 后通常已回到 UI 上下文
    }
    finally
    {
        if (ReferenceEquals(_captureCts, cts))
            _captureCts = null;

        if (ReferenceEquals(_captureTask, task))
            _captureTask = null;

        cts.Dispose(); // 对应任务已经结束后再释放
    }
}

private async void btnStop_Click(object? sender, EventArgs e)
{
    CancellationTokenSource? cts = _captureCts;
    Task? task = _captureTask;

    if (cts is null || task is null)
        return;

    cts.Cancel();

    try
    {
        await task; // 等待任务真正结束，而不只是发出取消请求
    }
    catch (OperationCanceledException)
    {
        // 预期的取消结果
    }
}
~~~

说明：
- 这里 Task.Run 用来模拟后台工作；纯异步等待本身不需要 Task.Run。真实项目应按任务性质选择执行方式。
- frame 只由后台循环访问，因此此处不需要 Interlocked；若多个线程都修改同一计数器，则需要 Interlocked 或锁。
- UI 更新通过 Ui/BeginInvoke 封送；已排队的 UI 委托不会因为 CTS.Cancel 自动撤回。
- 真实窗体关闭时，应先请求取消并等待任务结束，再释放窗体和设备资源。
- 真实相机 SDK 的阻塞采集调用不一定支持 CancellationToken；应调用 SDK 明确提供的停止/中断 API，并按 SDK 约定处理回调退出和图像缓冲区所有权。
- 真实设备的急停、限位、驱动器安全输入和 PLC 硬件互锁不能由上位机 CancellationToken 代替。

## 9. SynchronizationContext、await 上下文恢复与 Progress<T>

### 9.1 SynchronizationContext 是什么

SynchronizationContext 是 .NET 的调度抽象，提供 Post 和 Send 等方法，把回调安排到某个执行环境中。它本身不是线程，也不负责创建线程。

- SynchronizationContext.Current：获取当前线程关联的同步上下文；没有上下文时可能为 null。
- 在 WinForms 正常运行的 UI 线程中，当前上下文通常是 WindowsFormsSynchronizationContext。
- 在线程池工作线程中，Current 通常为 null，但不要把这当成所有自定义调度环境下的绝对规则。
- 应在 WinForms UI 上下文已经建立后（例如 Shown 事件中）捕获上下文；不要假设 Form 构造函数中一定已经有 WindowsFormsSynchronizationContext。

~~~csharp
private SynchronizationContext? _uiContext;

protected override void OnShown(EventArgs e)
{
    base.OnShown(e);
    _uiContext = SynchronizationContext.Current;
}
~~~

### 9.2 Post 与 Send

| 方法 | 调度方式 | 调用方是否等待 |
|---|---|---|
| Post | 异步投递回调 | 不等待回调执行完成 |
| Send | 同步发送回调 | 通常等待回调完成；同线程场景可能直接执行 |

WinForms 的 WindowsFormsSynchronizationContext 中，Post / Send 与 Control.BeginInvoke / Invoke 的用途相近，但不是所有上下文实现下都能视为完全等价。

- Post 不会创建新线程；它仍依赖目标上下文有机会处理回调。UI 线程正在 Sleep 或执行长时间同步代码时，Post 的回调也只能等待。
- Send 会同步等待 UI 线程执行回调。若 UI 线程同步等待工作线程，而工作线程又 Send 回 UI 线程，就可能形成死锁。
- 相机 SDK 的工作线程回调需要更新状态时，可以将轻量 UI 更新 Post 回 UI 上下文；不要在回调线程直接操作 WinForms 控件。

### 9.3 await 为什么通常能回到 UI 线程

在 WinForms UI 事件处理器中，执行到一个尚未完成的 await 时，默认会尝试捕获当前的 SynchronizationContext。异步操作完成后，后续代码通常会被安排回之前捕获的 UI 上下文。

~~~csharp
private async void btnInspect_Click(object? sender, EventArgs e)
{
    lblStatus.Text = "检测中";

    string result = await Task.Run(() =>
    {
        RunImageInspection(); // 仅示意：耗时计算在工作线程执行
        return "检测完成";
    });

    // 默认恢复到 WinForms UI 上下文
    lblStatus.Text = result;
}
~~~

需要同时记住以下边界：

1. Task.Run 负责把委托提交给线程池；await 本身不创建后台线程。
2. 原生异步 I/O 通常直接 await，不需要为了“异步”再包一层 Task.Run。
3. 若 await 时任务已经完成，后续代码可能直接在当前线程继续执行，不一定发生线程切换。
4. 若没有捕获 UI 上下文，或使用了 ConfigureAwait(false)，就不能假设后续代码回到 UI 线程。
5. 没有 SynchronizationContext 时，await continuation 还可能受当前 TaskScheduler 影响；不要把 await 简化成“永远回主线程”。

### 9.4 ConfigureAwait(false)

~~~csharp
await ReadDeviceAsync().ConfigureAwait(false);
~~~

ConfigureAwait(false) 表示不要求 await 后续代码恢复到之前捕获的 SynchronizationContext。它不代表强制切换到线程池，也不保证一定换线程。

- WinForms 事件处理器后面还要更新控件时，通常保留默认 await 行为。
- 不负责 UI 的设备通信、数据处理等库代码，可以根据设计使用 ConfigureAwait(false)，减少对调用方 UI 上下文的依赖。
- 使用 ConfigureAwait(false) 后，不要直接访问 WinForms 控件；需要 UI 更新时应显式切回 UI 上下文。

### 9.5 Progress<T>：从后台任务汇报进度

Progress<T> 在构造时捕获当前 SynchronizationContext。若在 WinForms UI 线程创建，之后即使从工作线程调用 Report，回调也会被投递到捕获的 UI 上下文。若创建时没有 SynchronizationContext，回调通常在线程池线程执行。

~~~csharp
private async void btnBatchInspect_Click(object? sender, EventArgs e)
{
    // 在 UI 线程创建，捕获 WinForms SynchronizationContext
    var progress = new Progress<int>(percent =>
    {
        progressBar1.Value = Math.Clamp(percent, 0, 100);
        lblStatus.Text = $"检测进度：{percent}%";
    });

    await Task.Run(() =>
    {
        for (int i = 1; i <= 100; i++)
        {
            InspectOneItem();
            progress.Report(i);
        }
    });
}
~~~

- Report 负责报告进度，不保证回调在 Report 返回前已经执行。
- Progress<T> 适合批次进度、初始化进度等相对低频的状态更新。
- 不要把每一帧相机图像都当作普通进度事件无节制地 Report。高帧率采集应考虑限频、合并更新、只保留最新帧或有界队列，避免 UI 回调积压。
- 相机 SDK 图像缓冲区可能被重复使用。是否需要复制、何时释放或如何延长缓冲区生命周期，必须按具体 SDK 文档执行；不能把已失效的帧数据交给 UI 线程。

### 9.6 UI 上下文投递与窗体生命周期

~~~csharp
private void PostToUi(Action action)
{
    var context = _uiContext;
    if (context is null || IsDisposed || Disposing)
        return;

    context.Post(_ =>
    {
        if (!IsDisposed && !Disposing)
            action();
    }, null);
}
~~~

这是简化示例：检查状态与 Post 之间仍可能发生窗体关闭竞争；回调也可能在窗口开始关闭后才执行。真实项目应在关闭时先停止相机/设备事件源、解除事件订阅、取消并等待后台任务收尾，再释放窗体和 SDK 资源。不要仅靠捕获异常来掩盖持续发生的生命周期问题。

## 10. 源码阅读补充：extern

在 .NET 源码中可能看到：

~~~csharp
public static extern int CurrentManagedThreadId
{
    [MethodImpl(MethodImplOptions.InternalCall)]
    get;
}
~~~

extern 表示成员实现不在当前 C# 方法体中。InternalCall 表示实现由 .NET Runtime 内部提供。它是运行时库实现细节，日常业务代码一般不需要自己声明 InternalCall。另一类 extern 常与 DllImport/P/Invoke 一起用于调用原生 DLL；extern 本身不等同于 DllImport。

## 11. 推荐练习顺序

1. ThreadSafetyDemo：用 Parallel.For 多线程执行 counter++，观察结果可能小于期望值；改用 Interlocked.Increment，验证结果。
2. LockDemo：用同一把 private readonly object 锁保护一组必须一起更新的字段。
3. CancellationDemo：每 500ms 输出计数，3 秒后 Cancel，捕获 OperationCanceledException 并 await 任务结束。
4. 修改 Task.Run 示例：分别传/不传 Task.Run 的 Token；比较任务开始前取消和开始后取消。
5. 修改 Task.Delay：分别使用 Task.Delay(500) 与 Task.Delay(500, token)，观察取消响应差异。
6. WinFormsThreadDemo：在后台线程更新 Label，观察跨线程检查；再分别用 Invoke 和 BeginInvoke，比较调用方是否等待。
7. 给 WinForms 模拟采集加 Stop，验证停止后可以重新开始，并在 FormClosing 时先取消、等待，再释放资源。

## 12. 面试口述（P0）

- **WinForms 消息循环是什么？** UI 线程持续获取和分发窗口消息；Application.Run 启动消息循环，UI 线程执行耗时同步工作会延迟处理后续消息。
- **InvokeRequired 是什么？** 判断调用线程是否需要封送到控件句柄所属线程；句柄尚未创建时结果有边界，不能盲目视为 UI 线程检测器。
- **BeginInvoke 和 Invoke 的区别？** 都把委托安排到控件所属线程；Invoke 同步等待执行完成，BeginInvoke 提交后立即返回。
- **BeginInvoke 会创建新线程吗？** 不会；它安排 UI 线程执行委托。
- **Task 和 Thread 的区别？** Thread 是执行线程；Task 是工作/异步操作及其完成状态、结果和异常的抽象，Task 不等于专属线程。
- **Task.Run 默认在哪里执行？** 默认提交给 .NET 线程池调度。
- **Task.Delay 和 Thread.Sleep 的区别？** Sleep 阻塞当前线程；Delay 异步等待，不需要线程在整个等待期休眠。
- **为什么 counter++ 不线程安全？** 它是读、改、写复合操作，多个线程可能覆盖彼此的更新。
- **Interlocked 与 lock 怎么选？** 单变量简单原子操作用 Interlocked；多个操作需要整体互斥时用同一把 lock。
- **volatile 能保证 ++ 原子性吗？** 不能；volatile 提供特定可见性和顺序语义，不把复合读改写变成原子操作。
- **CancellationTokenSource 与 CancellationToken 的区别？** CTS 发起/管理取消请求；Token 传递给任务，供任务检查取消状态。
- **Cancel 会强制终止线程吗？** 不会；它发出协作取消请求，工作代码需要检查 Token 或调用支持取消的 API。
- **Task.Run 的 Token 能中断已运行的委托吗？** 不能自动中断；它可阻止尚未启动的工作，已运行委托需要主动检查 Token。
- **何时 Dispose CTS？** 相关任务完成并不再使用其取消资源后；Dispose 不等于 Cancel。

### WinForms 消息循环与 SynchronizationContext（P0）

- **P0：Windows 消息循环是什么？** UI 线程持续获取并分发窗口消息；UI 线程执行长时间同步工作会延迟输入、重绘和其他消息处理。
- **P0：Application.Run(form) 会创建 UI 线程吗？** 不会；它在调用它的当前线程上运行 WinForms 消息循环。
- **P0：Control 与 HWND 有什么区别？** Control 是 .NET 托管控件对象；HWND 是 Windows 原生窗口句柄，可能延迟创建，也可能在生命周期中重建。
- **P0：为什么读取 Handle 不是单纯查询？** Handle getter 可能触发原生句柄创建；只检查状态应使用 IsHandleCreated。
- **P0：InvokeRequired 返回 false 就能证明当前是 UI 线程吗？** 不能；控件及父级句柄都未创建时，可能找不到可比较的句柄而返回 false。
- **P1：WndProc 是什么？** 窗口处理消息的入口；重写时通常调用 base.WndProc(ref m) 保留 WinForms 默认处理。
- **P0：SynchronizationContext 是线程吗？** 不是；它是把回调安排到特定执行环境的调度抽象。
- **P0：Post 与 Send 有什么区别？** Post 异步投递且不等待；Send 同步发送并等待，跨线程互相等待时可能死锁。
- **P0：await 为什么在 WinForms 中通常回到 UI 线程？** 默认捕获当前 UI SynchronizationContext；未完成任务结束后，后续通常安排回该上下文。
- **P1：ConfigureAwait(false) 会强制切换到线程池吗？** 不会；它只是不要求恢复捕获的上下文。
- **P0：Progress<T> 为什么能更新 UI？** 它在构造时捕获当前 SynchronizationContext，Report 会把回调安排到该上下文。
- **P0：Progress<T> 适合承载每帧相机图像吗？** 不适合无节制高频上报；应限频/合并帧/使用有界队列，并遵守 SDK 缓冲区生命周期。

## 13. 参考资料

- [Microsoft Learn：WinForms 跨线程控件调用](https://learn.microsoft.com/en-us/dotnet/desktop/winforms/controls/how-to-make-thread-safe-calls)
- [Microsoft Learn：Control.InvokeRequired](https://learn.microsoft.com/en-us/dotnet/api/system.windows.forms.control.invokerequired)
- [Microsoft Learn：Control.BeginInvoke](https://learn.microsoft.com/en-us/dotnet/api/system.windows.forms.control.begininvoke)
- [Microsoft Learn：Control.Invoke](https://learn.microsoft.com/en-us/dotnet/api/system.windows.forms.control.invoke)
- [Microsoft Learn：ThreadPool](https://learn.microsoft.com/en-us/dotnet/api/system.threading.threadpool)
- [Microsoft Learn：Task.Run](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.run)
- [Microsoft Learn：Task.Delay](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.delay)
- [Microsoft Learn：CancellationToken](https://learn.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken)
- [Microsoft Learn：CancellationTokenSource](https://learn.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource)
- [Microsoft Learn：volatile](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/volatile)
- [Microsoft Learn：Interlocked](https://learn.microsoft.com/en-us/dotnet/api/system.threading.interlocked)
- [Microsoft Learn：lock statement](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/lock)

- [Microsoft Learn：SynchronizationContext](https://learn.microsoft.com/en-us/dotnet/api/system.threading.synchronizationcontext)
- [Microsoft Learn：WindowsFormsSynchronizationContext](https://learn.microsoft.com/en-us/dotnet/api/system.windows.forms.windowsformssynchronizationcontext)
- [Microsoft Learn：Progress<T>](https://learn.microsoft.com/en-us/dotnet/api/system.progress-1)
- [Microsoft Learn：ConfigureAwait FAQ](https://devblogs.microsoft.com/dotnet/configureawait-faq/)
- [Microsoft Learn：Application.Run](https://learn.microsoft.com/en-us/dotnet/api/system.windows.forms.application.run)
- [Microsoft Learn：Control.Handle](https://learn.microsoft.com/en-us/dotnet/api/system.windows.forms.control.handle)
- [Microsoft Learn：Control.WndProc](https://learn.microsoft.com/en-us/dotnet/api/system.windows.forms.control.wndproc)

> 学习状态：待学习 / 待验证。已进行文字讲解和概念问答，但尚未记录用户独立编译运行上述练习，也未连接真实相机 SDK 或 PLC。
