# C# 命名空间、访问控制、继承、抽象类与接口

> **P0｜待学习 / 待验证｜2026-11-05**
>
> 面向 .NET 8 WinForms 工业上位机与机器视觉应用。示例中的相机类是教学用封装，不代表 Basler、海康或大华 SDK 的真实 API。

## 1. namespace：组织类型名称

```csharp
namespace IndustrialVision.Camera;

public class CameraService
{
    public void Connect()
    {
        Console.WriteLine("连接相机");
    }
}
```

- **namespace** 声明类型所属的命名空间；**IndustrialVision.Camera** 是一个限定命名空间名称。
- 点号 **.** 是命名空间名称的层级分隔符，不是访问对象属性/字段的成员访问运算符。
- **namespace IndustrialVision.Camera;** 是 C# 10 引入的文件级命名空间写法；也可以使用传统花括号写法。
- 命名空间是代码组织和名称解析机制，不是文件夹、对象、程序集或访问权限边界。
- 文件夹与 namespace 通常保持对应，方便维护，但不是编译器强制要求。

常见命名空间：

| 命名空间 | 可能放置的类型 |
|---|---|
| IndustrialVision.Camera | 相机服务、相机适配器 |
| IndustrialVision.PLC | PLC 服务、通信封装 |
| IndustrialVision.Inspection | 检测流程、检测结果 |
| IndustrialVision.UI | WinForms 窗体与界面协调 |

两个不同命名空间可以包含同名类型：

```csharp
namespace IndustrialVision.Camera
{
    public class Device { }
}

namespace IndustrialVision.PLC
{
    public class Device { }
}
```

它们的完整名称不同，因此不会仅因类名相同而冲突。

### 1.1 using 的作用与边界

```csharp
using IndustrialVision.Camera;

namespace IndustrialVision.UI;

public class MainForm
{
    public void Start()
    {
        CameraService camera = new CameraService();
        camera.Connect();
    }
}
```

- **using IndustrialVision.Camera;** 让当前文件可以省略类型名前面的命名空间限定名。
- 不写 using 也可以使用完整名称：**IndustrialVision.Camera.CameraService**。
- using 不会创建类型、不等于添加项目/程序集引用，也不会绕过 internal 等访问限制。
- 若类型来自另一个项目/程序集，仍需正确添加项目引用、程序集引用或 NuGet 包。

## 2. namespace 与 internal：两套不同规则

程序集（Assembly）是编译产物，通常是 .dll 或 .exe。普通 .NET 项目通常生成一个主要程序集，但项目与程序集并非任何情况下都严格一对一。

```csharp
namespace IndustrialVision.Camera;

internal class CameraHelper
{
    public void Open()
    {
        Console.WriteLine("打开相机");
    }
}
```

- namespace 回答“这个类型叫什么、属于哪组名称”。
- internal 回答“哪些代码可以访问这个类型或成员”：默认仅当前程序集内的代码可访问。
- 同一程序集中的不同 namespace 可以访问 internal 类型。
- 不同程序集即使声明相同 namespace，也不能因此访问对方的 internal 类型。
- using 只简化名称，不会授予访问权限。
- InternalsVisibleTo 可以显式允许指定友元程序集访问 internal 成员，常见于测试程序集；它是例外机制，不改变 internal 的默认规则。

常见封装方式：

```csharp
namespace IndustrialVision.Camera;

internal class CameraSdkHelper
{
    public void OpenDevice()
    {
        // 在这里封装厂商 SDK 调用
    }
}

public class CameraService
{
    public void Connect()
    {
        var helper = new CameraSdkHelper();
        helper.OpenDevice();
    }
}
```

外部程序集通过 public 的 CameraService 使用功能，而不直接依赖内部辅助类 CameraSdkHelper。

## 3. 常见访问修饰符

