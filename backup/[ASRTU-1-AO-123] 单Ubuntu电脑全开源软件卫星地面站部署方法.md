<img width="3634" height="2003" alt="Image" src="https://github.com/user-attachments/assets/bae85381-8970-42c5-a863-7d40803b3589" />

<img width="889" height="459" alt="Image" src="https://github.com/user-attachments/assets/f2734ccb-1ae0-4b45-bc88-9ec96ca41448" />

# 1. 硬件配置

电台：IC-9700 USB接IF输出到电脑，同时接USB转CIV到9700的CIV接口
旋转器：G5500 控制接口 GS-2320B 通过USB连接到电脑
电脑配置：i3-5015U 8GB RAM 128GB SSD

# 2. 系统配置

> ASRTU-1网盘链接：
https://1drv.ms/f/c/a7c3226f3f26758d/Eu6cZ-zqR9pOll54FyFmgX4BcsO14ehxuyQgCw6y0rBLwA
>
>包括linux livecd、windows解调软件、图像校正软件、离线解析软件、信号录音等，动态更新

将livecd通过rufus制作u盘启动盘，确保u盘没有坏块且≥16GB
通过U盘启动后选择Try or install Ubuntu
然后直接选择install _**注意install过程中用户名必须选ubuntu**_

进入系统后桌面图标显示不能直接运行，用这个`chmod -R 775 ~/Desktop`

正常登录vpn，启动gnuradio，与常规阿斯图livecd操作没区别

# 3. 地面站软件配置

如果要用一台电脑实现地面站，需要同时接通：
1. 电台的中频输入输出，gnuradio控制
2. 电台的PTT控制，gnuradio通过9700的USB控制
3. 电台的多普勒，gpredict通过usb转civ控制
4. 旋转器，gpredict通过usb控制

## 3.1. GNURadio相关配置

9700接上USB，选择IF OUT，电脑上用arecord -l显示声卡
card 1，device 0 则gnuradio中设置hw:1,0

<img width="218" height="99" alt="Image" src="https://github.com/user-attachments/assets/8898cf93-f299-439f-b8e9-5f30c5175b5d" />

<img width="231" height="117" alt="Image" src="https://github.com/user-attachments/assets/9ab29c1a-7b29-4885-a1f7-da812341e1b0" />

记得开电脑声音……

如果遇到声卡占用，可以`sudo asla force-reload`

PTT要选择9700 ptt对应的端口

<img width="266" height="120" alt="Image" src="https://github.com/user-attachments/assets/a8d4a52b-3b83-46af-8912-94e7fd7cdd03" />

## 3.2. Gpredict相关配置

livecd中自带了gpredict和hamlib，gpredict对旋转器和电台的控制均通过hamlib实现

首先需要配置hamlib

### 3.2.1 旋转器配置

首先通过`rotctl -l`列出所有支持的旋转器

<img width="529" height="348" alt="Image" src="https://github.com/user-attachments/assets/132e8b6c-2807-497a-9e4b-84013038644b" />

可以看到GS-232B是603

> 居然还支持一些赤道仪
> <img width="532" height="341" alt="Image" src="https://github.com/user-attachments/assets/9b22b554-f66e-405e-8127-8f824420f765" />

然后运行命令

```bash
rotctld -m 603 -r /dev/ttyUSB2
```

> -m 设备代号 -r 地址 -s 波特率（缺省自动选择设备支持的最大）

启动旋转器控制的监听，将会自动监听gpredict的输出给旋转器

然后配置gpredict

<img width="1138" height="907" alt="Image" src="https://github.com/user-attachments/assets/41b56c37-3e6a-4a18-b7d9-dd9939810655" />

> locakhost和4533是hamlib默认的

<img width="794" height="444" alt="Image" src="https://github.com/user-attachments/assets/937cb2f2-f2ad-4578-b489-39f328ca1cd6" />

<img width="1193" height="525" alt="Image" src="https://github.com/user-attachments/assets/bdbd3645-5cff-4e79-8c01-0b455659b578" />

选择Engage，过一段时间不会自动跳掉即表明配置成功

### 3.2.2 电台配置

通过`rigctl -l`显示支持的电台

<img width="532" height="372" alt="Image" src="https://github.com/user-attachments/assets/c4e35127-e20e-4026-b923-fb0815ef2a7f" />

可以看到9700是3081

然后运行一下指令

```bash
rigctld -m 3081 -s 9600 -c 162 -r /dev/ttyUSB3
```

> -m 设备代号 -s 波特率 -c CIV地址(十进制) -r 端口
> 由于用usb转CIV所以波特率的9600，CIV地址需要看9700电台设置是什么，比如此时地址是A2H，指令参数就是十进制162
> 测试发现最好把-r /dev/ttyUSB3放到最后，否则有时候报非法参数

这样就启动了hamlib控制电台的监听，自动监听gpredict的控制

然后配置gpredict

<img width="1138" height="908" alt="Image" src="https://github.com/user-attachments/assets/2526a99b-e8aa-4eb0-abce-ecb8ee9625cd" />

localhost和4532是hamlib默认的，如果发现上下行相反就切换up/down

<img width="794" height="444" alt="Image" src="https://github.com/user-attachments/assets/c8418be7-5c98-41c8-a7bc-a508c4cf0751" />

<img width="1257" height="642" alt="Image" src="https://github.com/user-attachments/assets/57d775dc-4b75-4726-a1ec-fe2bbadaf78b" />

点击engage，过一段时间发现不会自动跳掉即配置成功

# 4. PS
善用hamlib的 -h 可以解决不少问题

# 5.其他的坑
 
## 5.1设置完地面站地址之后，一定还要在configure里面加上

<img width="380" height="358" alt="Image" src="https://github.com/user-attachments/assets/6fc8a0e0-3870-4f8c-bd30-d703195932cc" />

## 5.2 pulseaudio最好直接干掉

```bash

systemctl --user stop pulseaudio.socket
systemctl --user stop pulseaudio.service

systemctl --user disable pulseaudio.socket
systemctl --user disable pulseaudio.service

```

## 5.3 .desktop问题

修改目录所有者，然后给权限

```bash

ubuntu@ubuntu-PC-MKM27CZG1:~$ sudo chown -R ubuntu:ubuntu ~/Desktop
ubuntu@ubuntu-PC-MKM27CZG1:~$ ls -ld ~/Desktop
drwxrwxrwx 3 ubuntu ubuntu 4096  8月  4 20:51 /home/ubuntu/Desktop

ubuntu@ubuntu-PC-MKM27CZG1:~$ chmod 755 ~/Desktop

```

## 5.4 USB不确定可以用lsusb辅助

```bash

ubuntu@ubuntu-PC-MKM27CZG1:~$ lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 005: ID 0403:6001 Future Technology Devices International, Ltd FT232 Serial (UART) IC
Bus 001 Device 014: ID 08bb:2901 Texas Instruments PCM2901 Audio Codec
Bus 001 Device 013: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
Bus 001 Device 012: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
Bus 001 Device 011: ID 0451:2046 Texas Instruments, Inc. TUSB2046 Hub
Bus 001 Device 010: ID 0403:6001 Future Technology Devices International, Ltd FT232 Serial (UART) IC
Bus 001 Device 002: ID 8087:0a2a Intel Corp. Bluetooth wireless interface
Bus 001 Device 016: ID 1c4f:0002 SiGma Micro Keyboard TRACER Gamma Ivory
Bus 001 Device 015: ID 093a:2510 Pixart Imaging, Inc. Optical Mouse
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub


```
