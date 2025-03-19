# 1、协议编写

首先是添加对应的文件，在其中实现具体协议内容

![Image](https://github.com/user-attachments/assets/dbad9609-1388-4afd-b8ef-83f3968511d8)

![Image](https://github.com/user-attachments/assets/adbb7862-4322-40a5-8550-0da78ae6aaed)

在这里添加MCMI的下位机接收ID

![Image](https://github.com/user-attachments/assets/324e87cc-7abd-426a-9d54-d84d38610303)

![Image](https://github.com/user-attachments/assets/3e13c42a-44fd-4936-8383-33a3b28827d8)

**需要在`LOS_SYS.h`中添加新增的头文件！！**，否则不会编译新增内容

![Image](https://github.com/user-attachments/assets/b2897fad-cd34-4d8e-a463-bec2253d0eee)

# 2、遥控部分代码添加

先在这里添加

![Image](https://github.com/user-attachments/assets/2b3bab93-3405-4ee5-94f5-f659738003f2)

![Image](https://github.com/user-attachments/assets/a1cce5fb-5f73-4ef7-a518-4f92dc917f61)

这里有两处需要添加

![Image](https://github.com/user-attachments/assets/89d8d793-9e16-4d37-94f2-25d80c42ed6b)

![Image](https://github.com/user-attachments/assets/46858ca3-0dbe-4c4e-b5d0-17c99a53d4af)

![Image](https://github.com/user-attachments/assets/065f9737-e7f0-4aa2-b08e-7a3f82827fed)

可以临时通过修改指令来测试

![Image](https://github.com/user-attachments/assets/abcd410a-b9c9-4812-8810-aa2241507c0a)

# 3、遥测部分代码添加

首先需要在haicogen里配置msgbox，重点是activate、txrx、id、enable int

![Image](https://github.com/user-attachments/assets/a402f7da-ac4e-4473-a776-c70e214a2d84)

然后删除生成的多余文件，实测tilayer里面，inc有两个、src有三个