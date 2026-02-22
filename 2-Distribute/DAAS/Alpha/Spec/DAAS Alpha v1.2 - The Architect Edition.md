这是一份经过深度整合的 **DAAS Alpha v1.2 (The Architect Edition) 产品规格说明书**。

这一版本不仅仅是功能的叠加，更是**系统架构的质变**。我们引入了设计模式（Strategy Pattern）来实现因子的解耦，并增加了回测模块（Backtesting），使您的系统具备了自我验证和进化的能力。

您可以按顺序将底部的提示词发送给 Cursor，完成从 v1.1 到 v1.2 的平滑升级。

---


| **维度**   | **内容**                                                                       |
| -------- | ---------------------------------------------------------------------------- |
| **版本代号** | **v1.2 (The Architect)**                                                     |
| **核心目标** | **架构解耦 + 胜率增强 + 历史回测**                                                       |
| **策略逻辑** | **Alpha 三叉戟** (RPS强动量 + PEG高性价比 + 量比资金流) + **MA20趋势防御**                      |
| **技术架构** | Python, Streamlit, Pandas (Vectorized), **Strategy Pattern (Factor Engine)** |
|          |                                                                              |

## 1. 系统目录结构 (Directory Structure)

v1.2 最大的变化在于 `src/factors/` 的模块化。

Plaintext

```
daas_alpha/
├── data/               # 存放 daas.db 和 cache.csv
├── src/
│   ├── factors/        # [NEW] 因子工厂 (设计模式)
│   │   ├── __init__.py
│   │   ├── base.py     # 抽象基类 BaseFactor
│   │   ├── momentum.py # RPS 因子
│   │   ├── technical.py# MA, VolumeRatio 因子
│   │   └── engine.py   # FactorPipeline 执行引擎
│   ├── data_provider.py# Tushare Pro (主力) + Eastmoney (公告)
│   ├── strategy.py     # [Refactor] 只负责选股规则，不负责计算
│   ├── backtest.py     # [NEW] 向量化回测引擎
│   ├── database.py     # DB 模型
│   └── monitor.py      # AI 分析 (保持不变)
├── app.py              # Streamlit 入口 (侧边栏 + 3大模块)
└── requirements.txt
```

## 2. 核心模块详述

### 模块 A: 因子工厂 (`src/factors/`)

- **设计模式:** 策略模式 (Strategy Pattern)。
    
- **BaseFactor:** 定义 `compute(df) -> df` 接口。
    
- **Concrete Factors:**
    
    - `RPSFactor(window=60)`: 计算全市场排名 (0-100)。
        
    - `MAFactor(window=20)`: 计算均线及乖离率。
        
    - `VolumeRatioFactor(window=5)`: 计算量比。
        
    - `PEFilterFactor(max_pe=30)`: 标记低估值状态。
        

### 模块 B: 策略层 (`src/strategy.py`)

- **输入:** 经过因子工厂处理后的 `enriched_df`。
    
- **逻辑 (Alpha Trident):**
    
    1. **RPS > 85**: 强者恒强。
        
    2. **0 < PE < 30**: 规避泡沫与亏损 (PEG Proxy)。
        
    3. **Vol_Ratio > 1.5**: 资金进场确认。
        
    4. **Close > MA20**: 趋势多头保护。
        
- **输出:** 排序后的选股列表。
    

### 模块 C: 时光机 (`src/backtest.py`)

- **原理:** Pandas 向量化回测 (Vectorized Backtesting)。
    
- **逻辑:**
    
    - T日触发 `Strong Buy` 信号。
        
    - T+1日 开盘买入。
        
    - 持有 `N` 天 (默认5天) 后收盘卖出。
        
- **指标:** 策略总收益 vs 沪深300收益、胜率 (Win Rate)。
    

### 模块 D: UI 升级 (`app.py`)

- **Sidebar:** 增加 "⏳ 时光机 (Backtest)" 选项。
    
- **Hunter Tab:** 展示详细因子数据 (RPS, 量比等列)。
    
- **Backtest Tab:** 包含日期选择器、参数滑块、收益曲线图。
    

---

# 🚀 给 Cursor 的构建提示词列表 (Step-by-Step)

为了保证代码质量，建议**分步**发送以下 Prompt。不要一次性发完，每完成一步测试通过后再发下一步。

### Step 1: 基础设施与因子架构 (The Foundation)

Markdown

