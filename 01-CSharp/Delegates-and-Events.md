# C# 委托（Delegate）与事件（Event）

> P0｜待学习 / 待验证｜2026-11-05。面向 .NET 8 / WinForms 工业上位机；相机 SDK 的具体 API、线程和缓冲区规则以厂商及版本文档为准。

## 一句话理解

- **Delegate（委托）**：一种类型安全的可调用对象类型，可以保存符合签名的方法，并通过委托调用它们。
- **Event（事件）**：基于委托的通知成员；发布者负责触发，外部对象通常只能订阅和退订。
- **EventArgs**：事件携带的数据对象；`sender` 表示触发事件的对象。
- **Callback（回调）**：把方法交给另一个对象，由对方在合适的时机调用。委托是 C# 实现回调的常见方式。

## 1. Delegate：把方法作为值传递

### 定义委托类型

```csharp
public delegate int Calculate(int a, int b);
```

这行定义的是一种新的委托类型，不是一个普通方法。它规定了可绑定方法的签名：接收两个 `int` 参数，返回一个 `int`。

```csharp
static int Add(int a, int b) => a + b;
static int Subtract(int a, int b) => a - b;

Calculate calculate = Add;       // 绑定方法组，不是立即执行 Add
int sum = calculate(100, 50);    // 通过委托调用 Add

calculate = Subtract;            // 同一个变量可以改绑兼容方法
int difference = calculate(100, 50);
```

绑定的方法必须与委托签名兼容，主要检查参数数量、参数类型/顺序、参数传递方式和返回类型。委托可以绑定静态方法，也可以绑定实例方法；绑定实例方法时，委托还会保留目标对象引用。

### Action、Func 与 Predicate

日常代码通常不必为每一种签名都自定义委托：

| 类型 | 规则 | 示例 |
|---|---|---|
| `Action` | 无参数、无返回值 | `Action log = WriteLog;` |
| `Action<T>` | 一个 T 参数、无返回值 | `Action<string> log = WriteLog;` |
| `Func<T, TResult>` | 一个 T 参数，返回 TResult | `Func<int, string> format = Format;` |
| `Func<T1, T2, TResult>` | 两个输入参数，最后一个类型是返回值 | `Func<int, int, int> add = Add;` |
| `Predicate<T>` | 一个 T 参数，返回 bool | `Predicate<int> isEven = n => n % 2 == 0;` |

记忆：`Action` 不返回值；`Func` 最后一个泛型参数是返回类型；`Predicate<T>` 是针对一个 T 值的 bool 判断。需要表达清晰业务语义或特定事件契约时，可以定义自己的委托类型。

### 多播委托

委托可以保存多个处理方法：

```csharp
Action notify = ShowImage;
notify += StartInspection;
notify += WriteLog;

notify();              // 按调用列表顺序调用
notify -= ShowImage;    // 移除一个匹配项
notify();
```

注意：

- `+=` / `-=` 对委托变量表示组合/移除调用列表中的处理器。
- 同一个处理器重复添加，可能会被调用多次；一次 `-=` 只移除一个匹配项。
- 多播委托有返回值时，直接调用只得到最后一个处理方法的返回值；事件通常采用 void 返回类型。
- 某个处理方法抛出未处理异常时，通常会中断这次多播调用，后续处理器不会继续执行。

### Callback（回调）

```csharp
public delegate void ImageCallback(byte[] imageData);

public class CameraSimulator
{
    public void Capture(ImageCallback callback)
    {
        byte[] imageData = new byte[1024]; // 仅为教学模拟数据
        callback?.Invoke(imageData);
    }
}

// 调用方把处理方法交给相机模拟类
camera.Capture(OnImageCaptured);
```

相机类不需要知道调用方方法的具体实现，只需要在采集完成时调用传入的委托。这就是回调。上例是同步调用，不代表真实相机 SDK 一定采用相同线程或生命周期。

## 2. Event：受控的通知入口

### 为什么不直接公开委托字段？

若把委托字段公开：

```csharp
public Action? ImageCaptured;
```

外部代码可能订阅、取消订阅、覆盖整个调用列表，甚至直接调用它。这样外部对象就能伪造“相机采集完成”的通知。

改成事件：

```csharp
public event Action? ImageCaptured;
```

外部调用方通常只能：

```csharp
camera.ImageCaptured += OnImageCaptured; // 订阅
camera.ImageCaptured -= OnImageCaptured; // 退订
```

外部不能直接给事件赋值或调用 `Invoke()`。声明事件的类型内部负责触发它。派生类也不能直接触发基类声明的事件，通常由基类提供 `protected` 的 `OnXxx` 方法。

**核心：event 不是另一种独立的回调底层；它是对委托调用列表的封装，控制谁能订阅、退订和触发。**

### 编译器为字段式事件做了什么？

```csharp
public event EventHandler? Connected;
```

