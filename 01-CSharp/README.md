# C# / .NET 笔记索引

面向 C# 工业上位机与工业机器视觉应用的基础知识。

## 专题

- [WinForms 布局、对话框与数据绑定](WinForms-Layout-and-Data-Binding.md)：TableLayoutPanel、FlowLayoutPanel、DataGridView 与 ListBox、模态对话框、DialogResult、using/Dispose、is/not/or 模式、BindingList<T>、INotifyPropertyChanged、BindingSource、ListChanged 源码、Filter/Sort 能力边界、DisplayMember/DisplayText 和对话框验证交互。

- [C# 委托与事件](Delegates-and-Events.md)：Delegate、Action/Func/Predicate、多播委托、Callback、event、EventHandler<TEventArgs>、WinForms 与相机事件、线程与退订生命周期。

- [命名空间、访问控制、继承、抽象类与接口](OOP-Types-Access-Modifiers-and-Polymorphism.md)：namespace、using、Assembly、internal/protected、继承、abstract、virtual/override、interface 与组合。
- [WinForms 窗体生命周期与可空类型](WinForms-Lifecycle-and-Nullability.md)：partial、构造方法、InitializeComponent、Dispose、IDisposable、Nullable<T>、null 运算符。
- [异步编程与 UI 线程](Async-Events.md)：Task、async/await、取消、UI 线程与 TaskCompletionSource；委托/事件基础见上方专题。
- [线程、线程池、WinForms UI 封送与协作式取消](Threading-and-Cancellation.md)：Thread/Task/ThreadPool、消息循环、Invoke/BeginInvoke、竞态条件、Interlocked/lock/volatile、CancellationToken。

- [C# 参数传递：ref、out 与 in](Parameter-Passing-Ref-Out-In.md)：默认按值传递、引用类型参数、ref/out/in、Try 模式与参数选择。

## 学习建议

1. 先理解 C# 类型、对象、构造和资源生命周期。
2. 再学习 WinForms 事件、异步任务、线程与 UI 更新。
3. 把这些知识用于相机 SDK 采集、PLC 通信和检测流程协调。
4. 所有设备资源的停止与释放顺序，以实际 SDK 文档和设备型号为准。
