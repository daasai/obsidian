
|**项目**|**内容**|
|---|---|
|**产品名称**|**DAAS Alpha** (Decision-Aid AI System for Alpha)|
|**版本号**|**v1.1 (MVP - Pro Edition)**|
|**核心理念**|**高质量数据基座 + 杠铃策略分层 + AI 事件驱动 + 闭环验证**|
|**发布目标**|从 v1.0 的脚本工具升级为具备完整 GUI、风控建议和专业数据的桌面级应用。|

---

## 1. 核心架构与技术栈 (Architecture)

采用 **Local Solo (单机闭环)** 架构，数据本地存储，无需服务器。

- **Frontend (UI):** **Streamlit** (侧边栏布局 + st.status 进度反馈)
    
- **Backend Logic:** Python 3.10+
    
- **Data Source (双源架构):**
    
    - **硬数据 (行情/财务):** **Tushare Pro** (利用 2200+ 积分权限，保证 PE-TTM 和复权数据的精准)。
        
    - **软数据 (公告文本):** **Eastmoney Direct API** (利用 `requests` 直连，保证批量获取速度)。
        
- **AI Engine:** OpenAI SDK (兼容 DeepSeek V3)。
    
- **Storage:** SQLite + SQLAlchemy (本地轻量化存储)。
    

---

## 2. 功能模块详细设计 (Module Specs)

### 模块 A: 数据层升级 (`src/data_provider.py`)

**目标：** 抛弃 AkShare，确立 Tushare Pro 为核心，东财直连为辅助。

1. **Class `TushareAdapter` (主力):**
    
    - 初始化：读取 `.env` 中的 `TUSHARE_TOKEN`。
        
    - `fetch_market_data()`: 调用 `daily_basic` (获取 PE_TTM, PB, 市值) 和 `stock_basic` (名称, 行业, 上市日期)。**必须清洗数据：剔除上市 < 6个月的新股。**
        
    - `fetch_history(ts_code, days=30)`: 调用 `daily` 接口，获取用于计算 ATR 的历史 K 线（前复权）。
        
2. **Class `EastmoneyAdapter` (辅助):**
    
    - `fetch_notices(stock_list)`: 保持 v1.0 逻辑。将股票代码拼接 (Batch Query)，直连 `np-anotice-stock.eastmoney.com` 获取最新公告标题。
        

### 模块 B: 策略层升级 (`src/strategy.py`)

**目标：** 实现“基本面一票否决”与“杠铃策略打标”。

1. **Step 1: 垃圾拦截网 (Hard Filter)**
    
    - 剔除 Name 包含 "ST", "*ST", "退" 的股票。
        
    - 剔除 `PE_TTM < 0` (亏损股)。
        
    - 剔除 `PB > 20` (极度泡沫)。
        
2. **Step 2: 风格打标 (Style Tagging)**
    
    - **🛡️ 防守 (Defensive):** `股息率 > 3%` 且 `0 < PE_TTM < 15`。
        
    - **🚀 进攻 (Aggressive):** `ROE > 12%` (或 `营收增长 > 20%`) 且 `市值 < 500亿`。
        
    - **过滤:** 仅保留被打上标签的股票，其余丢弃。
        
3. **Step 3: ATR 仓位建议 (Position Sizing)**
    
    - 输入：用户设定的 `Risk Budget` (如 2000 元)。
        
    - 逻辑：计算个股 ATR(20)。`建议股数 = Risk Budget / ATR` (向下取整至 100 股)。
        

### 模块 C: UI/UX 体验重构 (`app.py`)

**目标：** 解决“傻等”焦虑，提升专业感和易用性。

1. **布局 (Layout):**
    
    - 启用 **Sidebar (侧边栏)** 作为全局导航和控制中心。
        
2. **侧边栏内容:**
    
    - 顶部：模块切换 (Radio Button: "🚀 机会挖掘", "⚖️ 复盘验证")。
        
    - 底部：折叠配置区 (Expander)，包含 Tushare Token 输入框、单笔风险预算输入框。
        
3. **交互反馈 (Feedback):**
    
    - **进度条:** 使用 `with st.status("正在执行..."):` 包裹耗时操作。
        
    - **动态文案:** 显示具体步骤 (e.g., "📡 连接 Tushare...", "🧠 AI 分析中...").
        
