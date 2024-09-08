# 1. KISS是啥 

## 1.1. EA4GPZ / M0HXM 的[网站](https://destevez.net/)上的文章摘录 **_KISS, HDLC, AX.25 and friends_** 

- https://destevez.net/2016/06/kiss-hdlc-ax-25-and-friends/
- [自己翻译](https://axsye.github.io/blog/post/%E3%80%90KISS%2C%20HDLC%2C%20AX.25%20and%20friends%E3%80%91%20EA4GPZ%20-%20M0HXM%20-de-wen-zhang-fan-yi.html)

> KISS部分原文：
> KISS The KISS protocol was originally designed to interface a computer with a TNC (Terminal Network Controller). Think of the TNC as a sort of modem that does some modulation and low level framing functions. The idea of KISS is to move most of the processing to the host computer. Before KISS, a TNC usually performed some AX.25 related tasks. However, a KISS TNC only works with HDLC and passes raw AX.25 frames to the host. The KISS protocol is described [here](http://www.ax25.net/kiss.aspx). It provides a way to separate frames and to send commands to the TNC. Since this protocol is so simple, it is also used in many situations where no TNC is involved. In this case, it is just used to divide a stream of data into frames. Thus, a series of packets can be saved to a file “in KISS format”. These packets are usually AX.25 frames, but this need not be the case. Also, different pieces of software can exchange packets using the KISS format.
> 
> The end of a frame is marked by the byte 0xC0. The start of a frame need not be explicitly marked (but it is usually also marked with 0xC0). Several consecutive 0xC0 bytes may appear. This doesn’t mean that there are empty frames between them. Empty frames are not allowed. The first byte of any frame is used to send commands to the TNC. When a TNC is not involved, the first byte is 0x00. Then the real frame bytes come after it. The byte 0xC0 can not appear anywhere inside a frame. If the data contains this byte, it is replaced by 0xDB 0xDC. If 0xDB appears in the data, it is replaced by 0xDB 0xDD.

### 翻译：

### KISS协议

**KISS协议**最初是为计算机与TNC（终端网络控制器）之间的接口而设计的。可以将TNC看作一种调制解调器，它负责一些调制和低级帧处理的功能。KISS的理念是将大部分的处理任务转移到主机计算机上。在KISS协议出现之前，TNC通常会执行一些与AX.25相关的任务。然而，KISS TNC只处理HDLC，并将原始的AX.25帧传递给主机。KISS协议的详细描述请见[这里](http://www.ax25.net/kiss.aspx)。该协议提供了一种分离帧和向TNC发送命令的方法。由于这个协议非常简单，它也被应用于许多不涉及TNC的场景中。在这些情况下，它仅用于将数据流分隔为帧。因此，一系列数据包可以被保存为“KISS格式”的文件。这些数据包通常是AX.25帧，但并不局限于此。此外，不同的软件也可以使用KISS格式来交换数据包。

帧的结束标志是字节0xC0。帧的开始不需要明确标记（但通常也用0xC0标记）。多个连续的0xC0字节可能出现，这并不表示它们之间存在空帧。空帧是不被允许的。任何帧的第一个字节用于向TNC发送命令。当没有涉及TNC时，第一个字节是0x00，实际的帧数据字节跟在后面。字节0xC0不能出现在帧的内部。如果数据中包含该字节，它会被替换为0xDB 0xDC。如果数据中出现0xDB，它会被替换为0xDB 0xDD。


---

## 1.2. 基本知识——来自 BH2VJW 的 Packet Radio科普系列笔记

>这blog网页真的好看

- https://tccmu.com/2022/07/26/packet/

## 1.3.来自Bing

The **KISS protocol** (Keep It Simple, Stupid) is used in amateur radio to facilitate communication between a computer and a Terminal Node Controller (TNC). It simplifies the process by removing complex protocol handling from the TNC and shifting it to the host computer¹².

Here are some key points about the KISS protocol:
- **Purpose**: It allows the TNC to handle basic tasks while the host computer manages more complex protocol functions².
- **Functionality**: The protocol uses simple commands to control the TNC and transmit data¹.
- **Common Use**: It's often used with AX.25 packet radio frames and is popular in TCP/IP operations over amateur radio².



源: 与 Copilot 的对话， 2024/9/6
[1] KISS (amateur radio protocol) - Wikipedia. (https://en.wikipedia.org/wiki/KISS_(amateur_radio_protocol))
(2) KISS Protocol - AX.(http://www.ax25.net/kiss.aspx)
(3) KISS, HDLC, AX.25 and friends – Daniel Estévez. (https://destevez.net/2016/06/kiss-hdlc-ax-25-and-friends/).



The KISS protocol has several practical applications, particularly in the realm of amateur radio and related fields. Here are a few notable ones:

1. **Packet Radio**: KISS is widely used in packet radio communications, where it helps in transmitting digital data over radio frequencies. It simplifies the interface between the computer and the TNC, making it easier to handle AX.25 packet frames.

2. **Satellite Communications**: Amateur radio operators use KISS for communicating with satellites. It allows for efficient data transmission and reception, which is crucial for satellite telemetry and control.

3. **Emergency Communications**: In emergency situations, KISS can be used to set up quick and reliable communication networks. Its simplicity and efficiency make it ideal for transmitting critical information during disasters.

4. **TCP/IP over Radio**: KISS is often employed in TCP/IP operations over amateur radio, enabling the use of internet protocols over radio links. This is useful for setting up wireless networks in remote areas.

5. **Telemetry and Remote Sensing**: KISS can be used in telemetry systems to transmit sensor data from remote locations to a central monitoring station. This is useful in various fields, including environmental monitoring and scientific research.



In the context of the KISS protocol, **TNC** stands for **Terminal Node Controller**. A TNC is a device used in amateur radio to convert digital data from a computer into audio signals that can be transmitted over radio waves, and vice versa. Essentially, it acts as a modem for radio communications¹².

Here are some key functions of a TNC:
- **Packet Assembler/Disassembler**: It assembles data into packets for transmission and disassembles received packets back into data.
- **Modem**: It modulates digital data into audio signals for transmission and demodulates received audio signals back into digital data.
- **Protocol Handling**: It can handle various protocols, such as AX.25, which is commonly used in amateur radio packet communications².

Would you like to know more about how TNCs work or their applications in amateur radio?
¹: [Wikipedia](https://en.wikipedia.org/wiki/KISS_%28amateur_radio_protocol%29)
²: [AX.25](https://www.ax25.net/kiss.aspx)

源: 与 Copilot 的对话， 2024/9/6

(3) The KISS TNC – too simple, too stupid? – owenduffy.net. https://owenduffy.net/blog/?p=2969.
(4) The KISS TNC: A Simple Communications Protocol. https://minnie.tuhs.org/Seminars/Dual/section3_15.html.

## 1.4. 个人理解（需要确认）

### KISS协议

KISS（Keep It Simple, Stupid）协议主要用于在计算机和TNC（终端网络控制器）之间建立接口。KISS的设计理念是将大部分处理工作转移到主机计算机上，简化TNC的任务。KISS TNC只处理HDLC（高级数据链路控制）并将原始AX.25帧传递给主机。

### HDLC协议

HDLC是一种数据链路层协议，通常用于X.25等协议中。在AX.25中，HDLC的作用是处理帧的封装和帧完整性的校验。HDLC使用NRZ-I编码方式，并通过比特填充机制避免长时间的连续1，从而保持接收器的同步。

### AX.25协议

AX.25是专为业余无线电设计的数据链路层协议，主要用于分组无线电通信。AX.25的帧处理和FCS校验与HDLC协议类似。Linux支持AX.25协议，可以通过配置AX.25端口和使用相关工具如`socat`、`kissnetd`等来实现AX.25通信。Wireshark还可以用于捕获和分析AX.25帧。

### 如何协同工作

这些协议在业余分组无线电通信中协同工作：KISS用于简化主机与TNC之间的接口，HDLC负责数据的帧封装和完整性校验，而AX.25则是实际应用于无线电通信的协议层。通过GNU Radio的gr-kiss模块，可以方便地处理这些协议的帧，进行自定义数据处理、分析和测试。



---


# 2. 研究KISS的具体实现方法

## 2.1. 相关文章

### 2.1.1. 基于ATMega328微控制器的APRS Kiss TNC: 从原理到C语言实现的完整指南

- https://blog.csdn.net/m0_57781768/article/details/133174158

### 2.1.2.  HB9CVV 的英文PDF教程

- 原始链接 http://www.pestingers.net/pages-images/heathkit/radio-equipment/kiss-tnc.pdf
- 以防失效的本地版本 [kiss-tnc.pdf](https://github.com/user-attachments/files/16908903/kiss-tnc.pdf)



## 2.2. GitHub 仓库——kiss3 - Python KISS Module

> 来自GitHub的python包 https://github.com/python-aprs/kiss3

kiss3 is a Python Module that implements the [KISS Protocol](https://en.wikipedia.org/wiki/KISS_(TNC)) for communicating with KISS-enabled devices (such as Serial or TCP TNCs) and provides support for encoding and decoding AX.25 frames.

### Versions

- 8.x branch from python-aprs as kiss3, supports python 3.6+

Previous versions were released by ampledata as kiss:

- 7.x.x branch and-on will be Python 3.x ONLY.
- 6.5.x branch will be the last version of this Module that supports Python 2.7.x

### Installation

Install from pypi using pip: `pip install kiss3`

### Usage Examples

Read & print frames from a TNC connected to '/dev/ttyUSB0' at 1200 baud:

```python
import kiss

def p(x): print(x)  # prints whatever is passed in.

k = kiss.SerialKISS('/dev/ttyUSB0', 1200)
k.start()  # inits the TNC, optionally passes KISS config flags.
k.read(callback=p)  # reads frames and passes them to `p`.
```
See also: examples/ directory.