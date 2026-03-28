# 项目结构分析：每日股票分析系统（Daily Stock Analysis）

## 概述

这是一个功能完善的 **A股/港股/美股** 智能分析系统，基于 Python 构建，集成了多数据源、AI 分析引擎、多渠道通知和交易策略。

---

## 顶层目录结构

```
daily_stock_analysis-main/
├── main.py                 # 主入口：调度器 & CLI
├── server.py               # FastAPI 后端服务入口
├── webui.py                # Web UI 启动器
├── src/                    # 核心业务逻辑
├── data_provider/          # 多数据源集成层
├── api/                    # REST API 层（FastAPI）
├── bot/                    # 机器人命令系统
├── strategies/             # 交易策略（YAML 配置）
├── apps/                   # 桌面端 & Web 应用
├── scripts/                # 工具脚本
├── docker/                 # Docker 部署
├── .github/workflows/      # CI/CD
├── requirements.txt        # Python 依赖
├── pyproject.toml          # 项目配置
└── docs/                   # 文档
```

---

## 各模块目的与依赖关系

### 1. 入口层

| 文件 | 目的 |
|---|---|
| `main.py` | 主调度器，支持 CLI 参数（`--debug`, `--dry-run`, `--webui`, `--serve`），启动定时分析任务 |
| `server.py` | FastAPI 后端服务，通过 `uvicorn` 运行 |
| `webui.py` | Web 管理界面启动器 |

**依赖关系：** `main.py` → `src/core/pipeline.py`、`src/scheduler.py`、`src/webui_frontend.py`

---

### 2. `src/` — 核心业务逻辑

| 模块 | 目的 |
|---|---|
| `core/pipeline.py` | **分析管道**，编排整个分析流程（数据获取 → 技术分析 → AI 分析 → 通知） |
| `analyzer.py` | **AI 分析引擎**，通过 LiteLLM 统一调用 Gemini/Claude/OpenAI/DeepSeek 等模型 |
| `stock_analyzer.py` | **技术趋势分析**，计算 MA、成交量、动量等技术指标 |
| `market_analyzer.py` | **市场整体分析**，生成市场摘要报告 |
| `market_context.py` | **市场角色与准则上下文** |
| `market_review.py` | **每日市场复盘**（市场概览、板块排名） |
| `notification.py` | **多渠道通知服务**：企业微信、飞书、Telegram、Discord、Slack、邮件、PushPlus、钉钉 |
| `storage.py` | **数据库操作**，基于 SQLite + SQLAlchemy ORM |
| `search_service.py` | **新闻搜索**，集成 Tavily、SerpAPI、Bocha、Brave、MiniMax |
| `config.py` | **配置管理**，支持多 AI 模型配置 |
| `scheduler.py` | **任务调度器**，定时执行每日分析 |
| `auth.py` | **认证鉴权** |
| `agent/` | **Agent 系统**，多轮策略问答 |
| `data/` | 数据模型与映射 |
| `repositories/` | 数据访问层 |
| `schemas/` | Pydantic 请求/响应模型 |
| `services/` | 业务逻辑服务 |
| `utils/` | 工具函数 |

**核心数据流：**

```
main.py → StockAnalysisPipeline
  ├── DataFetcherManager (data_provider/)
  ├── StockTrendAnalyzer (stock_analyzer.py)
  ├── GeminiAnalyzer (analyzer.py) → LiteLLM → AI 模型
  └── NotificationService (notification.py) → 各通知渠道
```

---

### 3. `data_provider/` — 数据源层

采用 **优先级 + 自动降级** 策略：

| 优先级 | 模块 | 数据源 |
|---|---|---|
| 0 | `efinance_fetcher.py` | eFinance（东方财富） |
| 1 | `akshare_fetcher.py` | AkShare（东方财富爬虫） |
| 2 | `tushare_fetcher.py` | Tushare Pro API |
| 2 | `pytdx_fetcher.py` | 通达信行情服务器 |
| 3 | `baostock_fetcher.py` | 证券宝 |
| 4 | `yfinance_fetcher.py` | Yahoo Finance（兜底） |

其他关键文件：

- `base.py` — 基础 Fetcher 类，实现降级逻辑
- `tickflow_fetcher.py` — TickFlow API，增强市场复盘
- `fundamental_adapter.py` — 基本面数据适配器
- `realtime_types.py` — 实时数据类型定义

---

### 4. `api/` — REST API 层

```
api/
├── app.py           # FastAPI 应用配置
├── deps.py          # 依赖注入
├── middlewares/     # 自定义中间件
└── v1/
    ├── endpoints/   # 路由处理器
    └── schemas/     # API 数据模型
```

