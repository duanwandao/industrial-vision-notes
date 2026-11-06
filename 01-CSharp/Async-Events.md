# C# 异步编程与 UI 线程

> P0｜待学习｜2026-10-31｜Q013、Q016–018、Q040–041、Q056–059。项目应用见[双轴 Jog](../04-Industrial-Projects/Dual-Axis-Jog.md)。

一句话：事件通知某个动作或状态已经发生；Task 表示一次异步操作及其完成结果；await 让当前方法在等待期间交还执行权。

## 从 JS/TS 迁移

| C# | JS/TS 类比与差异 |
|---|---|
| `Task<T>` | 类似 Promise<T>，代表结果；不等于线程，也不一定已经启动（显式构造的 Task 可能尚未启动） |
| `await task` | JS 也可以 await Promise 变量；方法调用只是取得任务的一种表达式 |
| `async` | 方法先同步运行至尚未完成的 await；关键字本身不创建后台线程 |
| `Task.Run` | 将工作调度到线程池；不像普通 JS Promise 执行器，不是在调用线程直接完成整段委托 |
| `delegate` | 有类型约束的可调用对象，可包含多个处理器；不是 JS 的动态函数类型 |
| `event` | 限制外部只能订阅/退订的通知入口，外部不能直接触发；不等于任意可赋值函数字段 |

## 最小离线示例

适用于支持 C# async 的 .NET；只演示任务，不涉及 PLC。

```csharp
static async Task<int> GetNumberAsync()
{
    await Task.Delay(100);
    return 42;
}
// 放在另一个 async 方法内部：
Task<int> task = GetNumberAsync();
int number = await task;
```

`await` 传播任务失败；用 try/catch 包住 await 才能在本流程处理异常。`_ = Task.Run(...)` 只是丢弃返回值，不会观察异常，也不会安排关闭时等待收尾。普通异步方法优先返回 Task；WinForms 事件签名要求 void 时使用 async void，并在内部捕获异常。

## UI 线程与事件（Q013、Q040–041、Q058–059）

WinForms 的消息循环负责鼠标、重绘和窗口消息。在 UI 线程 Sleep、同步网络调用或 `.Result/.Wait()` 会卡界面，后者还可能与上下文恢复互相等待。原生异步 I/O 优先直接 await；包装同步通信的 Task.Run 会占线程池线程，仍需超时和串行访问。

UI 事件中的 await 默认捕获当前 WinForms 同步上下文，后续通常回 UI 线程；这不是所有 await 的通用保证。后台线程不能直接修改控件；可用 Invoke/BeginInvoke 将委托封送到控件所属线程。Invoke 同步等待，BeginInvoke 提交后立即返回。InvokeRequired 在控件句柄尚未创建时有边界，不能无条件视为 UI 线程检测器。当前 IndustrialVisionInspection 使用 .NET 8 WinForms；Control.InvokeAsync 是 .NET 9 新增 API，不能直接用于 .NET 8。

`button.MouseDown += Handler` 是订阅；`button.Enabled = false` 是属性赋值。Click 发生于一次点击语义，按住 Jog 应分析 MouseDown/MouseUp 等订阅，而非只找 Click。订阅重复会执行多次；长期发布者持有短期订阅者时应适时 `-=`。

`event` 的 add/remove 访问器中 `value` 就是本次传入的委托；`Events.AddHandler(key, value)` 按键保存处理器，不是修改 PLC 值。IsDisposed 继承自 Control 相关层次，表示控件已释放；检查它不能保证后续异步结束前窗体仍存活。阅读源码先确认目标框架版本，转到定义可能只显示元数据；反编译不是作者原始注释/源码。

## 取消与手动完成（Q016、Q057）

CancellationTokenSource 发出协作取消请求；操作必须检查 token 或调用支持 token 的 API。Cancel 不强杀线程，Dispose 也不等于 Cancel。C# 8 的 `using var cts = ...` 在作用域结束释放资源；旧语言版本可用 using 块。

TCS 把外部事件转换成 Task，类似保存 Promise 的 resolve/reject。`await tcs.Task` 不忙等；`TrySetResult` 可以应对重复释放通知。可考虑 `TaskCreationOptions.RunContinuationsAsynchronously` 避免完成信号时内联执行后续，但这属于设计选择，不是原仓库现状。TCS 自身线程安全不意味着多个 UI 状态字段的组合也线程安全。

## 常见错误与自测

await 不等于后台线程；停止等待不等于撤销已经发送的指令；Task.Run 不会自动令通信库线程安全。先在 UI 线程读取轴、方向和 NumericUpDown.Value，再交给后台通信，避免后台访问控件。

自测：在纯延迟模拟中，分别让释放发生在写速度前、写速度中、写启动中，解释哪些路径应跳过启动，哪些仍需停止。面试题见[复习题](../05-Interview/P0-Review.md)。

## 核对资料

- [Microsoft：WinForms 事件](https://learn.microsoft.com/en-us/dotnet/desktop/winforms/forms/events)
- [Microsoft：TaskCompletionSource<T>](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskcompletionsource-1)
- [Microsoft：跨线程操作控件](https://learn.microsoft.com/en-us/dotnet/desktop/winforms/controls/how-to-make-thread-safe-calls)


## 相关专题

委托与事件的基础机制见 [C# 委托与事件](Delegates-and-Events.md)。本篇聚焦 Task、取消和 UI 线程。

窗体构造、设计器初始化、资源释放和可空类型见 [C# partial、WinForms 窗体生命周期与可空类型](WinForms-Lifecycle-and-Nullability.md)。尤其要区分取消后台任务、停止设备采集与 Dispose 资源：它们不是同一件事。


## 深入专题：线程、线程池与取消

本篇先建立异步编程和 UI 线程的基础。以下内容另见 [C# 线程、线程池、WinForms UI 封送与协作式取消](Threading-and-Cancellation.md)：

- UI 消息循环、InvokeRequired、Invoke、BeginInvoke；
- Thread / Task / ThreadPool、Task.Run、Task.Delay 与 Thread.Sleep；
- 竞态条件、Interlocked、lock、volatile；
- CancellationTokenSource、CancellationToken、协作式取消与 WinForms 停止流程。
