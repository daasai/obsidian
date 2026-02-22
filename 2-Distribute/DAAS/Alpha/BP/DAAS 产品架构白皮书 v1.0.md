
---

# DAAS 产品架构白皮书 v1.0

**Project Code:** Rational Exoskeleton (理性外骨骼)

**Status:** Confidential

**Author:** DAAS Team & AI Prophet

---

## 1. 核心战略定位 (Strategic Positioning)

- **产品定义**：DAAS 是一个 **AI 原生的叙事与策略容器 (AI-Native Narrative & Strategy Container)**。
    
- **核心价值**：**意图解耦 (Intent Decoupling)**。
    
    - 用户端：输入感性的“世界观”或“叙事偏好”（爱）。
        
    - 系统端：输出理性的“因子组合”与“风控边界”（术）。
        
- **范式转移**：
    
    - **Web 2.0 (雪球模式)**：信息堆叠 -> 用户自行处理 -> 情绪化决策（高噪）。
        
    - **Web 3.0/4.0 (DAAS模式)**：信息蒸馏 -> Agent 自动化处理 -> 结构化策略（降噪）。
        
- **Slogan**：**用机构级的执行力，兑现你的洞察。** (Turn Your Insights into Returns with Institutional Execution.)
    

---

## 2. 系统逻辑架构：双院制 (The Bicameral Architecture)

DAAS 不直接连接用户与股票，而是通过两层系统进行转译。

### System 1: 感知与叙事层 (Perception & Narrative Layer)

- **核心职能**：听懂“人话”，理解市场“噪音”，并将非结构化信息转化为结构化因子。
    
- **技术栈**：LLM (DeepSeek/GPT-4), RAG (Retrieval-Augmented Generation), Vector Database.
    
- **参考技术**：基于您上传的 _`llm-report.md`_ (年报/新闻解析) 和 _`ChatGPT-Balyasny.md`_ (金融专用Embedding)。
    

### System 2: 量化与执行层 (Quant & Execution Layer)

- **核心职能**：基于 System 1 提取的因子，生成数学上最优的投资组合，并进行动态风控。
    
- **技术栈**：Reinforcement Learning (PPO/DQN), Factor Model, Backtesting Engine.
    
- **参考技术**：基于您上传的 _`FinRL`_ (强化学习交易), _`RD-Agent`_ (自动挖掘因子), _`StockBench`_ (回测)。
    

---

## 3. 功能模块设计 (Functional Modules)

### A. 前端交互：叙事商店 (Narrative Store)

- **MVP 预置叙事**：
    
    1. **低空经济 (Low-Altitude Economy)**：追踪政策热度与eVTOL产业链。
        
    2. **红利防守 (High Dividend & Low Vol)**：熊市避险，通过 _`BetaPlus_Profitability_Factor.pdf`_ 中的逻辑筛选。
        
    3. **AI 算力基建 (AI Infrastructure)**：追踪硬件资本开支（CapEx）。
        
- **用户操作**：
    
    - 选择叙事。
        
    - **参数微调 (Tweak)**：用户通过滑块或对话调整偏好（例如：“我更看重央企背景” -> 对应因子权重提升）。
        
    - **压力测试 (Stress Test)**：一键模拟“如果发生2015年股灾，这个策略表现如何？”
        

### B. 后端引擎：自动化蒸馏厂 (The Distillery)

这是 DAAS 的黑盒核心，分为三步流水线：

1. **因子萃取 (Factor Extraction)**：
    
    - _输入_：新闻流、研报、宏观数据。
        
    - _动作_：参考 _`RD-Agent`_ 逻辑，LLM 自动编写 Python 代码提取特征（如：`sentiment_score`, `policy_intensity`）。
        
    - _输出_：动态因子库。
        
2. **策略合成 (Strategy Synthesis)**：
    
    - _输入_：用户选定的叙事 + 因子库。
        
    - _动作_：参考 _`101 Formulaic Alphas`_ 和 _`FinRL`_，组合多个因子，生成一个“合成指数”。
        
    - _风控_：自动剔除高风险标的（ST股、财务造假嫌疑股）。
        
