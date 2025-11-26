# GitHub Actions 自动化工作流

[![Sync with Upstream](https://github.com/CJackHwang/AIstudioProxyAPI/actions/workflows/sync-upstream.yml/badge.svg)](https://github.com/CJackHwang/AIstudioProxyAPI/actions/workflows/sync-upstream.yml)
[![Create Docker Release](https://github.com/CJackHwang/AIstudioProxyAPI/actions/workflows/docker-release.yml/badge.svg)](https://github.com/CJackHwang/AIstudioProxyAPI/actions/workflows/docker-release.yml)

本文档介绍了为 AI Studio Proxy API 项目配置的自动化 GitHub Actions 工作流。

## 📋 工作流概览

项目包含两个主要的 GitHub Actions 自动化工作流，实现了从代码同步到镜像构建的全自动化闭环：

### 1. 🔄 自动同步上游仓库 (`sync-upstream.yml`)

**功能**: 每天自动同步 fork 的仓库与上游仓库（CJackHwang/AIstudioProxyAPI），确保您的私有部署始终保持最新。

**特性**:
- ✅ **每日同步**: 每天UTC 20点（北京时间次日凌晨4点）自动运行
- ✅ **完整同步**: 不仅同步 tags，也同步 main 分支的最新 commits
- ✅ **自动触发构建**: 同步成功后，自动触发 Docker 构建工作流
- ✅ **手动触发**: 支持在 Actions 页面手动触发同步

**触发方式**:
- ⏰ 定时触发：每天UTC 20点
- 👆 手动触发：在 Actions 页面手动运行

### 2. 🐳 Docker Release (`docker-release.yml`)

**功能**: 自动构建 Docker 镜像并发布到 GitHub Container Registry (GHCR)。

**支持两种构建模式**:

| 模式 | 触发条件 | Tag 格式 | 说明 |
|------|---------|----------|------|
| **稳定版发布** | 上游发布新 Tag (如 `v1.2.0`) | `v1.2.0`, `latest` | 对应上游正式版本，自动创建 GitHub Release |
| **日常构建 (Nightly)** | 日常同步 (无新Tag) 或 定时/手动 | `v2024.11.26-a1b2c3d` | 对应 main 分支最新代码，**不创建** GitHub Release |

**特性**:
- ✅ **双重标签**: 稳定版同时打上版本号和 `latest` 标签
- ✅ **自动 Changelog**: 发布稳定版时自动生成详细变更日志
- ✅ **多架构支持**: 自动构建适配不同架构的镜像
- ✅ **灵活触发**: 支持 Tag 推送、定时任务、手动触发等多种方式

## 🚀 使用指南

### 场景一：保持最新 (推荐)

您无需做任何操作。
1. 工作流每天自动同步上游代码。
2. 自动触发构建最新的 Docker 镜像。
3. 您只需定期拉取 `latest` 镜像即可享受最新功能。

```bash
# 拉取最新镜像（包含最新 commits）
docker pull ghcr.io/<your-username>/aistudioproxyapi:latest
```

### 场景二：使用特定稳定版

如果您希望使用经过验证的稳定版本：

```bash
# 拉取特定版本
docker pull ghcr.io/<your-username>/aistudioproxyapi:v1.2.0
```

### 场景三：手动触发更新

如果您知道上游刚更新了重要功能，不想等第二天自动同步：

1. 进入 GitHub 仓库的 **Actions** 页面。
2. 选择 **Sync with Upstream** 工作流。
3. 点击 **Run workflow**。
4. 等待同步完成，它会自动触发 Docker 构建。
5. 构建完成后，拉取镜像即可。

## 🐳 Docker 镜像使用

### 运行容器

```bash
docker run -d \
    --restart always \
    -p 8880:2048 \
    -p 8881:3120 \
    -v "/path/to/auth_profiles":/app/auth_profiles \
    -v "/path/to/.env":/app/.env:ro \
    --name ai-studio-proxy \
    ghcr.io/<your-username>/aistudioproxyapi:latest
```

### 环境变量与配置

推荐将 `.env` 文件挂载到容器中，这样可以方便地管理配置。

| 本地路径 | 容器路径 | 说明 |
|---------|---------|------|
| `/path/to/auth_profiles` | `/app/auth_profiles` | **必需**。存放认证文件 |
| `/path/to/.env` | `/app/.env:ro` | **推荐**。配置文件 |
| `/path/to/certs` | `/app/certs` | **可选**。自签名证书 |

## ⚙️ 权限配置

为了让工作流正常运行，请确保仓库权限设置正确：

1. 进入仓库 **Settings** > **Actions** > **General**。
2. 在 **Workflow permissions** 下，选择 **Read and write permissions**。
3. 勾选 **Allow GitHub Actions to create and approve pull requests**。

## 📁 文件结构

```
.github/
├── workflows/
│   ├── sync-upstream.yml      # 负责代码同步
│   └── docker-release.yml     # 负责镜像构建与发布
└── README.md                   # 本说明文档
```

## 📝 更新日志

- **2025-11-26**: 优化文档，明确区分稳定版发布与日常自动构建流程。
- **2025-11-26**: 初始版本，创建自动同步和 Docker 发布工作流。
