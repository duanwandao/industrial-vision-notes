# C# SerialPort 与 SemaphoreSlim：设备串口通信实战

> **P0 / P1｜待学习 / 待验证｜2026-11-12**
> 面向 C# 工业上位机、LED 光源控制器和工业相机周边设备。重点是理解通信 API、协议分帧、异步并发和设备确认之间的边界。

## 1. 先区分四个概念

| 概念 | 负责什么 | 示例 |
|---|---|---|
| 串口接口/API | 打开 COM 端口、收发字节 | .NET SerialPort |
| 操作系统驱动 | 让 Windows 识别并管理串口硬件 | USB 转串口芯片驱动 |
| 物理接口 | 电气信号和接线规范 | UART TTL、RS-232、RS-485 |
| 通信协议 | 约定命令、帧边界、校验、响应 | 厂商 ASCII 协议、Modbus RTU |

SerialPort 不是 Modbus，也不等于 RS-232/RS-485。它提供串口字节收发能力；协议层还要解析站号、命令、长度、校验、响应和错误码。

## 2. SerialPort 常见生命周期

### 2.1 创建与配置

~~~csharp
using System.IO.Ports;
using System.Text;

var port = new SerialPort
{
    PortName = "COM3",
    BaudRate = 9600,
    DataBits = 8,
    Parity = Parity.None,
    StopBits = StopBits.One,
    Handshake = Handshake.None,
    Encoding = Encoding.ASCII,
    ReadTimeout = 1000,
    WriteTimeout = 1000
};
~~~

所有参数都应以设备手册为准。COM3 只是示例；USB 转串口重新插拔后端口号可能改变。

### 2.2 打开、判断、关闭与释放

~~~csharp
port.Open();

if (port.IsOpen)
{
    // 仅说明本机 SerialPort 句柄处于打开状态
}

port.Close();
port.Dispose();
~~~

IsOpen == true 不代表设备在线、接线正确或协议响应正常。要判断设备是否可用，需要按协议发送探测/查询命令并验证响应。

通常一个设备服务在连接期间复用同一个串口实例；不要每发一条命令都重新 Open/Close。退出时按资源生命周期关闭并 Dispose。

## 3. Write 家族：发送什么数据？

### 3.1 Write(string)

~~~csharp
port.Encoding = Encoding.ASCII;
port.Write("HELLO");
~~~

把字符串按 Encoding 转成字节后发送。适用于文本协议。ASCII 下 HELLO 对应十六进制 48 45 4C 4C 4F。

### 3.2 Write(byte[], offset, count)

~~~csharp
byte[] frame = { 0x01, 0x03, 0x00, 0x02 };
port.Write(frame, 0, frame.Length);
~~~

参数依次为数组、起始索引、发送字节数。例如 Write(frame, 1, 2) 发送 03 00。

注意：

~~~csharp
port.Write("01 03");
~~~

发送的是字符 0、1、空格、0、3；而 byte[] { 0x01, 0x03 } 发送两个原始字节。二者不是同一帧。

### 3.3 Write(char[], offset, count)

~~~csharp
char[] chars = { 'A', 'B', 'C', 'D' };
port.Write(chars, 1, 2); // 发送 "BC"
~~~

字符数组也会经过 Encoding 转换。常见文本协议可使用 string；二进制协议优先使用 byte[]。

### 3.4 WriteLine(string)

~~~csharp
port.NewLine = "\r\n";
port.WriteLine("STATUS");
~~~

WriteLine 会在文本后追加 NewLine，上例等价于发送 STATUS\r\n。不能假设所有设备都要求 CR/LF；结束符必须按协议手册确定。

- \r：Carriage Return，回车，ASCII 0D。
- \n：Line Feed，换行，ASCII 0A。
- \r\n：两个控制字符，常见于文本行协议。

## 4. 接收方式与帧边界

串口是字节流，不是消息队列。一次读取或一次事件不等于一条完整设备响应。

### 4.1 ReadLine()

~~~csharp
port.NewLine = "\r\n";
port.ReadTimeout = 1000;

port.WriteLine("STATUS");
string response = port.ReadLine();
~~~

ReadLine 等待直到读到 NewLine 指定的结束符；超时会抛出 TimeoutException。它是同步读取，会阻塞调用线程，不要直接在 WinForms UI 线程等待。

只适用于设备确实使用行结束符的文本协议。

### 4.2 ReadExisting()

~~~csharp
string chunk = port.ReadExisting();
~~~

读取当前已经到达串口接收缓冲区、且可按当前 Encoding 解码的数据，并立即返回。它不保证拿到完整响应：可能只收到帧的一部分，也可能一次读到多帧或多条数据。

ASCII 文本协议可以把片段追加到应用层缓冲区，再按协议规则找完整帧。二进制协议不应随意通过 string/Encoding 转换，应使用 byte[] 读取和字节级解析。

### 4.3 DataReceived 事件

~~~csharp
port.DataReceived += Port_DataReceived;

private void Port_DataReceived(
    object sender,
    SerialDataReceivedEventArgs e)
{
    string chunk = port.ReadExisting();
    // 把 chunk 交给接收缓存/解析器
}
~~~

- 事件处理程序通常运行在非 UI 线程；不能直接更新 WinForms 控件，需要 Invoke/BeginInvoke 或异步 UI 调度。
- 事件触发时只代表有数据可读，不代表完整帧已经到达。
- 一次响应可能拆成多次事件；多条响应也可能一起到达。
- 对同一个 SerialPort 应设计唯一读取入口。不要让 DataReceived/ReadExisting 与另一个线程的 ReadLine 或 Read 同时竞争同一批数据。

