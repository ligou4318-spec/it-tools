# Cloudflare Pages 企业级部署指南
## Enterprise-Grade Deployment Architecture

> 🏢 **架构设计原则**: Google SRE + Microsoft Azure 企业级标准
> 🚀 **核心特性**: 全球边缘计算 + 零停机部署 + 自动化 CI/CD

---

## 📋 目录

1. [架构概览](#架构概览)
2. [前置条件](#前置条件)
3. [Cloudflare 配置](#cloudflare-配置)
4. [GitHub Secrets 配置](#github-secrets-配置)
5. [部署流程](#部署流程)
6. [验证部署](#验证部署)
7. [故障排查](#故障排查)

---

## 🏗️ 架构概览

### 部署架构图

```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐
│   GitHub    │ ───▶ │  GitHub      │ ───▶ │  Cloudflare │
│  Repository │      │   Actions    │      │   Pages     │
└─────────────┘      └──────────────┘      └─────────────│
                                                      │
                                                      ▼
                                          ┌──────────────────┐
                                          │  Global Edge CDN │
                                          │  (300+ Locations)│
                                          └──────────────────┘
```

### 部署模式

- **生产部署**: 推送到 `main` 分支自动触发
- **预览部署**: Pull Request 自动创建预览环境
- **手动部署**: GitHub Actions 手动触发

---

## ✅ 前置条件

### 必需账号

1. **GitHub 账号**: `https://github.com`
2. **Cloudflare 账号**: `https://dash.cloudflare.com`

### 必需工具

```bash
# Node.js 22+
node --version

# pnpm 9+
pnpm --version

# Git
git --version
```

---

## 🔧 Cloudflare 配置

### 步骤 1: 创建 Cloudflare Pages 项目

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com)
2. 进入 **Pages** 服务
3. 点击 **创建项目**
4. 选择 **连接到 Git**

### 步骤 2: 连接 GitHub 仓库

```
设置源                    │
─────────────────────────────────────────
连接 Git                 │ GitHub
                         │
仓库: ligou4318-spec/it-tools
生产分支: main
构建命令: pnpm run build:cf
构建输出目录: dist
```

### 步骤 3: 配置环境变量

在 Cloudflare Pages 设置中添加：

```bash
# Production Environment
NODE_VERSION = 22
PNPM_VERSION = 9.11.0
NODE_ENV = production
```

### 步骤 4: 获取 API Token

1. 进入 [Cloudflare API Tokens](https://dash.cloudflare.com/profile/api-tokens)
2. 点击 **创建令牌**
3. 使用 **Cloudflare Pages** 模板
4. 配置权限:
   - **Account**: Cloudflare Pages: Edit
   - **Zone**: Zone: Read (如果有自定义域名)
   - **Zone Resources**: Include - All zones
5. 复制生成的 Token

### 步骤 5: 获取 Account ID

1. 在 Cloudflare Dashboard 右侧可以看到 **Account ID**
2. 点击复制

---

## 🔐 GitHub Secrets 配置

### 在 GitHub 仓库中添加 Secrets

进入: `https://github.com/ligou4318-spec/it-tools/settings/secrets/actions`

添加以下 Secrets:

| Secret 名称 | 值 | 说明 |
|-------------|-----|------|
| `CLOUDFLARE_API_TOKEN` | 你的 API Token | Cloudflare API 令牌 |
| `CLOUDFLARE_ACCOUNT_ID` | 你的 Account ID | Cloudflare 账户 ID |

### 配置步骤

1. 点击 **New repository secret**
2. Name: `CLOUDFLARE_API_TOKEN`
3. Value: `<粘贴你的 API Token>`
4. 点击 **Add secret**

重复步骤添加 `CLOUDFLARE_ACCOUNT_ID`

---

## 🚀 部署流程

### 方式 1: 自动部署 (推荐)

```bash
# 1. 提交代码
git add .
git commit -m "feat: Your feature description"

# 2. 推送到 main 分支
git push origin main

# ✅ GitHub Actions 自动触发部署
# ✅ 约 2-3 分钟后部署完成
```

### 方式 2: 手动触发部署

1. 进入 GitHub Actions 页面
2. 选择 **Deploy to Cloudflare Pages**
3. 点击 **Run workflow**
4. 选择环境: `production` 或 `preview`
5. 点击 **Run workflow**

### 方式 3: PR 预览部署

```bash
# 1. 创建新分支
git checkout -b feature/new-feature

# 2. 提交更改
git add .
git commit -m "feat: Add new feature"

# 3. 推送到远程
git push origin feature/new-feature

# 4. 在 GitHub 创建 Pull Request
# ✅ 自动创建预览部署
```

---

## ✅ 验证部署

### 检查部署状态

1. **GitHub Actions**:
   ```
   https://github.com/ligou4318-spec/it-tools/actions
   ```

2. **Cloudflare Pages**:
   ```
   https://dash.cloudflare.com -> Pages -> it-tools
   ```

### 验证网站功能

```bash
# 检查主页面
curl -I https://it-tools.pages.dev

# 检查关键资源
curl -I https://it-tools.pages.dev/assets/index-*.js

# 检查 Service Worker
curl -I https://it-tools.pages.dev/sw.js
```

### 检查响应头

```bash
# 应该包含以下头部
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Cache-Control: public, max-age=31536000, immutable
CF-Cache-Status: HIT
```

---

## 🛠️ 故障排查

### 问题 1: 构建失败

**症状**: GitHub Actions 构建失败

**解决方案**:
```bash
# 本地测试构建
pnpm install
pnpm run build:cf

# 检查 Node.js 版本
node --version  # 应该是 v22+
```

### 问题 2: API Token 无效

**症状**: 部署时报错 `Invalid API Token`

**解决方案**:
1. 检查 Secret 名称是否正确: `CLOUDFLARE_API_TOKEN`
2. 重新生成 API Token
3. 确认 Token 权限包含 **Cloudflare Pages: Edit**

### 问题 3: Account ID 错误

**症状**: 部署时报错 `Invalid Account ID`

**解决方案**:
1. 在 Cloudflare Dashboard 复制正确的 Account ID
2. 检查 Secret 名称: `CLOUDFLARE_ACCOUNT_ID`

### 问题 4: 构建输出目录错误

**症状**: 部署成功但页面 404

**解决方案**:
```bash
# 检查 wrangler.toml 配置
pages_build_output_dir = "dist"

# 确认构建命令
pnpm run build:cf
```

### 问题 5: 环境变量未生效

**症状**: 应用行为异常

**解决方案**:
1. 在 Cloudflare Pages 设置中检查环境变量
2. 确保环境变量名称正确
3. 重新部署以应用环境变量

---

## 📊 性能优化

### 启用 Cloudflare 缓存

```toml
# wrangler.toml 中已配置
[[headers]]
for = "/assets/*"
[headers.values]
  Cache-Control = "public, max-age=31536000, immutable"
```

### 启用 Brotli 压缩

Cloudflare 自动启用，无需配置。

### 启用 HTTP/3

在 Cloudflare Dashboard -> Network -> HTTP/3: **On**

---

## 🔍 监控和日志

### Cloudflare Analytics

1. 进入 Cloudflare Pages 项目
2. 点击 **Analytics** 标签
3. 查看访问量、性能指标

### GitHub Actions 日志

1. 进入 Actions 页面
2. 点击具体的工作流运行
3. 查看详细日志

---

## 📚 最佳实践

### 1. 分支策略

```
main (生产)          ───▶ 自动部署到生产环境
  │
  ├─ develop (开发)  ───▶ PR 时创建预览环境
  │
  └─ feature/* (功能) ───▶ PR 时创建预览环境
```

### 2. 提交规范

```bash
# 功能开发
git commit -m "feat: Add new feature"

# Bug 修复
git commit -m "fix: Resolve deployment issue"

# 性能优化
git commit -m "perf: Optimize bundle size"

# 文档更新
git commit -m "docs: Update deployment guide"
```

### 3. 发布流程

```bash
# 1. 更新版本号
npm version patch  # 1.0.0 -> 1.0.1
npm version minor  # 1.0.0 -> 1.1.0
npm version major  # 1.0.0 -> 2.0.0

# 2. 推送标签
git push origin main --tags

# 3. 触发生产部署
```

---

## 🎯 下一步

1. ✅ 配置 Cloudflare Pages 项目
2. ✅ 添加 GitHub Secrets
3. ✅ 推送代码触发部署
4. ✅ 验证部署成功
5. ✅ 配置自定义域名（可选）

---

## 📞 支持

- **Cloudflare 文档**: https://developers.cloudflare.com/pages
- **GitHub Actions 文档**: https://docs.github.com/actions
- **项目 Issues**: https://github.com/ligou4318-spec/it-tools/issues

---

**部署架构版本**: v2.0.0
**最后更新**: 2024-01-15
**维护者**: Technical Architecture Team