这是字段式事件（field-like event）。编译器会生成事件的委托存储和 `add` / `remove` 访问器：

- 外部 `+=` 调用 `add` 访问器。
- 外部 `-=` 调用 `remove` 访问器。
- 事件声明所在类型内部可以触发事件。

也可以手写自定义事件访问器：

```csharp
private EventHandler? _connectedHandlers;

public event EventHandler Connected
{
    add { _connectedHandlers += value; }
    remove { _connectedHandlers -= value; }
}
```

自定义访问器适用于需要自定义存储/订阅管理的情况；一般业务代码优先使用字段式事件。这里的 `value` 是本次传入的处理委托。

## 3. sender、EventArgs 与 EventHandler<TEventArgs>

.NET 常见事件处理器约定：

```csharp
void Handler(object? sender, EventArgs e)
```

- `sender`：事件发布者/触发者，通常由发布者传入 `this`。
- `e`：事件数据。无额外数据时可使用 `EventArgs.Empty`。
- `EventArgs`：事件参数基类；需要携带数据时，定义自己的派生类。
- `EventHandler<TEventArgs>`：.NET 提供的通用事件委托，避免每种事件都手写 delegate。

```csharp
public sealed class ImageCapturedEventArgs : EventArgs
{
    public byte[] ImageData { get; }
    public DateTime CapturedAt { get; }

    public ImageCapturedEventArgs(byte[] imageData)
    {
        ImageData = imageData;
        CapturedAt = DateTime.Now;
    }
}
```

注意：`get;` 只限制外部替换 `ImageData` 属性引用，不会让数组内容自动不可变；真实项目若需要跨线程/跨回调保存图像，必须按 SDK 规则复制或取得有明确所有权的缓冲区。

## 4. 工业相机事件完整示例（同步教学模拟）

### Camera.cs

```csharp
public class Camera
{
    public event EventHandler<ImageCapturedEventArgs>? ImageCaptured;

    public void Capture()
    {
        Console.WriteLine("相机开始采集...");

        // 模拟数据，不是真实相机 SDK 缓冲区
        byte[] imageData = new byte[1024];
        var args = new ImageCapturedEventArgs(imageData);

        Console.WriteLine("图像采集完成");
        OnImageCaptured(args);
    }

    protected virtual void OnImageCaptured(ImageCapturedEventArgs e)
    {
        ImageCaptured?.Invoke(this, e);
    }
}
```

### 订阅与处理

```csharp
var camera = new Camera();

camera.ImageCaptured += Camera_ImageCaptured;
camera.ImageCaptured += WriteCaptureLog;

camera.Capture();

camera.ImageCaptured -= WriteCaptureLog;

static void Camera_ImageCaptured(
    object? sender,
    ImageCapturedEventArgs e)
{
    Console.WriteLine($"收到图像，长度：{e.ImageData.Length}");

    if (sender is Camera source)
        Console.WriteLine($"事件来自：{source.GetType().Name}");
}

static void WriteCaptureLog(object? sender, ImageCapturedEventArgs e)
{
    Console.WriteLine($"采集时间：{e.CapturedAt}");
}
```

调用流程：`Capture()` 生成/取得图像 → 创建事件参数 → `OnImageCaptured(args)` → `ImageCaptured?.Invoke(this, args)` → 依次调用订阅者。

### 为什么用 OnImageCaptured？

这是 .NET 常见的事件发布模式：

- 对外事件名使用过去式/已发生语义，如 `ImageCaptured`、`Connected`、`AlarmRaised`。
- `OnImageCaptured` 是内部触发入口。
- `protected virtual` 允许派生类在需要时扩展触发行为。
- 子类若重写并希望继续通知订阅者，应调用 `base.OnImageCaptured(e)`。

## 5. WinForms Button.Click 与工业相机事件

两者使用同一套 .NET 委托/事件模型：

| 项目 | WinForms 按钮 | 相机模拟类 |
|---|---|---|
| 发布者 | Button | Camera |
| 事件 | Click | ImageCaptured |
| 订阅 | `button.Click += Handler` | `camera.ImageCaptured += Handler` |
| 处理器 | 常见为 `EventHandler` | 常见为 `EventHandler<ImageCapturedEventArgs>` |
| 触发条件 | 用户点击 | 采集完成 |

`Button.Click` 通常使用 `EventHandler`，`sender` 是触发事件的按钮，`e` 是 `EventArgs`。鼠标事件还可能使用 `MouseEventHandler` 和 `MouseEventArgs`，其中包含鼠标按钮、坐标等数据。

工业相机 SDK 的设计因厂商、产品系列和 SDK 版本而异：有的提供 .NET event，有的提供 delegate callback、回调接口或轮询 API。不能仅凭“相机回调”就假设所有 SDK 的 API 名称、触发线程和图像内存生命周期相同；必须查对应版本文档。

## 6. 事件线程、UI 更新与订阅生命周期

