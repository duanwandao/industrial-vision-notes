# 2026-11-14 C# WinForms 消息循环与同步上下文学习记录

> 原始学习记录｜来源：本轮对话中的文字讲解与问答｜状态：待学习 / 待验证

## 本轮问题与学习内容

- 继续学习 Invoke 与 BeginInvoke：Invoke 同步等待 UI 委托执行完成；BeginInvoke 只异步投递，不等待。
- 用户正确判断：如果 Thread.Sleep(2000) 写在 Invoke/BeginInvoke 的委托内部，Sleep 仍在 UI 线程执行；把 Invoke 改为 BeginInvoke 并不能消除委托本身造成的 UI 卡顿。
- 学习控件句柄：区分 .NET Control 对象与 Windows HWND；IsHandleCreated 用于检查句柄状态；读取 Handle 可能触发创建；句柄可能在生命周期中销毁或重建。
- 学习 InvokeRequired 的边界：句柄和父级句柄都未创建时，InvokeRequired 可能返回 false，不能把 false 无条件解释为当前线程就是 UI 线程。对话中已纠正这一点。
- 学习 Windows 消息循环：Application.Run 在当前线程运行 WinForms 消息循环；消息循环处理窗口消息；WndProc 是窗口消息处理入口；UI 线程执行长时间同步代码会延迟消息处理。
- 学习 SynchronizationContext：Current 获取当前线程上下文；WinForms UI 线程通常是 WindowsFormsSynchronizationContext；Post 异步投递；Send 同步发送。
- 学习 await 默认上下文捕获、ConfigureAwait(false) 的边界，以及 Progress<T> 在构造时捕获上下文并向 UI 汇报低频进度。
- 工业视觉边界：高帧率相机图像不能无节制地逐帧投递 UI；需考虑限频、合并最新帧或有界队列。相机 SDK 缓冲区是否复用、复制与释放时机必须按具体 SDK 文档确认。

## 待验证事项

1. 在 .NET 8 WinForms 项目中比较构造函数、Load、Shown 阶段的 IsHandleCreated 与 SynchronizationContext.Current。
2. 用后台任务分别调用 SynchronizationContext.Post 和 Send，观察线程 ID 与 UI 响应；单独理解 Send 的死锁风险，不要在 UI 线程中制造真实死锁。
3. 比较 await Task.Delay、await Task.Run、ConfigureAwait(false) 前后线程 ID。
4. 在 UI 线程创建 Progress<int>，后台线程调用 Report，观察进度回调执行线程。
5. 将上述机制整合到 IndustrialVisionInspection 的模拟检测流程；真实相机 SDK 的回调线程、停止采集和缓冲区所有权待确认型号与文档后验证。

## 学习状态

本轮为文字讲解和概念问答；尚未记录独立编译运行、断点观察或真实相机 SDK 验证。不要因笔记已整理而标记为已掌握。
