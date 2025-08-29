---
title: Docker Registry
description: Docker 仓库管理和镜像分发
publishedDate: 2025-08-27
tags:
  - documentation
  - docker
---

## Docker Registry 简介

Docker Registry 是一个用于存储、管理和分发 Docker 镜像的服务端应用程序。它是 Docker 生态系统中的核心组件，为 Docker 镜像提供了集中化的存储和分发机制。

### 核心功能

Docker Registry 主要提供以下功能：

- 镜像存储：提供安全、可靠的镜像存储服务
- 镜像分发：支持镜像的上传（push）和下载（pull）操作
- 版本管理：通过标签（Tag）系统管理镜像的不同版本
- 访问控制：提供身份验证和授权机制
- API 接口：基于 RESTful API 的镜像管理接口

### 技术架构

Registry 采用分层存储架构：

```
Registry Server
├── Storage Backend (存储后端)
│   ├── 本地文件系统
│   ├── 云存储 (S3, Azure, GCS)
│   └── 分布式存储
├── HTTP API Layer (API 层)
│   ├── Registry API v2
│   └── 身份验证中间件
└── Web UI (可选)
    └── 管理界面
```

### Registry 类型

#### 1. 公共 Registry

- Docker Hub：官方公共注册中心，包含官方和社区镜像
- 其他公共服务：如 Quay.io、Google Container Registry 等

#### 2. 私有 Registry

- 企业内部部署：满足安全性和合规性要求
- 云服务商提供：如 AWS ECR、Azure ACR、阿里云 ACR
- 自建 Registry：使用开源方案如 Harbor、Distribution

### 应用场景

- 持续集成 / 持续部署 (CI/CD)：作为构建流水线的镜像仓库
- 微服务架构：集中管理服务镜像的不同版本
- 容器化应用分发：向不同环境分发应用镜像
- 镜像安全扫描：集成安全扫描和漏洞检测
- 多环境管理：隔离开发、测试、生产环境的镜像

## Registry、Repository 和 Image 关系

为了准确理解 Docker 镜像管理体系，需要明确三个核心概念的层次关系：

### 概念层次结构

```
Registry (注册中心)
├── Repository A (仓库)
│   ├── Image:tag1 (镜像版本 1)
│   ├── Image:tag2 (镜像版本 2)
│   └── Image:latest (最新版本)
├── Repository B
│   ├── Image:v1.0
│   └── Image:v2.0
└── Repository C
    └── Image:stable
```

### 详细定义

#### Registry (注册中心)

Registry 是最高层级的服务实体，负责：

- 管理多个镜像仓库
- 提供统一的访问入口
- 处理身份验证和授权
- 维护镜像的元数据信息

示例：

- `docker.io` (Docker Hub)
- `registry.example.com` (企业私有 Registry)

#### Repository (仓库)

Repository 是存储同一应用不同版本镜像的集合：

- 按应用或服务划分
- 包含多个带标签的镜像版本
- 支持版本历史管理

命名格式：`[registry_host]/[namespace]/repository_name`

示例：

- `docker.io/library/nginx` (官方 nginx 仓库)
- `registry.example.com/myapp/backend` (私有应用仓库)

#### Image (镜像)

Image 是仓库中的具体镜像实例：

- 通过 Tag 进行版本标识
- 包含完整的应用运行环境
- 具有唯一的 SHA256 摘要值

完整标识格式：`registry_host/namespace/repository_name:tag`

示例：

- `nginx:1.24.0`
- `ubuntu:22.04`
- `registry.example.com/myapp/backend:v1.2.3`

### 默认值

当省略部分信息时，Docker 会使用默认值：

- 默认 Registry: `docker.io`
- 默认 Namespace: `library` (官方镜像)
- 默认 Tag: `latest`

因此 `docker pull ubuntu` 等价于 `docker pull docker.io/library/ubuntu:latest`

## Docker Registry 的作用

### 1. 集中化镜像管理

Docker Registry 为组织提供了统一的镜像管理平台：

- 统一存储：所有 Docker 镜像存储在中央位置
- 标准化命名：遵循统一的镜像命名规范
- 版本控制：通过标签系统管理镜像版本历史
- 元数据管理：存储镜像的创建时间、大小、层信息等

### 2. 高效镜像分发

Registry 优化了镜像的分发过程：

- 分层传输：只传输变更的镜像层，减少网络开销
- 并行下载：支持多层并行下载，提升传输效率
- 断点续传：网络中断后可从断点继续传输
- 缓存机制：本地缓存减少重复下载

### 3. 安全和权限控制

Registry 提供完善的安全机制：

- 身份验证：支持多种认证方式（基本认证、Token、LDAP 等）
- 访问控制：基于用户和团队的细粒度权限管理
- 镜像签名：支持 Docker Content Trust 确保镜像完整性
- 漏洞扫描：集成安全扫描工具检测镜像漏洞

### 4. 持续集成 / 持续部署支持

Registry 是 CI/CD 流水线的重要组成部分：

- 自动化构建：CI 系统自动构建并推送镜像
- 环境隔离：不同环境使用不同标签的镜像
- 回滚支持：快速回滚到历史版本
- 部署追踪：记录镜像的部署历史

### 5. 多环境镜像管理

Registry 支持复杂的多环境部署场景：