ReadExisting 适合读取当前可用文本片段，不负责协议分帧。接收缓存还要有最大长度限制、帧完整性检查和异常处理。

## 5. 缓冲区清理：DiscardInBuffer / DiscardOutBuffer

~~~csharp
port.DiscardInBuffer();
port.DiscardOutBuffer();
~~~

- DiscardInBuffer：丢弃串口接收缓冲区中尚未读取的数据。
- DiscardOutBuffer：丢弃串口发送缓冲区中尚未发送的数据。

它们不是每条命令前都应该调用的“清理模板”。无条件清空可能丢失上一条命令的有效响应，或丢弃仍待发送的数据。只有在明确的协议恢复/初始化步骤、并确认丢弃行为安全时才使用。

区分两种缓存：
1. SerialPort/驱动内部收发缓冲区；
2. 应用程序自己维护的 StringBuilder / List<byte> 接收帧缓存。

清空应用层缓存不会自动清空串口内部缓存，反之亦然。

## 6. 为什么需要 SemaphoreSlim？

同一个串口通常一次只允许一个“命令—响应”事务占用读取通道。假设两个按钮同时发命令：

- A：设置亮度；
- B：读取亮度。

如果两条命令并发发送，响应可能交错或被错误地归给另一条命令。

使用单许可信号量：

~~~csharp
private readonly SemaphoreSlim _serialGate = new(1, 1);

private async Task SendTransactionAsync(
    byte[] command,
    CancellationToken cancellationToken)
{
    await _serialGate.WaitAsync(cancellationToken);
    try
    {
        // 发送命令，并等待/解析与本命令对应的完整响应
    }
    finally
    {
        _serialGate.Release();
    }
}
~~~

- new SemaphoreSlim(1, 1)：初始许可 1、最大许可 1，即同一时刻一个事务进入。
- WaitAsync：异步等待许可，不像同步 Wait() 那样阻塞当前线程等待。
- Release：归还许可。
- finally：确保已获取许可后，无论正常返回还是发生异常都会释放。
- 若 WaitAsync(cancellationToken) 在获得许可前就被取消，方法不会进入后面的 try/finally，因此不会错误地 Release。

SemaphoreSlim(1,1) 与 lock 的区别：lock 适合短小同步临界区，不能在 lock 块内 await；SemaphoreSlim 支持异步等待，适合保护跨 await 的异步事务。

信号量只对共用同一实例、遵守同一约定的代码路径生效；它不自动解决响应帧边界、迟到响应、设备断线和事务关联。若协议没有 RequestId/TransactionId，上一笔超时后迟到的响应仍可能污染下一笔事务，需要明确的排空/同步恢复策略。

## 7. ACK、发送成功与设备执行成功

需要分清三个层次：

1. SerialPort.Write 没抛异常：数据写入调用完成，不等于设备已接收。
2. 收到协议定义的 ACK：设备/协议端给出确认；具体语义由手册决定。
3. 设备状态回读或传感器反馈符合目标：更接近验证设备动作确实生效。

不能把“Write 成功”直接标记为“亮度设置成功”。结果对象可以记录命令、原始响应和是否收到有效 ACK，但字段名/文档必须说明 ACK 的实际含义。

## 8. 工业设备通信的推荐职责

- UI：显示状态、采集用户输入、发起操作；不拼接协议帧。
- 设备适配器（应用层设备驱动）：提供 Connect、SetBrightness、GetBrightness 等业务化 API。
- 串口传输层：负责 COM 端口打开、收发和连接生命周期。
- 协议层：负责命令编码、校验、分帧、ACK/错误码解析。
- 结果模型：记录本次操作的输入、命令、响应、确认状态和错误信息。

具体项目可以把这些职责拆成不同类，也可以先放在一个小服务内；重点是职责边界清楚，避免 UI、串口和协议解析互相耦合。

## 9. 练习顺序

1. 不接设备，写一个 SerialPort 参数配置与端口枚举界面。
2. 用虚拟串口对发送 ASCII 文本，观察 Write 与 WriteLine 的字节差异。
3. 模拟把 OK\r\n 拆成 O、K\r、\n 三次到达，验证接收缓存能否拼出完整帧。
4. 同时点击“设置亮度”和“读取亮度”，验证 SemaphoreSlim 能否串行化事务。
5. 模拟无响应、错误 ACK、迟到响应和串口断开，验证超时与恢复策略。
6. 只有确认设备型号、手册、接口和安全前提后，才连接真实控制器。

## 10. 学习状态

本轮为文字讲解、协议代码阅读与设计讨论；没有记录用户独立完成虚拟串口测试，也没有确认真实设备型号和协议全部细节。实际项目中的帧结束符、ACK 内容、响应长度、超时和重试必须以对应控制器手册/SDK 为准。

## 参考资料

- [Microsoft Learn：SerialPort](https://learn.microsoft.com/dotnet/api/system.io.ports.serialport)
- [SerialPort.DataReceived](https://learn.microsoft.com/dotnet/api/system.io.ports.serialport.datareceived)
- [SerialPort.ReadExisting](https://learn.microsoft.com/dotnet/api/system.io.ports.serialport.readexisting)
- [SerialPort.ReadLine](https://learn.microsoft.com/dotnet/api/system.io.ports.serialport.readline)
- [SerialPort.Write](https://learn.microsoft.com/dotnet/api/system.io.ports.serialport.write)
- [SerialPort.WriteLine](https://learn.microsoft.com/dotnet/api/system.io.ports.serialport.writeline)
- [SemaphoreSlim.WaitAsync](https://learn.microsoft.com/dotnet/api/system.threading.semaphoreslim.waitasync)
