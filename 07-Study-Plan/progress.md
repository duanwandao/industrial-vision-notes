# 学习进度

2026-11-05（跨语言编程抽象与软件设计）：新增[跨语言编程基础专题](../08-Programming-Fundamentals/Abstraction-and-Software-Design.md)，区分语言语法、编译器转换、运行时/类库、框架/SDK、设计模式与软件架构；用属性、泛型、接口、委托/事件、async/await、using 和工业相机采集流程说明抽象的价值、边界与代价。**状态保持待学习 / 待验证**：本轮为概念整理，尚未完成 ICamera / MockCamera 替换实现练习，也未验证相机 SDK 的真实回调线程与缓冲区生命周期。下一步通过模拟相机练习依赖抽象，并解释事件线程与 UI 调度的区别。


2026-11-05（C# Delegate 与 Event）：新增 [C# 委托与事件专题](../01-CSharp/Delegates-and-Events.md)，梳理自定义委托、方法签名匹配、Action/Func/Predicate、多播委托、Callback、event 的外部访问限制、add/remove、sender、EventArgs、EventHandler<TEventArgs>、WinForms Click 与相机采集事件、同步触发线程、UI 跨线程更新和退订生命周期；Anki 新增 Q198–Q210（13 张），P0 面试复习同步补充。**状态保持待学习 / 待验证**：目前为文字讲解与代码阅读，尚未记录用户独立完成 DelegateDemo、CameraEventDemo 或 WinForms 跨线程实验。下一步完成三个委托练习、相机事件多订阅者/退订实验，再验证 UI 线程切换。

2026-11-05（C# namespace、访问控制、继承、抽象类与接口）：新增 [C# 面向对象专题](../01-CSharp/OOP-Types-Access-Modifiers-and-Polymorphism.md)，梳理 dotted namespace、file-scoped namespace、using、Assembly/internal、protected、继承/base、abstract、virtual/override、interface、多接口实现、组合与多态；Anki 新增 Q184–Q197（14 张），P0 面试复习同步补充。**状态保持待学习 / 待验证**：本轮是文字讲解与代码阅读，尚未记录用户独立运行示例、跨程序集访问实验或 Visual Studio 调试验证。下一步完成 namespace/internal 双程序集实验，并实现 DeviceBase + ICamera + InspectionService 模拟项目。

2026-11-05（C# partial、WinForms 生命周期与可空类型）：新增 [C# 专题笔记](../01-CSharp/WinForms-Lifecycle-and-Nullability.md) 和 [C# 笔记索引](../01-CSharp/README.md)，梳理 partial、构造方法、InitializeComponent、InitializeLayout/Behavior、OnLoad、Dispose、IDisposable、Close/Dispose、Nullable<T>、可空引用注解及 ?. / ?? / ??=；Anki 新增 Q170–Q183（14 张），面试复习同步补充。**状态保持待学习 / 待验证**：目前是文字讲解与代码阅读，尚未记录 Visual Studio 断点观察、控制台示例运行或真实设备 SDK 释放验证。下一步在 .NET 8 WinForms 模拟项目中验证构造/关闭顺序，并单独练习 IDisposable/using。

2026-11-04（串口、以太网与 RS-485；求职方向校准）：补充[PLC 通信](../02-PLC/Communication.md)中的串口/UART、RS-485 差分接口、以太网协议分层及 Modbus RTU/TCP 对比；新增[工业机器视觉上位机学习路线](../03-Machine-Vision/README.md)、面试复习和 Anki Q162–Q169（8 张）。用户明确目标是 C# 上位机机器视觉岗位，因此将 UART 底层与电路设计降为 P2，将 RS-485/Modbus RTU 基础配置定为 P1；P0 聚焦 C#/.NET、异步与 UI 线程、工业相机 SDK/图像采集、成像与视觉算法、PLC 通信握手和部署诊断。**本轮仅整理理论，没有进行串口接线、抓包、相机采集或 PLC 通信实测。**

