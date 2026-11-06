# 2026-11-06 C# 抽象、多态与 ref 参数覆盖报告

> **P0｜待学习 / 待验证｜2026-11-06**
>
> 本报告记录本轮文字学习覆盖范围，不代表用户已独立完成代码实验或掌握全部知识。

## 1. 本轮内容

- 回顾抽象类、抽象成员：不能直接实例化；可包含字段、属性、构造器和已实现方法；非抽象子类必须实现抽象成员。
- 区分 abstract、virtual、override、new、sealed override 和 sealed class。
- 解释 ref、out、in 的参数传递语义。
- 区分值类型默认按值传递、引用类型默认按值传递对象引用、ref 引用参数。
- 用工业相机对象和 WinForms/.NET 示例联系真实上位机开发。

## 2. 关键结论

### 抽象与多态

| 关键字 | 准确含义 |
|---|---|
| abstract class | 抽象类，不能直接实例化；可包含状态、构造器和已实现成员 |
| abstract member | 没有具体实现；非抽象派生类必须实现 |
| virtual | 提供默认实现，允许派生类重写 |
| override | 重写继承来的 virtual / abstract / 可继续重写的 override 成员 |
| new（成员修饰符） | 隐藏基类同名成员，不等于 override；不建立虚分派 |
| sealed override | 重写并禁止更下一级子类继续重写该成员；当前类仍可被继承 |
| sealed class | 禁止其他类继承该类 |

没有 virtual、abstract 或可继续重写的 override 时，子类不能使用 override。可以用 new 显式隐藏同名成员；若通过基类类型变量调用非虚成员，会执行基类成员。对虚成员的 override 调用则根据实际对象类型进行运行时分派。上述规则也适用于属性。

### ref / out / in

| 关键字 | 调用前要求 | 方法内行为 |
|---|---|---|
| 普通参数 | 提供一个值 | 操作形参副本 |
| ref | 变量必须已赋值 | 可读取并修改调用方变量 |
| out | 可未初始化 | 必须在正常返回前赋值，赋值前不能读取 |
| in | 变量必须已赋值 | 通过只读引用访问，不允许给参数本身重新赋值 |

普通引用类型参数复制的是引用值：可以修改同一对象，但重新给形参赋新对象不会替换调用方变量。使用 ref Camera 才能替换调用方的引用变量。in 不是对象深度不可变保证。ref 不是裸指针，也不是线程安全机制。

## 3. 推荐实践顺序

1. 创建 AbstractRefPractice Console 项目：DeviceBase 抽象类，包含 Name、IsConnected、abstract Connect 和 virtual Disconnect；创建 Camera 派生类实现 Connect。
2. 创建 Animal / Dog：分别用非虚父类方法 + new 隐藏，以及 virtual + override 重写，比较 Animal a = new Dog() 的调用结果。
3. 给 Dog 的 override 加 sealed，尝试再派生类 override，观察编译错误；再单独尝试 sealed class 的继承限制。
4. 创建 ParameterPassingPractice：验证普通 int、ref int、out int、in struct。
5. 创建 Camera 类：比较普通参数修改对象属性、普通参数重新赋形参、ref Camera 替换外部引用。
6. 运行 int.TryParse，解释 bool 返回值和 out 结果。

## 4. Anki 与面试复习

- Anki：新增 Q211–Q223，共 13 张。
- P0 面试复习：新增抽象类与多态修饰符、sealed、ref/out/in、引用类型参数行为等口述题。
- 正式笔记：
  - [C# 面向对象专题](../01-CSharp/OOP-Types-Access-Modifiers-and-Polymorphism.md)
  - [C# 参数传递专题](../01-CSharp/Parameter-Passing-Ref-Out-In.md)

## 5. 验证边界与下一步

- 本轮完成文字讲解、源码阅读和示例设计。
- 尚未记录用户独立创建 Console 项目、编译运行示例或观察编译错误。
- 尚未将任何知识点标记为已掌握；状态保持**待学习 / 待验证**。
- 下一步先完成第 2、4、5 项练习，再用自己的话解释 new 与 override、普通引用类型参数与 ref 的区别。

## 6. 参考资料

- [Microsoft Learn：abstract](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract)
- [Microsoft Learn：virtual](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual)
- [Microsoft Learn：override](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override)
- [Microsoft Learn：new modifier](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/new-modifier)
- [Microsoft Learn：sealed](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed)
- [Microsoft Learn：ref / out / in](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters)
