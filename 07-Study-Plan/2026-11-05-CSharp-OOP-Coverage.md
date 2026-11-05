# 2026-11-05｜C# namespace、访问控制、继承、抽象类与接口覆盖报告

## 本轮目标

把本轮 C# 面向对象学习内容整理为可检索、可复习、可用于 .NET 工业上位机代码阅读的专题笔记，并与已有 WinForms 生命周期/可空类型知识衔接。

## 新增与修改

- 新增 01-CSharp/OOP-Types-Access-Modifiers-and-Polymorphism.md：namespace、using、程序集与 internal、访问修饰符、protected、继承、base、abstract、virtual/override、interface、组合与多态。
- 更新 01-CSharp/README.md：增加 C# 面向对象专题入口。
- 新增 00-Inbox/2026-11-05_CSharp-namespace-internal-protected-abstract-interface学习记录.md：保留原始问题、核心结论和待验证事项。
- 更新 06-Anki/CSharp.csv：新增 Q184–Q197，共 14 张卡片。
- 更新 05-Interview/P0-Review.md：增加 namespace/程序集、internal/protected、继承、抽象类和接口口述题。
- 更新 07-Study-Plan/progress.md：记录本轮学习内容、状态和下一步练习。
- 更新根目录 README.md：将本轮覆盖报告设为最新入口。

## 知识覆盖

| 模块 | 覆盖点 |
|---|---|
| 命名空间 | dotted namespace、file-scoped namespace、namespace 与文件夹、using |
| 程序集与访问控制 | Assembly、internal、namespace 与 internal 的边界、InternalsVisibleTo |
| 访问修饰符 | public、private、protected、internal、protected internal、private protected |
| 继承 | 基类/派生类、is-a、单继承、多接口、base 构造方法 |
| 多态 | virtual、override、抽象方法的运行时分派 |
| 抽象类 | 不可实例化、字段/属性/构造方法/具体方法、abstract 与 virtual |
| 接口 | 能力约定、实现接口、接口变量、多个接口、现代 C# 默认实现 |
| 设计方式 | 抽象类与接口选择、组合（has-a）与继承（is-a） |
| 项目联系 | IContainer/Container、ICamera、DeviceBase、InspectionService |

## Anki 与面试

- Anki：Q184–Q197，共 14 张；归入 06-Anki/CSharp.csv。
- 面试：增加 P0 口述题，覆盖 namespace 与程序集的区别、internal/protected 的访问边界、abstract/virtual/override、接口与抽象类的选择、单继承与多接口、组合与继承。

## 状态与边界

- 新增内容保持**待学习 / 待验证**；本轮为文字讲解与代码阅读，尚无用户独立运行示例或跨程序集验证的证据。
- 相机类是教学示例；没有调用真实 Basler/海康/大华 SDK，没有连接相机或 PLC。
- 没有修改 PLC 程序、设备配置或真实设备行为。
- 不把示例架构当作所有工业 SDK 的固定设计；实际 SDK 的接口、类层级和资源管理以具体版本文档为准。

## 下一步实践

1. 创建 .NET 8 控制台练习，验证 namespace、using 与两个程序集的 internal 访问边界。
2. 实现 DeviceBase、ICamera、BaslerCamera 和 InspectionService，练习抽象类、接口、多态与构造注入。
3. 在 Visual Studio 中通过“转到定义”查看 IContainer、Container、IDisposable 的接口/实现关系。
4. 完成练习并反馈结果后，再将掌握状态从待学习/待验证调整为已验证。
