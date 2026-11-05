# 2026-11-05 C# Delegate 与 Event 学习记录

> 来源：本次对话中的 C# 委托与事件学习。原始问题摘要保留在此，系统整理见 [C# 委托与事件](../01-CSharp/Delegates-and-Events.md)。

## 用户本轮学习主题

- 学习 C# Delegate：自定义委托、签名匹配、方法绑定、Action、Func、Predicate、Lambda、多播委托与回调。
- 使用 DelegateDemo 练习：自定义 Calculate、使用 Func<int, int, int>、模拟 Camera 图像采集回调。
- 学习 C# Event：event 关键字的作用、+= 订阅、-= 退订、外部不能直接触发事件、sender、EventArgs、EventHandler<TEventArgs>。
- 对比 WinForms button.Click 与工业相机图像采集事件。
- 关注事件触发线程、WinForms UI 线程切换、退订与资源生命周期。

## 本轮核心结论

1. Delegate 是类型安全的可调用对象类型，可以保存符合签名的方法并调用；Action/Func/Predicate 是常用内置委托。
2. Event 基于委托，限制外部只能订阅/退订；事件声明类型负责触发事件。
3. sender 表示触发事件的对象，EventArgs 或派生类携带事件数据。
4. 普通事件处理器通常在触发事件的线程同步执行；event 不等于异步，也不自动切换 UI 线程。
5. 工业相机 SDK 的事件/回调 API、触发线程和图像缓冲区生命周期依具体厂商和 SDK 版本，必须查手册。

## 待实践

- [ ] 完成 DelegateDemo 三项练习。
- [ ] 创建 Camera + ImageCapturedEventArgs 事件模拟。
- [ ] 添加两个事件订阅者，验证多播调用与取消订阅。
- [ ] 在外部尝试调用 event，确认编译器限制。
- [ ] 在 WinForms 模拟项目中验证跨线程 UI 更新。
- [ ] 确认实训相机品牌、型号和 SDK 版本后，再对照真实 API。

> 状态：待学习 / 待验证。尚无独立运行代码或真实设备采集证据。