| 修饰符 | 基本访问范围 |
|---|---|
| public | 所有能够访问该类型的代码 |
| private | 当前类型内部 |
| protected | 当前类型及派生类 |
| internal | 当前程序集内部 |
| protected internal | 当前程序集内部 **或** 派生类（OR） |
| private protected | 当前程序集内部的派生类（AND） |

注意：
- 对普通顶层 class，未写访问修饰符时默认是 internal；类成员未写访问修饰符时通常默认是 private。
- protected 不能用于普通顶层 class；它主要用于类成员和嵌套类型等场景。
- protected 的跨程序集派生类访问还有 C# 的受保护访问规则：不能把它理解为外部代码可以通过任意一个基类对象随意访问。
- public 成员仍受其外层类型可访问性的约束。

## 4. 继承（Inheritance）

继承用于表达“是一种（is-a）”关系，让派生类复用基类的状态和行为。

```csharp
public class Device
{
    public string Name { get; set; }

    public void Connect()
    {
        Console.WriteLine($"{Name} 连接设备");
    }
}

public class Camera : Device
{
    public void Capture()
    {
        Console.WriteLine("采集图像");
    }
}

var camera = new Camera
{
    Name = "Basler"
};

camera.Connect(); // 继承自 Device
camera.Capture(); // Camera 自己的方法
```

- Camera : Device 表示 Camera 继承 Device。
- Device 是基类（Base Class），Camera 是派生类（Derived Class）。
- C# 一个类只能有一个直接基类，但可以实现多个接口。
- 继承适合复用稳定的共同状态和行为；不要仅为了少写几行代码而建立不自然的继承关系。

### 4.1 protected：给派生类开放成员

```csharp
public abstract class DeviceBase
{
    public bool IsConnected { get; protected set; }

    protected void WriteLog(string message)
    {
        Console.WriteLine(message);
    }
}

public class Camera : DeviceBase
{
    public void MarkConnected()
    {
        IsConnected = true;       // 派生类可访问 protected set
        WriteLog("相机已连接");    // 派生类可调用 protected 方法
    }
}
```

外部代码可以读取 camera.IsConnected，但不能直接设置它的 protected setter，也不能直接调用 WriteLog()。这能让基类控制状态修改入口，同时允许派生类完成必要操作。

### 4.2 base：调用基类构造方法或成员

```csharp
public class Device
{
    public string Name { get; }

    public Device(string name)
    {
        Name = name;
    }

    public virtual void Disconnect()
    {
        Console.WriteLine($"{Name} 断开连接");
    }
}

public class Camera : Device
{
    public Camera(string name) : base(name)
    {
    }

    public override void Disconnect()
    {
        Console.WriteLine("停止相机采集");
        base.Disconnect();
    }
}
```

- **: base(name)** 调用基类构造方法，先完成基类部分的初始化。
- **base.Disconnect()** 调用基类的实现。
- **virtual** 表示基类提供默认实现，并允许派生类重写。
- **override** 表示派生类重写继承来的虚方法或抽象方法。

## 5. abstract：抽象类与抽象方法

抽象类使用 **abstract class** 声明，不能直接创建实例，通常作为其他类的基类。

抽象类可以包含：
- 实例字段、属性、构造方法；
- 有方法体的普通方法；
- 可由子类选择重写的 virtual 方法；
- 没有方法体、要求非抽象子类实现的 abstract 方法。

```csharp
public abstract class DeviceBase
{
    protected DeviceBase(string name)
    {
        Name = name;
    }

    public string Name { get; }

    public bool IsConnected { get; protected set; }

    // 没有方法体：非抽象子类必须实现
    public abstract void Connect();

    // 有默认实现：子类可以选择重写
    public virtual void Disconnect()
    {
        IsConnected = false;
        Console.WriteLine($"{Name} 已断开");
    }

    protected void WriteLog(string message)
    {
        Console.WriteLine($"{Name}: {message}");
    }
}
```

abstract void Connect(); 是抽象方法；它没有方法体。非抽象派生类必须用 override 实现所有继承来的抽象成员；如果暂时不实现，派生类本身也必须声明为 abstract。

