# 功能操作指南：首页 / 回测 / 问股 / 持仓

本文档详细介绍股票智能分析系统中四大核心功能的使用方法：**首页分析**、**回测验证**、**Agent 问股** 和 **持仓管理**。

---

## 目录

- [一、首页分析](#一首页分析)
  - [1.1 功能说明](#11-功能说明)
  - [1.2 首页布局](#12-首页布局)
  - [1.3 搜索与分析股票](#13-搜索与分析股票)
  - [1.4 查看分析报告](#14-查看分析报告)
  - [1.5 管理分析历史](#15-管理分析历史)
  - [1.6 智能导入股票](#16-智能导入股票)
  - [1.7 自选股配置](#17-自选股配置)
  - [1.8 定时自动分析](#18-定时自动分析)
  - [1.9 API 调用方式](#19-api-调用方式)
- [二、回测验证](#二回测验证)
  - [2.1 功能说明](#21-功能说明)
  - [2.2 前置条件](#22-前置条件)
  - [2.3 Web 界面操作](#23-web-界面操作)
  - [2.4 API 调用方式](#24-api-调用方式)
  - [2.5 结果解读](#25-结果解读)
  - [2.6 配置参数说明](#26-配置参数说明)
- [三、Agent 问股](#三agent-问股)
  - [3.1 功能说明](#31-功能说明)
  - [3.2 前置条件](#32-前置条件)
  - [3.3 Web 界面操作](#33-web-界面操作)
  - [3.4 支持的分析策略](#34-支持的分析策略)
  - [3.5 常用提问示例](#35-常用提问示例)
  - [3.6 API 调用方式](#36-api-调用方式)
  - [3.7 Bot 端使用](#37-bot-端使用)
- [四、持仓管理](#四持仓管理)
  - [4.1 功能说明](#41-功能说明)
  - [4.2 Web 界面操作](#42-web-界面操作)
  - [4.3 API 调用方式](#43-api-调用方式)
  - [4.4 CSV 批量导入交易记录](#44-csv-批量导入交易记录)
  - [4.5 支持的券商](#45-支持的券商)
  - [4.6 风险分析](#46-风险分析)
- [五、数据存储说明](#五数据存储说明)

---

## 一、首页分析

### 1.1 功能说明

首页是系统的核心入口，提供**股票搜索、AI 分析、报告查看、历史管理**一站式体验。登录后默认进入首页，可快速对任意股票发起 AI 分析并查看决策仪表盘报告。

核心能力：
- 智能搜索：支持股票代码、名称、拼音联想
- 一键分析：输入股票后快速生成 AI 决策仪表盘
- 实时进度：通过 SSE 流式推送显示分析进度
- 报告展示：结构化展示分析结论、买卖点位、操作检查清单
- 历史管理：查看、筛选、批量删除分析历史
- 多源导入：支持图片、CSV/Excel、剪贴板粘贴导入股票

### 1.2 首页布局

```
┌─────────────────────────────────────────────┐
│  [搜索框（股票代码/名称自动补全）]  [推送] [分析] │  ← 顶部操作栏
├─────────────────────────────────────────────┤
│                                             │
│  ┌─ 任务面板 ──────────────────────────┐    │  ← 进行中的分析任务
│  │  ⏳ 600519 处理中...                  │    │
│  │  ⏳ 000858 排队中...                  │    │
│  └──────────────────────────────────────┘    │
│                                             │
│  ┌─ 分析报告展示区 ────────────────────┐    │  ← 选中历史后显示
│  │  股票概要 / 策略建议 / 新闻情报      │    │
│  │  技术指标 / AI 模型信息              │    │
│  └──────────────────────────────────────┘    │
│                                             │
├──────────┬──────────────────────────────────┤
│ 分析历史  │  历史记录列表（无限滚动加载）     │  ← 左侧历史面板
│ ☑ 600519 │  ☑ 000858  2025-01-15           │
│ ☐ 300750 │  ☐ 601318  2025-01-14           │
│ ☐ 002594 │     ...                         │
│          │  [全选] [删除选中]               │
└──────────┴──────────────────────────────────┘
```

### 1.3 搜索与分析股票

#### 通过搜索框分析

1. 在首页顶部搜索框中输入股票代码或名称
2. 系统会自动弹出联想补全列表（支持代码、名称、拼音）
   - 输入 `600519` → 显示 "贵州茅台 600519"
   - 输入 `茅台` → 显示 "贵州茅台 600519"
   - 输入 `mt` → 拼音联想匹配
3. 从下拉列表中选择目标股票，或直接输入完整代码
4. 点击 **"分析"** 按钮发起分析
5. 分析任务创建后，在任务面板中可看到实时进度

#### 支持的市场

| 市场 | 代码格式 | 示例 |
|------|----------|------|
| A 股（沪市） | 600xxx / 601xxx / 603xxx | 600519（贵州茅台） |
| A 股（深市） | 000xxx / 002xxx / 300xxx | 000858（五粮液） |
| 港股 | 0xxxxx / 5xxxxx | 00700（腾讯控股） |
| 美股 | 字母代码 | AAPL（苹果） |

#### 分析进度说明

分析过程中，任务面板会实时显示进度：

| 状态 | 含义 |
|------|------|
| 排队中（pending） | 任务已创建，等待处理 |
| 处理中（processing） | AI 正在分析，调用工具获取数据 |
| 已完成（completed） | 分析完成，可点击查看报告 |
| 失败（failed） | 分析出错，可查看错误信息 |

### 1.4 查看分析报告

分析完成后，点击对应的历史记录即可查看完整报告。报告包含以下模块：

| 模块 | 内容 |
|------|------|
| **概要** | 股票基本信息、一句话核心结论 |
| **策略建议** | 精确买卖点位、操作检查清单、仓位建议 |
| **技术面** | 均线排列、MACD、RSI 等技术指标分析 |
| **筹码分布** | 持仓成本分布、集中度分析 |
| **新闻情报** | 最新相关新闻（最多 8 条） |
| **基本面** | 估值、盈利、机构持仓等数据 |
| **AI 模型** | 使用的 AI 模型及数据来源信息 |

#### 报告操作

- **追问 AI**：在报告基础上继续提问，进入 Agent 问股对话
- **查看完整报告**：展开查看 Markdown 格式的完整分析内容

### 1.5 管理分析历史

左侧历史面板展示所有历史分析记录，支持以下操作：

| 操作 | 说明 |
|------|------|
| 查看报告 | 点击历史记录查看对应分析报告 |
| 单选 | 勾选单条记录 |
| 全选 | 点击"全选"勾选当前可见的所有记录 |
| 批量删除 | 勾选后点击"删除选中"按钮，支持确认对话框 |
| 无限滚动 | 向下滚动自动加载更多历史记录 |

### 1.6 智能导入股票

系统支持三种方式批量导入股票进行分析：

#### 图片导入

1. 截取包含股票代码/名称的图片（如自选股截图、研报截图）
2. 通过导入功能上传图片
3. AI Vision 模型自动识别图片中的股票代码
4. 返回识别结果及置信度，确认后发起分析

**支持格式：** JPG、PNG、WebP、GIF（最大 5MB）

#### CSV / Excel 导入

1. 准备包含股票代码或名称的 CSV / Excel 文件
2. 上传文件或直接粘贴文本内容
3. 系统自动解析并提取股票代码
4. 确认后批量发起分析

**支持格式：** CSV、Excel (.xlsx)、纯文本（最大 2MB）

#### 剪贴板粘贴

1. 从任意来源复制包含股票代码/名称的文本
2. 在导入界面直接粘贴
3. 系统自动解析文本中的股票信息

#### 置信度说明

| 置信度 | 含义 | 建议 |
|--------|------|------|
| 高 | 代码明确匹配 | 可直接使用 |
| 中 | 名称匹配但需确认 | 建议人工确认 |
| 低 | 模糊匹配 | 需要手动选择正确股票 |

### 1.7 自选股配置

通过 `.env` 文件中的 `STOCK_LIST` 配置自选股列表，系统定时分析时会对该列表中的所有股票进行分析。

**配置方式：**

```bash
# .env 文件
STOCK_LIST=600519,000858,300750,601318,002594
```

**说明：**
- 股票代码以英文逗号分隔
- 代码自动转为大写格式
- 未配置时默认分析 600519、000001、300750
- 修改后系统会热加载，无需重启

### 1.8 定时自动分析

系统支持定时自动分析所有自选股，并通过配置的渠道推送结果。

**配置方式：**

在 `.env` 文件中配置定时任务：

```bash
# 定时分析开关
SCHEDULER_ENABLED=true

# 执行时间（24小时制，默认 18:00）
SCHEDULE_TIME=18:00
```

**自动分析流程：**

```
定时触发 → 读取 STOCK_LIST → 逐个发起分析 → 生成报告
  → 大盘复盘分析 → 推送到通知渠道（微信/飞书/Telegram 等）
```

**推送渠道配置：** 详见 `docs/` 目录下各 Bot 配置文档。

### 1.9 API 调用方式

#### 发起分析

```bash
# 分析单只股票
curl -X POST "http://localhost:8000/api/v1/analysis/analyze" \
  -H "Content-Type: application/json" \
  -d '{
    "stock_code": "600519",
    "stock_name": "贵州茅台",
    "report_type": "detailed",
    "async_mode": true,
    "selection_source": "autocomplete"
  }'
```

**参数说明：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `stock_code` | string | 股票代码（与 stock_codes 二选一） |
| `stock_codes` | string[] | 批量股票代码 |
| `stock_name` | string | 股票名称（用于记录） |
| `report_type` | string | 报告类型：`simple` / `detailed` / `full` / `brief` |
| `async_mode` | boolean | 是否异步（多只股票建议 true） |
| `force_refresh` | boolean | 是否忽略缓存强制刷新 |
| `selection_source` | string | 来源：`manual` / `autocomplete` / `import` / `image` |
| `notify` | boolean | 是否推送通知 |
| `original_query` | string | 用户原始输入 |

#### 查看任务状态

```bash
# 任务列表
curl "http://localhost:8000/api/v1/analysis/tasks?status=completed&limit=10"

# 单个任务状态
curl "http://localhost:8000/api/v1/analysis/status/{task_id}"

# SSE 实时推送
curl "http://localhost:8000/api/v1/analysis/tasks/stream"
```

#### 图片导入

```bash
curl -X POST "http://localhost:8000/api/v1/stocks/extract-from-image" \
  -F "file=@stock_screenshot.png"
```

#### CSV/文本导入

```bash
# 上传文件
curl -X POST "http://localhost:8000/api/v1/stocks/parse-import" \
  -F "file=@stock_list.csv"

# 粘贴文本
curl -X POST "http://localhost:8000/api/v1/stocks/parse-import" \
  -H "Content-Type: application/json" \
  -d '{"text": "600519 贵州茅台\n000858 五粮液\n300750 宁德时代"}'
```

#### 获取实时行情

```bash
curl "http://localhost:8000/api/v1/stocks/600519/quote"
```

#### 获取历史 K 线

```bash
curl "http://localhost:8000/api/v1/stocks/600519/history?period=daily&count=60"
```

---

## 二、回测验证

### 2.1 功能说明

回测功能用于**评估历史 AI 分析建议的准确率**。系统会自动将过去生成的分析记录与后续实际市场走势进行对比，模拟止盈止损交易，并生成详细的胜率统计报告。

核心能力：
- 自动评估历史分析记录的准确性
- 模拟带止盈止损的交易
- 统计方向胜率、盈亏比等关键指标
- 支持按股票代码、评估窗口筛选

### 2.2 前置条件

回测功能需要先有历史分析记录。确保：
1. 系统已正常运行并生成过分析结果（`analysis_history` 表中有数据）
2. 分析记录产生至少 **14 天**以上（默认 `min_age_days=14`），以便有足够的后续行情数据进行评估
3. 对应股票有足够的行情数据

### 2.3 Web 界面操作

1. **进入回测页面**
   - 在左侧导航栏中点击 **"回测"**
   - 或访问路径 `/backtest`

2. **设置回测参数**
   | 参数 | 说明 | 默认值 |
   |------|------|--------|
   | 股票代码 | 输入特定股票代码进行筛选，留空则回测所有股票 | 全部 |
   | 评估窗口 | 评估分析后多少个交易日内的表现（1-120 天） | 10 天 |
   | 强制重算 | 勾选后会重新计算已有结果 | 关闭 |

3. **执行回测**
   - 点击 **"运行回测"** 按钮
   - 系统会逐条处理分析记录，期间可看到进度提示
   - 运行完成后页面自动刷新结果

4. **查看结果**
   - **左侧面板**：显示总体绩效（胜率、方向准确率、平均收益率等）
   - **主内容区**：回测结果明细表格，包含以下列：
     - 代码、分析日期、操作建议、预期方向
     - 实际涨跌幅、模拟收益率
     - 止盈/止损触发情况、最终判定（胜/负/中性）

### 2.4 API 调用方式

#### 触发回测运行

```bash
# 回测所有股票
curl -X POST "http://localhost:8000/api/v1/backtest/run" \
  -H "Content-Type: application/json" \
  -d '{}'

# 回测指定股票
curl -X POST "http://localhost:8000/api/v1/backtest/run" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "600519",
    "eval_window_days": 10,
    "min_age_days": 14,
    "limit": 200,
    "force": false
  }'
```

**响应示例：**

```json
{
  "processed": 150,
  "saved": 150,
  "completed": 120,
  "insufficient": 25,
  "errors": 5
}
```

#### 查询回测结果

```bash
# 分页查询回测结果
curl "http://localhost:8000/api/v1/backtest/results?page=1&limit=20"

# 按股票筛选
curl "http://localhost:8000/api/v1/backtest/results?code=600519&eval_window_days=10"
```

#### 查询绩效指标

```bash
# 总体绩效
curl "http://localhost:8000/api/v1/backtest/performance"

# 指定股票绩效
curl "http://localhost:8000/api/v1/backtest/performance/600519"
```

### 2.5 结果解读

| 指标 | 含义 |
|------|------|
| `direction_accuracy_pct` | 方向准确率（预期涨跌与实际涨跌一致的比例） |
| `win_rate_pct` | 胜率（考虑止盈止损后的盈利比例） |
| `avg_stock_return_pct` | 平均股票涨跌幅 |
| `avg_simulated_return_pct` | 平均模拟交易收益率 |
| `stop_loss_trigger_rate` | 止损触发率 |
| `take_profit_trigger_rate` | 止盈触发率 |
| `outcome` | 单条判定：`win`（盈利）/ `loss`（亏损）/ `neutral`（中性） |

**中性区间**：涨跌幅在 ±2% 以内的判定为中性（可通过 `backtest_neutral_band_pct` 配置）。

### 2.6 配置参数说明

在 `.env` 或 `src/config.py` 中可调整：

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `backtest_eval_window_days` | 10 | 评估窗口（交易日） |
| `backtest_min_age_days` | 14 | 分析记录最小天数 |
| `backtest_engine_version` | v1 | 回测引擎版本 |
| `backtest_neutral_band_pct` | 2.0 | 中性区间百分比 |

---

## 三、Agent 问股

### 3.1 功能说明

Agent 问股是一个 **AI 驱动的多轮对话分析系统**，支持自然语言提问，内置多种专业分析策略，可实时获取行情数据并生成结构化分析报告。

核心能力：
- 自然语言交互，支持多轮对话
- 内置多种分析策略（均线金叉、缠论、波浪理论等）
- 实时流式输出，带进度提示
- 自动获取实时行情、技术指标、基本面数据
- 上下文记忆，支持追问

### 3.2 前置条件

1. 已配置 LLM API 密钥（如 `AIHUBMIX_KEY`、`GEMINI_API_KEY` 等）
2. 已配置数据源（Tushare Token 或 AkShare）
3. 后端服务和前端均正常运行

### 3.3 Web 界面操作

1. **进入问股页面**
   - 在左侧导航栏中点击 **"问股"** 或 **"Chat"**
   - 或访问路径 `/chat`

2. **选择分析策略**
   - 在输入框上方有一个策略选择下拉菜单
   - 可选择不同的分析策略（见下方策略列表）
   - 也可让 AI 自动选择

3. **输入问题**
   - 在输入框中用自然语言描述你的问题
   - 可参考下方的"快捷问题"快速提问

4. **查看分析过程**
   - 分析过程中会实时显示进度：
     - 💭 **思考中** — AI 正在规划分析步骤
     - 🔧 **调用工具** — 正在获取行情/指标数据
     - 📝 **生成报告** — 正在撰写最终分析
     - ✅ **完成** — 分析完成

5. **追问与多轮对话**
   - 在同一会话中可继续追问
   - AI 会记住之前的分析上下文

6. **管理会话历史**
   - 左侧会显示历史会话列表
   - 可切换查看不同会话

7. **导出分析结果**
   - 支持将对话导出到通知渠道（微信、Telegram 等）

### 3.4 支持的分析策略

| 策略名称 | 标识 | 说明 |
|----------|------|------|
| 多头趋势（默认） | `bull_trend` | 识别上升趋势和回调买入机会，基于 MA5/MA10/MA20 均线排列 |
| 缠论分析 | `chan_theory` | 缠论形态识别、笔段划分、中枢分析 |
| 波浪理论 | `wave_theory` | 艾略特波浪理论分析，波浪计数和形态识别 |
| 箱体震荡 | `box_oscillation` | 区间震荡交易分析，支撑阻力位识别 |
| 情绪周期 | `emotion_cycle` | 市场情绪周期分析，恐惧/贪婪指标 |
| 缩量回踩 | `shrink_pullback` | 缩量回调买入策略，量价配合分析 |
| 量能突破 | `volume_breakout` | 放量突破形态识别，确认价格走势 |
| 龙头战法 | `dragon_head` | 龙头股识别，板块动量分析 |

### 3.5 常用提问示例

**基础分析：**
- `分析 600519` — 对贵州茅台进行全面分析
- `茅台现在能买吗？` — 自然语言提问
- `用缠论分析茅台` — 指定策略分析

**进阶提问：**
- `600519 和 000858 哪个更适合现在买入？`
- `帮我看看白酒板块最近的表现`
- `这只股票的止盈止损位应该设在哪里？`
- `最近有什么利好消息？`

**追问示例：**
- `它的 RSI 指标怎么看？`
- `如果大盘下跌，这个股票会受多大影响？`
- `结合基本面分析一下估值是否合理`

### 3.6 API 调用方式

#### 同步分析

```bash
curl -X POST "http://localhost:8000/api/v1/analysis/analyze" \
  -H "Content-Type: application/json" \
  -d '{
    "stock_code": "600519",
    "report_type": "detailed",
    "async_mode": false
  }'
```

#### 异步分析（推荐，适合多只股票）

```bash
curl -X POST "http://localhost:8000/api/v1/analysis/analyze" \
  -H "Content-Type: application/json" \
  -d '{
    "stock_codes": ["600519", "000858", "601318"],
    "report_type": "simple",
    "async_mode": true
  }'
```

**响应示例（异步）：**

```json
{
  "task_ids": ["task_001", "task_002", "task_003"],
  "status": "pending",
  "message": "3 analysis tasks queued"
}
```

#### Agent 对话

```bash
# 普通对话
curl -X POST "http://localhost:8000/api/v1/agent/chat" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "分析一下 600519 的走势",
    "session_id": "my-session-001"
  }'

# 流式对话（SSE）
curl -X POST "http://localhost:8000/api/v1/agent/chat/stream" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "用缠论分析 600519",
    "session_id": "my-session-001",
    "skills": ["chan_theory"]
  }'
```

#### 查看分析任务状态

```bash
# 查看任务列表
curl "http://localhost:8000/api/v1/analysis/tasks?status=completed&limit=10"

# SSE 实时推送
curl "http://localhost:8000/api/v1/analysis/tasks/stream"

# 查看单个任务
curl "http://localhost:8000/api/v1/analysis/status/task_001"
```

#### 查看可用策略

```bash
curl "http://localhost:8000/api/v1/agent/skills"
```

#### 查看会话历史

```bash
curl "http://localhost:8000/api/v1/agent/chat/sessions"
```

### 3.7 Bot 端使用

如果配置了 Telegram / 微信 / 飞书等 Bot，也可以直接通过 Bot 进行问股：

```
/ask 分析 600519
/ask 用缠论分析茅台
/ask 现在大盘适合加仓吗
```

Bot 配置方式详见 `docs/bot/` 目录下的各 Bot 配置文档。

---

## 四、持仓管理

### 4.1 功能说明

持仓管理模块提供完整的投资组合管理功能，支持多账户、多市场（A股/港股/美股），包含交易记录、资金流水、公司行动（分红/拆股）的完整记录，以及实时盈亏计算和风险分析。

核心能力：
- 多账户管理（支持 A 股、港股、美股）
- 交易记录手动录入和 CSV 批量导入
- 资金流水管理（出入金记录）
- 公司行动记录（分红、拆股）
- 实时持仓快照与盈亏计算
- 风险分析（集中度、回撤、止损预警）
- 支持 FIFO 和平均成本两种计价方式

### 4.2 Web 界面操作

#### Step 1：创建账户

1. 进入 **持仓管理** 页面
2. 点击 **"新建账户"** 按钮
3. 填写账户信息：

| 字段 | 说明 | 示例 |
|------|------|------|
| 账户名称 | 自定义名称 | "主账户" |
| 券商 | 可选，记录所属券商 | "华泰证券" |
| 市场 | cn（A股）/ hk（港股）/ us（美股） | cn |
| 基础货币 | CNY / HKD / USD | CNY |

#### Step 2：录入交易记录

**手动录入：**

1. 在交易记录页面点击 **"新增交易"**
2. 填写交易详情：

| 字段 | 说明 | 示例 |
|------|------|------|
| 账户 | 选择目标账户 | 主账户 |
| 股票代码 | 证券代码 | 600519 |
| 交易日期 | 成交日期 | 2025-01-15 |
| 方向 | buy（买入）/ sell（卖出） | buy |
| 数量 | 成交数量（股） | 100 |
| 成交价格 | 每股价格 | 1680.00 |
| 手续费 | 交易费用（默认 0） | 5.00 |
| 印花税 | 印花税（默认 0） | 0 |
| 备注 | 可选备注 | 买入茅台 |

#### Step 3：管理资金流水

1. 在资金管理页面点击 **"新增记录"**
2. 填写资金变动信息：

| 字段 | 说明 | 示例 |
|------|------|------|
| 账户 | 选择目标账户 | 主账户 |
| 日期 | 发生日期 | 2025-01-15 |
| 方向 | in（转入）/ out（转出） | in |
| 金额 | 资金数额 | 500000 |
| 备注 | 可选 | 初始入金 |

#### Step 4：记录公司行动

对于分红、拆股等公司行动，需要在对应页面录入：

**现金分红：**

| 字段 | 说明 | 示例 |
|------|------|------|
| 股票代码 | 600519 | — |
| 生效日期 | 除权除息日 | 2025-06-20 |
| 行动类型 | cash_dividend | 现金分红 |
| 每股分红 | 分红金额 | 27.53 |

**股票拆分：**

| 字段 | 说明 | 示例 |
|------|------|------|
| 行动类型 | split_adjustment | 拆股 |
| 拆股比例 | 拆分比例 | 2（1拆2） |

#### Step 5：查看持仓快照

- 在持仓概览页面查看：
  - 总资产（现金 + 持仓市值）
  - 总盈亏（已实现 + 未实现）
  - 个股持仓明细（数量、成本、市值、浮动盈亏）
  - 手续费和税费合计

#### Step 6：查看风险分析

在风险分析页面查看：
- **持仓集中度**：单只股票占总资产比例
- **行业集中度**：按行业分类的持仓分布
- **最大回撤**：历史最大亏损幅度
- **止损预警**：接近止损线的持仓提示

### 4.3 API 调用方式

#### 账户管理

```bash
# 创建账户
curl -X POST "http://localhost:8000/api/v1/portfolio/accounts" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "主账户",
    "broker": "华泰证券",
    "market": "cn",
    "base_currency": "CNY"
  }'

# 查看账户列表
curl "http://localhost:8000/api/v1/portfolio/accounts"

# 更新账户
curl -X PUT "http://localhost:8000/api/v1/portfolio/accounts/1" \
  -H "Content-Type: application/json" \
  -d '{"name": "A股主账户"}'

# 停用账户
curl -X DELETE "http://localhost:8000/api/v1/portfolio/accounts/1"
```

#### 交易记录

```bash
# 新增交易
curl -X POST "http://localhost:8000/api/v1/portfolio/trades" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": 1,
    "symbol": "600519",
    "trade_date": "2025-01-15",
    "side": "buy",
    "quantity": 100,
    "price": 1680.00,
    "fee": 5.00,
    "note": "买入贵州茅台"
  }'

# 查询交易记录
curl "http://localhost:8000/api/v1/portfolio/trades?account_id=1&page=1&page_size=20"

# 按日期范围查询
curl "http://localhost:8000/api/v1/portfolio/trades?account_id=1&date_from=2025-01-01&date_to=2025-03-01"

# 删除交易记录
curl -X DELETE "http://localhost:8000/api/v1/portfolio/trades/101"
```

#### 资金流水

```bash
# 新增资金记录
curl -X POST "http://localhost:8000/api/v1/portfolio/cash-ledger" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": 1,
    "event_date": "2025-01-15",
    "direction": "in",
    "amount": 500000,
    "currency": "CNY",
    "note": "初始入金"
  }'

# 查询资金流水
curl "http://localhost:8000/api/v1/portfolio/cash-ledger?account_id=1"
```

#### 公司行动

```bash
# 记录分红
curl -X POST "http://localhost:8000/api/v1/portfolio/corporate-actions" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": 1,
    "symbol": "600519",
    "effective_date": "2025-06-20",
    "action_type": "cash_dividend",
    "market": "cn",
    "currency": "CNY",
    "cash_dividend_per_share": 27.53
  }'
```

#### 持仓快照与风险

```bash
# 获取持仓快照
curl "http://localhost:8000/api/v1/portfolio/snapshot?account_id=1&cost_method=fifo"

# 获取风险分析报告
curl "http://localhost:8000/api/v1/portfolio/risk?account_id=1"
```

### 4.4 CSV 批量导入交易记录

系统支持从券商导出的 CSV 文件批量导入交易记录，流程如下：

#### Step 1：查看支持的券商

```bash
curl "http://localhost:8000/api/v1/portfolio/imports/csv/brokers"
```

#### Step 2：预览解析结果（试运行）

```bash
curl -X POST "http://localhost:8000/api/v1/portfolio/imports/csv/parse" \
  -F "broker=huatai" \
  -F "file=@trades.csv"
```

系统会将 CSV 解析为标准格式，返回解析后的交易列表供确认。

#### Step 3：确认导入

```bash
# 正式导入
curl -X POST "http://localhost:8000/api/v1/portfolio/imports/csv/commit" \
  -F "account_id=1" \
  -F "broker=huatai" \
  -F "dry_run=false" \
  -F "file=@trades.csv"

# 试运行（不写入数据库，仅校验）
curl -X POST "http://localhost:8000/api/v1/portfolio/imports/csv/commit" \
  -F "account_id=1" \
  -F "broker=huatai" \
  -F "dry_run=true" \
  -F "file=@trades.csv"
```

系统自动进行**去重处理**（基于交易 UID 和哈希值），避免重复导入。

### 4.5 支持的券商

| 券商 | broker 标识 | 别名 |
|------|------------|------|
| 华泰证券 | `huatai` | — |
| 中信证券 | `citic` | `zhongxin` |
| 招商证券 | `cmb` | `cmbchina`、`zhaoshang` |

### 4.6 风险分析

风险分析接口返回以下维度：

| 维度 | 说明 |
|------|------|
| 持仓集中度 | 单只股票市值占总资产比例 |
| 行业集中度 | 按行业分类的持仓占比 |
| 最大回撤 | 选定周期内的最大亏损幅度 |
| 止损预警 | 当前价格接近预设止损线的持仓 |

---

## 五、数据存储说明

所有功能数据统一存储在 SQLite 数据库中：

| 功能 | 数据库表 | 说明 |
|------|----------|------|
| 首页分析 | `analysis_history` | 分析历史记录和报告 |
| 回测 | `backtest_results`、`backtest_summaries` | 回测结果和汇总统计 |
| 问股 | `analysis_history`、`conversation_messages` | 分析历史和对话记录 |
| 持仓 | `portfolio_accounts`、`portfolio_trades`、`portfolio_positions` 等 | 账户、交易、持仓数据 |
| 通用 | `stock_daily`、`news_intel`、`fundamental_snapshot` | 行情、新闻、基本面数据 |

数据库文件位置：`data/stock_analysis.db`

查看方式：
- **图形化工具**：使用 [DB Browser for SQLite](https://sqlitebrowser.org/) 打开数据库文件
- **命令行**：`sqlite3 data/stock_analysis.db`
- **Python**：`import sqlite3; conn = sqlite3.connect("data/stock_analysis.db")`
