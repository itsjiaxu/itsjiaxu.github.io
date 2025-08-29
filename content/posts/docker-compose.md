---
title: Docker compose
description: Docker compose
publishedDate: 2025-08-24
tags:
  - documentation
  - docker
---

## 简介

Docker Compose 是一个用于定义和管理多容器应用的工具，通过在项目根目录配置 compose.yaml 来描述应用所需的服务、网络和卷等资源。通过简单的一条命令，用户即可批量构建、启动或停止整个应用环境，极大地简化了容器编排和部署流程。

## 安装

### Debian 系

使用包管理器安装

```bash
sudo apt-get update
sudo apt-get install docker-compose-plugin
docker compose version
```

手动安装

```bash
# 下载 Docker Compose v2
curl -SL https://github.com/docker/compose/releases/download/v2.35.1/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/lib/docker/cli-plugins/docker-compose
# 添加执行权限
chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
# 检查 Docker Compose 版本
docker compose version
```

官方示例如下。Docker 安装在当前用户的 $HOME 目录下，若要全局安装，删掉前两行，把~/.docker/cli-plugins 替换成 / usr/local/lib/docker/cli-plugins；若要切换版本，修改 v2.35.1 以指定安装的版本

```bash
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.35.1/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
```

下载 Docker Compose v1

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

1.29.2 是 v1 系列的最后一个版本，后续不再更新，推荐使用 v2 版本。

## 卸载

### Debian系

```bash
sudo apt-get remove docker-compose-plugin
```

手动卸载

```bash
rm /usr/local/lib/docker/cli-plugins/docker-compose
```

## 命令

查看运行状态

显示当前项目的容器

```
docker compose ps
```

两种 ps 方式的区别

| 功能                   | docker compose ps       | docker ps           |
| ---------------------- | ----------------------- | ------------------- |
| 显示内容范围           | 当前 Compose 项目的容器 | 所有运行中的容器    |
| 是否依赖 Compose 项目  | ✅ 是                   | ❌ 否               |
| 是否需要在项目目录执行 | ✅ 推荐                 | ❌ 不需要           |
| 支持服务名等逻辑视图   | ✅ 显示服务、端口等     | ❌ 显示底层容器属性 |

构建容器

默认构建 compose.yaml 配置的所有内容，也可以在后面加参数指定构建的内容

```bash
docker compose build
```

启动容器

一般加上 - d 使服务在后台运行

```bash
docker compose up -d
```

选项：

- `-d`：后台运行
- `-f`：指定 Compose 文件路径

停止容器

停止所有服务容器，但不会删除容器

```bash
docker compose stop
```

停止并删除容器

停止所有服务，并删除容器、网络、默认卷等资源

```bash
docker compose down
```

重启容器

重启当前 Compose 项目中定义的所有服务容器

```bash
docker compose restart
```

查看日志

显示所有服务的日志，加上 -f 可以查看实时日志，Ctrl + C 退出

```bash
docker compose logs
```

## 构建 YAML 文件

环境变量

> By leveraging environment variables and interpolation in Docker Compose, you can create versatile and reusable configurations, making your Dockerized applications easier to manage and deploy across different environments.

使用环境变量可以动态地配置容器，避免在镜像中存储敏感信息，并有利于集中管理配置。

1. 使用 environment 属性

```bash
services:
  webapp:
    environment:
      DEBUG: "true"
```

2. 使用 env_file 属性

在项目根目录下创建. env 文件，Docker Compose 会自动识别并加载，并将它们应用到 compose.yaml 文件中的服务配置中。在 Compose 文件中可以通过 env_file 显式指定环境变量文件

```bash
services:
  webapp:
    env_file: - ./web.env
```

3. 使用 `docker compose run -e`

这种方法用于临时运行某个服务的容器，常用于执行一次性任务

```bash
docker compose run -e DEBUG=1 web python console.py
```
