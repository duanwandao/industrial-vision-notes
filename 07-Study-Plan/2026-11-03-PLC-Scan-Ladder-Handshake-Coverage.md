# 2026-11-03｜PLC 扫描周期、梯形图与视觉握手增量报告

> 本批内容默认**待学习 / 待验证**。本次只维护知识库，没有修改 PLC 程序、设备配置，也没有连接或控制真实设备。

## 覆盖范围与落点

| 对话主题 | 整理位置 | 本次补充 |
|---|---|---|
| 扫描周期、输入映像区 | [PLC 基础](../02-PLC/PLC-Basics.md) | I 输入快照、Q 输出映像、M 内部位、通信/诊断任务及立即 I/O 例外 |
| 物理 NO/NC 与逻辑 NO/NC | [PLC 基础](../02-PLC/PLC-Basics.md) | 以物理 NC 停止按钮为例，解释 I0.1=1 的接线前提 |
| 梯形图符号、串联/并联 | [PLC 基础](../02-PLC/PLC-Basics.md) | 触点判断、普通线圈写入、AND/OR |
| P/N 正负跳变 | [PLC 基础](../02-PLC/PLC-Basics.md) | 0→1、1→0；一个扫描周期；不等同于脉冲捕捉 |
| STL | [PLC 基础](../02-PLC/PLC-Basics.md) | Statement List、LD/A/O/=、EU/ED 与 LAD 对照 |
| S/R、普通线圈、自锁 | [PLC 基础](../02-PLC/PLC-Basics.md) | 置位/复位保持、位数参数、自锁公式和执行顺序 |
| 软件/硬件互锁 | [PLC 基础](../02-PLC/PLC-Basics.md) | 正反转示例、冲突请求仲裁与安全边界 |
| 短暂置位与应用层握手 | [PLC 通信](../02-PLC/Communication.md) | Trigger/TriggerAck、Busy/Ready、Done/ResultAck、任务编号 |
| 面试与记忆卡 | [P0 面试复习](../05-Interview/P0-Review.md)、[Anki CSV](../06-Anki/PLC.csv) | 新增 10 道口述题、20 张卡（Q097–Q116） |

原始问题摘要：[2026-11-03 Inbox](../00-Inbox/2026-11-03_PLC扫描与梯形图学习问题.md)。

## 本批关键纠错

1. **输入映像区不是所有 PLC 内存的统称**：I 是数字量输入映像，Q 是数字量输出映像，M 是内部标志位；C# 写 M 不会模拟物理 I 输入。
2. **物理常闭不等于逻辑常闭**：在示例 NC 停止回路中，未按下时 I0.1=1，所以程序用常开逻辑触点检查“停止回路健康”。
3. **P/N 不等于高速脉冲捕捉**：P/N 检测程序观察到的边沿；短物理脉冲仍可能在输入采样/滤波环节丢失。
4. **S/R 的参数不是数值或时间**：例如 S M0.0, 1 中 1 表示从该地址开始操作 1 个位。
5. **普通互锁不是安全回路**：上位机按钮禁用和普通 PLC 逻辑都不能替代适当的安全控制设计。
6. **通信写成功不等于业务完成**：HslCommunication 返回成功不能证明 PLC 业务逻辑已接受命令，必须有应用层确认和结果确认。

## 官方资料与版本边界

主要依据：
- [Siemens S7-200 SMART V3 System Manual, V3.0.1（2025-06）](https://support.industry.siemens.com/cs/attachments/109978364/S7-200_SMART_system_manual_en-US.pdf)：第 6 章（扫描与 I/Q 映像）、第 7.3.7 节（STL 编辑器）、第 10.1 节（位逻辑、边沿检测、置位/复位）。
- [Siemens S7-200 SMART V2.8 System Manual](https://support.industry.siemens.com/cs/attachments/109745610/s7-200_SMART_system_manual_en-US.pdf)：如课堂设备/软件为 V2.8，应以匹配版本手册核对。

握手地址和流程是本知识库中的教学约定，不是西门子标准通信协议；实际工程必须与 PLC 程序、相机 SDK、上位机状态机和现场电气设计统一。

## 后续练习

1. 离线画出停止回路、启动自锁、正反转互锁网络，逐扫描推演 I0.0/I0.1/M0.0/Q0.0。
2. 用状态表验证 P/N：0→0、0→1、1→1、1→0。
3. 在离线模拟器中实现 Trigger 保持到 TriggerAck、Done 保持到 ResultAck；注入延迟、重复请求、断线和重连。
4. 面试口述时说明：物理接点、逻辑触点、输入映像、M 位、上位机轮询是不同层次。

未进行真实 PLC 写入、下载、参数配置、相机采集或运动操作。
