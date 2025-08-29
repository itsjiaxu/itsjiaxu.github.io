---
title: Docker volume
description: Docker volume
publishedDate: 2025-08-24
tags:
  - documentation
  - docker
---

## 简介

容器的生命周期是短暂的，当容器被删除时，其内部文件系统中的数据也会随之丢失。为了实现数据的持久化存储，Docker 提供了三种主要机制：卷（Volume）、绑定挂载（Bind Mount）和内存文件系统挂载（tmpfs Mount）。

## Volume

卷（Volume）是 Docker 用于持久化数据的首选方式。它是由 Docker 创建和管理的、位于宿主机文件系统特定部分（默认为 `/var/lib/docker/volumes/`）的存储区域。

### 特性

- Docker 管理：卷的生命周期独立于任何容器，由 Docker 引擎统一管理。即使容器被删除，卷及其数据也会被保留。
- 平台无关性：卷屏蔽了宿主机的目录结构，使得数据卷在不同操作系统（Linux, Windows, macOS）之间更具可移植性。
- 易于共享：多个容器可以同时挂载并读写同一个卷，方便在容器间共享数据。
- 支持驱动：卷支持使用不同的驱动（Volume Driver），允许你将数据存储在远程主机、云存储或其他存储后端上。

### 卷的类型

1.  命名卷：创建时指定一个明确的名称。这是最常用和推荐的方式，便于管理和识别。
2.  匿名卷：创建时不指定名称，Docker 会为其分配一个唯一的、随机的哈希值作为名称。通常用于临时存储，当容器被删除时，若启动时使用了 `--rm` 选项，匿名卷也会被自动删除。

### 卷的初始化

- 空卷挂载到非空目录：如果将一个新创建的空卷挂载到容器内一个已经存在文件或目录的路径上（例如 `nginx` 容器的 `/usr/share/nginx/html`），Docker 会自动将该目录下的内容复制到卷中。
- 不存在的卷：如果在启动容器时指定了一个尚不存在的命名卷，Docker 会自动为你创建它。

### 常用命令

与绑定挂载不同，卷拥有一套完整的命令行工具进行独立管理。

| 命令                                  | 描述                                                 |
| :------------------------------------ | :--------------------------------------------------- |
| `docker volume create <volume-name>`  | 创建一个命名卷。                                     |
| `docker volume ls`                    | 列出所有卷。                                         |
| `docker volume inspect <volume-name>` | 查看卷的详细信息，包括其在宿主机上的真实路径。       |
| `docker volume rm <volume-name>`      | 删除一个或多个指定的卷。卷正在被容器使用时无法删除。 |
| `docker volume prune`                 | 删除所有未被任何容器使用的卷，释放磁盘空间。         |

---

## Bind Mount

绑定挂载（Bind Mount）是将宿主机上的一个文件或目录直接挂载到容器中的一种方式。它的历史比卷更悠久，功能强大但需谨慎使用。

### 特性

- 直接映射：容器内的路径直接指向宿主机的文件系统路径，对容器内该路径的任何修改都会立刻反映在宿主机上，反之亦然。
- 依赖宿主机：强依赖于宿主机的目录结构，降低了应用的可移植性。
- 安全风险：容器可以通过绑定挂载访问宿主机的文件系统，包括敏感的系统文件，可能带来安全隐患。

如果将宿主机的目录挂载到容器内一个非空目录，宿主机目录会覆盖容器内的原有内容。容器内的文件不会丢失，只是在挂载期间无法访问，卸载后会恢复。这类似于在 Linux 系统中将一个 U 盘挂载到非空的 `/mnt` 目录，原目录内容会被临时隐藏。

---

## Tmpfs 挂载

Tmpfs挂载是一种特殊的挂载方式，它将数据存储在宿主机的内存中，而不是磁盘上。

### 核心特性

- 非持久化：当容器停止时，tmpfs 挂载点的数据会完全丢失。
- 极高性能：由于数据直接在内存中读写，速度非常快。
- 安全性：适用于存储临时性的、不希望持久化的敏感数据，如令牌或会话数据。

---

## 使用场景

Volume vs. Bind Mount vs. Tmpfs

