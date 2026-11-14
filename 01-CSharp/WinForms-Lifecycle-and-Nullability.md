# C# partial、WinForms 窗体生命周期与可空类型

> **P0｜待学习 / 待验证｜2026-11-05**
>
> 面向 C# / .NET WinForms 工业上位机开发。示例结合 IndustrialVisionInspection 项目；项目中的实际设备 SDK 版本和释放契约仍需按设备资料确认。

## 1. partial：同一个类型拆分到多个文件

`partial` 是 C# 关键字，允许把同一个类型的声明分散在多个源文件。编译时这些声明会组合成**同一个类型**，不是两个对象，也不是继承关系。

```csharp
// Person.cs
public partial class Person
{
    public string Name = "张三";
    public void SayHello() => Console.WriteLine("你好");
}
```

```csharp
// Person.Age.cs
public partial class Person
{
    public int Age = 26;
    public void Introduce()
        => Console.WriteLine($"我叫{Name}，今年{Age}岁");
}
```

```csharp
var person = new Person();
person.SayHello();
person.Introduce();
```

第二个文件可以访问第一个文件声明的成员，包括 private 成员，因为它们是同一个类的不同部分。

规则与边界：
- 各部分必须属于同一命名空间、同一类型名和同一泛型参数列表，并使用 `partial`。
- 所有部分必须参与同一次编译，不能在两个独立程序集里再期待编译器合并。
- class、struct、interface、record 都支持 partial；partial 方法有额外规则，本节暂不展开。
- WinForms 常用它分离业务代码和设计器代码。不要把手写业务逻辑放进可能被设计器重新生成的文件。

## 2. 构造方法、InitializeComponent 与 Dispose

### 2.1 程序入口

```csharp
[STAThread]
private static void Main()
{
    ApplicationConfiguration.Initialize();
    Application.Run(new Form1());
}
```

`new Form1()` 创建窗体对象并触发构造过程；构造完成后，`Application.Run` 启动消息循环并显示主窗体。消息循环负责分发鼠标、键盘、重绘等窗口消息。

### 2.2 构造方法（Constructor）

构造方法名称与类名相同，没有返回类型，在创建对象时自动执行：

```csharp
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        if (LicenseManager.UsageMode != LicenseUsageMode.Designtime)
        {
            InitializeLayout();
            InitializeBehavior();
        }
    }
}
```

- `new Form1()` 会触发 `Form1()`；调用方不直接调用构造方法。
- 派生窗体构造方法主体执行前，基类构造过程已经完成；字段初始化器也有自己的顺序，不要把全部初始化都简单理解为构造方法中的第一行。
- 构造方法适合建立初始状态、初始化控件和准备依赖，不适合执行长时间阻塞操作。
- 当前项目的 `LicenseManager.UsageMode != LicenseUsageMode.Designtime` 用于避免设计器加载时启动运行期行为；它不是所有设计器场景的万能检测方案。

### 2.3 InitializeComponent()

`InitializeComponent()` **不是 C# 关键字，也不是构造方法**，它是 WinForms 设计器通常生成的普通方法。设计器生成的实现会创建控件/组件、设置属性、建立控件父子关系，并可能订阅事件。

当前项目 `Form1.Designer.cs` 的代码：

```csharp
private void InitializeComponent()
{
    components = new System.ComponentModel.Container();
    SuspendLayout();

    AutoScaleDimensions = new SizeF(11F, 24F);
    AutoScaleMode = AutoScaleMode.Font;
    ClientSize = new Size(800, 450);
    Name = "Form1";
    Text = "Form1";

    ResumeLayout(false);
}
```

当前 Designer 文件只初始化窗体基础属性和组件容器；主要界面由 `Form1.Layout.cs` 的 `InitializeLayout()` 动态创建，再通过 `Controls.Add(root)` 加入窗体。

| 方法 | 职责 |
|---|---|
| 构造方法 `Form1()` | 安排窗体初始化顺序 |
| `InitializeComponent()` | 初始化设计器管理的窗体/控件/组件 |
| `InitializeLayout()` | 当前项目自定义创建布局和控件 |
| `InitializeBehavior()` | 当前项目注册事件、创建运行期组件并启动行为 |
| `OnLoad()` | 窗体加载阶段的逻辑，发生在构造完成后、显示过程中 |

不要假设所有 WinForms 项目都采用动态布局；常见项目会把大量控件初始化代码放在 `InitializeComponent()` 中。

### 2.4 SuspendLayout 与 ResumeLayout

```csharp
SuspendLayout();
// 批量修改布局属性、添加子控件
ResumeLayout(true);
```