2026-11-04（交流电、三相电机、电容与继电器/接触器）：在已有电学基础上补充[交流瞬时值/有效值与电容](../02-PLC/Electrical-Fundamentals.md)、[三相电流/中性线/缺相](../02-PLC/AC-Motors-and-VFD.md)、[RCD 三相检测与电机保护](../02-PLC/Electrical-Protection.md)，并新增[继电器与接触器](../02-PLC/Relays-and-Contactors.md)。面试复习新增 15 题，Anki 新增 Q143–Q161（19 张）。用户正确回答三相三线电机无需 N、运行中缺相可能继续转动并过热、过载故障原因未排除时不应自动重启；负电流/有效值、电容充放电、接触器线圈排查仍需复述。**未进行真实接线、带电测量或电机故障实验。**


2026-11-04（工业电气与 PLC 输入补充）：从近期文字学习整理 [24V 电源、功率、配线](../02-PLC/Industrial-DC-Power-and-Wiring.md)、[PNP/NPN 输入与光耦](../02-PLC/Digital-Inputs-PNP-NPN.md)、[4 极电机/VFD](../02-PLC/AC-Motors-and-VFD.md) 和 [短路、过载及保护](../02-PLC/Electrical-Protection.md)；[覆盖报告](2026-11-04-Industrial-Electrics-Coverage.md) 记录重点误区与后续练习。用户已明确答对固定电压下电阻翻倍使电流/功率减半、两组电火炉耗电量相同两题；但 NPN 回路、保护配合、VFD 内部机制仍需复述验证。**未进行实机接线和故障测试，不把全部内容标为掌握。**

2026-11-03（电学基础补充）：新增[电学基础](../02-PLC/Electrical-Fundamentals.md)，系统梳理电荷、电场、电势差、电流、电阻、欧姆定律，以及开路仍有电压但无持续电流的情形；PLC 基础增加关联入口。面试复习新增 6 题，Anki 新增 Q111–Q120（10 张）。内容均为**待学习 / 待验证**，未进行带电测量或实机接线。

2026-11-03：完善 [PLC 基础](../02-PLC/PLC-Basics.md) 中的输入映像区、物理/逻辑触点、扫描、P/N、STL、S/R、自锁与互锁；在 [PLC 通信](../02-PLC/Communication.md) 中增加 PLC—C# 视觉检测应用层握手。面试复习新增 10 题，Anki 修订 Q007/Q008/Q92，并新增 Q097–Q110（14 张）。内容均为**待学习 / 待验证**；地址表和握手流程是教学示例，未连接或控制真实设备。

2026-11-02：新增 [丝杆/导程/脉冲当量](../02-PLC/Mechanics-and-Pulse-Equivalent.md)、[运动控制板卡/回零/定位](../02-PLC/Motion-Controller-and-Homing.md)、[STEP7 运动向导/在线监控](../02-PLC/STEP7-Motion-Wizard-Online.md)、[模拟量量程换算](../02-PLC/Analog-Scaling.md)，并补充 PLC 四大硬件部分与 MCU 区别。面试与 Anki 新增复习点 Q079–Q096。均为**待学习 / 待验证**；未进行真实 PLC 控制与在线验证。


2026-11-01：新增步进驱动器、数字电平/脉冲、PUL/DIR/ENA、绕组电流控制与细分换算内容，详见[步进驱动器与脉冲控制增量报告](2026-11-01-Stepper-Driver-Pulse-Review.md)及[运动控制主笔记](../02-PLC/Motion-Control.md)。已通过照片识读驱动器面板标识，但具体型号、电机铭牌、实际拨码状态、PLC 输出映射与接线仍待核实；学习状态保持待学习/待验证。

2026-10-31：已建立[四主题笔记与 Q001–Q063 追踪](2026-10-31-Coverage.md)，知识状态仍为待学习/待验证。当前 P0 顺序：电学基础 → PLC 基础 → 运动控制 → 通信 → 异步事件 → Jog；关键缺口为 PLC 工程/实际硬件参数及离线异常路径验证。运动与 Jog 停止职责按本批提升为 P0。

状态仅在有明确反馈或自测/实践证据时升级。

