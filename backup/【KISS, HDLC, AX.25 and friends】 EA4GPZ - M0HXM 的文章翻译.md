# KISS, HDLC, AX.25 and friends

- EA4GPZ / M0HXM 的[网站](https://destevez.net/)上的文章 **_KISS, HDLC, AX.25 and friends_**

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

---

# 网站上的问答

## Jeremy Kitchen - May 28, 2019 at 05:28 UTC

> So, I’m confused. The PDF you link to there of the AX.25 spec says there are 2 bytes in the frame just after the control field that are the FCS. However, you also say that they aren’t there. And in my exploration of real-world usage so far (sniffing the packets of my Winlink client checking in), I concur, that FCS field isn’t being sent with any packet I’ve seen.
> 
> Is the spec wrong? Or is that a newer version than most implementations use? Or something else?
> 
> I’m working on an AX.25 library for Swift so I can do some packet stuff on my Mac/iOS devices and am getting to the FCS portion of the frame 🙂
> 
> Thanks!

我有点困惑。你链接的AX.25规范的PDF中提到，在控制字段之后的帧中有2个字节是FCS。然而，你也说它们并不存在。而且，在我对实际使用情况的探索中（监听我的Winlink客户端的检查包），我也发现，在我见到的所有包中都没有看到FCS字段。

是规范错了吗？还是说这是大多数实现使用的较新版本？还是其他原因？

我正在为Swift开发一个AX.25库，以便我可以在Mac/iOS设备上处理一些数据包的事情，目前正在处理帧中的FCS部分 🙂

谢谢！

---

## destevez - May 28, 2019 at 17:21 UTC

> Hi Jeremy,  
> Sorry about my wording. Perhaps it was confusing. In the terminology I follow in this post (of course there are different ways to explain things), the `0x7e` flags and the CRC form part of the HDLC frame. The AX.25 frame (addresses, PID, etc.) is transmitted as the payload of the HDLC frame, and so it doesn’t include the `0x7e` flags nor CRC. So in this post, I don’t consider the CRC to form part of the AX.25 frame, but rather of the HDLC frame.
> 
> This is consistent with the common use of AX.25 frames through a KISS interface. As a matter of fact, CRCs are not transmitted through the KISS interface.

你好，Jeremy，  
抱歉我的措辞可能有些混淆。在这篇文章中，我使用的术语（当然有不同的解释方法）是：`0x7e`标志和CRC是HDLC帧的一部分。AX.25帧（地址、PID等）作为HDLC帧的有效载荷传输，因此不包括`0x7e`标志和CRC。因此，在本文中，我不认为CRC是AX.25帧的一部分，而是HDLC帧的一部分。

这与通过KISS接口使用AX.25帧的常见方式一致。实际上，CRC不会通过KISS接口传输。

---

## Jeremy Kitchen - May 28, 2019 at 21:36 UTC

> Thanks for the reply!
> 
> So an HDLC frame is like a different wrapper around the AX.25 frame but also affects the contents? The spec says there’s a protocol field and an FCS field, so I’m confused 🙂 I don’t have any experience with HDLC or even know where I would use that, I’m working from a KISS TNC, so maybe things are very different, then 🙂
> 
> If there’s no CRC with KISS+AX.25, how is packet integrity verified? Or is that left as an exercise for the layer 3 or layer 7 implementation?

谢谢你的回复！

所以HDLC帧就像是AX.25帧的一个不同的封装，但也会影响内容吗？规范中提到有一个协议字段和一个FCS字段，所以我有点困惑 🙂 我没有HDLC的经验，甚至不知道在哪里使用它，我是从KISS TNC开始的，所以可能情况非常不同，对吗？🙂

如果KISS+AX.25没有CRC，如何验证数据包的完整性？还是说这部分工作留给了第3层或第7层实现？

---

## destevez - May 28, 2019 at 21:42 UTC

> In AX.25, the physical (OSI L1) and link (OSI L2) layers are somewhat mixed up, so I prefer to distinguish them and say that HDLC is the physical layer, while AX.25 is the link layer. This makes sense, as AX.25 can then be sent on top of other physical layers, such as KISS. (Note that however this correspondence doesn’t follow the OSI model 100% accurately).
> 
> An HDLC frame doesn’t affect the contents of an AX.25 frame. It just encapsulates it.
> 
> There is no CRC in AX.25 on top of KISS. Packet integrity is taken for granted, since this is used for a serial link between a host and a TNC, for routing AX.25 between different software, or for storage of AX.25 frames. Of course, when transmitting AX.25 over the air, it is encapsulated on HDLC, so a CRC is added.

在AX.25中，物理层（OSI L1）和链路层（OSI L2）有些混杂，因此我更喜欢将它们区分开来，并说HDLC是物理层，而AX.25是链路层。这是有意义的，因为AX.25可以在其他物理层（如KISS）之上发送。（注意，这种对应关系并不完全符合OSI模型的100%精确性。）

HDLC帧不会影响AX.25帧的内容。它只是将其封装。

在KISS之上的AX.25中没有CRC。数据包的完整性被默认假定，因为这是用于主机和TNC之间的串行链接，用于在不同软件之间路由AX.25，或用于存储AX.25帧。当然，当通过无线传输AX.25时，它被封装在HDLC中，因此会添加CRC。

---

## Jeremy Kitchen - May 29, 2019 at 00:12 UTC

> OH! So the TNC actually adds its own CRCing before sending it over the air? And that’s the HDLC stuff? Interesting! Thanks for the info!

哦！所以TNC实际上在通过无线发送之前会添加自己的CRC？这就是HDLC的作用吗？有意思！谢谢提供的信息！


---

## destevez - May 29, 2019 at 05:53 UTC

> Yes, the CRC is usually added/checked by the TNC.

是的，CRC通常由TNC添加或检查。

---

# 讨论和理解

## 个人理解（需要确认）

### KISS协议

KISS（Keep It Simple, Stupid）协议（就添加了个帧尾，以及数据区出现帧尾的规避方法）主要用于在计算机和TNC（终端网络控制器）之间建立接口。KISS的设计理念是将大部分处理工作转移到主机计算机上，简化TNC的任务。KISS TNC（使用KISS和计算机通信的TNC）只处理HDLC（高级数据链路控制，其中的CRC和帧头帧尾）并将原始AX.25帧传递给主机。KISS用在短距离串口通信，不添加CRC。

### HDLC协议

HDLC是一种数据链路层协议，通常用于AX.25等协议中。在X.25（和AX.25不完全是同一个）中，HDLC的作用是处理帧的封装和帧完整性的校验。HDLC使用NRZ-I编码方式，并通过比特填充机制避免长时间的连续1，从而保持接收器的同步。HDLC用在TNC发送数据时添加，并由另一个TNC接受后解码，是对AX.25外面套了个壳用于无线传输时的可靠性提升。

### AX.25协议

AX.25是专为业余无线电设计的数据链路层协议，主要用于分组无线电通信。AX.25的帧处理和FCS校验与HDLC协议类似。Linux支持AX.25协议，可以通过配置AX.25端口和使用相关工具如`socat`、`kissnetd`等来实现AX.25通信。Wireshark还可以用于捕获和分析AX.25帧。

### 如何协同工作

这些协议在业余分组无线电通信中协同工作：KISS用于主机与TNC之间的接口；HDLC用于TNC和TNC间通信，负责数据的帧封装和完整性校验；而AX.25则是实际应用于无线电通信的协议层，是KISS和HDLC的内容。通过GNU Radio的gr-kiss模块，可以方便地处理这些协议的帧，进行自定义数据处理、分析和测试。

通过无线电完成两台计算机之间的通信，通常的流程是这样的：

1. **计算机1** -> 原始的 **AX.25(数据)** 封包成 **KISS(AX.25(数据))** 通过串行或USB接口发送到 **TNC1**。
2. **TNC1** -> 将计算机1的 **KISS(AX.25(数据))** 进行解包成 **AX.25(数据)** ，转换成 **HDLC(AX.25(数据))** 调制，转换为无线电信号 **调制(HDLC(AX.25(数据)))** 。
3. **无线电设备1** -> 通过天线将调制后的信号 **调制(HDLC(AX.25(数据)))** 发送到空气中。
4. **无线电设备2** -> 另一端的接收无线电设备通过天线接收到信号 **调制(HDLC(AX.25(数据)))** 。
5. **TNC2** -> 将接收到的无线电信号 **调制(HDLC(AX.25(数据)))** 解调、转换回数字数据 **AX.25(数据)** ，变成 **KISS(AX.25(数据))** 发送到 **计算机2**。
6. **计算机2** -> 通过串行或USB接口从 **TNC2** 接收 **KISS(AX.25(数据))** 解包后得到 **AX.25(数据)** 。

所以完整的链路是：

**计算机1 → TNC1 → 无线电设备1 → 无线电链路（空气） → 无线电设备2 → TNC2 → 计算机2**

TNC1和TNC2分别负责数据的调制和解调，而无线电设备负责实际的信号传输和接收。无线电链路是通过空气中的无线电波完成的。