- `SuspendLayout()` 暂时挂起当前控件的布局逻辑，适合批量设置布局属性时减少重复布局计算。
- `ResumeLayout(true)` 恢复布局，并请求执行挂起的布局逻辑。
- 它们不暂停线程、消息循环或绘制；复杂嵌套布局还要考虑子容器自己的布局。

### 2.5 Dispose：释放窗体资源

当前 Designer 文件中的典型实现：

```csharp
protected override void Dispose(bool disposing)
{
    if (disposing && (components != null))
    {
        components.Dispose();
    }

    base.Dispose(disposing);
}
```

- `protected`：受保护访问级别。
- `override`：重写基类虚方法。
- `disposing` 为 `true` 时执行显式释放路径，可以释放托管资源及其拥有的资源；为 `false` 时不应依赖其他托管对象仍可用，通常只处理本类型直接拥有的非托管资源。
- `components.Dispose()` 释放组件容器中登记的组件。
- `base.Dispose(disposing)` 让基类继续清理它管理的资源；重写时不要漏掉。

当前项目的 `InitializeBehavior()` 中：

```csharp
components ??= new Container();

var clockTimer = new System.Windows.Forms.Timer(components)
{
    Interval = 1000
};
```

把 Timer 传给 `Timer(components)`，会将它登记到组件容器中；释放容器时，容器会释放它管理的组件。

区分两类资源：
- **非可视组件**：例如 Timer，可以由 `components` 容器管理。
- **可视控件**：例如 PictureBox、RichTextBox，通常属于窗体的 `Controls` 控件树；窗体基类负责释放其子控件。不是每个可视控件都必须放进 `components`。

### 2.6 IDisposable、Close 与 Dispose

一个类仅仅定义了名为 `Dispose()` 的普通方法，并不意味着 C# 会自动调用它。通常通过实现 `IDisposable` 提供确定性释放能力；`using` / `using var` 会在作用域结束时（包括异常退出）调用 `Dispose()`：

```csharp
using var stream = File.OpenRead(path);
// 使用 stream；离开作用域时自动 Dispose
```

| 操作 | 含义 |
|---|---|
| `Close()` | 请求关闭窗体，会经过关闭事件流程 |
| `Dispose()` | 释放窗体对象及其拥有的资源 |
| `using` | 对 IDisposable 对象提供作用域结束时释放的语法 |