**依赖关系：** API 层 → `src/services/`、`src/repositories/`、`src/schemas/`

---

### 5. `bot/` — 机器人系统

| 模块 | 目的 |
|---|---|
| `dispatcher.py` | 命令分发器 |
| `handler.py` | Webhook 处理器 |
| `platforms/` | 平台适配器：飞书、钉钉、企业微信、Telegram |
| `commands/` | 命令处理器 |

**依赖关系：** Bot → `src/agent/`、`data_provider/`

---

### 6. `strategies/` — 交易策略

11 个内置策略（YAML 配置）：

| 策略文件 | 说明 |
|---|---|
| `bull_trend.yaml` | 牛市趋势 |
| `ma_golden_cross.yaml` | 均线金叉 |
| `chan_theory.yaml` | 缠论 |
| `wave_theory.yaml` | 波浪理论 |
| `volume_breakout.yaml` | 放量突破 |
| `emotion_cycle.yaml` | 市场情绪周期 |
| `box_oscillation.yaml` | 箱体震荡 |
| `bottom_volume.yaml` | 底部放量 |
| `dragon_head.yaml` | 龙头战法 |
| `one_yang_three_yin.yaml` | 一阳三阴 |
| `shrink_pullback.yaml` | 缩量回踩 |

---

### 7. `apps/` — 应用层

- `dsa-web/` — Web 应用（React + FastAPI）
- `dsa-desktop/` — 桌面应用（Electron）

---

### 8. `scripts/` — 工具脚本

- `fetch_tushare_stock_list.py` — 从 Tushare 获取最新股票列表
- `generate_stock_index.py` — 生成搜索索引
- `check_ai_assets.py` — 检查 AI 模型资产
- `build-*.sh/ps1` — 多平台构建脚本

---

## 模块依赖关系总图

```
                    main.py / server.py
                          │
                    ┌─────┴──────┐
                    │   API 层    │
                    │  (api/)    │
                    └─────┬──────┘
                          │
                    ┌─────┴──────┐
                    │  核心逻辑   │
                    │  (src/)    │
                    └──┬──┬──┬───┘
                       │  │  │
           ┌───────────┘  │  └───────────┐
           │              │               │
    ┌──────┴──────┐ ┌────┴────┐  ┌──────┴──────┐
    │  数据源层    │ │ AI 引擎 │  │  通知服务   │
    │(data_provider)│ │(LiteLLM)│  │(notification)│
    └─────────────┘ └─────────┘  └─────────────┘
                                        │
                              ┌─────────┴─────────┐
                              │  企业微信 │ 飞书 │ TG │ ...  │
                              └───────────────────┘
```

---

## 关键技术栈

- **后端框架：** FastAPI + Uvicorn
- **数据存储：** SQLite + SQLAlchemy
- **AI 集成：** LiteLLM（统一接口，支持 Gemini/Claude/OpenAI/DeepSeek/Ollama）
- **数据源：** eFinance、AkShare、Tushare、PyTDX、Baostock、YFinance
- **部署：** Docker + GitHub Actions CI/CD

---

## 核心特性

1. **多市场支持** — A股、港股、美股
2. **多数据源** — 6 个数据源，自动降级容错
3. **AI 分析** — 统一 LLM 接口，支持 Gemini/Claude/OpenAI/DeepSeek
4. **多渠道通知** — 8+ 通知平台
5. **交易策略** — 11 个内置技术分析策略
6. **交互式 Agent** — 多轮策略问答
7. **回测能力** — 历史分析准确性测试
8. **Web 管理界面** — 全功能管理面板
9. **桌面客户端** — 跨平台桌面应用
10. **Docker 部署** — 容器化部署支持

---

## 启动方式

```bash
# 正常模式运行
python main.py

# 调试模式
python main.py --debug

# 试运行（不发送通知）
python main.py --dry-run

# 启动 Web UI
python main.py --webui

# 仅启动 API 服务
python main.py --serve

# FastAPI 后端服务
uvicorn server:app --reload --host 0.0.0.0 --port 8000
```

---

## 环境配置

关键环境变量（参见 `.env.example`）：

| 类别 | 变量 |
|---|---|
| AI 模型 | `GEMINI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `AIHUBMIX_KEY` |
| 股票列表 | `STOCK_LIST`（逗号分隔的股票代码） |
| 数据源 | `TUSHARE_TOKEN`, `TICKFLOW_API_KEY` |
| 搜索引擎 | `TAVILY_API_KEYS`, `SERPAPI_API_KEYS` |
| 通知渠道 | `WECHAT_WEBHOOK_URL`, `FEISHU_WEBHOOK_URL` 等 |
| 调度配置 | `SCHEDULE_ENABLED`, `SCHEDULE_TIME` |
