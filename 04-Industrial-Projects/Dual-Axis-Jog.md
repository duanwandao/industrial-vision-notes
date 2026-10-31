# 双轴 WinForms / PLC 项目源码导读

> P0｜待学习、待离线验证｜2026-10-31｜Q038–052、Q054、Q056–059（其他问题见覆盖表）。
> 本文为源码阅读结果，不代表用户实践经历、设备验收或安全认证。

## 证据与入口

只读核对提交 `0aea306f162b1b2d184e33ca9073fd31ecc2087c`（2026-10-31 获取）。已读 [Program.cs](https://github.com/duanwandao/plc-axis-control/blob/0aea306f162b1b2d184e33ca9073fd31ecc2087c/Program.cs)、[Form1.cs](https://github.com/duanwandao/plc-axis-control/blob/0aea306f162b1b2d184e33ca9073fd31ecc2087c/Form1.cs)、[Designer](https://github.com/duanwandao/plc-axis-control/blob/0aea306f162b1b2d184e33ca9073fd31ecc2087c/Form1.Designer.cs)、[PlcHelper](https://github.com/duanwandao/plc-axis-control/blob/0aea306f162b1b2d184e33ca9073fd31ecc2087c/PlcHelper.cs)、[PlcAddr](https://github.com/duanwandao/plc-axis-control/blob/0aea306f162b1b2d184e33ca9073fd31ecc2087c/PlcAddr.cs)、[csproj](https://github.com/duanwandao/plc-axis-control/blob/0aea306f162b1b2d184e33ca9073fd31ecc2087c/PlcAxisControl.csproj) 和 [sln](https://github.com/duanwandao/plc-axis-control/blob/0aea306f162b1b2d184e33ca9073fd31ecc2087c/PlcAxisControl.sln)；未运行或改动源仓库。

.sln 组织项目，.csproj 指定编译项、引用和目标 .NET Framework 4.8。项目声明 HslCommunication 12.1.2.0、SunnyUI 3.9.9.0；这是引用元数据，未检查 DLL 实际文件版本。HintPath 指向 libs，而 bin/Debug 是输出位置，不能把输出目录当作依赖管理。NuGet 管包版本及依赖，项目引用表达源码项目关系，DLL 引用指向程序集；程序集含元数据及可能的 IL，反编译不保证还原原源码。

Program.Main 使用 STAThread，经 Application.Run(new Form1()) 启动窗体消息循环；InitializeComponent 创建控件，构造函数补充事件订阅。标题“步进轴 1 控制”不能证明程序只控制轴 1，实际由单选框选择。

## 地址和通信链

PlcAddr 实际是 internal class，成员为 public const string，并不是素材中的 static class。常量是地址文本，不能当作该位当前值。统一命名便于维护，但字符串不能自动保证地址类型、PLC 程序一致性。

| 用途 | 轴 1 | 轴 2 |
|---|---|---|
| 回零请求 | M0.0 | M0.1 |
| Jog 正 / 负 | M3.0 / M3.1 | M3.4 / M3.5 |
| 手动速度 | VD2108 | VD2112 |
| 当前位置 | VD100 | VD2024 |

Axis1HomeDone 在源码被注释；Axis2HomeDone=M10.1 存在，但本窗体未使用完成信号。地址表存在不证明 PLC 中已有相应程序。

ConnectPLC 创建 S200Smart 对象，代码示例 IP=192.168.0.2、端口=102、连接超时 5000 ms、接收超时 10000 ms；这些不是通用设备配置。连接由 UI 同步调用，可能阻塞。连接/断开未使用 sync 锁，通信读写使用锁；关闭事件没有调用 DisconnectPLC。

ReadPosition 选择轴地址→ReadFloat→检查 IsSuccess→返回 Content，否则抛异常。RefreshTick 用 refreshing 避免重入，后台读取后检查窗体及轴选择再更新文字；显示单位 mm 来自界面约定，未有 PLC 工程验证实际量纲。

## JogMouseDown 的真实顺序

1. 过滤非左键、已有 jogRelease、jogFault、未连接、刷新定时器未启用。
2. 在 UI 线程固定轴/方向/速度；保存按钮，创建 TCS，并用局部 release 保留本轮信号对象。
3. 暂停刷新，禁用轴选择和回零。停止计时器不撤销已经开始的 RefreshTick。
4. Task.Run 写速度；失败抛异常。若写速度期间已经释放，直接 return，仍会进入 finally。
5. 在尝试写 true **之前**设置 attemptedStart；写入响应失败也可能实际启动，所以 finally 仍尝试写 false。
6. 启动成功后 await release.Task，等待释放信号。
7. catch 标记 jogFault 并显示错误；finally 在 attemptedStart 时尝试复位命令位，清理字段，窗体未释放才恢复 UI。出现故障不恢复刷新和回零许可。

MouseUp 仅匹配当前按钮时释放；MouseCaptureChanged 在对应按钮丢失捕获时释放；Deactivate、FormClosing 也释放。ReleaseJog 用 TrySetResult，重复释放不抛重复完成异常。未发现素材提到的空 btnJogNegative_Click / btnJogPositive_Click，不能把旧描述套到此提交。

## 已实现和边界

- 已有鼠标驱动 Jog、位置刷新、回零请求。Home 持锁写 true→Sleep(200)→写 false，没有等待回零完成。
- Designer 中 btnMove、btnStop 没有绑定 Click；源码也未找到其控制处理。存在按钮不代表定位/停止功能已实现。
- 状态灯当前是 Designer 初值，RefreshTick 只更新位置/运行文字，未读取限位等灯的状态。
- FormClosing 只释放 TCS，不取消本次关闭，也不等待异步 finally；因此不能承诺窗体退出前复位写入完成。
- catch 中模态 MessageBox 在 finally 前显示，可能推迟发送停止；这是源码审阅发现的风险，未做运行复现。
- 速度写入后的释放检查与实际启动写入之间仍有时间窗口；写启动期间释放后仍需等写调用返回再复位。停止可能受超时、排队和网络影响。
- 普通软件复位不是已停止证据。缺 PLC 工程、驱动器型号、心跳/互锁和实际反馈，不能判断硬件停止行为。

以上仅记录待改进点，未修改控制程序。原理见 [异步事件](../01-CSharp/Async-Events.md)、[通信](../02-PLC/Communication.md)、[运动控制](../02-PLC/Motion-Control.md)。

## 安全的验证练习

用假的通信对象记录调用及时间，模拟写速度失败、启动已执行但响应丢失、提前释放、重复释放、关闭和停止失败；断言每次尝试启动都有停止尝试，且 UI 不把失败报告为完成。该练习只能验证软件顺序，不能证明设备安全。