| 关键字 | 是否提供方法体 | 子类是否必须重写 |
|---|---|---|
| abstract | 否 | 是（非抽象子类） |
| virtual | 是 | 否 |
| override | 重写父类实现 | 用于实现/替换继承的虚或抽象成员 |

抽象类不能直接实例化，但可以通过派生类对象来使用：

```csharp
public class BaslerCamera : DeviceBase
{
    public BaslerCamera() : base("Basler")
    {
    }

    public override void Connect()
    {
        // 教学示意：真实项目中调用对应厂商 SDK
        IsConnected = true;
        WriteLog("调用相机 SDK 完成连接");
    }
}

DeviceBase device = new BaslerCamera();
device.Connect();
```

变量声明类型是 DeviceBase，实际对象类型是 BaslerCamera。调用虚方法时会根据实际对象类型执行重写后的实现，这体现了运行时多态。

## 6. interface：接口与能力约定

接口用来描述一个类型应当提供哪些能力。实现接口的类需要实现接口要求的成员（除非接口提供默认实现，或实现类本身是抽象类）。

```csharp
public interface ICamera
{
    void Capture();

    void SetExposure(double exposure);
}
```

实现接口：

```csharp
public class BaslerCamera : ICamera
{
    public void Capture()
    {
        Console.WriteLine("Basler 采集图像");
    }

    public void SetExposure(double exposure)
    {
        Console.WriteLine($"设置曝光时间：{exposure}");
    }
}
```

- 接口不能直接实例化；可以声明接口类型变量，引用实现该接口的对象。
- 一个类可以实现多个接口。
- 接口可以继承多个接口。
- 接口可以声明方法、属性、事件和索引器；不能声明实例字段或构造方法。
- 现代 C# 接口还支持默认实现、静态抽象成员等能力；日常业务设计仍常把接口作为调用方依赖的能力约定。
- 接口成员通常隐式为 public；显式接口实现是另一个需要单独学习的语法。

示例：

```csharp
ICamera camera = new BaslerCamera();
camera.Capture();
camera.SetExposure(1200);
```

变量的静态类型是 ICamera，实际对象是 BaslerCamera。调用方依赖接口，而不是具体相机品牌类。

## 7. 抽象类与接口如何选择

| 对比 | 抽象类（abstract class） | 接口（interface） |
|---|---|---|
| 主要表达 | 一类对象的共同基础 | 一种能力/约定 |
| 能否直接实例化 | 不能 | 不能 |
| 实例字段 | 可以 | 不可以 |
| 构造方法 | 可以 | 不可以 |
| 普通方法实现 | 可以 | 现代 C# 可提供默认接口实现 |
| 抽象成员 | 可以 | 可以声明接口成员契约 |
| 一个类可采用多个 | 只能有一个直接基类 | 可以实现多个接口 |
| 常见用途 | 共享状态、构造逻辑和基础实现 | 跨不同类型统一调用方式、降低耦合 |

判断时问自己：
1. 几个类是否属于同一类对象，并需要共享字段、属性、构造过程和代码？考虑抽象基类。
2. 几个类是否只需要提供相同能力，不一定属于同一继承体系？考虑接口。
3. 两者可以一起用：类继承一个基类，同时实现多个接口。

## 8. 工业视觉示例：抽象基类 + 相机接口 + 组合

