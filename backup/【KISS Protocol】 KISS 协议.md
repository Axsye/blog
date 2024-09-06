# 1. KISS是啥 from Bing:
The **KISS protocol** (Keep It Simple, Stupid) is used in amateur radio to facilitate communication between a computer and a Terminal Node Controller (TNC). It simplifies the process by removing complex protocol handling from the TNC and shifting it to the host computer¹².

Here are some key points about the KISS protocol:
- **Purpose**: It allows the TNC to handle basic tasks while the host computer manages more complex protocol functions².
- **Functionality**: The protocol uses simple commands to control the TNC and transmit data¹.
- **Common Use**: It's often used with AX.25 packet radio frames and is popular in TCP/IP operations over amateur radio².



源: 与 Copilot 的对话， 2024/9/6
[1] KISS (amateur radio protocol) - Wikipedia. (https://en.wikipedia.org/wiki/KISS_(amateur_radio_protocol))
(2) KISS Protocol - AX.(http://www.ax25.net/kiss.aspx)
(3) KISS, HDLC, AX.25 and friends – Daniel Estévez. (https://destevez.net/2016/06/kiss-hdlc-ax-25-and-friends/).

---

The KISS protocol has several practical applications, particularly in the realm of amateur radio and related fields. Here are a few notable ones:

1. **Packet Radio**: KISS is widely used in packet radio communications, where it helps in transmitting digital data over radio frequencies. It simplifies the interface between the computer and the TNC, making it easier to handle AX.25 packet frames.

2. **Satellite Communications**: Amateur radio operators use KISS for communicating with satellites. It allows for efficient data transmission and reception, which is crucial for satellite telemetry and control.

3. **Emergency Communications**: In emergency situations, KISS can be used to set up quick and reliable communication networks. Its simplicity and efficiency make it ideal for transmitting critical information during disasters.

4. **TCP/IP over Radio**: KISS is often employed in TCP/IP operations over amateur radio, enabling the use of internet protocols over radio links. This is useful for setting up wireless networks in remote areas.

5. **Telemetry and Remote Sensing**: KISS can be used in telemetry systems to transmit sensor data from remote locations to a central monitoring station. This is useful in various fields, including environmental monitoring and scientific research.

---

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

---

# 2. 研究KISS的具体实现方法

## 2.1. 相关文章
### 2.1.1. 基本知识——来自 BH2VJW 的 Packet Radio科普系列笔记

>这blog网页真的好看

- https://tccmu.com/2022/07/26/packet/

### 2.1.2. 基于ATMega328微控制器的APRS Kiss TNC: 从原理到C语言实现的完整指南

- https://blog.csdn.net/m0_57781768/article/details/133174158

### 2.1.3.  HB9CVV 的英文PDF教程

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