```bash
# 开发环境
myapp:dev-20250827-a1b2c3d

# 测试环境
myapp:test-v2.1.0-rc1

# 生产环境
myapp:prod-v2.0.5
```

### 6. 团队协作

Registry 促进团队间的协作：

- 镜像共享：团队成员可以共享自定义镜像
- 标准化环境：确保团队使用一致的运行环境
- 知识沉淀：通过镜像文档化应用的部署需求
- 权限管理：不同团队拥有不同的访问权限

## 公共 Registry 和私有 Registry

### 公共 Registry

#### Docker Hub

Docker Hub 是最广泛使用的公共 Docker Registry：

特点：

- 官方镜像：提供经过验证的官方镜像
- 社区镜像：用户和组织贡献的开源镜像
- 免费使用：提供免费的公共仓库
- 自动构建：支持从 GitHub/Bitbucket 自动构建镜像

使用示例：

```bash
# 拉取官方 nginx 镜像
docker pull nginx:latest

# 拉取用户上传的镜像
docker pull username/custom-app:v1.0
```

#### 其他公共 Registry

| Registry                  | 提供商  | 特点                     |
| ------------------------- | ------- | ------------------------ |
| Quay.io                   | Red Hat | 企业级安全特性，漏洞扫描 |
| Google Container Registry | Google  | 集成 GCP 生态系统        |
| Amazon ECR Public         | AWS     | 集成 AWS 服务            |
| GitHub Container Registry | GitHub  | 与 GitHub 深度集成       |

### 私有 Registry

#### 企业私有 Registry 的优势

1. 安全性

- 内网部署，避免敏感镜像暴露
- 细粒度的访问控制
- 符合企业安全合规要求
- 支持镜像签名和验证

2. 性能优势

- 内网传输速度快
- 减少外网带宽使用
- 本地缓存提高访问速度
- 支持就近访问

3. 自主控制

- 完全控制存储和访问策略
- 自定义认证和授权机制
- 灵活的配置和扩展
- 数据主权控制

#### 私有 Registry 解决方案

1. Docker Distribution (Registry)
   官方开源的基础 Registry 实现：

```bash
# 快速部署基础 Registry
docker run -d \
  -p 5000:5000 \
  --name registry \
  -v registry-data:/var/lib/registry \
  registry:2
```

2. Harbor
   功能完整的企业级 Registry：

特性：

- Web UI 管理界面
- 基于角色的访问控制
- 镜像漏洞扫描
- 镜像签名验证
- 审计日志
- 镜像复制

3. 云服务 Registry

| 服务                      | 提供商    | 特点                           |
| ------------------------- | --------- | ------------------------------ |
| Amazon ECR                | AWS       | 完全托管，集成 IAM             |
| Azure Container Registry  | Microsoft | 集成 Azure AD，geo-replication |
| Google Container Registry | Google    | 集成 Cloud IAM，漏洞扫描       |
| 阿里云容器镜像服务        | 阿里云    | 国内访问优化，安全扫描         |

#### 私有 Registry 部署架构

```
企业私有 Registry 架构
┌─────────────────────────────────────┐
│              Load Balancer          │
└─────────────┬───────────────────────┘
              │
    ┌─────────┴─────────┐
    │                   │
┌───▼────┐         ┌───▼────┐
│Registry│         │Registry│
│Node 1  │         │Node 2  │
└───┬────┘         └───┬────┘
    │                  │
    └─────────┬────────┘
              │
    ┌─────────▼─────────┐
    │   Shared Storage  │
    │  (NFS/S3/Ceph)    │
    └───────────────────┘
```

### 选择建议

#### 使用公共 Registry 的场景

- 开源项目或学习环境
- 使用标准的开源镜像
- 小型项目或个人开发
- 对安全性要求不高的应用

#### 使用私有 Registry 的场景

- 企业生产环境
- 包含敏感信息的镜像
- 需要严格的访问控制
- 对网络性能有较高要求
- 符合合规性要求的场景

## 快速搭建私有 Registry

### 基础 Registry 部署

```bash
# 创建数据目录
mkdir -p /opt/registry/data
mkdir -p /opt/registry/certs
mkdir -p /opt/registry/auth

# 生成 SSL 证书 (生产环境建议使用正式证书)
openssl req -newkey rsa:4096 -nodes -sha256 \
  -keyout /opt/registry/certs/domain.key \
  -x509 -days 365 \
  -out /opt/registry/certs/domain.crt

# 创建认证文件
docker run --entrypoint htpasswd registry:2 \
  -Bbn admin password > /opt/registry/auth/htpasswd

# 启动 Registry
docker run -d \
  --name private-registry \
  --restart=always \
  -p 443:443 \
  -v /opt/registry/data:/var/lib/registry \
  -v /opt/registry/certs:/certs \
  -v /opt/registry/auth:/auth \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -e REGISTRY_AUTH=htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_REALM=Registry \
  registry:2
```

### 使用私有 Registry

```bash
# 登录私有 Registry
docker login your-registry.com

# 推送镜像
docker tag nginx:latest your-registry.com/nginx:latest
docker push your-registry.com/nginx:latest

# 拉取镜像
docker pull your-registry.com/nginx:latest
```

通过以上配置，您就拥有了一个功能完整、安全可靠的私有 Docker Registry。
