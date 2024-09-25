总结之前的经验来看，只有少数情况下zt可以建立点对点，基本上还是靠中转。自建moon中转效果不明显，可能是需要换个docker镜像。根本解决办法是自建planet。

> 使用docker自建planet教程
> https://blog.csdn.net/u012153104/article/details/131829837?type=blog&rId=131829837&refer=APP&source=koude11

> github仓库https://github.com/xubiaolin/docker-zerotier-planet

**不能解析地址，ip变动需要全部重新配置**
**一键部署只支持Linux，windows下要自己想办法**

这是一开始采用的命令
```powershel
docker run -d -v zerotier-app:/app -v zerotier-one:/var/lib/zerotier-one -p 9994:9994 -p 9994:9994/udp -p 3443:3443 --name zerotier-planet --restart unless-stopped xubiaolin/zerotier-planet:latest
```

这样配置有bug：
管理页面可以登陆但是进入网络配置就会报错
Error retrieving list of networks on this controller: RequestError: connect ECONNREFUSED 127.0.0.1:9994
![image](https://github.com/user-attachments/assets/59d6c47f-0915-4f4b-b1ec-e2458cd9bfc1)

这是后来从deploy.sh中截取的命令，猜测对应替换掉其中的地址可以避免上述报错，但是由于这个镜像不能解析域名，过于飞舞，域名解析有多个issue提议，但是甚至不在开发计划中，疑似作者想通过卖他自己的服务赚米，不想尝试了

```powershell
docker run -d \
        --name ${CONTAINER_NAME} \
        -p ${ZT_PORT}:${ZT_PORT} \
        -p ${ZT_PORT}:${ZT_PORT}/udp \
        -p ${API_PORT}:${API_PORT} \
        -p ${FILE_PORT}:${FILE_PORT} \
        -e IP_ADDR4=${ipv4} \
        -e IP_ADDR6=${ipv6} \
        -e ZT_PORT=${ZT_PORT} \
        -e API_PORT=${API_PORT} \
        -e FILE_SERVER_PORT=${FILE_PORT} \
        -v ${DIST_PATH}:/app/dist \
        -v ${ZTNCUI_PATH}:/app/ztncui \
        -v ${ZEROTIER_PATH}/one:/var/lib/zerotier-one \
        -v ${CONFIG_PATH}:/app/config \
        --restart unless-stopped \
        ${DOCKER_IMAGE}
```