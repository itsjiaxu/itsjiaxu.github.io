---
title: Docker image
description: Docker image
publishedDate: 2025-08-15
tags:
  - documentation
  - docker
---

## 概述

Docker 镜像是一种只读的模板，用于创建 Docker 容器。
它包含了运行应用所需的所有内容：

- 操作系统文件（如 Ubuntu、Alpine 等）
- 应用程序及其依赖库
- 配置文件
- 启动命令

可以理解为：

镜像 = 环境快照 + 应用程序

容器 = 镜像的运行实例

Docker 镜像由多个层组成。每个层都是 Dockerfile 中构建指令的结果。层按顺序堆叠，每个层都是一个增量，表示应用于前一层的更改。

---

## 特点

1. 分层结构（Layered Architecture
   - Docker 镜像由多层文件系统组成，每一层都是只读的。
   - 通过 ** 联合文件系统（UnionFS）** 叠加在一起，形成一个完整的镜像。
   - 优势：镜像层可复用，减少存储空间和下载时间。

2. 不可变性（Immutable
   - 镜像本身不会被修改，一旦创建即固定。
   - 如果需要修改，需基于原镜像构建新镜像。

3. 可移植性（Portable
   - 镜像可以在任何安装了 Docker 的系统上运行，不受底层环境影响。

---

## 操作

### 拉取镜像

```bash
docker pull [OPTIONS] [registry address[:Port]/]repository[:Tag]
```

- **Registry Address**：地址的格式一般是 <域名 / IP>[: 端口号]，默认仓库是 Docker Hub。
- **Repository**：仓库名是两段式名称，格式为 <用户名>/< 软件名 >。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。
- **Tag**：软件版本，默认为 `latest`。

### 推送镜像

```bash
docker login
docker push [OPTIONS] NAME[:TAG]
```

用途：将本地镜像上传到远程镜像仓库。

注意：如果想把本地的镜像推送到 Docker Hub，需要先给它打上你的 Docker Hub 用户名标签。

### 保存镜像

```bash
docker image save [OPTIONS] IMAGE
```

使用 `-o` 指令指定写入的文件，也可以使用重定向输出的方式保存镜像：`docker save hello-world > helloworld`

**警告**：保存镜像会覆盖同名文件！

使用 gzip 压缩：`docker save alpine | gzip > alpine-latest.tar.gz`

### 加载镜像

```bash
docker load [OPTIONS]
```

使用 `-i` 指令读取加载的文件，也可以使用重定向输入的方式加载镜像：`docker save hello-world < helloworld`

### 查看镜像

```bash
docker images
```

### 删除镜像

```bash
docker rmi <image id>:<tag>
```

### 标记镜像

一般用于推送到远程仓库前标记地址和用户名

```bash
docker tag my-image:1.0 myrepo/my-image:1.0
```

## 镜像构建

Docker build 实现了 client-server 架构，调用构建时，Buildx 客户端会向 BuildKit 后端发送构建请求。BuildKit 解析构建指令并执行构建步骤。构建输出会被发送回客户端或上传到 Docker Hub 等仓库。

Client 由 Buildx 提供，作为用户命令行接口，负责接收用户输入的构建指令、配置参数，并将构建请求发送至构建引擎。Server 由 BuildKit 提供，作为实际的构建引擎，负责解析构建指令、执行构建步骤、管理缓存，并最终生成镜像。

### Buildx

Buildx 是 build 的 CLI，`docker build` 命令是其封装。调用 build 时，buildx 会接收用户输入的构建指令、配置参数，并将构建请求发送至 BuildKit。Buildx 还可以创建和管理 BuildKit、管理镜像仓库、并行构建。

`docker build` 和 `docker buildx build` 的区别

| 特性                  | `docker build`                                   | `docker buildx build`                              |
| --------------------- | ------------------------------------------------ | -------------------------------------------------- |
| ** 底层调用 **        | 调用默认 Buildx Builder（随 Docker Engine 提供） | 调用 Buildx，使用当前设定的默认 Builder            |
| **Builder 选择 **     | 总是使用默认 Builder（为旧版 CLI 保持兼容）      | 检查并使用用户设置的 Builder，可为本地或远程       |
| ** 多平台构建 **      | 仅限本地平台                                     | 支持 `--platform` 多平台构建                       |
| ** 缓存导入 / 导出 ** | 基础功能有限                                     | 支持 `--cache-from` / `--cache-to`                 |
| ** 并行构建 **        | 单一 Builder                                     | 多 Builder 并行                                    |
| ** 镜像输出 **        | 默认导入本地缓存                                 | 支持推送仓库（`--push`）、导出文件（`--output`）等 |
| ** 适用场景 **        | 简单构建，兼容旧 CLI                             | 高级构建功能、跨平台、多节点                       |

