# C# 语法进阶：对象构造、record、模式匹配与 unsafe 指针

> **状态：待学习 / 待验证｜2026-11-12**
> 本篇整理近期阅读 WinForms 工具方法、设备通信结果类型和 .NET string.Trim() 源码时遇到的语法。示例用于理解语言机制，不代表已经在用户项目中独立编译或验证。

## 1. 表达式体成员：`=>`

方法只有一个表达式时，可以用表达式体语法：

~~~csharp
public static int Add(int a, int b)
    => a + b;
~~~

等价于：

~~~csharp
public static int Add(int a, int b)
{
    return a + b;
}
~~~

`=>` 在表达式体成员和 Lambda 中都出现，但含义要结合语法位置判断：这里是方法体简写，不是在声明 Lambda。

## 2. 目标类型推断的 `new()` 与对象初始化器

~~~csharp
Label label = new Label
{
    Name = "lblTitle",
    Text = "设备状态",
    AutoSize = true
};
~~~

可简写为：

~~~csharp
Label label = new()
{
    Name = "lblTitle",
    Text = "设备状态",
    AutoSize = true
};
~~~

编译器从左侧目标类型 Label 推断 `new()` 创建 Label 实例。对象初始化器 `{ Name = ..., Text = ... }` 在构造对象后设置可访问的字段/属性；它不意味着 Label 一定有接收这些成员作为参数的构造函数。

### 2.1 拆解 WinForms Label 工厂方法

~~~csharp
public static Label Label(
    string name,
    string text,
    float size = 10,
    bool bold = false)
    => new()
    {
        Name = name,
        Text = text,
        AutoSize = true,
        ForeColor = Text,
        Font = new Font(
            "Microsoft YaHei UI",
            size,
            bold ? FontStyle.Bold : FontStyle.Regular),
        Anchor = AnchorStyles.Left,
        Margin = new Padding(0, 2, 0, 2)
    };
~~~

- 第一个 Label 是返回类型；第二个 Label 是方法名。若此方法位于 UiTheme，可用 UiTheme.Label(...) 调用。
- static 表示通过类型调用，不需要创建 UiTheme 实例。
- size = 10、bold = false 是可选参数；调用方可以省略，或用命名参数只指定 bold: true。
- bold ? FontStyle.Bold : FontStyle.Regular 是三元条件运算符，返回两个 FontStyle 值之一。
- ForeColor = Text 右侧的 Text 是外围作用域中的颜色成员（例如 UiTheme.Text），不是小写参数 text；C# 区分大小写。
- new Padding(left, top, right, bottom) 设置外边距；此处是左右 0、上下 2。Margin 是外边距，Padding 是内边距。
- AnchorStyles.Left 表示控件锚定到容器左侧；布局行为仍取决于父容器尺寸和其他属性。

## 3. 位置 record、sealed 与 with

~~~csharp
public sealed record LightBrightnessWriteResult(
    int Channel,
    int Brightness,
    string Command,
    string Response,
    bool Acknowledged);
~~~

这是位置 record class 声明：

- record 声明用于承载数据的记录类型；默认是引用类型。record struct 才是值类型。
- 括号中的参数会生成构造函数参数和对应的公共 init 属性。
- init 允许在构造/初始化阶段设置属性，通常不能在普通后续赋值中修改。
- sealed 禁止其他类型继承该记录。
- record 默认按成员值进行相等性比较；普通 class 默认比较引用身份（除非重写相等性）。
- with 会复制一个 record 并修改指定成员；这是浅复制，引用类型属性仍可能共享内部对象。

~~~csharp
var first = new LightBrightnessWriteResult(
    1, 80, "SET80", "$", true);

var changed = first with { Brightness = 100 };
~~~

Acknowledged 应表达“收到符合设备协议的确认响应”，不能仅因为 SerialPort.Write 没有抛异常就设为 true；ACK 也不一定证明物理亮度已经达到目标值。

## 4. switch 表达式与模式匹配

~~~csharp
Color color = level switch
{
    "ERROR" => UiTheme.Ng,
    "WARN" => Color.DarkOrange,
    "TX" => Color.DeepSkyBlue,
    "RX" => Color.MediumSeaGreen,
    _ => Color.FromArgb(226, 232, 240)
};
~~~

- level switch { ... } 是 switch 表达式，会产生一个值，因此可以直接赋值。
- 每个分支称为 arm；模式 => 结果。
- _ 是丢弃模式，匹配其他未命中的情况，类似传统 switch 的 default。
- 这里的 => 是 switch arm 分隔符，不是 Lambda。

关系模式与组合模式：

~~~csharp
if (channel is < 1 or > ChannelCount)
{
    throw new ArgumentOutOfRangeException(nameof(channel));
}
~~~

等价于：

~~~csharp
if (channel < 1 || channel > ChannelCount)
{
    throw new ArgumentOutOfRangeException(nameof(channel));
}
~~~

< 1、> ChannelCount 是关系模式；or 是模式组合符。模式里的 or/and/not 与布尔逻辑运算符 ||/&&/! 不是同一套语法。

