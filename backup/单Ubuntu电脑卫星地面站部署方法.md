<img width="3634" height="2003" alt="Image" src="https://github.com/user-attachments/assets/bae85381-8970-42c5-a863-7d40803b3589" />

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

进入系统后桌面图标显示不能直接运行，这个自行搜索解决办法

正常登录vpn，启动gnuradio，与常规阿斯图livecd操作没区别

# 3. 地面站软件配置

如果要用一台电脑实现地面站，需要同时接通：
1. 电台的中频输入输出，gnuradio控制
2. 电台的PTT控制，gnuradio通过9700的USB控制
3. 电台的多普勒，gpredict通过usb转civ控制
4. 旋转器，gpredict通过usb控制

9700接上USB，选择IF OUT，电脑上用arecord -l显示声卡
card 1，device 0 则gnuradio中设置hw:1,0

