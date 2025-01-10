## 以随便做的一个demo项目为例

# 1. 文件结构以及dockerfile编写

## 1.1. 项目根目录下的文件结构如图

![image](https://github.com/user-attachments/assets/c4238a97-d708-48b4-b350-70282169a445)

这个项目里的npm指令是直接在根目录运行的，故指令生成的`package.json` `package-lock.json`都在根目录。
前后端只在文件里有区分，实际的耦合还是偏多，部署时只用了一个docker镜像，没有分离成两个镜像，只能用于小项目。

## 1.2. dockerfile

```dockerfile
# 使用 Node.js 基础镜像
FROM node:lts-alpine3.21

# 设置工作目录
WORKDIR /app

# 复制代码
COPY src .

# 安装后端依赖
RUN npm install

# 暴露后端端口
EXPOSE 3000

# 启动服务
CMD ["node", "backend/tle2orbit.js"]
```

这里的dockerfile是最基本的，也没用docker-compose。为了减少不必要的文件，只把src目录下的前后端文件复制到/app。
基础镜像的选择实际是比较复杂的，为了体积小传的快因此用的`node:lts-alpine3.21`
运行`docker build -t orbit-display:test .`指令前，需要注意的是dockerfile里的`RUN npm install`命令。复制的目录（`/src`）下必须要有`package.json` `package-lock.json`文件，因此对于前后端分离成两个docker的情况下，npm命令可能需要在`frontend`和`backend`两个文件中分别运行（？

## 1.3. 前后端分离成两个docker是更好的方法

---
> AI
- 前端用nginx，后端用node.js来分别创建docker镜像，此时后端可以不用配置静态文件服务来把前端的文件推出去
- 相当于其实前后端没什么联系，只是通过约定好的API交互。
- Dockerfile 中执行 `npm install` 时，`package.json` 文件必须存在于工作目录中。如果没有正确复制，安装依赖会失败。
- 如果 `package.json` 文件不在 Dockerfile 中定义的工作目录`（WORKDIR /app）`下，也会导致错误。
- 这种部署方法需要在一开始就考虑，在运行npm的时候需要在前端和后端目录里分别运行，如果像这个项目一样是在项目根目录运行的npm会导致package.json在根目录不在复制的文件里，从而无法编译。

使用 Docker 来部署你的前后端应用非常方便，它可以将应用及其依赖环境封装在容器中，确保一致性和易管理性。

### **1. 准备环境**

1. 确保云服务器已安装 Docker 和 Docker Compose：
   ```bash
   sudo apt update
   sudo apt install docker.io docker-compose -y
   ```
2. 启用 Docker 服务：
   ```bash
   sudo systemctl enable docker
   sudo systemctl start docker
   ```

### **2. 创建项目结构**

在服务器或本地创建一个项目目录，整理文件结构：
```plaintext
docker-deployment/
    ├── frontend/
    │   ├── OrbitDisplay.html
    │   ├── cesium_script.js
    │   ├── Cesium/          # 如果有 Cesium 的依赖文件
    │   ├── Dockerfile       # 前端的 Dockerfile
    ├── backend/
    │   ├── server.js        # 假设后端入口是 server.js
    │   ├── package.json     # Node.js 后端的依赖文件
    │   ├── Dockerfile       # 后端的 Dockerfile
    ├── docker-compose.yml   # Docker Compose 配置文件
```

### **3. 前端 Dockerfile**

在 `frontend/` 目录下创建 `Dockerfile`：
```dockerfile
# 基于 Nginx 作为 Web 服务器
FROM nginx:latest

# 将前端代码复制到 Nginx 默认的静态文件目录
COPY . /usr/share/nginx/html

# 暴露 Nginx 默认端口
EXPOSE 80

# 启动 Nginx
CMD ["nginx", "-g", "daemon off;"]
```

### **4. 后端 Dockerfile**

在 `backend/` 目录下创建 `Dockerfile`：
```dockerfile
# 使用 Node.js 基础镜像
FROM node:16

# 设置工作目录
WORKDIR /app

# 复制后端代码到容器
COPY . .

# 安装后端依赖
RUN npm install

# 暴露后端端口
EXPOSE 3000

# 启动后端服务
CMD ["node", "server.js"]
```

### **5. Docker Compose 配置**

在项目根目录创建 `docker-compose.yml` 文件：
```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
    ports:
      - "80:80" # 将容器的 80 端口映射到主机的 80 端口
    volumes:
      - ./frontend:/usr/share/nginx/html # 便于本地开发时实时同步

  backend:
    build:
      context: ./backend
    ports:
      - "3000:3000" # 将容器的 3000 端口映射到主机的 3000 端口
    volumes:
      - ./backend:/app # 便于本地开发时实时同步
    depends_on:
      - frontend
```

### **6. 构建和运行容器**

1. 在项目根目录下运行以下命令构建镜像：
   ```bash
   docker-compose build
   ```

2. 启动服务：
   ```bash
   docker-compose up -d
   ```

3. 查看运行中的容器：
   ```bash
   docker ps
   ```

### **7. 测试部署**

1. 访问前端页面：
   ```
   http://<your-server-ip>:80
   ```

2. 验证前端是否能正常调用后端接口：
   - 检查浏览器的网络请求（Network）。
   - 确保后端返回正确的 JSON 数据。

---

### **8. 备选优化**

- **Nginx 反向代理**：
  可以在 `frontend` 容器内的 Nginx 配置中添加反向代理，将后端 API 转发到 `backend` 容器：
  ```nginx
  location /api/ {
      proxy_pass http://backend:3000/;
  }
  ```
  修改 `OrbitDisplay.html` 中的请求路径为 `/api/orbit`。

- **持久化存储**：
  将数据（如日志、上传文件）挂载到主机：
  ```yaml
  volumes:
    - ./backend-data:/data
  ```

- **环境变量**：
  使用 `.env` 文件管理配置，避免将敏感信息硬编码在 Dockerfile 中。

---

### **最终目录结构**
```plaintext
docker-deployment/
    ├── frontend/
    │   ├── OrbitDisplay.html
    │   ├── cesium_script.js
    │   ├── Cesium/
    │   ├── Dockerfile
    ├── backend/
    │   ├── server.js
    │   ├── package.json
    │   ├── Dockerfile
    ├── docker-compose.yml
```

通过以上步骤，可以用 Docker 将前端和后端打包为独立的服务并部署到云服务器上，便于管理和扩展。

---

# 2. docker镜像的编译

对于没有用docker-compose的项目，项目根目录下运行即可

```bash
docker build -t orbit-display:test .
```

![image](https://github.com/user-attachments/assets/873abe7e-3885-4308-8298-aaed5ec3e1a8)

计算机上没有基础镜像的会自动先下载基础镜像，因此可以提前下载基础镜像把步骤分开来。
如果使用了docker desktop则需要开着它编译。

编译好后就会出现你的镜像
![image](https://github.com/user-attachments/assets/9e332252-7b89-457a-beca-2b7816fc766e)

# 3. docker镜像的部署

## 3.1. 最简单的方法是直接复制镜像文件

```bash
docker save -o <镜像文件名>.tar <镜像名>:<标签>
```

```bash
docker save -o Orbirt-Display.tar orbit-display:test
```

![image](https://github.com/user-attachments/assets/ef60e14e-8480-4f11-a95d-a6b2d078dbf4)

这会直接把镜像.tar文件保存到你运行指令的目录下，复制到服务器上，运行

```bash
docker load -i <文件名>.tar
```

无需解压就能加载镜像

加载完成后可以通过以下命令验证：

```bash
docker images
```

然后通过这个镜像创建容器

```bash
docker run -d --name orbit-display-http -p 80:3000 orbit-display:test
```

这样就在http的默认80端口创建了名字是orbit-display-http的容器

运行

```bash
docker start orbit-display-http
```

就能启动容器

可以使用以下指令查看运行状态等

```bash
docker stats orbit-display-http 
docker inspect orbit-display-http 
```

## 3.2. 使用Docker Hub——更加常用的方法

---
> AI

push到Docker Hub上是更常用的办法

#### **1. 在 Docker Hub 上准备镜像**

##### **登录 [Docker Hub](https://hub.docker.com/)**
- 注册 Docker Hub 账号：[Docker Hub](https://hub.docker.com)
- 登录 Docker Hub：
  ```bash
  docker login
  ```
  输入用户名和密码登录。

##### **推送镜像到 Docker Hub**
如果镜像尚未上传到 Docker Hub，可以先将其推送：

1. 给镜像打标签：
   ```bash
   docker tag <本地镜像名>:<标签> <DockerHub用户名>/<仓库名>:<标签>
   ```
   示例：
   ```bash
   docker tag my-app:latest myusername/my-app:1.0.0
   ```

2. 推送到 Docker Hub：
   ```bash
   docker push <DockerHub用户名>/<仓库名>:<标签>
   ```
   示例：
   ```bash
   docker push myusername/my-app:1.0.0
   ```

#### **2. 在服务器上部署镜像**

##### **登录 Docker Hub**
在服务器上登录 Docker Hub：
```bash
docker login
```

##### **拉取镜像**
从 Docker Hub 拉取镜像：
```bash
docker pull <DockerHub用户名>/<仓库名>:<标签>
```
示例：
```bash
docker pull myusername/my-app:1.0.0
```

##### **运行容器**
使用拉取的镜像运行容器：
```bash
docker run -d --name <容器名> -p <宿主机端口>:<容器端口> <DockerHub用户名>/<仓库名>:<标签>
```
示例：
```bash
docker run -d --name my-app-container -p 8080:3000 myusername/my-app:1.0.0
```

- `-d`：以守护模式运行容器（后台运行）。
- `--name`：指定容器名称。
- `-p`：将宿主机端口映射到容器端口。

#### **3. 检查部署状态**

##### **查看运行中的容器**
```bash
docker ps
```
输出示例：
```plaintext
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                    NAMES
abc12345xyz    myusername/my-app:1.0.0  "npm start"             5 seconds ago    Up 5 seconds    0.0.0.0:8080->3000/tcp   my-app-container
```

##### **访问应用**
- 如果服务器有公网 IP：在浏览器中访问 `http://<服务器IP>:8080`。
- 如果服务器仅内网可用，可以在本地使用 `curl` 测试：
  ```bash
  curl http://localhost:8080
  ```

#### **4. 自动化部署（可选）**

为了简化未来更新，可以创建自动化流程。

##### **使用 Docker Compose**
编写一个 `docker-compose.yml` 文件：
```yaml
version: '3'
services:
  my-app:
    image: myusername/my-app:1.0.0
    container_name: my-app-container
    ports:
      - "8080:3000"
    restart: always
```
启动服务：
```bash
docker-compose up -d
```

##### **持续更新镜像**
可以通过配置 Webhook 或使用 CI/CD 工具（如 GitHub Actions）自动将最新镜像推送到 Docker Hub并更新服务器。

#### **5. 定期更新镜像**

如果镜像在 Docker Hub 上更新了，可以拉取最新版本并重新启动容器：

1. 拉取最新镜像：
   ```bash
   docker pull <DockerHub用户名>/<仓库名>:<标签>
   ```

2. 停止旧容器并删除：
   ```bash
   docker stop <容器名>
   docker rm <容器名>
   ```

3. 使用新镜像重新运行容器：
   ```bash
   docker run -d --name <容器名> -p <宿主机端口>:<容器端口> <DockerHub用户名>/<仓库名>:<标签>
   ```

---

## 3.3. 用于实际生产等也可以使用服务器提供商的流程

---
> AI

#### **传镜像到阿里云容器镜像服务**
如果需要将镜像上传到阿里云容器镜像服务（ACR），请按照以下步骤操作：

#### **1 登录阿里云容器镜像服务**
在本地或服务器上使用 `docker login` 登录阿里云 ACR：

```bash
docker login --username=<阿里云账号或子账号> registry.cn-<地域>.aliyuncs.com
```

例如：

```bash
docker login --username=myusername registry.cn-shanghai.aliyuncs.com
```

#### **2 给镜像打标签**
为镜像添加阿里云 ACR 的标签：

```bash
docker tag <镜像名>:<标签> <阿里云镜像仓库地址>/<命名空间>/<镜像名>:<标签>
```

例如：

```bash
docker tag my_image:latest registry.cn-shanghai.aliyuncs.com/my_namespace/my_image:latest
```

#### **3 推送镜像到 ACR**
推送镜像到阿里云 ACR：

```bash
docker push <阿里云镜像仓库地址>/<命名空间>/<镜像名>:<标签>
```

例如：

```bash
docker push registry.cn-shanghai.aliyuncs.com/my_namespace/my_image:latest
```

---