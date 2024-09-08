# EA4GPZ / M0HXM 的[网站](https://destevez.net/)上的文章 **_KISS, HDLC, AX.25 and friends_**

> 原文链接
> https://destevez.net/2016/06/kiss-hdlc-ax-25-and-friends/

### 翻译：

不久前，我将我的[gr-kiss](https://github.com/daniestevez/gr-kiss) out-of-tree GNU Radio模块上传到了Github。这个模块包含一组用于处理KISS、HDLC和AX.25协议的模块，这些协议是业余分组无线电中常用的。虽然有一些其他的OOT模块可以完成类似的工作，但我对它们的功能并不十分满意。在编写这个模块的过程中，我也注意到这些协议的文档有时并不太完善。因此，我将在这里简要描述这些协议，并解释它们如何协同工作。

### KISS协议

**KISS协议**最初是为计算机与TNC（终端网络控制器）之间的接口而设计的。可以将TNC看作一种调制解调器，它负责一些调制和低级帧处理的功能。KISS的理念是将大部分的处理任务转移到主机计算机上。在KISS协议出现之前，TNC通常会执行一些与AX.25相关的任务。然而，KISS TNC只处理HDLC，并将原始的AX.25帧传递给主机。KISS协议的详细描述请见[这里](http://www.ax25.net/kiss.aspx)。该协议提供了一种分离帧和向TNC发送命令的方法。由于这个协议非常简单，它也被应用于许多不涉及TNC的场景中。在这些情况下，它仅用于将数据流分隔为帧。因此，一系列数据包可以被保存为“KISS格式”的文件。这些数据包通常是AX.25帧，但并不局限于此。此外，不同的软件也可以使用KISS格式来交换数据包。

帧的结束标志是字节0xC0。帧的开始不需要明确标记（但通常也用0xC0标记）。多个连续的0xC0字节可能出现，这并不表示它们之间存在空帧。空帧是不被允许的。任何帧的第一个字节用于向TNC发送命令。当没有涉及TNC时，第一个字节是0x00，实际的帧数据字节跟在后面。字节0xC0不能出现在帧的内部。如果数据中包含该字节，它会被替换为0xDB 0xDC。如果数据中出现0xDB，它会被替换为0xDB 0xDD。

### HDLC（高阶数据链路控制）协议

**HDLC（高阶数据链路控制）协议**是一种数据链路层协议，用于X.25和其他协议中。在AX.25中，仅使用了HDLC的一些功能来进行帧处理和帧完整性检查。首先需要注意的是，HDLC通常采用NRZ-I编码方式。这意味着逻辑0位通过状态变化来表示（例如，从正频偏变为负频偏或反之），而逻辑1位则通过状态不变来表示。帧结束标志是序列01111110（或0x7e）。注意，其他NRZ-I的实现可能采用相反的约定。该标志也用于帧的起始标记，并作为帧之间的空闲信号。在传输帧之前，会发送多个0x7e字节以帮助接收器同步。同样，在帧传输完毕后，通常会多次发送0x7e字节以确保其中一个标志被无误接收。

HDLC通过限制数据中连续1的长度来避免长时间的1导致接收器失去同步（除了0x7e标志外，其包含6个连续的1）。每当数据中出现5个连续的1时，会在后面插入一个0，这被称为位填充。当然，当接收器检测到5个连续的1时，如果后续位是0，则应忽略；如果是1，则接收器应期望后续位是0，因为这必须是0x7e标志的最后一位。

HDLC的另一个重要特点是，每个字节的传输是从最低有效位开始的，这与许多其他协议相反。HDLC在AX.25中使用的最后一个功能是帧检验序列（FCS）。这是一个16位的校验和，附加在数据帧之后。校验和是使用CRC-16-CCITT计算的。FCS的发送顺序是最低有效字节优先（同时要记住，每个字节是最低有效位优先，并且执行了位填充）。

需要注意的是，由于HDLC使用NRZ-I编码，因此它对信号的极性不敏感。因此，信号极性可以反转而不会引起问题。正因如此，当使用BPSK传输HDLC帧时，不使用差分编码。

### AX.25协议

**AX.25协议**是一种业余无线电使用的数据链路层协议。其帧处理和帧校验序列（FCS）是使用上述的HDLC方式完成的。AX.25帧本身并不包括FCS或其他任何校验和。协议的详细信息可以参考[这里](https://www.tapr.org/pdf/AX25.2.2.pdf)。阅读该文档时，请记住，0x7e标志和FCS并不是真正属于AX.25帧的一部分。AX.25协议具有许多功能，这里不做详细描述。由于Linux能够处理AX.25协议，因此了解Linux如何用于处理AX.25是非常有用的。此外，[Wireshark](https://www.wireshark.org/)也可以用于检查和分析AX.25帧。

### 调制方式

了解AX.25 HDLC帧是如何在空中传输的也很重要。第一种方法是使用AFSK（音频频移键控）。这种方式通常在短波（HF）频段以300波特的速率使用。NRZ-I位以音频信号的形式传输，该信号在相距200Hz的两个音调之间进行频率切换。无线电设备设置为SSB（单边带）模式，因此实际的发射信号实际上是FSK（频移键控）。所使用的特定音调不是标准的，因此需要通过正确设置无线电拨盘频率来进行补偿。使用LSB（下边带）或USB（上边带）模式都可以，因为信号对极性反转不敏感。

第二种方法是使用FM AFSK。这通常在VHF和UHF频段以1200波特的速率使用。NRZ-I位以音频信号的形式传输，频率在1200Hz和2200Hz的音调之间切换。该音频信号在传输前进行FM调制。

第三种方法是使用[G3RUH FSK](http://www.amsat.org/amsat/articles/g3ruh/109.html)。这通常在UHF频段以9600波特的速率使用。HDLC位流在NRZ-I编码之前使用带有多项式1+x¹²+x¹⁷的乘法扰码器进行扰码。NRZ-I信号随后被整形（低通滤波），并直接驱动FM调制器，以产生FSK信号。

第四种方法是使用BPSK（相移键控）。这种方法在一些业余卫星中使用，速率为1000或1200波特。NRZ-I位以BPSK信号形式传输（未使用差分编码）。可以在计算机上将BPSK信号生成为音频信号，然后用它来驱动SSB发射机。

最后，[fldigi](http://www.w1hkj.com/)可以作为KISS TNC，允许以该程序支持的多种模式发送AX.25帧。然而，这些数字模式通常用于基于文本的聊天，很少用于AX.25。

[gr-kiss](https://github.com/daniestevez/gr-kiss)包含示例流程图，展示了1k2 FM AFSK、9k6 FSK和1k2 BPSK调制是如何工作的。

### 在Linux中连接AX.25

正如前面提到的，将AX.25系统与Linux连接有时是非常有用的。要使用Linux的AX.25功能，首先需要为每个AX.25接口声明一个“端口”，这些接口将由Linux处理。端口在`/etc/ax25/axports`中声明。

文件示例：

```plaintext
# /etc/ax25/axports
#
# 该文件的格式为：
#
# name callsign speed paclen window description
#
test    EA4GPZ-10    38400   2000    2   test
test2   EA4GPZ-9     38400   2000    2   test2
```

这是一个示例。如果这些端口不通过串行（RS232）端口连接到硬件TNC，则速率并不重要。

每个端口需要连接到一个串行端口或pty设备（一种虚拟串行端口）。如果我们想将一些AX.25软件连接到Linux的AX.25接口，可以使用两个工具：`kissnetd`和`socat`。`kissnetd`用于创建一对互相连接的pty设备。写入其中一个pty设备的所有内容都可以在另一个设备上读取，反之亦然。这对于AX.25软件能够读取和写入pty设备的情况非常适用。

例如，我们使用`kissnetd`将一个KISS文件发送到Linux AX.25接口。这可以用于通过Wireshark分析帧，通过让Wireshark捕获AX.25接口上的流量。

```bash
# kissnetd -p 2 &
kissnetd V 1.5 by Frederic RIBLE F1OAT - ATEPRA FPAC/Linux Project

等待客户端连接于:
/dev/pts/6 /dev/pts/7

# kissattach /dev/pts/6 test
AX.25端口test绑定到设备ax0
# cat file.kiss > /dev/pts/7
```

`socat`是一个更加灵活的工具。它也用于创建两种“设备”并将它们连接在一起。但是，它支持多种不同的设备：pty设备、TCP和UDP套接字、文件、管道、UNIX套接字等。例如，我们可以使用`socat`将pty设备与UDP套接字连接，然后通过UDP发送一个KISS文件。

```bash
# socat PTY,link=/tmp/serial UDP-LISTEN:1234 &
# kissattach /tmp/serial test
AX.25端口test绑定到设备ax0
$ nc -4u localhost 1234 < sats/firebird-4-2015032418.kiss 
nc: using datagram socket
```

作为另一个示例，我们可以使用`socat`设置一个TCP客户端，连接到[gr-kiss](https://github.com/daniestevez/gr-kiss)中的示例之一。

```bash
# 运行gr-kiss中的示例
# socat PTY,link=/tmp/serial TCP:localhost:52001 &
# kissattach /tmp/serial test
AX.25端口test绑定到设备ax0
```

Linux AX.25接口的基本使用

Linux AX.25接口可以像任何其他网络接口一样配置用于IPv4。然后可以使用常规的IP流量工具。此外，还有几个特定于AX.25的工具，可用于监控AX.25流量。请记住，Wireshark也可以捕获并分析AX.25接口中的流量。AX.25连接也可以通过这些工具来实现，这也是生成测试数据包的简便方法。 
