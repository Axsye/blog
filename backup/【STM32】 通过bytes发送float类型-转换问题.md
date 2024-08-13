# STM32对于float类型处理
![image](https://github.com/user-attachments/assets/e0cef6c9-7edd-4b3c-a03f-6d2b500c3759)
stm32采用IEEE 754, 但是是**高位字节在前**
从上图（https://www.h-schmidt.net/FloatConverter/IEEE754.html）
可以看出，对于windows，是低位在前。100对于stm32来说是0000C842，因此要注意发送端和接收端的区别
# 转换方法
经过测试，采用共用体的转换方法是实测最好用的方法。
保存在uint32中再采用(float)强制转换实测是不可行的，会变成该段数据按照uint方式解码的大小。
该文章有详细的阐释https://cloud.tencent.com/developer/article/2168376
## 实际应用
```C
// 通过共用体实现浮点数和字节之间的转换
typedef union
{
  float floatData;
  unsigned char byteData[4];
}FLOAT_BYTE;

FLOAT_BYTE data;
// 将data1的值转换为字节数组
data.byteData[0] = (data1 >> 24) & 0xFF;
data.byteData[1] = (data1 >> 16) & 0xFF;
data.byteData[2] = (data1 >> 8) & 0xFF;
data.byteData[3] = data1 & 0xFF;

stepper_x.position_des = data.floatData;
```