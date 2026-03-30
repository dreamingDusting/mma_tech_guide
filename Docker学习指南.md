# Docker学习指南

## 前言

嗨！今天来聊一个听起来很高大上，但其实非常实用的技术——Docker。如果你之前没接触过，可能会觉得"这啥玩意儿？跟我有啥关系？"

这么说吧，以前你帮同学装个软件，经常会听到："在我电脑上能运行啊！""你咋装的？""我忘了当时怎么弄的了..."

这就是传说中的"在我电脑上能运行"问题。Docker就是来解决这个问题的！它可以让你的应用"打包"好，不管在谁电脑上、啥系统上，都能跑起来，一模一样。

协会现在也在用Docker来部署服务，学好这个，你也能自己部署协会的网站了！

---

## 第一章：Docker到底是啥？

### 1.1 什么是Docker？

Docker是一个"容器化"平台。听起来有点抽象，让我用生活化的例子来解释：

**没有Docker之前：**
- 你在Windows上开发了一个Python项目
- 发给同学，他用的是Mac
- 告诉他"你装个Python3.8，再pip install这些包..."
- 同学捣鼓半天，最后来一句"还是报错"

**有了Docker之后：**
- 你把项目"打包"成一个"镜像"
- 同学直接运行这个镜像
- 搞定！完全一样的工作环境

简单理解，Docker就是一个"轻量级的虚拟机"，但比虚拟机快得多、省资源得多。

### 1.2 几个核心概念

在开始之前，先搞清楚几个名词：

**镜像（Image）**
- 想象成"模板"或者"蓝图"
- 比如Ubuntu镜像、Windows镜像、MySQL镜像
- 只读的，不能修改

**容器（Container）**
- 镜像的"实例"，就像类是对象的模板
- 可以创建、启动、停止、删除
- 相互隔离，互不影响

**仓库（Repository）**
- 存放镜像的地方
- 最常用的是Docker Hub（https://hub.docker.com/）
- 跟GitHub差不多意思

### 1.3 Docker vs 虚拟机

| 特性 | Docker | 虚拟机 |
|------|--------|--------|
| 启动速度 | 秒级 | 分钟级 |
| 占用空间 | MB级 | GB级 |
| 性能 | 接近原生 | 有损耗 |
| 隔离性 | 进程级隔离 | 系统级隔离 |

---

## 第二章：Docker基本操作

### 2.1 安装Docker

**Windows：**
去Docker官网下载Docker Desktop，安装就行。注意：需要WSL2支持。

**Mac：**
同样下载Docker Desktop，安装。

**Linux（以Ubuntu为例）：**
```bash
# 更新apt
sudo apt update

# 安装必要依赖
sudo apt install apt-transport-https ca-certificates curl software-properties-common

# 添加Docker官方GPG密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 添加Docker仓库
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# 启动Docker
sudo systemctl start docker
sudo systemctl enable docker

# 测试
sudo docker run hello-world
```

**检查安装：**
```bash
docker --version
docker-compose --version
```

### 2.2 第一个Docker容器

来，运行你的第一个容器！

```bash
# 运行一个hello-world镜像
docker run hello-world

# 运行一个Ubuntu镜像
docker run -it ubuntu bash

# 运行并命名
docker run -it --name my-ubuntu ubuntu bash

# 后台运行
docker run -d --name my-nginx nginx

# 端口映射（重要！）
# 格式：-p 主机端口:容器端口
docker run -d -p 8080:80 --name my-nginx nginx
```

参数解释：
- `-i`：交互式运行
- `-t`：分配一个终端
- `-d`：后台运行
- `-p`：端口映射
- `--name`：给容器起名字

### 2.3 常用命令

**查看容器：**
```bash
# 运行中的容器
docker ps

# 所有容器（包括停止的）
docker ps -a

# 只看容器ID
docker ps -q
```

**启动/停止/删除：**
```bash
# 启动容器
docker start my-nginx

# 停止容器
docker stop my-nginx

# 重启容器
docker restart my-nginx

# 删除容器（需要先停止）
docker rm my-nginx

# 强制删除（不管运行中）
docker rm -f my-nginx
```

