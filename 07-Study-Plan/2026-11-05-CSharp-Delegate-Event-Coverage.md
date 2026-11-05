# 2026-11-05｜C# Delegate 与 Event 覆盖报告

## 本轮目标

将近期学习的 C# 委托、内置泛型委托、回调和事件机制整理为独立专题，并连接到 WinForms 与工业相机 SDK 的实际使用场景。

## 新增与修改

- 新增 `01-CSharp/Delegates-and-Events.md`：Delegate、Action/Func/Predicate、多播委托、Callback、event、事件访问器、sender、EventArgs、EventHandler<TEventArgs>、WinForms/相机事件、线程与订阅生命周期。
- 新增 `00-Inbox/2026-11-05_CSharp-Delegate-Event学习记录.md`：保留本轮学习主题、核心结论与待实践任务。
- 更新 `01-CSharp/README.md`：增加委托与事件专题入口。
- 更新 `01-CSharp/Async-Events.md`：把主题范围聚焦到异步编程与 UI 线程，并链接新的委托/事件专题。
- 更新 `06-Anki/CSharp.csv`：新增 Q198–Q210，共 13 张卡片。
- 更新 `05-Interview/P0-Review.md`：增加 Delegate/Event 口述题。
- 更新 `07-Study-Plan/progress.md`：记录本轮学习内容、状态与下一步实践。
- 更新根目录 `README.md`：将本轮覆盖报告设为最新入口。

## 知识覆盖

| 模块 | 覆盖点 |
|---|---|
| Delegate | 自定义委托类型、方法签名匹配、方法组绑定、委托调用、实例方法目标引用 |
| 内置委托 | Action、Func、Predicate 与泛型参数顺序 |
| 多播委托 | 调用列表、+=、-=、重复订阅、返回值与异常行为 |
| Callback | 将处理方法交给设备对象，由设备在合适时机回调 |
| Event | event 的外部访问限制、字段式事件、add/remove、自定义事件访问器 |
| 事件参数 | sender、EventArgs.Empty、自定义 EventArgs、EventHandler<TEventArgs> |
| WinForms | Button.Click、MouseEventArgs 与事件处理方法 |
| 工业相机 | 模拟采集完成事件、多个订阅者、图像数据与采集时间 |
| 线程与生命周期 | 事件同步触发线程、UI 线程切换、图像缓冲区所有权、退订与对象生命周期 |

## Anki 与面试

- Anki：Q198–Q210，共 13 张，归入 `06-Anki/CSharp.csv`。
- 面试：增加 P0 口述题，覆盖委托与事件区别、+=/-=、sender/EventArgs、事件同步执行、UI 跨线程更新和退订原因。

## 状态与边界

- 所有新增内容保持**待学习 / 待验证**。本轮为文字讲解和代码示例，尚无用户独立运行 DelegateDemo/CameraEventDemo 的证据。
- 相机代码是同步教学模拟，不代表任何特定厂商 SDK 的真实 API、回调线程或图像缓冲区生命周期。
- 未连接真实相机、PLC 或运动控制设备；没有修改 PLC 程序、设备配置或实际控制行为。
- 事件通知不能替代 PLC/驱动器的硬件互锁、急停、限位等安全功能。

## 下一步实践

1. 完成 `DelegateDemo` 的自定义 Calculate、Func 和相机回调三个练习。
2. 新建 CameraEventDemo，使用 `EventHandler<ImageCapturedEventArgs>` 实现图像采集事件。
3. 订阅两个处理器，验证多播、重复订阅和退订行为。
4. 在 WinForms 模拟程序中从后台线程触发事件，验证 `InvokeRequired` / `BeginInvoke`。
5. 确认实训相机品牌、型号和 SDK 版本后，再对照厂商文档核实事件 API、线程模型和缓冲区所有权。
