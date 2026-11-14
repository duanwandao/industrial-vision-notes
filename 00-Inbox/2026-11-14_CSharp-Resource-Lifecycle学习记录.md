# 2026-11-14｜C# IDisposable、using 与异步资源释放学习记录

> 原始学习问题与本轮概念澄清记录。本文是学习素材，不代表用户已完成代码实验或掌握全部内容。

## 用户提出的问题

1. 学习 IDisposable、using、IAsyncDisposable 和异步资源释放。
2. “啥叫释放对象占用的资源？还是没很理解，意思是不光是内存对象的问题？”
3. 关于 _cts.Cancel() 只是通知后台任务、不能证明任务停止，用户追问：是否应该 await _cts.Cancel()？

## 整理后的关键结论

- C# 托管对象不只是托管内存；它可能包装 Windows 文件/串口/Socket 句柄、GDI+ 原生资源或工业相机 SDK 的原生资源。
- GC 回收不可达托管对象内存；Dispose 让对象所有者及时释放对象管理的资源。把变量设为 null 不会自动调用 Dispose。
- IDisposable 提供同步 Dispose；using / using var 在作用域退出时调用 Dispose，包括异常和提前 return 路径。
- 资源所有权决定谁负责释放：自己创建并拥有的资源通常由自己释放；借用外部资源不能擅自 Dispose。
- IAsyncDisposable 提供 ValueTask DisposeAsync；await using 会在作用域退出时等待异步释放完成。它不是所有 IDisposable 的默认替代品。
- CancellationTokenSource.Cancel() 返回 void，不能 await；它发出协作式取消请求，后台任务必须检查 Token 或调用支持取消的 API。
- .NET 8 的 CancellationTokenSource.CancelAsync() 返回 Task，等待取消回调 / 可取消操作注册完成；它不等于等待代表整个后台工作的 Task。
- 要确认后台任务真正结束，需要 await workerTask；相关任务结束后再 Dispose CTS。
- WinForms FormClosing 是同步事件；可在 async void 事件处理器中先设置 e.Cancel = true，再 await 服务清理，完成后重新 Close。Form.Dispose(bool) 仍是同步方法。

## 本轮待验证事项

- 用 TraceResource 验证 using 在正常、return、异常路径都会 Dispose。
- 用虚拟串口理解 Close 与 Dispose 的差异。
- 在 .NET 8 对比 Cancel、CancelAsync 和 await workerTask 的不同等待边界。
- 编写 IAsyncDisposable 检测服务，使用 await using 验证取消、等待和 CTS 释放顺序。
- 在 WinForms 模拟项目验证 FormClosing 异步收尾和重复关闭保护。
- 具体工业相机 SDK 的停止采集、回调退订、图像缓冲区归还和 Dispose 顺序待查实际厂商/版本文档。
