原始项目
https://github.com/guaguastandup/zotero-pdf2zh?tab=readme-ov-file

起因是不想要每台电脑都部署一遍后端，想着服务器用docker部署就完事了，结果花了一晚上，比一台一台部署浪费时间多多了（悲）

官方README下的docker部署方案不知道为什么总是有奇奇怪怪的问题，且有些更新不及时，有些只支持arm64，导致各种失败

直接上文件
> 最好在build前先pull一下python:3.12-slim

Dockerfile

```Dockerfile
# 使用 Python 3.12 官方镜像
FROM python:3.12-slim

# 设置工作目录
WORKDIR /app

# 1. 安装系统依赖 (添加了 libgl1 和 libglib2.0-0 以支持 OpenCV)
RUN apt-get update && apt-get install -y \
    wget \
    unzip \
    git \
    curl \
    libgl1 \
    libglib2.0-0 \
    && rm -rf /var/lib/apt/lists/*

# 2. 使用国内镜像安装 uv
RUN pip install uv -i https://pypi.tuna.tsinghua.edu.cn/simple

# 3. 下载项目文件
RUN wget https://ghproxy.net/https://raw.githubusercontent.com/guaguastandup/zotero-pdf2zh/refs/heads/main/server.zip \
    && unzip server.zip \
    && rm server.zip

# 进入 server 目录
WORKDIR /app/server

# 4. 安装基础依赖
RUN pip install --no-cache-dir -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

# 暴露端口
EXPOSE 8890

# 5. 启动命令
CMD ["python", "server.py", "--env_tool=uv", "--port=8890", "--mirror_source=https://pypi.tuna.tsinghua.edu.cn/simple/"]
```

docker-compose.yaml

```docker-compose
services:
  zotero-pdf2zh:
    build: .
    container_name: zotero-pdf2zh
    ports:
      - "8890:8890"
    volumes:
      # 持久化存储：避免容器重启后重新下载翻译引擎
      - ./data:/app/server/.venv
      - ./engines:/root/.cache/uv  # 如果使用 uv，缓存环境加速启动
    restart: always
    environment:
      - TZ=Asia/Shanghai
```

两个文件放到一个文件夹下

构建并启动服务：

```Bash

docker compose up -d
```

> 注：第一次启动时，容器内会执行依赖安装和引擎下载，可能需要几分钟。

查看运行日志（确认是否启动成功）：

```Bash

docker compose logs -f
```

20页字数比较多的pdf大约需要翻译5分钟，消耗0.25元token，对于docker所在的服务器平台资源消耗较高

<img width="3101" height="1789" alt="Image" src="https://github.com/user-attachments/assets/63eb03f2-ad9a-44b7-9990-8ee223c5d5e8" />

翻译结束的log

<img width="2156" height="1600" alt="Image" src="https://github.com/user-attachments/assets/064d37a5-a132-4476-8edd-5f091f3a6ef6" />

最后是翻译结果

<img width="2567" height="1699" alt="Image" src="https://github.com/user-attachments/assets/fb17353d-104b-45da-adb9-9fcb622d4efb" />

<img width="2541" height="1676" alt="Image" src="https://github.com/user-attachments/assets/a83ad746-d201-46a9-93ec-99796372fb60" />

<img width="3392" height="1903" alt="Image" src="https://github.com/user-attachments/assets/8b31a6c3-84d7-4069-a093-55ed6145b3ca" />

<img width="2688" height="1737" alt="Image" src="https://github.com/user-attachments/assets/654873a5-c80b-47f7-83d6-16ff61412aa4" />