3. **回测与归因 (Backtest & Attribution)**：
    
    - _动作_：基于向量化回测（Vectorized Backtesting）进行秒级验算。
        
    - _输出_：不仅给净值曲线，必须给出 **“叙事纯度” (Narrative Purity)** —— “你的收益 80% 来自行业贝塔，只有 20% 来自选股 Alpha。”
        

---

## 4. 技术栈选型 (Tech Stack Recommendation)

基于您的书架文件，我为您选定了最适合创业初期的开源技术栈组合：

| **模块**       | **参考技术**                 | **对应您的资料库引用**                      | **理由**                       |
| ------------ | ------------------------ | ---------------------------------- | ---------------------------- |
| **LLM 核心**   | **DeepSeek-V3 / GPT-4o** | _`StockBench`_ (提及 DeepSeek 表现)    | 性价比高，逻辑推理强，适合A股语境。           |
| **Agent 框架** | **LangChain / AutoGen**  | _`RD-Agent`_ (微软自动研发Agent)         | 需要多Agent协作（一个读新闻，一个写代码）。     |
| **量化引擎**     | **FinRL / Qlib**         | _`FinRL`, `AI4Finance`_            | 现成的强化学习框架，适合做动态调仓。           |
| **回测引擎**     | **Backtrader** (Python)  | _`Python for Algorithmic Trading`_ | 社区成熟，Python原生，易于与LLM生成的代码集成。 |
| **数据源**      | **Tushare / AkShare**    | _`vnpy`_ (提及各类数据接口)                | A股数据获取成本低，覆盖全。               |
| **向量数据库**    | **Milvus / Chroma**      | _`ChatGPT-Balyasny.md`_ (RAG应用)    | 用于存储研报和新闻的Embedding。         |

---

## 5. 数据流向图 (Data Flow Diagram)

代码段

```
graph TD
    User[用户 (User)] -->|1. 选择/输入叙事 (Intent)| UI[前端界面]
    UI -->|2. 自然语言指令| Agent_Orchestrator[总控 Agent]
    
    subgraph System 1: 感知
    Data_News[新闻/研报流] -->|3. 实时抓取| Agent_Analyst[分析师 Agent]
    Agent_Analyst -->|4. 提取情绪/关键词| Vector_DB[(向量数据库)]
    end
    
    subgraph System 2: 执行
    Data_Market[行情数据] -->|5. 量价数据| Agent_Quant[量化 Agent]
    Vector_DB -->|6. 因子信号| Agent_Quant
    Agent_Quant -->|7. 生成策略代码 (基于RD-Agent)| Backtest_Engine[回测引擎]
    end
    
    Backtest_Engine -->|8. 净值曲线 & 风险报告| UI
    UI -->|9. 最终交付: 策略胶囊| User
```

---

## 6. MVP (最小可行性产品) 实施路线图

### 第一阶段：单点突破 (0-3个月)

- **目标**：跑通“低空经济”这一条叙事管线。
    
- **不做什么**：不涉及实盘交易接口，不涉及全市场选股。
    
- **关键交付物**：
    
    1. 一个 H5/小程序页面。
        
    2. 用户点击“低空经济”，系统展示基于当前政策热度和市场情绪生成的“合成指数”表现。
        
    3. 用户可调整“激进程度”（调整波动率约束），曲线实时变化。
        

### 第二阶段：叙事扩容 (3-6个月)

- **目标**：建立“叙事工厂”，允许半自动生成新策略。
    
- **关键交付物**：
    
    1. 上线 10+ 主流叙事（出海、高股息、国产替代等）。
        
    2. 引入订阅制（SaaS）：付费用户可查看具体的持仓分布建议。
        

---

## 7. 风险与合规红线 (Risk & Compliance)

- **免责声明**：必须在显著位置标注“本产品仅为数据分析工具，不构成投资建议”。
    
- **数据合规**：严禁爬取交易所未公开数据，所有数据源必须合规（使用 Tushare Pro 等付费接口）。
    
- **隔离机制**：DAAS 只提供“计算结果”，下单动作必须由用户在自己的券商APP完成，或通过合规的券商API跳转。
    