**进入容器内部：**
```bash
# 进入运行中的容器
docker exec -it my-nginx bash

# 如果容器运行的是sh而不是bash
docker exec -it my-nginx sh
```

**查看日志：**
```bash
# 查看日志
docker logs my-nginx

# 实时查看日志
docker logs -f my-nginx

# 只看最后100行
docker logs --tail 100 my-nginx
```

**查看资源使用：**
```bash
# 查看运行中的容器
docker stats
```

### 2.4 镜像操作

```bash
# 搜索镜像
docker search nginx

# 拉取镜像
docker pull nginx:latest

# 查看本地镜像
docker images

# 删除镜像
docker rmi nginx:latest

# 推送镜像到Docker Hub
docker tag my-image:latest username/my-image:latest
docker push username/my-image:latest
```

---

## 第三章：Dockerfile——构建自己的镜像

### 3.1 什么是Dockerfile？

Dockerfile是一个"配方"文件，用来告诉Docker怎么构建你的镜像。里面写的就是一系列指令。

### 3.2 第一个Dockerfile

假设你要把一个Python项目做成Docker镜像：

```dockerfile
# 基础镜像（基于什么系统/环境）
FROM python:3.9-slim

# 设置工作目录
WORKDIR /app

# 复制文件
COPY requirements.txt .

# 安装依赖
RUN pip install -r requirements.txt

# 复制项目代码
COPY . .

# 暴露端口
EXPOSE 8080

# 启动命令
CMD ["python", "app.py"]
```

指令解释：
- `FROM`：基于哪个镜像
- `WORKDIR`：设置工作目录
- `COPY`：复制文件
- `RUN`：执行命令
- `EXPOSE`：声明端口
- `CMD`：容器启动时执行的命令

### 3.3 构建镜像

```bash
# 在Dockerfile所在目录执行
docker build -t my-python-app .

# 指定Dockerfile路径
docker build -t my-python-app -f /path/to/Dockerfile .

# 带标签
docker build -t my-app:1.0 .
```

### 3.4 一个完整的例子

假设你有一个Spring Boot项目：

```dockerfile
# 基础镜像
FROM maven:3.8-openjdk-8 AS builder

# 设置工作目录
WORKDIR /app

# 先复制pom.xml
COPY pom.xml .
# 下载依赖（利用Docker缓存）
RUN mvn dependency:go-offline

# 复制源码并构建
COPY src ./src
RUN mvn package -DskipTests

# 运行镜像
FROM openjdk:8-jre-slim

WORKDIR /app

# 从builder阶段复制jar包
COPY --from=builder /app/target/*.jar app.jar

# 暴露端口
EXPOSE 8080

# 启动
ENTRYPOINT ["java", "-jar", "app.jar"]
```

这个Dockerfile用了**多阶段构建**技术：
- 第一个阶段用Maven构建项目
- 第二个阶段只复制运行所需的jar包
- 大大减小最终镜像体积！

---

## 第四章：Docker Compose——编排多容器

### 4.1 为什么要用Docker Compose？

一个完整的Web应用通常不止一个容器：
- 前端容器
- 后端容器
- MySQL容器
- Redis容器
- Nginx容器

一个一个启动太麻烦了！Docker Compose就是用来"一键启动"整个应用的。

### 4.2 docker-compose.yml

创建一个`docker-compose.yml`文件：

```yaml
version: '3.8'

services:
  # MySQL数据库
  mysql:
    image: mysql:8.0
    container_name: mma-mysql
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: mma
      MYSQL_USER: mma_user
      MYSQL_PASSWORD: mma123
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - mma-network

  # Redis缓存
  redis:
    image: redis:7-alpine
    container_name: mma-redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - mma-network

  # 后端服务
  backend:
    build: ./mma_back_end
    container_name: mma-backend
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/mma?useSSL=false&serverTimezone=UTC
      SPRING_DATASOURCE_USERNAME: mma_user
      SPRING_DATASOURCE_PASSWORD: mma123
    depends_on:
      - mysql
      - redis
    networks:
      - mma-network

  # 前端
  frontend:
    build: ./mma_web
    container_name: mma-frontend
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - mma-network

  # Nginx反向代理
  nginx:
    image: nginx:latest
    container_name: mma-nginx
    ports:
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - backend
    networks:
      - mma-network

# 定义网络
networks:
  mma-network:
    driver: bridge

# 定义数据卷
volumes:
  mysql-data:
  redis-data:
```

