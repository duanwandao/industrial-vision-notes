# C# 参数传递：ref、out 与 in

> **P0｜待学习 / 待验证｜2026-11-06**
>
> 面向 .NET 8 / C# 工业上位机开发。本文解释 C# 参数传递语义；示例中的 Camera 是教学类，不代表真实厂商 SDK。

## 1. 默认规则：普通参数按值传递

~~~csharp
static void ChangeNumber(int number)
{
    number = 100;
}

int a = 10;
ChangeNumber(a);
Console.WriteLine(a); // 10
~~~

默认情况下，方法接收的是实参值的副本。对 number 赋值不会改变调用方的变量 a。

- 对值类型（如 int、double、struct），复制的是值。
- 对引用类型（如 class），复制的是对象引用；不是复制整个对象。

这两种情况都属于默认的按值传递，只是“值”本身不同。

## 2. ref：让方法通过引用访问调用方变量

~~~csharp
static void ChangeNumber(ref int number)
{
    number = 100;
}

int a = 10;
ChangeNumber(ref a);
Console.WriteLine(a); // 100
~~~

ref 使形参通过引用指向调用方传入的变量位置。方法可以读取原值，也可以修改调用方变量。

### 必须遵守的规则

1. 方法声明处写 ref：void Change(ref int number)。
2. 调用处也写 ref：Change(ref a)。
3. 调用前变量必须已明确赋值，因为方法可以读取原值。
4. 方法不强制必须修改它；是否修改由方法实现决定。
5. 传入的必须是可作为变量传递的存储位置；普通属性通常不能直接作为 ref 实参。

ref 是托管语言的按引用参数机制，不是让开发者直接操作裸指针。

## 3. out：通过参数输出结果

~~~csharp
static int Calculate(int a, int b, out int difference)
{
    difference = a - b;
    return a + b;
}

int difference;
int sum = Calculate(10, 20, out difference);

Console.WriteLine(sum);        // 30
Console.WriteLine(difference); // -10
~~~

一个方法可以通过 return 返回主要结果，再通过一个或多个 out 参数输出其他结果。

### out 规则

- 声明处和调用处都写 out。
- 调用前不要求变量已初始化。
- 方法必须保证在每条正常返回路径上都给 out 参数赋值。
- 方法在给 out 参数赋值之前不能读取它的值。

常见的 Try 模式：

~~~csharp
bool success = int.TryParse("123", out int number);
// success == true，number == 123

bool failed = int.TryParse("hello", out int other);
// failed == false，other == 0
~~~

Try 模式通常用 bool 表示操作是否成功，用 out 提供解析结果。

## 4. in：只读引用参数

~~~csharp
struct Point
{
    public double X;
    public double Y;
}

static double GetLength(in Point point)
{
    return Math.Sqrt(point.X * point.X + point.Y * point.Y);
}

var p = new Point { X = 3, Y = 4 };
double length = GetLength(in p); // 5
~~~

in 表示按只读引用传递参数：

- 调用方必须提供一个已赋值的参数。
- 方法内不能给 point 这个参数本身重新赋值。
- 调用时可以写 in；在很多调用场景下也可以省略。
- 它常用于较大的结构体，避免不必要的值复制；是否有性能收益应结合类型大小和实际测量判断。
- 对引用类型而言，in 不等于对象深度不可变：它限制参数引用本身被重新赋值，不会自动禁止通过该引用修改对象内部状态。

## 5. ref、out、in 对比

| 对比 | 普通参数 | ref | out | in |
|---|---|---|---|---|
| 参数传递 | 按值 | 按引用 | 按引用 | 按只读引用 |
| 调用前是否需初始化 | 需要提供值 | 必须已赋值 | 不需要 | 必须已赋值 |
| 方法内能否读取初始值 | 可以 | 可以 | 赋值前不可以 | 可以 |
| 方法内能否给参数重新赋值 | 可以（只影响形参副本） | 可以，影响调用方变量 | 必须赋值 | 不可以 |
| 典型目的 | 传入数据 | 修改调用方变量 | 输出额外结果 | 只读访问，特定场景避免复制 |

## 6. 最容易混淆的地方：引用类型参数与 ref

先定义一个教学类：

~~~csharp
class Camera
{
    public string Name { get; set; }
    public string Status { get; set; }

    public Camera(string name)
    {
        Name = name;
        Status = "未连接";
    }
}
~~~

### 6.1 普通引用类型参数：能改对象内容

~~~csharp
static void MarkConnected(Camera camera)
{
    camera.Status = "已连接";
}

var camera = new Camera("Basler-001");
MarkConnected(camera);

Console.WriteLine(camera.Status); // 已连接
~~~