| 知识点 | 状态 | 优先级 | 前置/备注 |
|---|---|---|---|
| 跨语言编程抽象与软件设计 | 待学习 / 待验证 | P1 | 区分语言、编译器、运行时/类库、框架/SDK、设计模式与架构；结合 ICamera / MockCamera 实践 |
|---|---|---|---|
| C#/.NET 与 WinForms 工程开发 | 待学习 / 待验证 | P0 | 结合已有编程基础，完成可维护的设备上位机界面 |
| 委托、事件、异步与 UI 线程 | 待学习 / 待验证 | P0 | Delegate/Event、Task、async/await、取消、回调、线程安全 |
| 工业相机 SDK 与图像采集 | 待学习 | P0 | 枚举/连接、采集生命周期、软硬触发、回调、缓冲区；型号待确认 |
| 成像基础：相机、镜头、光源、曝光 | 待学习 | P0 | 视野、工作距离、分辨率、景深和照明 |
| 图像处理与视觉算法 | 待学习 | P0 | OpenCV/HALCON 基础，按课程与目标岗位选工具 |
| PLC 通信、业务握手与异常恢复 | 待学习 / 待验证 | P0 | TCP/IP、实际 PLC 协议、超时/断线、请求确认与结果反馈 |
| 工业软件部署、日志与结果追溯 | 待学习 | P0 | 工控机部署、配置、日志、图像与检测结果保存 |
| RS-485 / Modbus RTU | 待学习 | P1 | 能配置串口参数、站号、寄存器并排查常见问题 |
| PLC I/O、24V、PNP/NPN、继电器/接触器 | 待学习 / 待验证 | P1 | 理解触发/反馈与电气安全边界，不以电工设计为求职主线 |
| 步进/伺服与运动控制 | 待学习 / 待验证 | P1 | 理解设备接口、状态和安全职责；深度按岗位需求扩展 |
| UART 芯片内部采样、RS-485 收发器电路与 PCB | 暂缓 | P2 | 当前岗位非核心，除非目标岗位明确要求底层硬件开发 |

## 优先知识缺口

| 缺口 | 为什么重要 | 优先级 | 前置/小实验 |
|---|---|---|---|
| C# 异步、UI 线程与任务生命周期 | 采集和通信不能卡住界面；停止、关闭、断线时要能安全收尾 | P0 | 用 Task 模拟耗时采集/通信，练取消、异常和 UI 回切 |
| 工业相机 SDK 采集流程 | 相机连接、触发、回调、图像缓冲区是视觉上位机的入口 | P0 | 先确认实训相机品牌/型号/接口，再按对应 SDK 示例枚举、连接、采图和停止 |
| 成像基础 | 图像质量受镜头、视野、工作距离、光源和曝光影响，算法无法弥补所有成像问题 | P0 | 用样图对比曝光/光照；核实相机传感器与镜头规格 |
| 视觉算法基础 | 检测、定位、测量和缺陷判断是岗位核心产出 | P0 | 从灰度/阈值/滤波/形态学/轮廓开始，再练模板匹配与标定 |
| PLC—上位机握手和故障恢复 | 避免漏请求、重复检测、结果错配和断线后状态混乱 | P0 | 先在模拟状态机中验证 RequestId、Ack、Done、超时和重连 |
| 工业软件部署、日志和追溯 | 设备现场需要可诊断、可恢复、可追溯 | P0 | 为模拟检测流程增加日志、参数配置、图像/结果保存和异常记录 |
| RS-485 / Modbus RTU 基础配置 | 现场可能遇到仪表、灯控器、变频器等串口设备 | P1 | 读设备手册，核对 A/B、波特率、校验、站号、终端和寄存器 |
| 24V 控制回路与 PLC I/O | 需要看懂触发输入、输出类型、公共端及设备反馈 | P1 | 按真实型号画纸面图；区分逻辑位、物理输出和反馈，不自行带电测量 |
| 步进/伺服与设备安全边界 | 需要理解运动设备的状态、报警、限位和安全职责 | P1 | 以接口和状态诊断为主；具体参数先核对型号手册 |
| UART 底层、电路设计与信号完整性 | 对当前上位机视觉岗位不是普遍前置要求 | P2 | 暂缓；只有岗位明确要求嵌入式/硬件开发时再深入 |

## 2026-10-31 STEP 7-Micro/WIN SMART 补充

[Q064–Q069 覆盖报告](2026-10-31-STEP7-MicroWIN-SMART-Coverage.md)：新增本地工程与 CPU 程序区别、上传/下载、MAIN/SBR/INT、RUN/STOP、限位信号排查。状态均为**待学习/待验证**；待确认上传日志、CPU 当前运行版本和左右限位实际地址。
