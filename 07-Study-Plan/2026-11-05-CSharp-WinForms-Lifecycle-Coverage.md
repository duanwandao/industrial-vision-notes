# 2026-11-05｜C# partial、WinForms 生命周期与可空类型覆盖报告

## 本轮目标

整理用户围绕 IndustrialVisionInspection 项目提出的 C# 问题，建立可复习、可实践、可用于 C# 工业上位机面试的知识链条。

## 新增与修改

- 新增 `01-CSharp/WinForms-Lifecycle-and-Nullability.md`：partial、构造方法、InitializeComponent、动态布局、SuspendLayout/ResumeLayout、Dispose、IDisposable、Close/Dispose、Nullable<T>、可空引用注解及问号运算符。
- 新增 `01-CSharp/README.md`：C# 专题入口。
- 更新 `01-CSharp/Async-Events.md`：增加窗体生命周期与资源释放专题的关联入口。
- 新增 `00-Inbox/2026-11-05_CSharp-partial-WinForms生命周期与可空类型学习记录.md`：保留本轮问题摘要、结论与待验证事项。
- 更新 `06-Anki/CSharp.csv`：新增 Q170–Q183 共 14 张卡片。
- 更新 `05-Interview/P0-Review.md`：增加 WinForms 生命周期与 C# 可空类型口述题。
- 更新 `07-Study-Plan/progress.md`：记录学习内容、状态与下一步验证任务。
- 更新根目录 `README.md`：增加最新增量入口和 C# 笔记索引入口。

## 知识覆盖

| 模块 | 覆盖点 |
|---|---|
| 类型组织 | partial 声明、同一类型、private 成员共享、设计器文件边界 |
| 窗体生命周期 | new、构造方法、InitializeComponent、InitializeLayout、InitializeBehavior、OnLoad、Close、Dispose |
| 布局 | SuspendLayout、ResumeLayout |
| 资源管理 | IContainer、Timer 注册、IDisposable、using、基类 Dispose |
| 可空类型 | Nullable<T>、int?、string?、IContainer?、HasValue、Value、GetValueOrDefault |
| 运算符 | ?.、??、??=，以及 null 与 0/空字符串的区别 |

## Anki 与面试

- Anki：Q170–Q183，共 14 张；一题一知，归入 CSharp.csv。
- 面试：补充 P0 口述题，重点检查 partial 与继承、构造与初始化顺序、组件容器与控件树、Close/Dispose、可空值类型与可空引用注解。

## 状态与边界

- 所有新增内容保持**待学习 / 待验证**；用户本轮提出了问题并接受讲解，但尚无代码运行或断点观察证据。
- 当前项目中的 Form1 代码仅作为代码结构示例，不代表已经连接工业相机或 PLC。
- 没有执行真实设备控制、PLC 程序修改、相机采集或现场资源释放测试。
- 对相机 SDK、PLC 通信库的具体清理顺序，必须以后续实际型号和厂商文档为准。

## 下一步实践

在 .NET 8 WinForms 模拟项目中设置断点观察构造与关闭顺序；再编写小型 IDisposable 示例，验证 using 在正常退出和异常退出时都会释放资源。