```
# Role
You are a Software Architect for a Quantitative System.
We are upgrading to DAAS Alpha v1.2 with a decoupled architecture.

# Task
Create the directory structure and the Factor Engine using the **Strategy Design Pattern**.

# Action 1: Create `src/factors/`
1. `src/factors/base.py`:
   - Abstract class `BaseFactor` with method `compute(self, df) -> df` and `name(self) -> str`.
2. `src/factors/momentum.py`:
   - Class `RPSFactor(window=60)`. Logic: Group by `trade_date` to rank `pct_change`.
3. `src/factors/technical.py`:
   - Class `MAFactor(window=20)`. Logic: Calc MA and boolean `above_ma`.
   - Class `VolumeRatioFactor(window=5)`. Logic: `vol / rolling_mean(vol)`.
4. `src/factors/fundamental.py`:
   - Class `PEProxyFactor(max_pe=30)`. Logic: boolean `is_undervalued` if `0 < pe_ttm < max_pe`.

# Action 2: Create `src/factors/engine.py`
- Class `FactorPipeline`.
- Method `add(factor)`.
- Method `run(df)`: Sequentially execute all factors.

# Constraint
Use Pandas vectorized operations strictly. No loops over rows.
```

### Step 2: 策略层重构 (The Logic)

Markdown

```
# Context
We have the `FactorPipeline` ready. Now we need to implement the "Alpha Trident" strategy.

# Task
Refactor `src/strategy.py`.

# Specs
Class `AlphaStrategy`:
- Init: Accepts `enriched_df` (which already has factor columns).
- Method `filter_alpha_trident()`:
  - Filter logic:
    1. `rps_60 > 85` (Momentum)
    2. `is_undervalued == True` (Value)
    3. `vol_ratio_5 > 1.5` (Liquidity)
    4. `above_ma_20 == True` (Trend)
  - Return: DataFrame sorted by `rps_60` descending.
  - Columns to keep in output: `ts_code`, `name`, `close`, `pe_ttm`, `rps_60`, `vol_ratio_5`, `strategy_tag` (Set to "🚀 强推荐").

# Note
This module should NOT calculate indicators. It only filters based on existing columns.
```

### Step 3: 数据层与回测引擎 (The Time Machine)

Markdown

```
# Context
We need Tushare Pro data and a Backtesting engine.

# Task 1: Update `src/data_provider.py`
- Ensure `TushareAdapter` fetches `pe_ttm`, `vol`, `close` correctly.
- Add `fetch_history_batch(start_date, end_date)`: Fetches daily data for ALL stocks (or CSI300 constituents) for backtesting. *Optimization: Implement caching to `data/cache.csv`.*

# Task 2: Create `src/backtest.py`
- Class `VectorBacktester`.
- Method `run(df, holding_days=5)`:
  1. Generate `buy_signal` (using the same logic as `AlphaStrategy` but vectorized for the whole timeline).
  2. Calculate returns: Buy at T+1 Open, Sell at T+1+N Close.
  3. Compute: `Win Rate`, `Total Return`, `Max Drawdown`.
  4. Compare against Benchmark (fetch CSI300 data).

# Constraint
Keep the backtest logic simple and fast using Pandas.
```

### Step 4: UI 组装 (The Assembly)

Markdown

```
# Context
All backend modules are ready. Now assemble the UI in `app.py`.

# Task
Rebuild `app.py` with Streamlit.

# UI Structure (Sidebar Navigation)
1. **Sidebar:**
   - Title: "🛰️ DAAS Alpha v1.2".
   - Navigation: Radio ["🚀 机会挖掘 (Hunter)", "⏳ 时光机 (Backtest)", "⚖️ 复盘验证 (Truth)"].
   - Settings Expander: Tushare Token input.

# Tab 1: Hunter
- Button "启动扫描".
- Process:
  - `st.status` showing progress steps.
  - `TushareAdapter` -> `FactorPipeline` -> `AlphaStrategy`.
- Display: DataFrame with columns [代码, 名称, 现价, RPS强度, 量比, PE(TTM), 建议]. Highlight RPS > 90.

# Tab 2: Backtest
- Inputs: Date Range, Holding Days Slider (1-20).
- Button "开始验证".
- Process: `VectorBacktester.run()`.
- Display:
  - 3 Metrics: 策略收益 vs 基准, 胜率, 交易次数.
  - Line Chart: Equity Curve.

# Localization
Ensure ALL text is in Chinese. Use Red for Up/Positive, Green for Down/Negative.
```

---

### 💡 迭代建议

1. **先跑通 Step 1 & 2:** 确保因子计算和筛选逻辑没错，这步不需要 UI 也能用 Jupyter 或简单的 Print 测试。
    
2. **回测数据量:** 在 Step 3 中，建议先只拉取 **沪深300成分股** 的历史数据进行测试，跑通后再尝试全市场，否则第一次拉取 5000 只股票的一年数据会非常慢（可能需要几分钟）。
    
3. **Token:** 确保 `.env` 文件或 Sidebar 中填入了您的 Tushare Token。
    

祝您的 v1.2 迭代顺利！这将是一个质的飞跃。