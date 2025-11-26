# GitHub Actions 自动化工作流

本文档介绍了为 AI Studio Proxy API 项目配置的自动化 GitHub Actions 工作流。

## 📋 工作流概览

项目包含两个主要的 GitHub Actions 自动化工作流：

### 1. 🔄 自动同步上游仓库 (`sync-upstream.yml`)

**功能**: 每3小时自动同步 fork 的仓库与上游仓库（CJackHwang/AIstudioProxyAPI）

**特性**:
- ✅ 每3小时自动运行（UTC时间 0,3,6,9,12,15,18,21点）
- ✅ 支持手动触发
- ✅ 智能检测是否需要同步
- ✅ 自动合并上游更新
- ✅ **自动同步上游tags到fork**
- ✅ 触发Docker Release工作流
- ✅ 生成详细的执行报告

**触发方式**:
- 定时触发：每3小时
- 手动触发：在 Actions 页面手动运行

**权限要求**:
- `contents: write` - 推送更改到仓库

**关键流程**:
```
上游仓库打tag → sync工作流检测 → 同步tag到fork → 触发docker-release → 构建镜像
```

### 2. 🐳 Docker Release (`docker-release.yml`)

**功能**: 自动构建 Docker 镜像并创建 GitHub Release

**特性**:
- ✅ 自动构建和推送 Docker 镜像到 GitHub Container Registry (GHCR)
- ✅ 创建包含详细变更日志的 GitHub Release
- ✅ 支持多标签（latest、版本号）
- ✅ 自动生成 changelog
- ✅ 支持手动触发（可选择不创建Release）

**触发方式**:
- 自动触发：fork收到同步的tag（来自sync工作流）
- 手动触发：在 Actions 页面手动运行

**权限要求**:
- `contents: read` - 读取仓库内容
- `packages: write` - 推送 Docker 镜像
- `packages: read` - 读取镜像信息

**使用场景**:
- 自动：上游tag → 自动构建镜像 + 创建Release
- 手动：仅构建镜像（不创建Release）

## 🚀 使用指南

### 方式一：自动同步上游代码

1. **自动触发**：
   - 每3小时自动检测上游仓库更新
   - 当检测到新tag时，自动同步到fork

2. **自动构建Docker**：
   - 上游仓库打tag → sync工作流同步tag → docker-release工作流自动构建
   - 无需手动干预，全自动化

3. **监控同步**：
   - 在 GitHub 仓库的 "Actions" 页面查看工作流执行状态
   - 每次执行后会在执行摘要中显示同步状态

### 方式二：手动触发同步

1. **手动运行同步**：
   - 进入 Actions 页面
   - 选择 "Sync with Upstream" 工作流
   - 点击 "Run workflow" 按钮

### 方式三：手动构建 Docker 镜像

1. **进入 Actions 页面**：
   - 选择 "Create Docker Release" 工作流
   - 点击 "Run workflow"

2. **填写参数**：
   - **Tag name**: 输入要构建的tag名称（如 v1.0.0）
   - **Create GitHub Release**: 选择是否创建GitHub Release
     - ✅ 选中：构建镜像 + 创建Release
     - ❌ 不选中：仅构建镜像

3. **获取镜像**：
   ```bash
   # 拉取最新版本
   docker pull ghcr.io/<your-username>/aistudioproxyapi:latest

   # 拉取特定版本
   docker pull ghcr.io/<your-username>/aistudioproxyapi:v1.0.0
   ```

### 完整自动化流程

**上游仓库发布新版本时**：

1. 原仓库维护者推送新tag（如 `v1.2.0`）
2. 每3小时，sync工作流自动检测并同步tag到fork
3. docker-release工作流自动触发，构建Docker镜像
4. 自动创建GitHub Release
5. 完成！您的fork现在有了对应的Docker镜像

**用户无需任何手动操作！** 🎉

## 📁 文件结构

```
.github/
├── workflows/
│   ├── sync-upstream.yml      # 自动同步上游仓库
│   └── docker-release.yml     # Docker 镜像发布
└── README.md                   # 本文档
```

## ⚙️ 配置说明

### 所需权限

确保您的 GitHub 仓库已启用以下设置：

1. **Workflow 权限**：
   - 进入仓库 Settings > Actions > General
   - 选择 "Read and write permissions"
   - 勾选 "Allow GitHub Actions to create and approve pull requests"（如果需要自动创建 PR）

2. **容器注册表访问**：
   - GitHub Container Registry (GHCR) 默认可用
   - 无需额外配置，使用 `GITHUB_TOKEN` 进行身份验证

### 可选配置

您可以在仓库 Secrets 中配置以下变量：

- `DOCKER_PROXY_ADDR`: Docker 构建时的代理地址（可选）

## 📊 监控和调试

### 查看工作流状态

1. 进入 GitHub 仓库
2. 点击 "Actions" 标签页
3. 查看工作流执行历史和日志

### 常见问题

**问题1**: 同步工作流失败
```
解决方案: 检查是否有 push 权限，确保分支保护规则允许 workflow 推送
```

**问题2**: Docker 构建失败
```
解决方案: 检查 Dockerfile 和依赖配置是否正确
```

**问题3**: 权限错误
```
解决方案: 检查仓库 Settings > Actions > General 中的工作流权限设置
```

## 🔧 自定义配置

### 修改同步频率

编辑 `.github/workflows/sync-upstream.yml` 中的 cron 表达式：

```yaml
on:
  schedule:
    # 每6小时运行一次
    - cron: '0 */6 * * *'
```

### 修改 Docker 镜像标签策略

编辑 `.github/workflows/docker-release.yml` 中的 metadata 配置：

```yaml
tags: |
  type=ref,event=tag,pattern={{tag}}
  type=raw,value=latest
  # 添加其他标签格式
```

## 📝 更新日志

- **2025-11-26**: 初始版本，创建自动同步和 Docker 发布工作流

## 🤝 贡献

如有问题或建议，请提交 Issue 或 Pull Request。

## 📄 许可证

本项目遵循与主项目相同的 AGPLv3 许可证。