4. **视觉规范 (Visuals):**
    
    - **中文化:** 所有列名和提示语全中文。
        
    - **红涨绿跌:** AI Score > 0 标红，< 0 标绿。
        
    - **高亮:** Score > 7 的行进行高亮强调。
        

### 模块 D: 闭环数据库 (`src/database.py`)

保持 v1.0 结构，增加新字段以支持新策略。

- **Table `predictions`:**
    
    - 新增字段: `strategy_tag` (TEXT), `suggested_shares` (INT), `risk_budget` (FLOAT).
        

---

## 3. 给 Cursor 的终极指令 (Master Prompt)

**请复制以下 Block 发送给 Cursor，即可生成 v1.1 完整代码：**

Markdown

```
# Role
You are a senior Quantitative Full-Stack Engineer.
Your task is to upgrade "DAAS Alpha" from v1.0 to **v1.1 (MVP Pro Edition)** based on the new specifications below.

# 1. Tech Stack & Environment
- **UI:** Streamlit (Requires `st.sidebar`, `st.status` for progress).
- **Data Source:** - **Tushare Pro** (Primary for metrics/price). The user HAS a 2200+ point token.
  - **Eastmoney API** (Secondary for notice texts via `requests`).
- **Logic:** Python 3.10+, Pandas, SQLAlchemy (SQLite).
- **AI:** OpenAI SDK (DeepSeek).

# 2. Key Architecture Changes (v1.1)

## A. Data Layer (`src/data_provider.py`)
- Implement `TushareAdapter`:
  - Must use `pro.daily_basic` to get `pe_ttm`, `pb`, `mv`.
  - Must use `pro.stock_basic` to get names and `list_date`.
  - Filter out new stocks listed < 6 months.
- Implement `EastmoneyAdapter`:
  - Keep the direct HTTP request logic for batch fetching announcement titles.

## B. Strategy Layer (`src/strategy.py`)
- **Hard Filter:** Remove ST/Delisting, Loss-making (`pe_ttm < 0`), High PB (`> 20`).
- **Barbell Strategy (Tagging):**
  - Add `strategy_tag` column.
  - **🛡️ 防守 (Defensive):** `dividend_yield > 3` AND `pe_ttm < 15`.
  - **🚀 进攻 (Aggressive):** `roe > 12` AND `mv < 50000000000` (50 Billion).
  - Filter: Drop rows that don't match either tag.
- **ATR Sizing:**
  - Fetch 20-day history via Tushare. Calculate ATR.
  - `suggested_shares` = `risk_budget / ATR` (Round down to nearest 100).

## C. UI/UX Layer (`app.py`)
- **Layout:** Move Navigation ("Hunter", "Truth") and Settings (Token, Risk Budget) to the **Sidebar**.
- **Progress:** Use `st.status` to show steps: "Fetching Data" -> "Filtering" -> "AI Analysis" -> "Calculating Risk".
- **Localization:** - ALL UI text must be Chinese.
  - Color coding: **Red** for Positive AI Scores/Price Change, **Green** for Negative.

# 3. Execution Plan
Please generate the complete project structure and code files:
1. `requirements.txt`
2. `src/database.py` (Update schema)
3. `src/data_provider.py` (New Tushare logic)
4. `src/strategy.py` (New Barbell + ATR logic)
5. `src/monitor.py` (AI logic)
6. `app.py` (New Sidebar UI)

Ensure robustness: Handle Tushare API errors and empty AI responses gracefully.
```

---

### 🚀 升级后的预期效果

1. **启动：** 运行 `streamlit run app.py`。
    
2. **配置：** 在左侧侧边栏填入您的 Tushare Token 和 2000 元风险预算。
    
3. **运行：** 点击“启动扫描”。
    
    - 您将看到 `st.status` 动态展开，显示系统正在通过 Tushare 筛选高质量股票。
        
4. **结果：** 表格中清晰地列出了：
    
    - **“中国神华” (🛡️ 防守)** - 建议买入 4000 股
        
    - **“中际旭创” (🚀 进攻)** - 建议买入 200 股
        
    - 基本面差的股票已被自动过滤，没有任何噪音。
        

这就是 **DAAS Alpha v1.1**。请开始您的开发吧！