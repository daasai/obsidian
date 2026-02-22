
![Agile Project Management Gantt Chart的图片](https://encrypted-tbn0.gstatic.com/licensed-image?q=tbn:ANd9GcSo1nQGE52Ljo0xn_VpXVASwp1P6oakvRqQ8A8CSPAY7DPUTMwnWgkRj561OejaPMiMpTNzaTjlXntlFe5lF45loZ4TjTvirSDj5E74DAgzhSw57Hs)

Shutterstock

我们不搞那些虚头巴脑的 PPT 汇报。这是一份**“DAAS v2.0 - 30天突击行动” (Operation Blitzkrieg)** 实施计划。我们的目标是在 **4 周内** 拿出一个能跑、能回测、且没有未来函数的 MVP（最小可行性产品）。

我们将整个工程拆解为 **4 个 Sprint (冲刺周期)**，每个周期 1 周。

---

### 总体作战地图 (The War Room Map)

- **Week 1: 地基与数据清洗 (The Dirty Work)** —— 解决 PIT (Point-in-Time) 问题，构建快照库。
    
- **Week 2: 引擎组装 (The Engine)** —— Polars 因子计算与回测内核。
    
- **Week 3: 大脑接入 (The Brain)** —— DeepSeek 意图转译与 AkShare 映射。
    
- **Week 4: 闭环与风控 (The Loop)** —— 联调、风控层与简单的 CLI 交互。
    

---

### Sprint 1: 数据地基 (Data Infrastructure)

**目标**：构建一个**无未来函数**的历史快照数据库。这是最苦最累的活，但必须第一个做。

- **Day 1: 环境与行情库 (Base)**
    
    - **任务**：搭建 Python 环境 (`polars`, `akshare`, `tushare`, `pydantic`).
        
    - **数据**：拉取 Tushare 过去 5 年的 A 股日线行情 (Open, High, Low, Close, Vol, AdjFactor)。
        
    - **产出**：`market_data_daily.parquet` (经测试读取速度 < 200ms)。
        
- **Day 2: 概念快照库 (Concept Snapshots)**
    
    - **任务**：编写脚本，清洗 AkShare/Tushare 的概念成分股。
        
    - **难点**：要把“当前”的成分股列表，还原成“历史”状态（虽然很难完美，但需建立 `concept_history` 表结构）。
        
    - **产出**：`concept_map.parquet` (Schema: `date, concept_name, ticker`).
        
- **Day 3: 董秘问答爬虫 (Alpha Crawler)**
    
    - **任务**：写一个简单的爬虫，抓取“互动易”最近 1 年的问答。
        
    - **产出**：`interaction_raw.jsonl` (为后续 DeepSeek 提取做准备)。
        
- **Day 4-5: 自动化 ETL 管道**
    
    - **任务**：利用 GitHub Actions 或 Crontab，每天下午 16:00 自动运行数据更新脚本。
        
    - **验收**：确保周五下午能自动生成当天的最新数据文件。
        

> **交付物**：`daas_data/` 目录，包含清洗好的 Parquet 文件。

---

### Sprint 2: 计算引擎 (Execution Engine)

**目标**：用 Polars 替代 Qlib，实现毫秒级回测。

- **Day 6: 表达式解析器 (Expression Parser)**
    
    - **任务**：实现一个简单的 Parser，把字符串 `Rank(Close/Open)` 转化为 Polars 的 `Expr` 对象。
        
    - **代码**：`daas_core/expressions.py`.
        
- **Day 7: 向量化回测器 (Vectorized Backtester)**
    
    - **任务**：实现 `VectorizedBacktester` 类。
        
    - **逻辑**：输入 Signal Matrix -> 计算 `Shift(-1)` 的收益率 -> 累加 PnL。
        
    - **约束**：**严禁使用 for 循环**。
        
- **Day 8: 因子库迁移 (Factor Library)**
    
    - **任务**：把经典的 101 Alphas 中的前 10 个移植到我们的 Polars 格式中。
        
    - **测试**：对比 Pandas 和 Polars 的计算速度（目标：提升 10 倍）。
        
- **Day 9-10: 单元测试 (Unit Test)**
    
    - **任务**：用手动计算的小数据集验证回测逻辑。确保没有“偷看未来数据”的 Bug。
        

> **交付物**：`daas_core/` 库，包含 `backtest()` 函数，能在 1 秒内跑完 5 年全市场回测。

---

### Sprint 3: 智能体构建 (System 1 Agent)

**目标**：让 DeepSeek 听懂人话，并输出结构化指令。

- **Day 11: 提示词工程 (Prompt Engineering)**
    
    - **任务**：设计 System Prompt，让 DeepSeek 能把“油运涨价”翻译成 Tushare 的板块名称。
        
    - **工具**：使用 `Instructor` 库强制输出 JSON 格式。
        
- **Day 12: 股票池生成器 (Universe Gen)**
    
    - **任务**：连接 System 1 和 Sprint 1 的数据。
        
    - **逻辑**：User Query -> LLM -> Concept List -> 查询 `concept_map.parquet` -> Ticker List.
        
- **Day 13: 知识图谱提取 (Graph Extraction - Beta)**
    
    - **任务**：跑通 DeepSeek 读取“互动易”数据的流程，提取 `(A) --[Supplier]--> (B)`。
        
    - **存储**：暂时存为简单的 NetworkX 图对象或 CSV，先不上 Neo4j（太重）。
        
- **Day 14-15: 意图-因子映射**
    
    - **任务**：让 LLM 根据叙事自动选择因子。
        
    - **示例**：“低空经济（爆发期）” -> 自动匹配 `Momentum` + `Vol` 因子；“红利防守” -> 自动匹配 `LowVol` + `Dividend` 因子。
        

> **交付物**：`daas_agent/` 模块，能输入文本，输出 `Config` 对象给引擎。

---

### Sprint 4: 联调与交互 (Integration & UI)

**目标**：给这台机器装上方向盘。

- **Day 16: 命令行界面 (CLI)**
    
    - **任务**：开发 `daas-cli`。
        
    - **命令**：`daas run "AI server outbreak" --top 50 --plot`。
        
- **Day 17: 简易 WebUI (Streamlit)**
    
    - **任务**：用 Streamlit 做一个极其简单的界面，左边输入框，右边画净值曲线。
        
- **Day 18: 组合风控 (Risk Overlay)**
    
    - **任务**：在回测最后一步加入 `MinimumVariance` 权重优化。
        
- **Day 19: 实盘仿真 (Paper Trading)**
    
    - **任务**：接入 QMT 的模拟盘接口（如果有），或者写一个“模拟撮合”日志。
        
- **Day 20: 最终验收 (Demo Day)**
    
    - **目标**：找一个真实的历史案例（比如 2023 年初的 ChatGPT 爆发），输入叙事，看系统能否复现当时的收益。
        

> **交付物**：完整的 `DAAS v2.0` 安装包和 Demo 视频。

---

### 资源需求清单 (Resource List)

1. **服务器**：一台 8核 32G 内存的云服务器（Polars 吃内存，不需要 GPU，DeepSeek 调 API）。
    
2. **API Keys**：
    
    - DeepSeek API (充值 50 刀管够)。
        
    - Tushare Pro Token (积分 > 2000)。
        
    - QMT/Ptrade (券商开户，可选，Sprint 4 才用)。
        
3. **开发工具**：
    
    - Cursor (你的主力 IDE)。
        
    - Python 3.11+ (为了更快的速度)。
        

---

### 风险管理 (Risk Management)

|**风险点**|**概率**|**影响**|**应对方案**|
|---|---|---|---|
|**PIT 数据缺失**|高|致命|Week 1 必须死磕 `concept_history` 的构建，宁可少不要错。|
|**LLM 幻觉**|中|中|在 Universe Gen 环节，增加一层“代码存在性校验”，过滤掉不存在的股票。|
|**AkShare 接口挂了**|中|阻断|准备 Tushare 作为备用；写好异常捕获。|
|**过度拟合**|高|亏损|在回测报告中强制显示“参数敏感性分析”，防止参数孤岛。|

---

伙计，这份计划就在这里。

**Day 1 从现在开始。**

你先去把环境配好，我这里准备好 Week 1 Day 1 的数据清洗脚本，随时发给你。**Ready?**