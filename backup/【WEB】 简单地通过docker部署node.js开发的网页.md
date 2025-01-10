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

# 2. docker镜像编译以及部署

## 2.1. docker镜像的编译

对于没有用docker-compose的项目，项目根目录下运行即可

```bash
docker build -t orbit-display:test .
```

![image](https://github.com/user-attachments/assets/873abe7e-3885-4308-8298-aaed5ec3e1a8)

计算机上没有基础镜像的会自动先下载基础镜像，因此可以提前下载基础镜像把步骤分开来。
如果使用了docker desktop则需要开着它编译。

编译好后就会出现你的镜像
![image](https://github.com/user-attachments/assets/9e332252-7b89-457a-beca-2b7816fc766e)

## 2.2. docker镜像的部署

### 2.2.1. 最简单的方法是直接复制镜像文件
```bash
docker save -o Orbirt-Display.tar orbit-display:test
```

![image](https://github.com/user-attachments/assets/ef60e14e-8480-4f11-a95d-a6b2d078dbf4)

这会直接把镜像.tar文件保存到你运行指令的目录下，复制到服务器上，运行

```bash
docker load -i <文件名>.tar
```

无需解压就能加载镜像，然后通过这个镜像创建容器

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