### BuildKit

BuildKit 是执行构建任务的守护进程。BuildKit 解析构建指令并执行构建步骤。在 BuildKit 执行构建时，Buildx 会监控构建状态并将进度打印到终端。如果构建需要来自客户端的资源，BuildKit 会向 Buildx 请求所需的资源。也就是说，BuildKit 不直接访问本地系统，而是按需向 Buildx 申请资源，这种方式比旧的 Builder 更高效，旧的 Builder 会直接拷贝整个本地文件系统。

### Dockerfile

`Dockerfile` 是一个文本文件，定义了构建镜像的步骤。文件默认名就是 Dockerfile ，使用默认名可以在 `docker build` 时不加参数。

常用指令
| 指令 | 描述 |
| --- | --- |
|`FROM <image>`| 定义了基础镜像 |
|`RUN <command>`| 镜像构建阶段执行命令 |
|`WORKDIR <directory>`| 为指令设置工作目录 |
|`COPY <src> <dest>`| 将构建上下文中的文件复制到容器指定路径中 |
|`CMD <command>`| 指定容器启动时默认执行的命令 |

示例：

```dockerfile
FROM node:18-alpine       # 基础镜像
WORKDIR /app              # 设置工作目录
COPY package*.json ./     # 复制依赖文件
RUN npm install           # 安装依赖
COPY . .                  # 复制项目文件
EXPOSE 3000               # 暴露端口
CMD ["npm", "start"]      # 容器启动命令
```

详细 dockerfile 用法见 [下篇文章](./docker-04-dockerfile.md)。

构建命令：

```bash
docker build -t myapp:1.0 .
```

当你运行 docker build . 命令时，这个点代表当前目录。Docker 会将这个目录下的所有文件和子目录作为构建上下文发送给 Docker 守护进程。

常用标签

- `-t`：给构建的镜像添加 tag，默认使用 latest
- `-f`：指定 dockerfile 的路径，当的 dockerfile 不叫 dockerfile 或不在当前目录时使用。
- `--no-cache`：不使用缓存构建镜像，每一步都强制重新构建。
- `pull`：总是尝试拉取最新的基础镜像。

### 4.2 从容器创建镜像

当你对一个正在运行的容器做了修改，比如安装了新的软件、更新了配置文件或者添加了新的文件，你可以把这些改动保存成一个新的镜像。这个过程就像是给容器拍了一张快照，然后用这张快照创建一个新的模板。

语法：`docker commit [OPTIONS] <container> [<repository>[:<tag>]]`

常用选项

- `-a`, `--author`：指定镜像的作者信息。
- `-m`, `--message`：为新镜像添加提交信息，解释你做了哪些改动。
- **`-p`, `--pause`**：在提交过程中暂停容器。这是默认行为。这样可以确保容器的状态在保存过程中是静态的，避免数据不一致。如果你确定容器中的应用状态不会被改变，也可以使用 `--no-pause`。

在大多数场景中，不推荐使用它来创建镜像。主要原因有：

- 不透明性：`docker commit` 创建的镜像缺乏透明度。你无法从镜像中轻松地看出它是如何构建出来的，因为所有的操作都只是一个快照，没有一个可重复的构建脚本。
- 不可重复性：想重建这个镜像，或者别人想基于此镜像进行修改，他们无法知道你具体做了哪些步骤。这违背了环境一致的核心理念。

`docker commit` 通常只用于快速测试或者临时保存一些改动，而在正式的开发和部署流程中，应该优先使用 `Dockerfile`。

## 5. 镜像优化技巧

1. 选择轻量镜像：优先使用 `alpine`、`node:alpine` 等轻量的镜像。
2. 避免不必要的依赖和文件：使用 `.dockerignore` 文件排除不需要的文件，`COPY` 指令尽可能精确。
3. 减少层数：合并多个 `RUN` 指令，减少镜像层。
4. 利用缓存：将变化频率高的文件放到 Dockerfile 的后面，避免频繁重建。
5. 多阶段构建：构建阶段使用完整环境，最终镜像仅保留运行所需文件。
