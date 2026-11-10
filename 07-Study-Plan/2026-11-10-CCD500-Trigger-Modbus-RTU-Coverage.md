# 2026-11-10 CCD500 光源触发、ST/SR 与 Modbus RTU 覆盖报告

> 状态：待学习 / 待验证。依据用户照片、用户现场反馈与文字讲解整理；没有执行真实接线、带电测量、PLC 程序改写或 Modbus 实机通信。

## 覆盖范围

- CCD500 LED 光源控制器的角色；CH 光源输出与 TR 外部触发输入。
- 常亮/外部触发、电平/边沿触发、Active High/Active Low 三类概念。
- 现场现象：TR1 接线标记 Q1.2；Q1.2 指示灯未亮；老师切到 LOW 后灯亮。
- S7-200 SMART ST 晶体管输出与 SR 继电器输出；Q1.2 逻辑状态与物理电平的区别。
- Windows COM3/COM4、设备管理器、SerialPort.GetPortNames。
- RS-232/RS-485 与 Modbus RTU 的分层关系。
- RTU 帧结构、站号、功能码、CRC16、寄存器、地址偏移、串口参数、t1.5/t3.5。
- C# SerialPort 与 Modbus 通信库的职责边界。

## 用户现场证据

- 用户确认 PLC 平台为 Siemens S7-200 SMART，但 CPU 完整型号未知。
- 用户报告 TR1 线缆标记 Q1.2，Q1.2 指示灯当时未亮。
- 用户报告老师把 CCD500 调到 LOW 后，小 LED 光源亮起。
- 小光源没有单独电源线；线缆实际连接和供电路径仍待追踪。
- CCD500 厂家、完整型号、原厂手册及触发端子规格均未确认。

## 关键结论与纠错

1. CH 是光源输出通道，TR 是外部触发输入；准确映射以控制器手册为准。
2. 外部触发模式不是“灯坏了/没供电”的同义词；控制器可能等待 PLC 或相机的硬件信号。
3. HIGH/LOW 可能涉及有效极性或特定工作模式；LOW 后灯亮不能单独证明 Active Low。
4. Q1.2 指示灯未亮不等于 TR1 实测为 0V。ST 晶体管关断可能呈高阻；SR 继电器关断是触点断开。
5. S7-200 SMART 标准型型号中，ST 对应 DC 电源/晶体管输出，SR 对应 AC 电源/继电器输出；完整接线仍按订货号手册。
6. COM3/COM4 是 Windows 串口设备名称，不是接口标准或 Modbus 站号。
7. RS-232/RS-485 是电气接口；Modbus RTU 是串行通信协议。RTU 常用 RS-485，也可以使用 RS-232。
8. C# SerialPort 只提供字节收发，不会自动处理 Modbus 地址、功能码、CRC 和 RTU 帧边界。
9. 当前没有 CCD500 支持 Modbus RTU 的证据；不得把硬接线触发输入当作串口协议接口。

## 关键示例

读取从站 1 的保持寄存器地址 0、数量 1：

```
01 03 00 00 00 01 84 0A
```

字段依次为从站地址 01、功能码 03、起始地址 0000、数量 0001、CRC 84 0A。示例响应值 0x0064（100）：

```
01 03 02 00 64 B9 AF
```

以上是协议教学示例，不代表实训 PLC 或 CCD500 的寄存器映射。

## 下一步实践

1. 拍清 CCD500 铭牌、端子、CH 光源插头及背面标签，取得原厂手册。
2. 确认 S7-200 SMART CPU 的完整 ST/SR 型号和订货号。
3. 画出 Q1.2—TR1—COM 的纸面回路，标出输出组公共端和触发输入参考端。
4. 在老师指导与安全条件满足后，分别记录 Q1.2 逻辑状态、TR1 实测电气状态和 LED 亮灭；不要短接或盲目送 24V。
5. 在 Windows 设备管理器识别一个 USB 串口转换器对应的 COM 号；先完成 COM 端口枚举练习。
6. 使用离线 Modbus RTU 模拟器或已知测试设备练习 03/06 功能码、CRC、超时和地址偏移；不要把未知 CCD500 当作 Modbus 设备。
7. C# WinForms 增加串口枚举、参数选择、连接状态和十六进制日志，再用成熟 Modbus RTU 库完成受控读写。

## 参考资料

- [Siemens S7-200 SMART System Manual V2.8](https://support.industry.siemens.com/cs/attachments/109745610/s7-200_SMART_system_manual_en-US.pdf)
- [Modbus Application Protocol Specification V1.1b3](https://www.modbus.org/file/secure/modbusprotocolspecification.pdf)
- [Modbus Serial Line Protocol and Implementation Guide V1.02](https://www.modbus.org/file/secure/modbusoverserial.pdf)
- [Microsoft SerialPort API](https://learn.microsoft.com/dotnet/api/system.io.ports.serialport)