### 事件不等于异步

普通事件触发通常是同步调用：发布者在调用 `Invoke` 时，按调用列表执行处理器，处理器未返回前发布者的这次调用通常不会继续。事件处理器运行在**触发事件的线程**上；`event` 不会自动创建线程，也不会自动切换到 UI 线程。

### WinForms 跨线程更新

如果相机 SDK 在采集线程触发事件，不能直接从该线程更新 WinForms 控件：

```csharp
private void Camera_ImageCaptured(
    object? sender,
    ImageCapturedEventArgs e)
{
    if (IsDisposed || !IsHandleCreated)
        return;

    if (InvokeRequired)
    {
        BeginInvoke(new Action(() => UpdateUi(e)));
        return;
    }

    UpdateUi(e);
}

private void UpdateUi(ImageCapturedEventArgs e)
{
    lblStatus.Text = $"图像长度：{e.ImageData.Length}";
}
```

这是简化示例。实际代码还要处理窗体关闭与 BeginInvoke 之间的竞态、异常和图像数据所有权；不要把 SDK 临时缓冲区直接跨线程长期保存。必要时在回调有效期内复制图像数据，并按 SDK 规定释放原始缓冲区。

### 为什么要退订？

发布者的事件调用列表会持有订阅者委托；实例方法委托通常也会引用目标对象。如果相机服务生命周期长于窗体，而窗体关闭后没有退订，可能使窗体无法及时被 GC 回收，或在关闭后继续收到回调。

```csharp
camera.ImageCaptured += Camera_ImageCaptured;

// 窗体关闭/释放前，在合适的生命周期中退订
camera.ImageCaptured -= Camera_ImageCaptured;
```

重复订阅会造成处理器重复执行。若用 Lambda 订阅，不能简单地再写一个内容相同的新 Lambda 来退订，因为那通常是另一个委托实例；应保存委托变量或使用具名方法。

## 7. 常见误区

1. `event` 不会让回调自动异步，也不会自动切换线程。
2. `+=` 是订阅，不是立即执行处理器。
3. `sender` 是事件发送者，不是事件数据；事件数据放在 `e` 中。
4. `EventArgs` 可以为空数据基类；有数据时使用自定义 EventArgs。
5. 公开委托字段与公开 event 不同：前者允许外部覆盖/调用，后者限制外部访问方式。
6. 事件处理器可能抛异常；一个处理器的未处理异常可能中断同一次多播调用。
7. 退订、停止采集、释放 SDK 资源是不同动作，具体顺序以设备 SDK 文档为准。
8. 上位机事件通知不能替代 PLC/驱动器的硬件互锁、急停、限位等安全功能。

## 8. 动手验证

使用 .NET 8 Console 项目：

1. 定义 `ImageCapturedEventArgs`，包含图像数据和采集时间。
2. Camera 声明 `event EventHandler<ImageCapturedEventArgs>`，在 Capture 完成后触发。
3. Program 订阅两个处理器，验证一次采集通知两个订阅者。
4. 使用 `-=` 取消其中一个订阅，再采集一次，比较输出。
5. 尝试在 Camera 外部调用 `camera.ImageCaptured?.Invoke(...)`，观察编译错误；再在 Camera 内部调用，观察成功。
6. 尝试重复订阅同一个处理器，观察它被调用的次数；再退订一次观察变化。

## 9. 面试口述

- **P0：Delegate 和 Event 的区别？** Delegate 是类型安全的方法引用/调用列表；Event 是基于委托的通知成员，限制外部只能订阅和退订，发布者控制触发。
- **P0：为什么事件使用 += 和 -=？** 分别调用事件的 add/remove 访问器，将处理委托加入或移出调用列表。
- **P0：sender 与 EventArgs 是什么？** sender 标识事件发布者；EventArgs 或其派生类承载事件数据。
- **P0：事件是异步的吗？** 不是。普通事件处理器通常在触发事件的线程同步执行；需要异步要由处理器或发布者另行设计。
- **P0：相机事件中为什么不能直接更新 Label？** SDK 可能在采集线程触发回调；WinForms 控件应通过 Invoke/BeginInvoke 等机制切回 UI 线程。
- **P1：为什么关闭窗体时要取消订阅？** 避免长生命周期发布者继续持有短生命周期订阅者，并防止关闭后仍处理事件。

## 10. 参考资料

- [Microsoft：Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [Microsoft：Events](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/)
- [Microsoft：EventHandler<TEventArgs>](https://learn.microsoft.com/en-us/dotnet/api/system.eventhandler-1)
- [Microsoft：Make thread-safe calls to controls](https://learn.microsoft.com/en-us/dotnet/desktop/winforms/controls/how-to-make-thread-safe-calls)

> 学习状态：待学习 / 待验证。以上相机代码为同步模拟示例，未连接真实工业相机，也未验证特定厂商 SDK。
