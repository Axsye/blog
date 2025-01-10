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
dockerfile是最基本的，为了减少不必要的，只把src目录下的前后端文件复制到/app。
运行`docker build -t orbit-display:test .`指令前，需要注意的是dockerfile里的`RUN npm install`命令。复制的目录（`/src`）下必须要有`package.json` `package-lock.json`文件，因此对于前后端分离成两个docker的情况下，npm命令可能需要在`frontend`和`backend`两个文件中分别运行（？