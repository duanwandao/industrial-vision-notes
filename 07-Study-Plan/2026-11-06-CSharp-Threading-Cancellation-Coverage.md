# 2026-11-06 C# 线程、UI 封送与协作式取消覆盖报告

> **P0｜待学习 / 待验证｜2026-11-06**
>
> 本报告记录本轮文字学习和源码阅读覆盖范围，不代表用户已独立完成代码实验或掌握全部知识。

## 1. 本轮覆盖内容

新增 [C# 线程、线程池、WinForms UI 封送与协作式取消专题](../01-CSharp/Threading-and-Cancellation.md)，覆盖：

- WinForms UI 线程、Windows 消息队列、Application.Run 与消息循环；UI 线程执行耗时同步工作会延迟消息处理。
- InvokeRequired、Invoke、BeginInvoke 的区别；Action、Lambda、UI 封送；控件句柄尚未创建、窗体关闭竞争和 Invoke 死锁边界。
- Thread、Task、ThreadPool、Task.Run 的关系；async/await 不等于创建线程；Task.Delay 与 Thread.Sleep 的区别。
- ThreadPool 的线程复用、任务调度、线程占用和图像处理队列积压风险。
- Visual Studio 调试器中的 lambda local；Lambda 捕获外部局部变量形成闭包；C# 12 集合表达式与数组初始化器。
- Race Condition / Lost Update；Interlocked 原子操作；lock 临界区；volatile 的可见性/内存顺序语义及其不能保证复合操作原子性的边界。
- CancellationTokenSource、CancellationToken、Cancel、ThrowIfCancellationRequested、可取消 Task.Delay、Task.Run 第二个 Token 参数的实际边界、CTS 释放时机。
- WinForms 模拟采集的开始/停止生命周期；真实相机 SDK 阻塞调用和停止接口的边界。
- extern、InternalCall 与 P/Invoke 的源码阅读补充。

## 2. 重要纠错与边界

1. **Task 不等于 Thread**：Task 可以由线程池执行，也可以表示异步 I/O、Task.Delay 或已完成结果，不保证一项 Task 对应一个专属线程。
2. **async 不会自动创建后台线程**：Task.Run 才会把工作提交给线程池；await 等待未完成操作时挂起当前方法，不阻塞 UI 线程。
3. **Task.Delay 不会占用一个线程整个等待期**；Thread.Sleep 会阻塞执行它的当前线程。
4. **BeginInvoke 不创建线程**：它将委托安排到控件所属线程；Invoke 同步等待，BeginInvoke 提交后不等待。
5. **InvokeRequired 有句柄创建边界**：控件句柄尚未创建时，不能将 false 无条件解释为当前线程就是 UI 线程。
6. **volatile 不等于线程安全**：它提供特定读写可见性和内存顺序语义，不会让 counter++ 原子化；计数用 Interlocked，复合状态用 lock/其他同步原语。
7. **Cancel 不会强杀线程**：任务要检查 Token 或调用支持取消的 API；Task.Run 的 Token 不能自动中断已开始运行的委托。
8. **CTS.Dispose 不等于 Cancel**：应在相关任务结束后释放 CTS；关闭窗体时应先请求取消并等待收尾。
9. **真实相机 SDK 不能凭教学模拟推断**：阻塞采集调用、回调线程、停止 API、图像缓冲区所有权和释放规则都需要对应 SDK 文档确认。
10. 普通上位机取消逻辑不能替代急停、硬件限位、驱动器安全输入和 PLC 硬件互锁。

## 3. 文件变更

- 新增 01-CSharp/Threading-and-Cancellation.md。
- 更新 01-CSharp/Async-Events.md，补充本专题入口并校准 .NET 8 / .NET 9 WinForms API 说明。
- 更新 01-CSharp/README.md，加入线程与取消专题索引。
- 更新 05-Interview/P0-Review.md，新增线程、线程池、UI 封送、线程安全和取消口述题。
- 更新 06-Anki/CSharp.csv，新增 Q224–Q243，共 20 张卡片。
- 更新 07-Study-Plan/progress.md，记录学习范围、未验证事项和下一步。
- 更新根目录 README.md，增加本轮最新增量入口。

## 4. 建议练习顺序

1. ThreadSafetyDemo：比较普通 counter++ 与 Interlocked.Increment。
2. CancellationDemo：练习 Cancel、Token 检查、Task.Delay(token)、await 收尾。
3. WinFormsThreadDemo：比较 Invoke 与 BeginInvoke，观察 UI 线程 ID 和后台线程 ID。
4. 模拟采集：实现开始、停止、停止后重新开始，以及窗体关闭时先取消再等待。
5. 最后结合真实相机 SDK 文档核对回调线程、缓冲区所有权和停止采集流程。

## 5. 验证状态

- 用户已进行文字问答和概念辨析；没有记录用户独立创建并运行 ThreadSafetyDemo、CancellationDemo 或 WinFormsThreadDemo。
- 没有连接真实工业相机、PLC 或驱动器；没有验证具体 SDK 的回调和停止语义。
- 所有新增内容保持**待学习 / 待验证**，不因笔记已整理而标记为已掌握。
