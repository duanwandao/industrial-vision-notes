# PLC 通信、类型与结果处理

> P0｜待学习｜2026-10-31｜Q010–012、Q034–036（项目所需部分）、Q043–047、Q049、Q053。

一句话：通信把双方约定的数据编码到协议中；网络连通、读写成功和设备动作成功是三层不同证据。

IP 定位网络接口，掩码帮助判断是否本地直达，MAC 用于链路转发，TCP 提供有序字节流，端口标识服务。同网段还可能因 VLAN、防火墙、冲突或服务未启用而不通；ping 仅验证相应 ICMP 可达，不证明 S7 服务和地址可访问。

VS 是开发工具，WinForms 是界面框架，S7netplus/HslCommunication 是协议库。不能因为都是西门子就推断所有 CPU 与库组合兼容；S7netplus 对 SMART 的具体支持须查版本，原项目实际用了 HslCommunication。以项目配置确认 API，不照搬最新文档。

## 数据协议与离线测试（Q012）

先约定 VW200 的类型、范围、读写权限和单位。2002 可用 16 位有符号数表达，但不能用 WriteFloat 写入同一 2 字节目标。用内存字节数组模拟 PLC 区域：写入两个字节后按同一字节序读回 2002，并测试越界、超时和写入失败；该模拟不证明真实协议握手兼容。实际 PLC 需审批后的测试环境和地址表，本次不连接。

`WriteFloat(address, 20)` 在只有 float 形参时发生 int→float 隐式转换；`20f` 明确 float，`(short)20` 明确 short。`short x = 20` 是范围内常量赋值；不存在通用 short 字面量后缀。若有 int/short/float 重载，重载决议取决于参数静态类型，不能靠运行时值猜测。TS 的 number 习惯不能直接套到 C# 和 PLC 二进制布局。

## 结果、异常与并发

原项目 ReadBool/ReadFloat 返回 OperateResult<T>，成功后才读 Content；WriteBool/WriteFloat 返回非泛型 OperateResult。失败结果与抛异常都要处理，不能仅 catch 而忽略 IsSuccess。

lock(sync) 串行化同一实例使用同一锁的路径，不是跨进程锁，也不保证未用锁的连接/断开安全。Monitor 同线程可重入，因此原项目 ReadPosition 中嵌套 ReadFloat 的同锁不是必然死锁；Home 持锁 Sleep 则会延迟其他请求。不要在 lock 中 await。

连接检查可提取“检查+加锁+调用”的 Execute<T>，写操作可能需非泛型重载；这是重构建议，未改项目或验证 API。连接状态、超时、断开必须纳入共同策略。`isConnect` 是历史布尔状态，不能代替实时读写结果。

工业用途：周期读取位置，失败时保留错误和时间戳；写命令后读取 PLC 应答/设备反馈。Q053 缺原测试工具上下文，数字 1 无法确定代表长度、站号还是值，维持待核实。

自测：区分“连接失败”“地址错误”“结果成功但轴没动”，每类列一条下一步证据。源码依据及配置见[项目笔记](../04-Industrial-Projects/Dual-Axis-Jog.md)。
