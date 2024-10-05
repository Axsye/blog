拿到板子之后，按照Quick Start Guide启动BIST发现INIT_B始终为红色，BoardUI连上没反应，FMC卡插上后VADJ FMC不亮
于是看油管上别人的BIST视频

> https://www.youtube.com/watch?v=u-6CQBHGNG8&list=PL6G4uDwVg2cljE07uKgnMy16SJH97KMYV&index=4

找到了遇到同样问题的帖子

> https://adaptivesupport.amd.com/s/question/0D52E00006iHqvxSAC/zcu111-power-on-failed?language=en_US

建议重新烧录QSPI
![image](https://github.com/user-attachments/assets/44fee2b2-3307-4462-bf93-d3e2769a93f9)

> Hi,
> It seems there is some problem with the power rails as multiple power and status LEDs are off and SCUI readings are askew.
> Please try restoring the flash and see it helps with the BIST.
> Tutorial- https://www.xilinx.com/member/forms/download/design-license.html?cid=0540b548-181d-4514-a120-19cbbcec3243&filename=xtp515-zcu111-restoring-flash-c-2018-3.pdf
> File- https://www.xilinx.com/member/forms/download/design-license.html?cid=497f9c5d-42a1-41c2-b1c6-2bab976583f9&filename=rdf0473-zcu111-restoring-flash-c-2018-3.zip
> If the issue persists , file a RMA SR with the Xilinx for further technical debug. 
> https://www.xilinx.com/support/quality/product-return.html
> Best Regards,
> Vatsal

烧录教程（需要装好vivado，启动TCL Shell）：
[xtp515-zcu111-restoring-flash-c-2018-3.pdf](https://github.com/user-attachments/files/17266452/xtp515-zcu111-restoring-flash-c-2018-3.pdf)
文件：
[rdf0473-zcu111-restoring-flash-c-2018-3.zip](https://github.com/user-attachments/files/17266453/rdf0473-zcu111-restoring-flash-c-2018-3.zip)

重新烧录后还是亮红灯，需要手动按一下SW3来重启，重启后INIT_B正常，VADJ FMC正常，电源 DS40 PS_DDR4_VTEAM有时启动后亮有时不亮（怀疑这里还是有问题）

