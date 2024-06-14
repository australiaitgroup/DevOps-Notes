# Class-05: Dockerization

## 目录

1. [Docker 基础](#docker-基础)
   - [DevOps 循环与课程概述](#devops-循环与课程概述)
   - [Docker 介绍](#docker-介绍)
     - [什么是 Docker](#什么是-docker)
     - [为什么使用虚拟化](#为什么使用虚拟化)
     - [容器与虚拟机的区别](#容器与虚拟机的区别)
       - [关键优势](#关键优势)
     - [为什么使用镜像](#为什么使用镜像)
   - [Docker 架构](#docker-架构)
   - [Docker 实践](#docker-实践)
     - [Docker CLI](#docker-cli)
     - [Docker 镜像](#docker-镜像)
       - [console-helloworld](#console-helloworld)
       - [Dockerfile 示例](#dockerfile-示例)
       - [web-nginx](#web-nginx)
       - [web-python-flask](#web-python-flask)
       - [console-dependency](#console-dependency)
     - [清理](#清理)
     - [Docker Compose](#docker-compose)
     - [Docker Registry](#docker-registry)
     - [镜像共享](#镜像共享)
     - [容器威胁模型](#容器威胁模型)
     - [容器镜像](#容器镜像)
2. [资料阅读](#资料阅读)
   - [更多阅读](#更多阅读)
   - [最佳实践](#最佳实践)
3. [练习](#练习)
   - [Docker CLI 手动练习](#docker-cli-手动练习)
   - [Dockerfile 手动练习](#dockerfile-手动练习)
   - [Docker Compose 手动练习](#docker-compose-手动练习)

## Docker 基础

### DevOps 循环与课程概述

- **编码和测试**
  - 如何配置环境？
  - 按步骤配置
  - 是否可以更好地配置？
- **部署代码到测试环境**
  - 解决“在我这里工作”问题
- **部署代码到生产环境**
  - 预发布和生产环境差异

通过打包代码和所有配置，构建一次，在本地、预发布和生产环境中运行，尽量保持环境一致。

### Docker 介绍

Docker 是一种 PaaS 产品，使用操作系统级别的虚拟化技术，将软件打包成独立的容器，这些容器可以在任何地方运行。

#### 什么是 Docker

- Docker 是一套 PaaS 产品，使用操作系统级别的虚拟化技术，将软件打包成独立的容器。
- 容器是彼此隔离的，包含自己的软件、库和配置文件。

#### 为什么使用虚拟化

- 资源利用：一台电脑上运行多个服务。
- 隔离：多个服务同时使用需要隔离环境，防止资源冲突。
- 资源限制：控制 CPU、内存等资源使用。

Docker 使开发人员能够轻松地打包、传输和运行任何应用程序，将其作为轻量级、可移植、自给自足的容器，可以在几乎任何地方运行。

#### 容器与虚拟机的区别

- 容器和虚拟机都将物理机器隔离成多个虚拟机器。
- 区别在于实现方式和层级：虚拟机虚拟化物理硬件，容器在操作系统上隔离出不同的环境。

#### 关键优势

- **可移植性**：容器可以在不同环境中一致运行。
- **应用中心化**：专注于应用的部署，而不是底层机器的配置。
- **自动构建**：开发人员可以自动化地从源代码构建容器。
- **版本控制**：类似于 Git 的版本控制功能，可以追踪容器的不同版本。
- **组件重用**：任何容器都可以作为基础镜像，创建更多专用组件。
- **共享**：通过 Docker Hub 分享和获取有用的镜像。

#### 为什么使用镜像

镜像包含运行容器所需的所有文件和配置信息。构建一次镜像，可以多次启动容器。

## Docker 架构

- Docker Daemon
- 客户端
  - CLI
  - GUI
- 容器
- 镜像
- 资源
  - 网络
  - 数据卷

## Docker 实践

### Docker CLI

CLI 是与 Docker Daemon 交互的命令行界面，极大简化了容器管理。

#### 常用命令

- 运行 Docker 容器

  ```sh
  docker run -d -p 80:80 --name demo nginxdemos/hello
  ```

  - `-d`：后台运行
  - `-p`：端口映射
  - `--name`：容器命名

- 列出 Docker 容器

  ```sh
  docker ps
  ```

- 列出 Docker 镜像

  ```sh
  docker images
  ```

- 查看容器日志

  ```sh
  docker logs -f demo
  ```

- 检查容器

  ```sh
  docker inspect demo
  ```

- 复制文件到容器

  ```sh
  docker cp your_file demo:/
  ```

- 进入容器并执行命令

  ```sh
  docker exec -it demo sh
  ```

- 停止或杀死容器

  ```sh
  docker stop demo
  docker kill demo
  ```

- 查看所有容器

  ```sh
  docker ps -a
  ```

- 删除容器

  ```sh
  docker rm demo
  ```

- 在不同端口运行相同容器
  ```sh
  docker run -p 8080:80 --name demo8080 nginxdemos/hello
  ```

### Docker 镜像

#### 创建镜像

使用 Dockerfile 创建自定义镜像。

#### console-helloworld

参考链接: [console-helloworld](https://github.com/JiangRenDevOps/DevOpsNotes/tree/master/WK3_Dockerisation/dockerfile/1.console-helloworld)

```sh
./run.sh
```

**分步操作**：

1. 构建镜像

   ```sh
   cd 1.console-helloworld
   docker build -t jr/console-helloworld .
   docker images
   ```

2. 运行镜像
   ```sh
   docker run --rm jr/console-helloworld
   ```

**Dockerfile**：

```Dockerfile
FROM ubuntu
ENTRYPOINT ["/bin/bash"]
CMD ["-c", "echo Hello world from command line."]
```

**run.sh**：

```sh
#!/bin/bash
set -e

cd "$(dirname "$0")"

echo "Building image ..."
docker build -t jr/console-helloworld .

echo
echo "Running container ..."
docker run --rm jr/console-helloworld
```

#### Dockerfile 示例

```Dockerfile
FROM python:3
WORKDIR /usr/src/app
COPY . .
RUN pip install --no-cache-dir -r ./src/requirements.txt
EXPOSE 5000
CMD ["python", "./src/app.py"]
```

#### web-nginx

参考链接: [web-nginx](https://github.com/JiangRenDevOps/DevOpsNotes/tree/master/WK3_Dockerisation/dockerfile/2.web-nginx)

```sh
cd 2.web-nginx
docker build -t jr/web-helloworld .
docker images
```

**Dockerfile**：

```Dockerfile
FROM nginx
COPY ./index.html /usr/share/nginx/html
```

#### web-python-flask

参考链接: [web-python-flask](https://github.com/JiangRenDevOps/DevOpsNotes/tree/master/WK3_Dockerisation/dockerfile/3.web-python-flask)

```sh
cd 3.web-python-flask
docker build -t jr/web-flask .
docker images
```

**Dockerfile**：

```Dockerfile
FROM tiangolo/uwsgi-nginx-flask:python3.7
RUN pip install requests
COPY ./app /app
```

#### console-dependency

参考链接: [console-dependency](https://github.com/JiangRenDevOps/DevOpsNotes/tree/master/WK3_Dockerisation/dockerfile/4.console-dependency)

**Dockerfile**：

```Dockerfile
FROM ubuntu
RUN apt-get update && apt-get install -y curl && apt-get autoclean
COPY app/main.sh .
ENTRYPOINT ["/bin/bash"]
CMD ["main.sh"]
```

**main.sh**：

```sh
#!/bin/bash
echo "Press Ctrl+C any time to stop the application."
echo
for i in $(seq 2); do
    for key in sy au ing ba lu pa ji san; do
        echo "Searching all cities that contains '${key}' keyword ..."
        curl http://citymatcher?city=${key} 2>/dev

/null
        echo
        echo
        sleep 3
    done
done
```

**构建与运行**：

```sh
cd 4.console-dependency
docker build -t jr/console-hello .
docker images
docker run --link webflask --rm jr/console-hello
```

### 清理

```sh
docker kill $(docker ps -q)
```

### Docker Compose

使用 Docker Compose 同时启动多个容器。

参考链接: [docker-compose](https://github.com/JiangRenDevOps/DevOpsNotes/tree/master/WK3_Dockerisation/docker-compose)

**docker-compose.yml**：

```yaml
version: "3"
services:
  console-helloworld:
    image: jr/console-helloworld
    build: ../dockerfile/1.console-helloworld
  web-nginx:
    image: jr/web-helloworld
    build: ../dockerfile/2.web-nginx
    ports:
      - "8080:80"
  web-python-flask:
    image: jr/web-citymatcher
    build: ../dockerfile/3.web-python-flask
    ports:
      - "80:80"
  console-dependency:
    image: jr/console-citymatcher
    build: ../dockerfile/4.console-dependency
    links:
      - web-python-flask:citymatcher
    depends_on:
      - web-python-flask
```

**启动 Compose**：

```sh
docker-compose up
```

### Docker Registry

Docker Hub 账户上可以推送自己的镜像。

参考链接: [docker-registry](https://github.com/JiangRenDevOps/DevOpsNotes/blob/master/WK3_Dockerisation/4.docker-registry.md)

### 镜像共享

```sh
docker pull myimage:1.0
docker tag myimage:1.0 myrepo/myimage:2.0
docker push myrepo/myimage:2.0
```

### 容器威胁模型

- 不安全的主机
- 含有漏洞的镜像
- 镜像中暴露秘密
- 容器逃逸漏洞

### 容器镜像

- 官方 Docker 镜像
- 使用工具扫描已知漏洞（如 Docker Scout）
  ```sh
  docker scout quickview jr/web-flask:latest
  ```

## 资料阅读

### 更多阅读

- [Docker 官方文档](https://docs.docker.com/engine/reference/builder/)
- [Docker Compose 文件](https://docs.docker.com/compose/compose-file/compose-file-v3/)
- [Docker Compose FAQ](https://docs.docker.com/compose/faq/)
- [Dockerfile 最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Dockerfile 教程](https://takacsmark.com/dockerfile-tutorial-by-example-dockerfile-best-practices-2018/#overview)
- [CI/CD 与 GitHub Actions](https://docs.docker.com/ci-cd/github-actions/)

### 最佳实践

- [Dockerfile 最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Dockerfile 教程](https://takacsmark.com/dockerfile-tutorial-by-example-dockerfile-best-practices-2018/#overview)
- [CI/CD 与 GitHub Actions](https://docs.docker.com/ci-cd/github-actions/)

## 练习

### Docker CLI 手动练习

通过实际操作练习不同的 Docker 命令，熟悉 Docker CLI 的使用。

#### 练习 1：了解 Docker 镜像和容器

1. 确保 Docker 已启动。
2. 熟练使用 Linux 命令，如 `ls`、`cd`、`copy`。
3. 掌握以下命令：
   - `build`
   - `run`（可加 `rm` 参数）
   - `stop`

#### Dockerfile 示例

```Dockerfile
FROM python:3
WORKDIR /usr/src/app
COPY . .
RUN pip install --no-cache-dir -r ./src/requirements.txt
EXPOSE 5000
CMD ["python", "./src/app.py"]
```

### Dockerfile 手动练习

通过编写和执行 Dockerfile，理解其工作原理。

#### 练习 1：Hello World

1. 编写简单的 Dockerfile，并使用 `docker build` 构建镜像。
2. 使用 `docker run` 运行容器。

```Dockerfile
FROM ubuntu
ENTRYPOINT ["/bin/bash"]
CMD ["-c", "echo Hello world from command line."]
```

### Docker Compose 手动练习

使用 Docker Compose 配置多容器应用。

#### 示例配置

```yaml
version: "3"
services:
  console-helloworld:
    image: jr/console-helloworld
    build: ../dockerfile/1.console-helloworld
  web-nginx:
    image: jr/web-helloworld
    build: ../dockerfile/2.web-nginx
    ports:
      - "8080:80"
  web-python-flask:
    image: jr/web-citymatcher
    build: ../dockerfile/3.web-python-flask
    ports:
      - "80:80"
  console-dependency:
    image: jr/console-citymatcher
    build: ../dockerfile/4.console-dependency
    links:
      - web-python-flask:citymatcher
    depends_on:
      - web-python-flask
```

启动 Compose：

```bash
docker-compose up
```
