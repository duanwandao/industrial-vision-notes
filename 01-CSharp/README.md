# C# / .NET 笔记索引

面向 C# 工业上位机与工业机器视觉应用的基础知识。

## 专题

- [WinForms 窗体生命周期与可空类型](WinForms-Lifecycle-and-Nullability.md)：partial、构造方法、InitializeComponent、Dispose、IDisposable、Nullable<T>、null 运算符。
- [异步、事件与 UI 线程](Async-Events.md)：Task、async/await、事件订阅、UI 线程、取消与 TaskCompletionSource。

## 学习建议

1. 先理解 C# 类型、对象、构造和资源生命周期。
2. 再学习 WinForms 事件、异步任务、线程与 UI 更新。
3. 把这些知识用于相机 SDK 采集、PLC 通信和检测流程协调。
4. 所有设备资源的停止与释放顺序，以实际 SDK 文档和设备型号为准。
