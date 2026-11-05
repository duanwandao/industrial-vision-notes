# 2026-11-05｜C# namespace、访问控制、继承、抽象类与接口学习记录

> 来源：本轮 ChatGPT 文字学习与用户提问。保留原始问题摘要；正式解释见 C# 专题笔记。
>
> 学习状态：待学习 / 待验证。本轮未记录用户运行示例、跨程序集编译验证或独立完成代码练习的证据。

## 原始问题摘要

1. namespace 为什么可以写成 IndustrialVision.Camera，点号表示什么？
2. namespace 和 internal 有什么关系？使用场景有什么区别？
3. 学习 protected 和继承（Inheritance）。
4. 学习 interface（接口），理解类如何实现接口、接口类型变量如何引用实现类对象。
5. 抽象类 abstract 与 interface 有什么区别？在工业视觉项目中如何组合使用？

## 本轮整理结论

- namespace 是类型名称的逻辑组织与名称解析机制；点号用于限定命名空间名称，不是对象成员访问符。文件夹与 namespace 通常对应，但不是强制绑定。
- using 用于简化类型名称，不等于添加程序集引用，也不会绕过访问权限。
- internal 默认限制在当前程序集内；同程序集不同 namespace 可访问，不同程序集即使 namespace 相同也不能默认访问。
- protected 面向当前类型和派生类；protected internal 是同程序集或派生类（OR），private protected 是同程序集内派生类（AND）。
- 继承表达 is-a 关系；C# 类只能有一个直接基类，但可以实现多个接口。base(...) 调用基类构造方法，base.Member() 调用基类成员。
- 抽象类不能直接实例化，可以包含字段、属性、构造方法、具体方法、virtual 方法和 abstract 方法。
- abstract 方法没有方法体，非抽象派生类必须 override；virtual 方法有默认实现，派生类可以选择 override。
- interface 主要定义能力约定；实现类需满足接口契约。现代 C# 接口也支持默认实现等高级能力，不能简单理解成“只能写方法声明”。
- 抽象类适合共享状态与基础实现；接口适合跨不同类型统一能力。一个类可以继承一个基类并实现多个接口。
- InspectionService 使用 ICamera 是组合/依赖关系（has-a），而 Camera : DeviceBase 是继承关系（is-a）。
- IContainer components = new Container() 是接口变量引用具体实现类对象的例子；IContainer 继承 IDisposable。

## 待验证事项

- 在 .NET 8 控制台项目中验证 file-scoped namespace 与 block-scoped namespace。
- 创建两个程序集，分别验证 internal 在同程序集不同 namespace、不同程序集相同 namespace 下的访问边界。
- 运行 DeviceBase / ICamera / BaslerCamera / InspectionService 示例，观察 abstract、virtual、override 和接口多态。
- 从外部代码尝试访问 protected setter 和 protected 方法，确认访问限制。
- 后续对接真实相机 SDK 时，必须以实际厂商、型号和 SDK 版本文档为准；本轮代码是教学结构，不代表已接入设备。

正式笔记：[C# 命名空间、访问控制、继承、抽象类与接口](../01-CSharp/OOP-Types-Access-Modifiers-and-Polymorphism.md)。