- 对以 `Show()` 显示的非模态窗体，正常关闭流程通常会释放窗体资源。
- 对 `ShowDialog()` 显示的模态窗体，关闭通常先隐藏窗体并返回对话结果；调用方应负责在不再使用时 Dispose，例如 `using var dialog = new SettingsForm();`。
- 不要因为调用 `Close()` 就认为后台任务、相机采集或 PLC 通信已安全停止；设备 SDK 和任务需要按各自契约停止、等待、退订并释放。
- 工业视觉软件关闭时，应先阻止新工作进入，再请求取消/停止采集，等待正在执行的工作结束，最后释放通信/相机对象。具体顺序以对应 SDK 文档为准。
- 资源不只是托管内存：SerialPort、Bitmap、相机 SDK 对象可能管理 OS 句柄或原生资源；GC 与 Dispose 分工不同。资源所有权、using / await using、IAsyncDisposable 和 Cancel/await Task 的完整说明见 [C# 资源生命周期专题](Resource-Lifecycle-and-Async-Disposal.md)。
- 窗体关闭不是安全回路；急停、硬件互锁、驱动器安全功能和危险能量隔离不能由 Close/Dispose 代替。

## 3. null 与可空类型

### 3.1 值类型：Nullable<T>

普通值类型不能直接赋值为 `null`：

```csharp
int count = 0;
// count = null; // 编译错误
```

`int?` 是 `Nullable<int>` 的简写，允许整数表示一个值或“没有值”。`Nullable<T>` 只用于非可空值类型 T。

```csharp
int? defectCount = 5;

bool hasResult = defectCount.HasValue; // true
int value = defectCount.Value;         // 5
int safeValue = defectCount.GetValueOrDefault();
```

- `HasValue`：是否有实际值。
- `Value`：读取实际值；没有值时访问会抛出 `InvalidOperationException`，所以应先判断。
- `GetValueOrDefault()`：有值时返回值，没有值时返回该值类型的默认值。
- `??`：为空时提供备用值。

### 3.2 工业检测例子：null 不等于 0

```csharp
int? defectCount = null; // 尚未得到检测结果
defectCount = 0;         // 已完成检测，缺陷数量为 0
defectCount = 5;         // 已完成检测，发现 5 个缺陷
```

| 值 | 业务含义 |
|---|---|
| `null` | 尚未检测 / 结果未知 |
| `0` | 已检测，缺陷数为零 |
| `5` | 已检测，发现 5 个缺陷 |

不能随意把 `null` 转成 0，否则可能把“尚未检测”误显示为“检测合格”。

### 3.3 引用类型：string?、IContainer?

```csharp
string? name = null;
IContainer? components = null;
```

`string?` / `IContainer?` 与 `int?` 不同：

- `int?` 是 `Nullable<int>`，运行时是可空值类型结构。
- `string?`、`IContainer?` 是可空引用类型注解，用于告诉编译器该引用可能为 null，帮助静态分析和产生警告。
- 可空引用类型功能从 C# 8.0 开始提供，通常需要启用 `<Nullable>enable</Nullable>` 或 `#nullable enable`。
- `string?` 不会变成 `Nullable<string>` 包装对象；运行时引用仍可能是 null。注解不会在运行时自动阻止 null，因此外部输入仍需验证。

### 3.4 四种常见问号写法

```csharp
IContainer? components = null;

// ??：左侧为 null 时，表达式结果使用右侧值
IContainer container = components ?? new Container();

// ??=：仅当左侧为 null 时，才把右侧赋给左侧
components ??= new Container();

// ?.：左侧不为 null 才调用成员
components?.Dispose();
```

| 写法 | 名称 | 作用 |
|---|---|---|
| `IContainer?` | 可空引用注解 | 说明引用可能为 null |
| `components?.Dispose()` | 空条件运算符 | 不为 null 才访问成员 |
| `components ?? new Container()` | 空合并运算符 | 为 null 时使用备用表达式 |
| `components ??= new Container()` | 空合并赋值运算符 | 为 null 时才赋值 |

`??` 和 `??=` 只判断 null，不判断空字符串、0 或 false：

```csharp
string name = "";
name ??= "默认名称"; // name 仍然是空字符串

int? count = 0;
bool hasValue = count.HasValue; // true
```

## 4. 常见误区

1. **把 partial 当成继承**：partial 是同一个类型的分文件声明；继承才是建立基类/派生类关系。
2. **认为 InitializeComponent 是语言关键字**：它只是设计器通常生成并由构造方法调用的普通方法。
3. **认为 Designer 文件一定包含全部 UI**：本项目实际 UI 主要在 InitializeLayout 中运行时创建。
4. **把 Close 和 Dispose 当成同义词**：关闭流程与资源释放相关，但模态窗体和后台资源要单独考虑。
5. **认为 components 管理所有控件**：容器管理登记其中的组件；可视控件由 Controls 父子树管理。
6. **认为写了 Dispose 方法就会自动调用**：需要由调用方、using 或窗体关闭/释放流程触发；普通同名方法不会自动执行。
7. **把 null 和 0 混为一谈**：前者表示没有结果，后者是有效数值。
8. **把 string? 当成 Nullable<string>**：Nullable<T> 只用于值类型；string? 是可空引用注解。
9. **认为问号会在运行时自动防止 null**：可空引用注解主要提供编译期分析，不替代运行时验证。

## 5. 动手验证

在当前 WinForms 项目中设置断点，观察：

1. `Program.Main` 中 `new Form1()`。
2. `Form1()` 构造方法。
3. `InitializeComponent()`。
4. `InitializeLayout()` 和 `InitializeBehavior()`。
5. `OnLoad()`。
6. 关闭窗体时的 `FormClosing` 与 `Dispose(bool)`。

再用控制台或调试窗口验证：

```csharp
int? defectCount = null;
Console.WriteLine(defectCount.HasValue); // false

defectCount = 0;
Console.WriteLine(defectCount.HasValue); // true
Console.WriteLine(defectCount.Value);    // 0

IContainer? components = null;
components ??= new Container();
Console.WriteLine(components is not null); // true
components?.Dispose();
```

**验证状态：** 本笔记根据文字讲解和仓库代码结构整理；尚未记录用户在 Visual Studio 中逐步断点验证、运行上述示例或检查真实设备 SDK 释放行为的证据。

## 6. 参考资料

- [Microsoft Learn：Partial Classes and Methods](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/partial-classes-and-methods)
- [Microsoft Learn：Nullable value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types)
- [Microsoft Learn：Nullable reference types](https://learn.microsoft.com/en-us/dotnet/csharp/nullable-references)
- [Microsoft Learn：Null-coalescing operators](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator)
- [Microsoft Learn：IDisposable interface](https://learn.microsoft.com/en-us/dotnet/api/system.idisposable)
- [Microsoft Learn：Windows Forms overview](https://learn.microsoft.com/en-us/dotnet/desktop/winforms/overview/)
