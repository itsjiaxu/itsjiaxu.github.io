---
title: Docker install
description: Installing docker on multiple platforms
publishedDate: 2025-08-14
tags:
  - documentation
  - docker
---

## Docker 安装

本篇将介绍如何在不同操作系统上安装和配置 Docker。

### Ubuntu

**注意：此脚本需要以 root 权限运行**

使用方法

```bash
sudo bash install-docker.sh
```

复制以下脚本并运行

```bash
#!/bin/bash

# --- Ubuntu Docker 安装脚本 ---
# 注意：此脚本需要以 root 权限运行：sudo bash install-docker.sh

# 检查是否以 root 权限运行
if [[ $EUID -ne 0 ]]; then
   echo "此脚本需要以 root 权限运行"
   echo "请使用: sudo bash $0"
   exit 1
fi

# 1. 卸载冲突的旧版本

echo "正在卸载可能冲突的旧版 Docker..."
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  apt-get remove -y $pkg
done

# 2. 添加必要的工具和 Docker GPG 密钥

echo "正在添加安装 Docker 所需的工具和密钥..."
apt-get update
apt-get install -y ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc

# 3. 添加 Docker 官方源

echo "正在添加 Docker 官方软件源..."
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update

# 4. 安装 Docker

echo "正在安装 Docker Engine、CLI 和相关插件..."
apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 5. 配置镜像源

echo "正在配置 Docker 镜像加速器..."
mkdir -p /etc/docker
tee /etc/docker/daemon.json > /dev/null <<EOF
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
EOF

# 6. 重启 Docker 服务

echo "正在重启 Docker 服务..."
systemctl restart docker

# 7. 将当前用户添加到 docker 组

echo "正在将指定用户添加到 docker 组..."
if [[ -n "$SUDO_USER" ]]; then
    usermod -aG docker $SUDO_USER
    echo "用户 '$SUDO_USER' 已添加到 docker 组"
else
    echo "请手动将用户添加到 docker 组: usermod -aG docker <username>"
fi
echo "操作完成，请退出当前终端并重新登录，以使组权限生效。"
```

### OpenCloudOS

```bash
yum install docker -y
```

安装完成后，配置镜像源。路径可能不存在，需要手动创建文件夹

```bash
vim /etc/docker/daemon.json
```

添加镜像源

```json
{
  "registry-mirrors": ["https://mirror.ccs.tencentyun.com"]
}
```

重启Docker

```bash
systemctl restart docker
```

检查是否安装成功

```bash
docker info
```

运行测试镜像，检查Docker是否可用

```bash
docker run hello-world
```

### Windows & macOS

下载[Docker Desktop](https://www.docker.com/products/docker-desktop/) 后安装

## 测试

```shell
sudo docker run hello-world
```