### 4.3 常用命令

```bash
# 启动所有服务
docker-compose up

# 后台启动
docker-compose up -d

# 停止所有服务
docker-compose down

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f

# 只启动某个服务
docker-compose up mysql

# 重新构建镜像
docker-compose build

# 删除镜像
docker-compose down --rmi local
```

---

## 第五章：实战——部署协会项目

### 5.1 协会现有的Docker配置

来看看协会是怎么用Docker的：

**目录结构：**
```
mma_docker/
├── docker-compose.yaml
```

**docker-compose.yaml：**
```yaml
version: '3'

services:
  mysql:
    image: mysql:8.0
    container_name: mma_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: mma
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - ./mma_mysql:/var/lib/mysql
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --default-authentication-plugin=mysql_native_password

  nginx:
    image: nginx:latest
    container_name: mma_nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/ssl:/etc/nginx/ssl
      - ./nginx/log:/var/log/nginx
    depends_on:
      - mysql
```

### 5.2 后端Dockerfile示例

这是协会后端的Dockerfile：

```dockerfile
FROM maven:3.8.6-openjdk-8 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn package -DskipTests

FROM openjdk:8-jre-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 5.3 前端Nginx配置示例

```nginx
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    # 前端静态文件
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API反向代理
    location /api/ {
        proxy_pass http://backend:8080/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 上传文件代理
    location /upload/ {
        proxy_pass http://backend:8080/upload/;
        proxy_set_header Host $host;
        client_max_body_size 100m;
    }
}
```

---

## 第六章：进阶内容

### 6.1 数据管理

```bash
# 查看数据卷
docker volume ls

# 创建数据卷
docker volume create my-data

# 查看数据卷详情
docker volume inspect my-data

# 删除未使用的数据卷
docker volume prune
```

### 6.2 网络管理

```bash
# 查看网络
docker network ls

# 创建网络
docker network create my-network

# 将容器连接到网络
docker network connect my-network container-name

# 查看网络详情
docker network inspect my-network
```

### 6.3 日志和监控

```bash
# 查看所有容器资源
docker stats

# 查看容器详细信息
docker inspect container-name

# 查看镜像历史
docker history image-name
```

### 6.4 清理

```bash
# 清理停止的容器
docker container prune

# 清理 dangling 镜像
docker image prune

# 清理所有未使用的数据
docker system prune

# 完全清理（包括volume）
docker system prune -a --volumes
```

### 6.5 Docker可视化工具

```bash
# Portainer（轻量级可视化工具）
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock --name portainer portainer/portainer-ce
```

安装后访问 http://localhost:9000 就可以用图形界面管理Docker了！

---

## 附录：推荐学习资源

### 官方文档
- Docker官方文档：https://docs.docker.com/
- Docker Hub：https://hub.docker.com/

### 视频教程
- 搜索"Bilibili Docker入门"有很多优质教程

### 实践
- 尝试把协会的项目用Docker部署一遍
- 练习用Docker运行各种服务（MySQL、Redis、Nginx等）

---

## 结语

Docker是现代开发者的必备技能！学会它之后：
- 部署项目变得超级简单
- 再也不怕"在我电脑上能运行"的问题
- 团队协作更高效

建议的学习路径：
1. 先在本地安装Docker
2. 运行几个现成的镜像（nginx、mysql等）
3. 尝试写一个简单的Dockerfile
4. 学习docker-compose
5. 尝试部署协会的项目

动手试试吧！有问题随时问~
