> 通过docker安装目前已经测试3.10版本连接PlutoSDR，理论上也可以正常安装3.8和3.9版本


# 1.乌拉圭的老哥做的dockerfile

Github仓库：
https://github.com/git-artes/docker-gnuradio

这个仓库里有3.7 3.8 3.9 3.10的dockerfile，每个文件对应不同的容器，可以同时运行多个不同版本的容器
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
docker run --name=gnuradio-310 --net=host --env="DISPLAY" --volume="$HOME/.Xauthority:/root/.Xauthority:rw" --device /dev/snd -v persistent:/home/gnuradio/persistent --device /dev/dri -v /dev/bus/usb/:/dev/bus/usb/ --privileged --group-add=audio -it ubuntu:gnuradio-releases bash
```

>  这步指定了网络使用主机网络、显示使用主机显示器、声音用主机的声音、设定了一个卷persistent用于永久储存容器里的数据、映射了主机的usb、给予了高权限、最后用了刚刚编译好的镜像

启动容器后，当前终端会立刻变成容器的终端，容器的用户名和密码都是gnuradio，之后就可以在容器中通过`gnuradio-companion`启动grc3.10了。

## 后续操作：

1. 每次重新启动主机，都需要启动一下容器`sudo docker start gnuradio-310`，然后使用`sudo docker exec -it gnuradio-310 /bin/bash`来进入容器的终端。

2. 可以通过`docker stats gnuradio-310`来查看容器资源占用

![Image](https://github.com/user-attachments/assets/dfadafa4-349b-4fec-a5b0-d6ad23640efb)

3. 容器的volume永久文件保存在主机的`/var/lib/docker/volumes/persistent/_data`，在容器里是`/home/gnuradio/persistent`，容器的永久文件只能保存在这里，其他位置重启后会丢失，可以从主机进行读写。

4. 容器里默认没有装alsa，grc的audio sink无法工作，可以通过apt安装alsa，然后通过`arecord -l`来列出容器的声卡，例如我的输出是
```bash
gnuradio@hp-elitebook:~$ arecord -l
**** List of CAPTURE Hardware Devices ****
card 1: Generic_1 [HD-Audio Generic], device 0: ALC245 Analog [ALC245 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```
  这里是card 1 device 0, 那么我在audio sink里应该这么配置：
  
  ![Image](https://github.com/user-attachments/assets/8ea7b9ee-261b-4a4e-89b4-c07da3e8d4e3)

5. 在容器内可以正常安装libiio，但是我尝试使用`iio_info -s`来搜索PlutoSDR时无法搜索到设备，这个问题目前无法解决，在容器内就是无法使用`iio_info -s`搜索设备。但是可以在主机上安装libiio然后搜索pluto，得到ip后填到容器内grc的PlutoSDR Source里，可以正常运行。参考（与我遇到的情况完全相同）：https://ez.analog.com/linux-software-drivers/f/q-a/562936/iio_info-does-not-work-in-docker-container-scanning-for-iio-contexts-failed-text-file-busy

## 最后效果

![Image](https://github.com/user-attachments/assets/91177c57-0db3-4f1a-9cf5-dca0dad28268)

# 2. dockerHub上的镜像

在dockerhub上也有gnuradio镜像，但是似乎不是为了在本地使用而创建的，没有仔细研究
https://hub.docker.com/r/gnuradio/ci/tags

![Image](https://github.com/user-attachments/assets/a5548dcb-9ab0-440b-9549-61bb79bcb930)