```csharp
public abstract class DeviceBase
{
    protected DeviceBase(string name) => Name = name;

    public string Name { get; }

    public bool IsConnected { get; protected set; }

    public abstract void Connect();

    public virtual void Disconnect()
    {
        IsConnected = false;
        Console.WriteLine($"{Name} 已断开");
    }
}

public interface ICamera
{
    void Capture();

    void SetExposure(double exposure);
}

public class BaslerCamera : DeviceBase, ICamera
{
    public BaslerCamera() : base("Basler")
    {
    }

    public override void Connect()
    {
        // 实际项目中在这里调用对应版本的厂商 SDK
        IsConnected = true;
    }

    public void Capture()
    {
        Console.WriteLine("采集一帧图像");
    }

    public void SetExposure(double exposure)
    {
        Console.WriteLine($"曝光时间：{exposure}");
    }
}

public class InspectionService
{
    private readonly ICamera _camera;

    public InspectionService(ICamera camera)
    {
        _camera = camera;
    }

    public void Start()
    {
        _camera.Capture();
    }
}

ICamera camera = new BaslerCamera();
var inspection = new InspectionService(camera);
inspection.Start();
```

设计意图：
- DeviceBase：设备共有的名称、连接状态和基础断开逻辑。
- ICamera：相机必须提供的采集与曝光能力。
- BaslerCamera：继承设备基础能力，并实现相机接口。
- InspectionService：拥有/使用一个相机（has-a），不继承相机；通过构造方法注入 ICamera，因此可替换不同实现。

这体现了组合（Composition）与继承（Inheritance）的区别：
- Camera is a Device：适合继承。
- InspectionService has a Camera：适合组合。

## 9. 与此前 WinForms 知识的联系

```csharp
IContainer? components = new Container();
```

- IContainer 是接口类型。
- Container 是具体类。
- new Container() 创建具体实现对象，再赋值给接口类型变量。
- IContainer 继承 IDisposable，因此可通过接口调用 components.Dispose()。

这与相机服务依赖 ICamera 的设计思想一致：调用方依赖抽象约定，具体实现可以替换。

## 10. 常见误区

1. 把 namespace 的点号当作对象成员访问符；它是命名空间限定名称的一部分。
2. 把文件夹结构当成 namespace 的强制规则；两者只是通常保持对应。
3. 认为相同 namespace 就能跨程序集访问 internal；访问权限由程序集边界决定。
4. 认为 using 能绕过 internal；using 只简化名称。
5. 认为 protected 就等于 internal；前者与继承关系有关，后者与程序集有关。
6. 为了复用代码而滥用继承；先判断是否真的是 is-a 关系。
7. 把 abstract 和 virtual 混淆；abstract 无实现并要求非抽象子类实现，virtual 有默认实现且允许重写。
8. 认为接口只能包含没有方法体的方法；现代 C# 接口支持默认实现等高级成员。
9. 认为一个类可以继承多个类；C# 只能有一个直接基类，但可以实现多个接口。
10. 把接口变量当成接口对象；接口不能直接实例化，变量引用的是实现类对象。

## 11. 动手练习

在独立控制台项目或当前 .NET 8 项目中完成：

1. 创建 IndustrialVision.Camera、IndustrialVision.PLC 两个 namespace，各声明一个同名 Device，练习完整名称和 using。
2. 在同一程序集创建 internal class CameraHelper，在另一个程序集尝试访问，观察编译器错误；再将类型改为 public 对比。
3. 创建 DeviceBase 抽象类，包含 Name、IsConnected、abstract Connect 和 virtual Disconnect。
4. 创建 ICamera 接口，包含 Capture；让 BaslerCamera 继承 DeviceBase 并实现 ICamera。
5. 创建 InspectionService(ICamera camera)，分别注入两个模拟相机实现，确保服务层代码不依赖品牌类。
6. 练习从外部代码访问 public getter、protected setter 和 protected 方法，观察访问限制。

**验证边界：** 本笔记依据文字讲解与代码示例整理；尚未记录用户在 Visual Studio 中运行上述练习或完成跨程序集编译验证。示例中的 SDK 调用均为占位说明，不代表真实相机 SDK 已接入。

## 12. 参考资料

- [Microsoft Learn：Namespaces](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/namespaces)
- [Microsoft Learn：Access modifiers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/access-modifiers)
- [Microsoft Learn：Inheritance](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance)
- [Microsoft Learn：abstract](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract)
- [Microsoft Learn：Interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces)
