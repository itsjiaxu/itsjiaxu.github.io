---
title: Docker笔记
description: Docker tutorial
publishedDate: 2025-06-30
tags:
  - documentation
---

## Docker 简介

Docker 是一个开源的容器化平台，用于将应用程序及其依赖项打包在一个轻量级、可移植的容器中，以实现跨环境的一致运行。相比传统虚拟机，Docker 容器更加轻量、启动更快，占用资源更少。 开发人员可以通过简单的配置文件定义应用的构建方式，并使用命令快速创建、管理和运行容器。

### Docker vs 虚拟机

###

## Docker 安装

### Ubuntu

卸载冲突的包

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

添加必要工具，设置 GPG key 存放目录并下载 Docker GPG 公钥

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

添加 Docker 官方源

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

安装 Docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

添加镜像源

```bash
sudo nano /etc/docker/daemon.json
```

```bash
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "https://mirror.ccs.tencentyun.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.m.daocloud.io",
    "https://docker.imgdb.de",
    "https://docker-0.unsee.tech",
    "https://docker.hlmirror.com",
    "https://docker.1ms.run",
    "https://func.ink",
    "https://lispy.org",
    "https://docker.xiaogenban1993.com"
  ]
}
```

重启 Docker，检查是否能拉到镜像

```bash
sudo systemctl restart docker
sudo docker run hello-world
```

将当前用户添加到 docker 组中，避免每次都要加 `sudo`

```bash
sudo usermod -aG docker $USER
```

### OpenCloudOS

```bash
sudo yum install docker -y
```

安装完成后，配置镜像源。路径可能不存在，需要手动创建文件夹

```bash
vim /etc/docker/daemon.json
```

添加镜像源

```bash
{
   "registry-mirrors": [
   "https://mirror.ccs.tencentyun.com"
  ]
}
```

重启Docker

```bash
sudo systemctl restart docker
```

检查是否安装成功

```bash
sudo docker info
```

运行测试镜像，检查Docker是否可用

```bash
docker run hello-world
```

### Windows & macOS

