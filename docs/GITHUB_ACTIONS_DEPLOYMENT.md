# GitHub Actions 部署指南

本文档详细介绍如何使用 GitHub Actions 部署每日股票分析系统。

---

## 📋 目录

1. [前置条件](#前置条件)
2. [工作流概览](#工作流概览)
3. [配置 Secrets 和 Variables](#配置-secrets-和-variables)
4. [部署方式](#部署方式)
5. [Docker 镜像部署](#docker-镜像部署)
6. [桌面应用发布](#桌面应用发布)
7. [常见问题](#常见问题)

---

## 前置条件

在开始之前，确保你已具备：

- GitHub 账号
- 已 Fork 或 Clone 本仓库
- 基本的 Git 操作知识
- 必要的 API Keys（见下方配置）

---

## 工作流概览

项目包含以下 GitHub Actions 工作流：

| 工作流文件 | 触发条件 | 用途 |
|-----------|---------|------|
| `ci.yml` | Pull Request | CI 门禁检查（代码质量、构建测试） |
| `daily_analysis.yml` | 定时 / 手动 | 每日股票分析任务 |
| `docker-publish.yml` | Tag 推送 / 手动 | 发布 Docker 镜像到 GHCR |
| `ghcr-dockerhub.yml` | 手动 | 多架构镜像推送到 GHCR + Docker Hub |
| `desktop-release.yml` | Tag 推送 / 手动 | 构建桌面应用安装包 |
| `create-release.yml` | 手动 | 创建 GitHub Release |
| `pr-review.yml` | Pull Request | PR 代码审查 |
| `stale.yml` | 定时 | 自动关闭不活跃 Issue/PR |

---

## 配置 Secrets 和 Variables

进入仓库 **Settings → Secrets and variables → Actions**，配置以下内容：

### 🔐 必需的 Secrets

#### AI 模型配置（至少配置一个）

| Secret 名称 | 说明 | 获取方式 |
|------------|------|---------|
| `GEMINI_API_KEY` | Google Gemini API Key | [Google AI Studio](https://aistudio.google.com/app/apikey) |
| `ANTHROPIC_API_KEY` | Claude API Key | [Anthropic Console](https://console.anthropic.com/) |
| `OPENAI_API_KEY` | OpenAI API Key | [OpenAI Platform](https://platform.openai.com/api-keys) |
| `AIHUBMIX_KEY` | AIHubMix Key（国内推荐） | [AIHubMix](https://aihubmix.com/) |

#### 数据源配置

| Secret 名称 | 说明 | 获取方式 |
|------------|------|---------|
| `TUSHARE_TOKEN` | Tushare Pro Token | [Tushare](https://tushare.pro/register) |

#### 搜索服务（至少配置一个）

| Secret 名称 | 说明 |
|------------|------|
| `TAVILY_API_KEYS` | Tavily API Keys（逗号分隔多个） |
| `SERPAPI_API_KEYS` | SerpAPI Keys |
| `BOCHA_API_KEYS` | 博查 API Keys |

#### 通知渠道（至少配置一个）

| Secret 名称 | 说明 |
|------------|------|
| `WECHAT_WEBHOOK_URL` | 企业微信机器人 Webhook |
| `FEISHU_WEBHOOK_URL` | 飞书机器人 Webhook |
| `TELEGRAM_BOT_TOKEN` | Telegram Bot Token |
| `TELEGRAM_CHAT_ID` | Telegram Chat ID |
| `PUSHPLUS_TOKEN` | PushPlus Token |
| `DISCORD_WEBHOOK_URL` | Discord Webhook URL |
| `SLACK_WEBHOOK_URL` | Slack Webhook URL |
| `EMAIL_SENDER` | 发件人邮箱 |
| `EMAIL_PASSWORD` | 邮箱授权码 |
| `EMAIL_RECEIVERS` | 收件人邮箱（逗号分隔） |

#### Docker Hub 配置（可选）

| Secret 名称 | 说明 |
|------------|------|
| `DOCKERHUB_USERNAME` | Docker Hub 用户名 |
| `DOCKERHUB_TOKEN` | Docker Hub Access Token |

### 📝 Variables（可选）

| Variable 名称 | 默认值 | 说明 |
|--------------|-------|------|
| `STOCK_LIST` | `600519` | 自选股代码（逗号分隔） |
| `REPORT_TYPE` | `simple` | 报告类型：`simple` / `detailed` |
| `MARKET_REVIEW_ENABLED` | `true` | 是否启用大盘复盘 |
| `MARKET_REVIEW_REGION` | `cn` | 市场区域：`cn` / `us` / `hk` |
| `MAX_WORKERS` | `1` | 并发工作进程数 |
| `TRADING_DAY_CHECK_ENABLED` | `true` | 是否检查交易日 |

---

## 部署方式

### 方式一：定时自动分析（推荐）

使用 `daily_analysis.yml` 工作流，每天自动运行股票分析。

**触发方式：**
- **定时触发**：周一至周五，北京时间 18:00（UTC 10:00）
- **手动触发**：Actions 页面选择 "每日股票分析" → Run workflow

**手动触发选项：**

| 选项 | 说明 |
|-----|------|
| `mode` | `full`（完整分析）、`market-only`（仅大盘）、`stocks-only`（仅股票） |
| `force_run` | 强制运行（跳过交易日检查） |

**配置步骤：**

1. 进入仓库 Settings → Secrets and variables → Actions
2. 配置至少一个 AI 模型的 API Key
3. 配置至少一个通知渠道
4. 配置 `STOCK_LIST` 变量（自选股代码）
5. 等待定时触发，或手动触发工作流

**查看结果：**
- 分析报告会上传到 Artifacts（保留 30 天）
- 通知会发送到配置的渠道

---

### 方式二：手动运行分析

1. 进入 **Actions** 页面
2. 选择 **"每日股票分析"** 工作流
3. 点击 **Run workflow**
4. 选择运行模式：
   - `full` - 完整分析（股票 + 大盘复盘）
   - `market-only` - 仅大盘复盘
   - `stocks-only` - 仅股票分析
5. 点击绿色的 **Run workflow** 按钮

---

## Docker 镜像部署

### 发布到 GitHub Container Registry (GHCR)

使用 `docker-publish.yml` 工作流：

**触发方式：**
1. 推送 Tag（如 `v1.0.0`）
2. 手动触发，输入 release tag

**步骤：**

```bash
# 创建并推送 tag
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

或者手动触发：
1. Actions → "Docker Publish" → Run workflow
2. 输入 tag 版本号（如 `v1.0.0`）

**拉取镜像：**

```bash
docker pull ghcr.io/jserlin/daily_stock_analysis:latest
```

### 发布到 GHCR + Docker Hub

使用 `ghcr-dockerhub.yml` 工作流（多架构支持）：

**前置条件：**
- 配置 `DOCKERHUB_USERNAME` Secret
- 配置 `DOCKERHUB_TOKEN` Secret

**触发方式：**
1. Actions → "Build and Push Multi-Arch Docker Images" → Run workflow
2. 输入镜像 tag（如 `v1.0.0` 或 `latest`）

**支持平台：**
- `linux/amd64`
- `linux/arm64`

**拉取镜像：**

```bash
# 从 GHCR 拉取
docker pull ghcr.io/jserlin/daily_stock_analysis:v1.0.0

# 从 Docker Hub 拉取
docker pull jserlin/daily_stock_analysis:v1.0.0
```

### 使用 Docker 运行

```bash
# 拉取镜像
docker pull ghcr.io/jserlin/daily_stock_analysis:latest

# 运行容器（需要传入环境变量）
docker run -d \
  --name stock-analysis \
  -e GEMINI_API_KEY="your-api-key" \
  -e STOCK_LIST="600519,000001" \
  -e WECHAT_WEBHOOK_URL="https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxx" \
  -v $(pwd)/data:/app/data \
  -v $(pwd)/logs:/app/logs \
  -v $(pwd)/reports:/app/reports \
  -p 8000:8000 \
  ghcr.io/jserlin/daily_stock_analysis:latest
```

**Docker Compose 示例：**

```yaml
version: '3.8'
services:
  stock-analysis:
    image: ghcr.io/jserlin/daily_stock_analysis:latest
    container_name: stock-analysis
    environment:
      - GEMINI_API_KEY=${GEMINI_API_KEY}
      - STOCK_LIST=600519,000001
      - WECHAT_WEBHOOK_URL=${WECHAT_WEBHOOK_URL}
      - SCHEDULE_ENABLED=true
      - SCHEDULE_TIME=18:00
    volumes:
      - ./data:/app/data
      - ./logs:/app/logs
      - ./reports:/app/reports
    ports:
      - "8000:8000"
    restart: unless-stopped
```

---

## 桌面应用发布

使用 `desktop-release.yml` 工作流构建桌面应用安装包。

**触发方式：**
1. 推送 Tag（如 `v1.0.0`）
2. 手动触发，输入 release tag

**构建产物：**

| 平台 | 文件名格式 |
|-----|-----------|
| Windows | `daily-stock-analysis-windows-installer-{tag}.exe` |
| Windows (免安装) | `daily-stock-analysis-windows-noinstall-{tag}.zip` |
| macOS (Intel) | `daily-stock-analysis-macos-x64-{tag}.dmg` |
| macOS (Apple Silicon) | `daily-stock-analysis-macos-arm64-{tag}.dmg` |

**步骤：**

```bash
# 1. 更新 CHANGELOG
# 编辑 docs/CHANGELOG.md，添加新版本的变更记录

# 2. 创建并推送 tag
git tag -a v1.0.0 -m "Release v1.0.0: 添加新功能"
git push origin v1.0.0
```

或者手动触发：
1. Actions → "Desktop Release" → Run workflow
2. 输入已存在的 release tag

---

## CI 门禁说明

`ci.yml` 工作流在 PR 时自动运行，包含以下检查：

| Job | 说明 |
|-----|------|
| `ai-governance` | 检查 AI 相关资源配置 |
| `backend-gate` | 后端代码质量检查（lint + import 测试） |
| `docker-build` | Docker 镜像构建测试 |
| `web-gate` | 前端代码 lint + build（仅当修改 `apps/dsa-web/` 时） |

---

## 常见问题

### 1. 定时任务没有执行

**可能原因：**
- GitHub Actions 默认在仓库无活动 60 天后禁用定时任务
- 配置了 `TRADING_DAY_CHECK_ENABLED=true` 但当天不是交易日

**解决方案：**
- 保持仓库活跃（定期提交代码）
- 手动触发一次，使用 `force_run: true` 跳过交易日检查

### 2. 分析失败：未配置 API Key

**错误信息：** `❌ 未配置任何 AI 模型 API Key`

**解决方案：**
1. 进入 Settings → Secrets and variables → Actions
2. 添加至少一个 AI 模型的 API Key（推荐 `GEMINI_API_KEY`）

### 3. 通知发送失败

**可能原因：**
- Webhook URL 配置错误
- Webhook 已失效

**解决方案：**
- 检查 Webhook URL 格式是否正确
- 重新生成 Webhook URL 并更新 Secret

### 4. Docker 镜像拉取失败

**可能原因：**
- 镜像不存在
- 权限问题（私有仓库）

**解决方案：**
```bash
# 登录 GHCR
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# 拉取镜像
docker pull ghcr.io/jserlin/daily_stock_analysis:latest
```

### 5. 桌面应用构建失败

**可能原因：**
- Tag 格式不正确（需要 `vX.Y.Z` 格式）
- CHANGELOG 中没有对应版本的记录

**解决方案：**
- 确保 tag 格式为 `v1.0.0`（语义化版本）
- 在 `docs/CHANGELOG.md` 中添加对应版本的变更记录

---

## 快速开始清单

- [ ] Fork 或 Clone 仓库
- [ ] 配置 AI 模型 API Key（至少一个）
- [ ] 配置通知渠道（至少一个）
- [ ] 配置 `STOCK_LIST` 变量
- [ ] 手动触发一次分析测试
- [ ] 检查通知是否正常接收
- [ ] （可选）配置 Docker Hub 发布
- [ ] （可选）配置桌面应用发布

---

## 相关链接

- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [项目 README](./README.md)
- [更新日志](./docs/CHANGELOG.md)
