> 通过docker安装目前已经测试3.10版本连接PlutoSDR，理论上也可以正常安装3.8和3.9版本

# 1.获取docker镜像

# 1.1.乌拉圭的老哥做的dockerfile

Github仓库：
https://github.com/git-artes/docker-gnuradio

以3.10版本安装为例

---
 原文：
If you want the release version of GNU Radio (currently 3.10): 

1. Install docker by following [Docker's installation guide](https://docs.docker.com/get-docker/) (though I've tested this on Ubuntu only, so for convenience a direct link to the corresponding [guide for Ubuntu](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)). 
2. Clone this repo: `git clone https://github.com/git-artes/docker-gnuradio.git`
3. Enter the `docker-gnuradio/gnuradio-releases` folder and execute `docker build -t ubuntu:gnuradio-releases .` (this step is necessary only once, or every time you modify *Dockerfile*) 
4. Run the container: 
```bash
docker run --net=host --env="DISPLAY" --volume="$HOME/.Xauthority:/root/.Xauthority:rw" --device /dev/snd -v persistent:/home/gnuradio/persistent --device /dev/dri -v /dev/bus/usb/:/dev/bus/usb/ --privileged --group-add=audio -it ubuntu:gnuradio-releases bash
```

You will then be in a comand line logged as user *gnuradio* who is a sudoer (password: *gnuradio*). The folder *persistent* in its home directory will persist even after you exit the container. **Remember**: everything else is erased after you exit the container. This means for instance that if you compile and install an OOT you will have to do that again every time you run the container (unless for instance you compile in the *persistent* folder). You may modify Dockerfile to avoid this. 

The host's USB should be accessible from the docker too, meaning you should be able to use any SDR hardware you plug into the PC.
---

实测所有docker命令前需要sudo
其中第四步运行容器时我使用了`sudo sudo .......`才成功运行
原文中第四步创建容器时没有指定容器名字，可以添加-name参数
```bash
docker run --name=gnuradio310 --net=host --env="DISPLAY" --volume="$HOME/.Xauthority:/root/.Xauthority:rw" --device /dev/snd -v persistent:/home/gnuradio/persistent --device /dev/dri -v /dev/bus/usb/:/dev/bus/usb/ --privileged --group-add=audio -it ubuntu:gnuradio-releases bash
```

>  这步指定了网络使用主机网络、显示使用主机显示器、声音用主机的声音、设定了一个卷persistent用于永久储存容器里的数据、映射了主机的usb、给予了高权限、最后用了刚刚编译好的镜像