| 特性         | Volume                                   | Bind Mount                               | Tmpfs 挂载                       |
| :----------- | :--------------------------------------- | :--------------------------------------- | :------------------------------- |
| 宿主机位置   | Docker 管理 (`/var/lib/docker/volumes/`) | 用户指定任意位置                         | 宿主机内存                       |
| 持久性       | 持久                                     | 持久                                     | 非持久 (随容器停止而消失)        |
| 生命周期管理 | Docker 命令 (`docker volume ...`)        | 手动管理                                 | 随容器生命周期                   |
| 可移植性     | 高                                       | 低 (依赖宿主机路径)                      | N/A                              |
| 数据共享     | 多个容器可安全共享                       | 多个容器可共享 (需注意并发问题)          | 无法在容器间共享                 |
| 安全性       | 较高 (与宿主机文件系统隔离)              | 较低 (可访问宿主机任意文件)              | 高 (内存中，不写盘)              |
| 性能         | 良好                                     | 极高                                     | 最高                             |
| 使用场景     | 数据库、应用数据、需要在容器间共享的配置 | 开发环境（挂载源码）、共享宿主机配置文件 | 存储临时敏感数据、缓存、会话状态 |

- 生产环境 & 数据持久化：优先选择 Volume。它更安全、易于管理和备份。
- 开发环境：Bind Mount 非常适合。你可以直接在 IDE 中修改代码，容器内的应用会立即热重载，无需重新构建镜像。
- 临时或敏感数据：Tmpfs 挂载，数据不会泄露到磁盘上。

---

## 语法

推荐使用 `--mount` 标志，因为它更明确、易读。`-v` 或 `--volume` 标志是旧语法，虽然简洁，但功能稍弱且易混淆。

### `--mount`

`--mount` 由多个键值对组成，格式为 `type=<type>,src=<source>,dst=<destination>[,options]`。

Volume:

```bash
# 挂载一个命名卷
docker run -d \
  --name my-nginx \
  --mount type=volume,src=nginx-data,dst=/usr/share/nginx/html \
  nginx

# 挂载一个匿名卷 (省略 src)
docker run -d --mount type=volume,dst=/app/data my-app
```

Bind Mount:

```bash
# 将当前目录下的 app 文件夹挂载到容器的 /app
docker run -d --mount type=bind,src="$(pwd)/app",dst=/app my-app
```

Tmpfs：

```bash
docker run -d --mount type=tmpfs,dst=/tmp/cache my-app
```

### `-v`/`--volume`

`-v` 由以冒号 `:` 分隔的字段组成。

Volume:

```bash
# 挂载命名卷
docker run -d -v nginx-data:/usr/share/nginx/html nginx

# 挂载匿名卷
docker run -d -v /app/data my-app
```

Bind Mount:

```bash
# 必须使用绝对路径
docker run -d -v /path/on/host:/path/in/container my-app
```

---

## 数据备份、恢复与迁移

卷使得数据管理变得非常方便。

### 备份

思路：启动一个新容器，挂载需要备份的卷，同时再挂载一个本地目录（用于存放备份文件），然后在容器内执行压缩命令。

```bash
# 假设 my-nginx 容器正在使用名为 nginx-data 的卷
# 将 nginx-data 卷的内容备份到当前目录下的 backup.tar 文件
docker run --rm \
  --mount type=volume,src=nginx-data,dst=/data \
  --mount type=bind,src="$(pwd)",dst=/backup \
  ubuntu \
  tar cvf /backup/backup.tar /data
```

### 恢复

思路与备份相反：启动一个新容器，挂载目标卷和存放备份文件的本地目录，然后在容器内执行解压命令。

```bash
# 创建一个新的目标卷
docker volume create new-nginx-data

# 将备份文件恢复到新卷中
docker run --rm \
  --mount type=volume,src=new-nginx-data,dst=/data \
  --mount type=bind,src="$(pwd)",dst=/backup \
  ubuntu \
  bash -c "cd /data && tar xvf /backup/backup.tar --strip 1"
```

现在，`new-nginx-data` 卷就包含了之前备份的所有数据，可以挂载给新的容器使用。这种方式也常用于在不同 Docker 主机之间迁移数据。