下载[Docker Desktop](https://www.docker.com/products/docker-desktop/) 后安装

## Docker 使用

### Volume

如果将一个空的卷挂载到容器中某个已经存在文件或目录的目录上，这些文件或目录会被默认复制到该卷中（挂载传播）。同样地，如果启动一个容器并指定了一个尚不存在的卷，系统会为你自动创建一个空卷，并将对应目录中的文件或目录默认复制到该卷中**。**

#### 创建和删除

与 bind mount 不同，容器可以在没有任何容器的情况下被创建和管理。

| 类型           | 是否必须配合容器使用 | 是否可以独立管理 | 常用命令支持         |
| -------------- | -------------------- | ---------------- | -------------------- |
| **Volume**     | ❌ 不需要容器        | ✅ 可以独立操作  | `docker volume` 命令 |
| **Bind Mount** | ✅ 必须配合容器使用  | ❌ 不可单独管理  | 没有专门命令         |

创建容器

```bash
docker volume create volume-name
```

列出容器

```bash
docker volume ls
```

查看容器信息

```bash
docker volume inspect volume-name
```

容器被设计用来持久化数据的，其生命周期独立于容器，容器删除后需要手动删除容器。使用`--rm`选项，容器被删除时，docker 引擎会自动删除其挂载的匿名卷，具名卷会被保留

```bash
docker volume rm volume-name
docker run --rm -v /foo -v awesome:/bar busybox top
```

删除所有未使用的卷

```bash
docker volume purne
```

#### 挂载

推荐使用 `--mount`，语义明确

```bash
docker run --mount type=volume,src=<volume-name>,dst=<mount-path>
docker run --volume <volume-name>:<mount-path>
```

##### --mount

由多个键值对组成，格式：`<key>=<value>`。键值对的顺序不重要。

```bash
docker run --mount type=volume[,src=<volume-name>],dst=<mount-path>[,<key>=<value>...]
```

以下是当 `--mount type=volume`时的选项

| 选项                           | 描述                                 |
| ------------------------------ | ------------------------------------ |
| `source`, `src`                | 挂载源，对应卷名。匿名卷省略该选项。 |
| `destination`, `dst`, `target` | 容器中的挂载路径。                   |
| `volume-subpath`               | 挂载到容器卷中子目录的路径。         |
| `readonly`, `ro`               | 以只读形式挂载到容器中               |

例子

```bash
docker run --mount type=volume,src=myvolume,dst=/data,ro,volume-subpath=/foo
```

##### --volume/-v

-v 由三个字段组成，以 `:`分隔，且顺序不能颠倒。匿名卷省略第一个选项，且同样支持 `ro`。

```bash
docker run -v [<volume-name>:]<mount-path>[:opts]
```

#### 备份和恢复

备份：运行一个一次性容器，挂载容器卷，将当前主机目录挂载到容器内部 `backup`目录中， 执行 `tar`命令，将备份文件保存到 `backup`（即当前主机目录） 下。

```bash
docker run --rm \
  --volumes-from container-name \
  -v $(pwd):/backup \
  ubuntu \
  tar cvf /backup/backup.tar /data/path
```

恢复：创建一个一次性容器，挂载要恢复数据的容器，将当前主机目录挂载为容器中的 `/backup`，解压文件到目标目录

```bash
docker run --rm \
  --volumes-from container-name \
  -v $(pwd):/backup \
  ubuntu \
  bash -c "cd /target/path && tar xvf /backup/backup.tar --strip 1"
```

例如，创建一个 nginx 容器

```bash
docker run -d \
  --name my-nginx \
  -p 8080:80 \
  -v nginx-html:/usr/share/nginx/html \
  nginx
```

备份数据卷内容

```bash
docker run --rm \
  --volumes-from my-nginx \
  -v $(pwd):/backup \
  ubuntu \
  tar cvf /backup/nginx-backup.tar /usr/share/nginx/html
```

创建一个新的 nginx 容器，恢复上面持久化的数据

```bash
docker run --rm \
  --volumes-from my-nginx \
  -v $(pwd):/backup \
  ubuntu \
  bash -c "cd /usr/share/nginx/html && tar xvf /backup/nginx-backup.tar --strip 1"
```

备份和恢复都是针对某一容器的卷操作的，备份相当于给卷拍快照，恢复相当于回滚。

#### Bind Mount

### Image

### Container

#### 重启策略

Docker 提供重启策略来控制容器退出时是否自动启动，或者 Docker 重启时是否自动启动。官方建议使用 `--restart`控制容器启动，避免使用进程管理器启动容器

| 标志                                                                                 | 描述                                                                                                                     |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| `no`                                                                                 | 不自动启动容器（默认）                                                                                                   |
| `<font style="background-color:rgb(231, 234, 239);">on-failure[:max-retries]</font>` | 只在容器因错误而非正常退出（退出码 ≠0 ）时新启容器。可选参数 `max-retries` 控制最大重启次数，**超过后就不再重启**        |
| `<font style="background-color:rgb(231, 234, 239);">always</font>`                   | 容器无论是否正常退出都会自动重启，如果手动停止，则仅当 Docker 守护程序重新启动或容器本身手动重新启动时才会重新启动       |
| `<font style="background-color:rgb(231, 234, 239);">unless-stopped</font>`           | 与 `always` 类似， 容器退出后自动重启 。不同之处在于，当**手动**停止容器时，即使 Docker 守护程序或主机重启后它也不会重启 |

对于启动时未设置重启策略的容器，可以使用 `update`命令修改

```bash
docker update --restart on-failure:3 container-name
```

为所有启动容器设置重启策略

```bash
docker update --restart unless-stopped $(docker ps -q)
```

#### 一个容器一个服务

### Registry

## Dockerfile

## Docker Compose

### 简介

Docker Compose 是一个用于定义和管理多容器应用的工具，通过在项目根目录配置`compose.yaml`来描述应用所需的服务、网络和卷等资源。通过简单的一条命令，用户即可批量构建、启动或停止整个应用环境，极大地简化了容器编排和部署流程。

### 安装

#### Ubuntu

```bash
sudo apt-get update
sudo apt-get install docker-compose-plugin
docker compose version
```

#### 手动安装

下载Docker Compose v2

```bash
curl -SL https://github.com/docker/compose/releases/download/v2.35.1/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/lib/docker/cli-plugins/docker-compose
```

添加执行权限

```bash
chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
```

检查Docker Compose版本

```bash
docker compose version
```

官方示例如下。Docker安装在当前用户的`$HOME`目录下，若要全局安装，删掉前两行，把`~/.docker/cli-plugins`替换成`/usr/local/lib/docker/cli-plugins`；若要切换版本，修改`v2.35.1`以指定安装的版本

```bash
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.35.1/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
```

下载Docker Compose v1

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

1.29.2是v1系列的最后一个版本，后续不再更新，推荐使用v2版本

### 卸载

#### Ubuntu

```bash
sudo apt-get remove docker-compose-plugin
```

#### 手动卸载

```bash
rm /usr/local/lib/docker/cli-plugins/docker-compose
```

### 命令

#### 查看运行状态

显示当前项目的容器

```bash
docker compose ps
```

##### 两种`ps`方式的区别

| 功能                   | `docker compose ps`     | `docker ps`         |
| ---------------------- | ----------------------- | ------------------- |
| 显示内容范围           | 当前 Compose 项目的容器 | 所有运行中的容器    |
| 是否依赖 Compose 项目  | ✅ 是                   | ❌ 否               |
| 是否需要在项目目录执行 | ✅ 推荐                 | ❌ 不需要           |
| 支持服务名等逻辑视图   | ✅ 显示服务、端口等     | ❌ 显示底层容器属性 |

#### 构建容器

默认构建`compose.yaml`配置的所有内容，也可以在后面加参数指定构建的内容

```bash
docker compose build
```

#### 启动容器

一般加上`-d`使服务在后台运行

```bash
docker compose up -d
```

选项：

- `-d`：后台运行
- `-f`：指定Compose文件路径

#### 停止容器

停止所有服务容器，但不会删除容器

```bash
docker compose stop
```

#### 停止并删除容器

停止所有服务，并**删除容器、网络、默认卷**等资源

```bash
docker compose down
```

#### 重启容器

重启当前 Compose 项目中定义的所有服务容器

```bash
docker compose restart
```

#### 查看日志

显示所有服务的日志，加入`-f`可以查看实时日志，Ctrl + C 退出

```bash
docker compose logs
```

### 构建YAML文件

#### 环境变量

> <font style="color:rgb(0, 0, 0);">By leveraging environment variables and interpolation in Docker Compose, you can create versatile and reusable configurations, making your Dockerized applications easier to manage and deploy across different environments.</font>

<font style="color:rgb(0, 0, 0);">使用环境变量可以动态地配置容器，避免在镜像中存储敏感信息，并有利于集中管理配置。</font>

1. 使用`environment`属性

```bash
services:
  webapp:
    environment:
      DEBUG: "true"
```

2. 使用`env_file`属性

在项目根目录下创建`.env`文件，Docker Compose 会自动识别并加载，并将它们应用到`compose.yaml`文件中的服务配置中。在Compose 文件中可以通过`env_file`显式指定环境变量文件

```bash
services:
  webapp:
    env_file:
      - ./web.env
```

3. 使用`docker compose run -e`

这种方法用于临时运行某个服务的容器，常用于执行一次性任务

```bash
docker compose run -e DEBUG=1 web python console.py
```

## 工具

docker run -> docker compose：[https://www.composerize.com/](https://www.composerize.com/)

<font style="color:rgb(0, 0, 0);"></font>