调用时复制的是对象引用。调用方的 camera 与方法参数 camera 指向同一个对象，因此通过参数修改对象属性，调用方能看到变化。

### 6.2 普通引用类型参数：重新给形参赋对象，不会替换外部变量

~~~csharp
static void ReplaceLocal(Camera camera)
{
    camera = new Camera("新相机");
}

var camera = new Camera("旧相机");
ReplaceLocal(camera);

Console.WriteLine(camera.Name); // 旧相机
~~~

方法内的形参只是调用方引用值的副本。重新给形参赋值，只改变副本本身。

### 6.3 ref 引用类型参数：可以替换调用方变量

~~~csharp
static void ReplaceCaller(ref Camera camera)
{
    camera = new Camera("新相机");
}

var camera = new Camera("旧相机");
ReplaceCaller(ref camera);

Console.WriteLine(camera.Name); // 新相机
~~~

这里 ref 让方法直接访问调用方的引用变量，因此可以让外部变量改为指向另一个对象。

**记住：**

- 普通引用类型参数：可以修改它指向的对象，但不能替换调用方的引用变量。
- ref Camera：既可以修改对象，也可以替换调用方的引用变量。

## 7. out _：丢弃不需要的输出

~~~csharp
Font font = GetCurrentFontAndDpi(out _);
~~~

如果方法有一个 out 参数，但当前调用方不需要它，可以使用丢弃符号 _。

例如 WinForms 内部方法同时返回 Font 和 DPI；只读取 Font 的属性 getter 可以忽略 DPI 输出。

## 8. 什么时候使用哪一种？

- **普通参数**：默认选择。只需要读取传入值，或通过引用类型参数修改对象内容时，通常不需要 ref。
- **ref**：调用方明确需要被调用方法修改或替换自己的变量时使用。不要为了“少写一个 return”而到处使用 ref。
- **out**：方法需要输出额外结果，尤其是 TryParse 这类“成功/失败 + 结果”的模式。
- **in**：确实需要只读引用语义，且大型结构体等场景经过评估有价值时使用。
- 如果要返回多个有明确业务含义的结果，也可以考虑 tuple 或结果类型；不必为了多个结果一律使用 out。

## 9. 工业上位机中的联系

- 协议解析可以采用 Try 模式：返回是否解析成功，通过 out 返回解析后的值。
- 设备对象通常通过普通引用类型参数传递；修改连接状态等对象内容，外部可以观察到变化。
- 只有当方法的职责确实是替换调用方持有的对象引用时，才考虑 ref Camera。
- 不要把 ref 当作线程安全机制；多个线程同时读写同一对象仍然需要同步设计。
- 不要把 ref 当作图像缓冲区所有权管理方案；图像生命周期仍以 SDK 文档为准。

## 10. 常见误区

1. 认为普通引用类型参数会复制整个对象；默认复制的是对象引用。
2. 认为普通引用类型参数能替换调用方变量；它只能重新赋值自己的形参副本。
3. 忘记在 ref / out 声明和调用两处都写对应关键字。
4. 使用 ref 时忘记调用前必须初始化变量。
5. 使用 out 时遗漏某条正常返回路径上的赋值。
6. 认为 in 会让引用类型对象完全不可变；它只限制参数引用本身重新赋值。
7. 认为 ref 就是裸指针；C# ref 是受类型系统和运行时管理的引用参数机制。
8. 把 ref 当作默认设计；多数业务 API 用普通参数、返回值或结果类型更清楚。

## 11. 动手练习

在 .NET 8 Console 项目 ParameterPassingPractice 中完成：

1. 写 ChangeNumber(int) 和 ChangeNumber(ref int)，比较调用后的变量。
2. 写 Calculate(int, int, out int)，通过 return 返回和，通过 out 返回差。
3. 使用 int.TryParse，分别测试成功、失败和 out var。
4. 创建 Camera 类；用普通参数修改 Status，观察外部对象变化。
5. 用普通参数给形参重新赋新 Camera，再用 ref Camera 重做，比较外部 Name。
6. 定义一个 Point struct，使用 in Point 读取坐标；尝试给参数重新赋值，观察编译器限制。
7. 在 WinForms / SDK 代码中找一个 out 参数，解释调用方是否真的需要接收它。

**完成标准：** 能不看笔记解释普通参数、ref、out、in 的区别，并通过运行结果证明“修改对象内容”与“替换调用方引用”不是一回事。

## 12. 参考资料

- [Microsoft Learn：ref keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref)
- [Microsoft Learn：out parameter modifier](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/out-parameter-modifier)
- [Microsoft Learn：in parameter modifier](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters#in-parameter-modifier)
- [Microsoft Learn：Method parameters](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters)