## 5. params char[]? 与 Trim

.NET 的一个 Trim 重载形如：

~~~csharp
public string Trim(params char[]? trimChars);
~~~

- char[]：字符数组。
- ?：在启用可空引用类型注解时，表示数组引用允许为 null；它不是“数组元素可空”。
- params：调用方可以传入一个 char[]，也可以在调用位置直接传入多个 char，编译器会构造参数数组。

~~~csharp
string normalized = response.Trim('\r', '\n', '\0', ' ');
~~~

此调用要求从字符串开头和结尾删除属于指定集合的字符：回车、换行、NUL 和普通空格。它不是删除一个连续子串，也不会删除字符串中间的这些字符。未列入集合的制表符 \t 不会被这次调用专门删除。

字符与字符串要区分：

~~~csharp
char nulCharacter = '\0'; // 一个 NUL 字符
string nulString = "\0";  // 含一个 NUL 字符的字符串
string? noString = null;  // 没有字符串引用
~~~

'\0' 不等于 null。Trim 返回处理后的字符串，不会原地修改原字符串；字符串是不可变类型。若字符串本来不需要裁剪，运行时实现也可能直接返回原实例，不应依赖对象引用是否相同。

## 6. 阅读 .NET Trim 源码：unsafe、指针与 fixed

示例源码：

~~~csharp
public unsafe string Trim(params char[]? trimChars)
{
    if (trimChars == null || trimChars.Length == 0)
    {
        return TrimWhiteSpaceHelper(TrimType.Both);
    }

    fixed (char* pTrimChars = &trimChars[0])
    {
        return TrimHelper(pTrimChars, trimChars.Length, TrimType.Both);
    }
}
~~~

### 6.1 unsafe

unsafe 表示此方法体允许使用指针等不安全操作。它不是“代码一定不安全”，而是编译器允许开发者直接操作内存地址；越界或错误地址可能导致内存破坏或进程崩溃。

项目需要允许不安全代码：

~~~xml
<PropertyGroup>
  <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
</PropertyGroup>
~~~

### 6.2 char*

~~~csharp
char* pTrimChars;
~~~

char* 是指向 char 数据的指针，存放内存地址。它不同于 char[]：

- char[] 是由 GC 管理的托管数组对象，知道自身长度。
- char* 是裸指针，只表示地址，不自动携带数组长度、边界或所有权信息。

因此 TrimHelper 还要接收 trimChars.Length。

C# char 是 16 位 UTF-16 代码单元。对 char* 做 p + 1 指针运算，会移动一个 char 元素的大小（通常 2 字节），不是简单加 1 字节。

### 6.3 & 取地址

~~~csharp
&trimChars[0]
~~~

先通过 trimChars[0] 取得数组首元素，再用 & 获取该元素的地址。数组为空时不能访问 [0]，因此源码先处理 null 和空数组。

### 6.4 fixed 固定托管对象

~~~csharp
fixed (char* pTrimChars = &trimChars[0])
{
    return TrimHelper(
        pTrimChars,
        trimChars.Length,
        TrimType.Both);
}
~~~

托管堆对象可能被 GC 移动。fixed 在其作用域内固定数组所在位置，使指针在该作用域中保持有效；退出作用域后解除固定，GC 可以再次移动对象。

fixed 不会复制数组，也不会永久锁住对象。固定区域应尽量短，不能把指针保存起来在作用域外继续使用。

### 6.5 为什么框架源码使用指针？

底层实现可以借助指针扫描连续内存，减少部分托管访问开销；是否更快仍取决于实现、JIT 和工作负载，不能认为 unsafe 一定更快。

日常 WinForms、SerialPort 和设备 SDK 集成通常不需要自己写指针。机器视觉中可能在 Bitmap 像素缓冲区、原生 C/C++ SDK、图像缓冲区互操作时遇到指针；应优先遵循库提供的安全 API、Span<T> / Memory<T> 和明确的缓冲区生命周期约定。

## 7. 自测

1. public static Label Label(...) => new() { ... }; 中两个 Label 分别是什么？为什么 new() 能推断类型？
2. record 与 record struct 的值/引用类型区别是什么？with 是否深复制？
3. level switch { ... } 为什么是表达式？_ 和 default 有何对应关系？
4. channel is < 1 or > ChannelCount 与 channel < 1 || channel > ChannelCount 有什么关系？
5. params char[]? 中 params、[]、? 各表示什么？
6. Trim('\r','\n') 是否会删除字符串中间的换行？'\0' 是否等于 null？
7. char* 为什么还需要单独传递长度？
8. fixed 的作用域结束后，为什么不能继续使用里面取得的指针？

## 8. 学习状态与边界

本轮以文字讲解和源码阅读为主。尚无用户独立运行 unsafe 示例、开启 AllowUnsafeBlocks 编译验证或对 .NET 具体版本源码逐行调试的证据。源码实现可能随 .NET 版本变化；以当前目标框架对应的运行时源码为准。
