# 2026-11-06 C# 抽象、多态与 ref 参数学习记录

> 来源：2026-11-06 与 ChatGPT 的连续学习对话。此文件保留本轮问题与学习线索；正式解释见对应专题笔记。

## 用户提出的问题

- .NET WinForms 的 Font 属性为什么声明为 public virtual Font Font？没有 virtual 时，子类还能否 override？
- sealed 是否意味着不能继续重写？sealed override 与 sealed class 有什么区别？
- out 参数是什么？为什么 GetCurrentFontAndDpi(out int fontDpi) 可以同时返回 Font 和 DPI？
- abstract 用来做什么？如何与 virtual、override、接口区分？
- 学习 ref，并理解它与 out、普通参数以及引用类型参数的区别。

## 本轮学习结论（待复述、待验证）

1. abstract class 不能直接实例化，可以包含字段、属性、构造器、普通方法、virtual 成员和 abstract 成员。非抽象派生类必须实现继承的抽象成员。
2. abstract 成员没有具体实现；virtual 成员有默认实现且允许重写；override 重写可重写成员。
3. 没有 virtual、abstract 或可继续重写的 override 时，子类不能使用 override；可以用 new 隐藏同名成员，但隐藏不等于多态重写。
4. sealed override 阻止更下一级子类继续重写该成员，但不禁止继承当前类；sealed class 禁止其他类继承它。
5. ref 在方法声明和调用处都要标记；调用前变量必须已赋值。方法通过引用访问调用方的变量，可以读取和修改它。
6. 普通引用类型参数复制的是对象引用：方法可以修改同一个对象的内容，但给形参重新赋一个对象，不会替换调用方的变量。使用 ref 才能替换调用方的引用变量。
7. out 是输出参数，调用前可不初始化，但方法必须在正常返回前赋值；in 是只读引用参数，不允许在方法内给参数重新赋值，但不代表对象深度不可变。
8. out _ 表示丢弃不需要的输出值。

## 后续实践

- 在 Console 项目中验证 abstract、virtual、override、new、sealed override 与 sealed class。
- 对比 ChangeNumber(int) 与 ChangeNumber(ref int)。
- 对比普通引用类型参数修改对象属性、重新赋形参，以及 ref Camera 替换外部引用。
- 对比 ref、out、in，运行 int.TryParse。
- 将示例放入 .NET 8 Console 项目独立编译运行；本记录本身不代表已完成验证。

## 正式笔记

- [C# 命名空间、访问控制、继承、抽象类与接口](../01-CSharp/OOP-Types-Access-Modifiers-and-Polymorphism.md)
- [C# 参数传递：ref、out 与 in](../01-CSharp/Parameter-Passing-Ref-Out-In.md)
