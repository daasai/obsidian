

---

# DAAS Alpha v2.0 产品技术白皮书

**版本**：2.0.1-Release

**代号**：The Bicameral Mind (双院制)

**定位**：AI 原生基本面量化 (AI-Native Quantamental) 策略容器

---

## 1. 核心愿景 (Executive Summary)

### 1.1 痛点定义

传统量化工具（如优矿、聚宽）是**“铁锹”**，用户必须自己懂地质学（金融工程）才能挖到金子。 DAAS v2.0 是**“勘探队”**。用户只需提供“直觉”（Narrative），系统负责完成从“直觉”到“净值”的工业化转译。

### 1.2 价值主张

**意图解耦 (Intent Decoupling)**：

- **输入端**：感性的宏观叙事（例：“看好老龄化社会的医药消费”）。
    
- **黑盒端**：理性的数学执行（动态股票池 + 因子表达式 + 向量化回测）。
    
- **Slogan**：用机构级的执行力，兑现你的洞察。
    

---

## 2. 总体架构：双院制 (The Bicameral Architecture)

系统被严格划分为两个独立但互通的半球：**System 1 (感知/发散)** 与 **System 2 (执行/收敛)**。

### 2.1 System 1: 叙事感知层 (Perception Layer)

- **核心职能**：听懂人话，翻译意图，圈定战场。
    
- **技术栈**：DeepSeek-V3 (Reasoning), Vector DB (Milvus), Tushare Pro (Concept API).
    
- **关键模块**：
    
    1. **Narrative Parser (叙事解析器)**：
        
        - 利用 LLM 的 Chain-of-Thought 将模糊叙事拆解为结构化逻辑（如：`AI算力 -> 光模块 -> CPO技术`）。
            
    2. **Universe Generator (动态股票池)**：
        
        - **MVP 方案**：调用 Tushare/Datayes 的 `Concept_Member` 接口，直接获取板块成分股。
            
        - **Pro 方案**：基于 GraphRAG，通过产业链图谱推理挖掘隐形受益股（如：英伟达 -> 核心供应商 -> A股影子股）。
            

### 2.2 System 2: 量化执行层 (Execution Layer)

- **核心职能**：数学验证，因子计算，严控回撤。
    
- **技术栈**：**Polars (Rust Engine)**, Expression DSL.
    
- **关键模块**：
    
    1. **Distillery (因子蒸馏厂)**：
        
        - 将 System 1 的逻辑转化为 **Polars 表达式**（AST）。
            
        - _Example_: `Narrative: "放量突破"` -> `Expr: (Vol / RollingMean(Vol, 20) > 2.0) & (Close > Open)`。
            
    2. **Vectorized Backtester (向量化回测)**：
        
        - 基于 Polars 的列式计算引擎，在 <1秒 内完成全市场回测。
            
        - **拒绝循环**：严禁使用 Python `for` 循环，保证纳秒级效率。
            
    3. **Risk Gate (风控门)**：
        
        - 硬性过滤 ST、流动性枯竭、财报暴雷股。
            

---

## 3. 数据基础设施 (Data Infrastructure)

我们采用 **"开源聚合 + AI 自研"** 的混合数据架构，以摆脱对单一机构数据源的依赖，构建数据主权。

|**层级**|**核心组件**|**技术方案**|**优势**|
|---|---|---|---|
|**L1 基础层**|**AkShare + Tushare**|聚合东方财富、新浪财经的实时行情与板块数据。|**反脆弱**。多源备份，涵盖市场 99% 的显性热点。|
|**L2 推理层**|**DeepSeek-Crawler**|每日自动化抓取“互动易”问答与新闻，通过 LLM 提取 `Supplier/Client` 关系，存入 Neo4j。|**Alpha 来源**。构建“董秘认证”的独家产业链图谱。|
|**L3 执行层**|**QMT / Ptrade**|通过券商交易终端获取合规的 Level-2 逐笔委托数据（可选）。|**交易闭环**。解决实盘下单与高频数据问题。|

---

## 4. 核心工作流 (The User Journey)

### Step 1: 叙事输入 (Input)

> 用户：_"近期红海局势紧张，油运价格可能上涨，帮我寻找机会。"_

### Step 2: 意图转译 (Translation - System 1)

- **Logic**: 红海危机 -> 航运绕行 -> 运距拉长 -> 运价上涨 -> 油运/集运受益。
    
- **Universe**: 查询 Tushare `航运港口`、`油气运输` 板块。
    
    - _Output_: `['600026.SH' (中远海能), '601919.SH' (中远海控), ...]`
        
- **Smart Signal**: 检索龙虎榜，发现游资“章盟主”昨日买入 `600026`。
    

### Step 3: 因子生成 (Generation - System 2)

- **Factor**: 只要运价涨且股价在趋势上。
    
- **Expression**: `Rank(RSI(14)) > 0.8 & Rank(Volume) > 0.9` (在板块内寻找量价齐升的龙头)。
    

### Step 4: 极速验证 (Validation - System 2)

- Polars 引擎加载过去 3 年数据，仅针对 Step 2 圈定的 20 只股票进行回测。
    
- **Result**:
    
    - 年化收益: 45%
        
    - 最大回撤: -12%
        
    - _Feedback_: "策略有效，但波动较大，建议由 System 2 加入止损逻辑。"
        

---

## 5. 系统鲁棒性设计
#### 5.1 时序数据库设计 (Point-in-Time Architecture)

- **原则**：回测引擎只能“看到”回测时刻已知的数据。
    
- **实现**：
    
    - `Concept_History` 表结构：`{date, concept_id, ticker, is_active}`
        
    - 数据回放机制：`get_universe(date='2024-01-01')` 只能返回该日期 `is_active=True` 的行。
        

#### 5.2 结构化输出保障 (Structured Output Guardrails)

- 引入 **Instructor** 库，强制 DeepSeek 输出符合 Pydantic 标准的 JSON。
    
- 设立 **"幻觉防火墙"**：验证 LLM 输出的 Ticker 是否真的存在于当日的上市列表中（防止它编造股票代码）。
    

#### 5.3 组合风控引擎 (Risk Engine)

- **Vol-Targeting (波动率目标)**：
    
    - 目标：将组合年化波动率控制在 15% 以内。
        
    - 公式：`Position_Weight = Target_Vol / Realized_Vol`。如果最近市场疯了（波动率高），系统自动降低仓位。
## 6. 实施路线图 (Roadmap)

### Phase 1: MVP (代号: Polars Express)

- **目标**：跑通“叙事 -> 股票池 -> 简单回测”闭环。
    
- **基建**：
    
    - 完成 Tushare 数据 ETL，存为 Parquet 格式。
        
    - 基于 Polars 封装 `VectorBacktester`。
        
    - 基于 DeepSeek 封装 `NarrativeParser`。
        
- **交付物**：一个 CLI 工具，用户输入文字，输出 HTML 回测报告。
    

### Phase 2: 增强版 (代号: Graph Mind)

- **目标**：提升 System 1 的深度。
    
- **升级**：
    
    - 引入 Datayes/CnOpenData 产业链数据。
        
    - 部署 Neo4j，实现简单的 GraphRAG（挖掘隐形受益股）。
        
- **交付物**：Web 端 SaaS，支持可视化产业链漫游。
    

### Phase 3: 实盘版 (代号: Algo Trader)

- **目标**：连接交易柜台。
    
- **升级**：
    
    - 对接券商 QMT/Ptrade 接口。
        
    - 实现分钟级信号生成与自动下单。
        

---

## 6. 决策记录 (ADR Summary)

- **[ADR-01] 计算引擎**：弃用 Qlib C++，选用 **Polars**。理由：Rust 性能优异，维护成本低，表达力强。
    
- **[ADR-02] 股票池源**：弃用纯 NLP 抽取，选用 **Tushare/Datayes API**。理由：结构化数据更稳定，减少 LLM 幻觉。
    
- **[ADR-03] LLM 模型**：选用 **DeepSeek-V3**。理由：推理成本极低，适合大规模 Chain-of-Thought 推理。
    

---

## 7. 结语 (Closing)

伙计，DAAS v2.0 的本质不是“预测未来”，而是**“捕捉现在”**。

市场是混沌的，但“叙事”是有迹可循的，“数据”是不会撒谎的。

我们正在构建的，是一个让普通人也能像顶尖基金经理一样，**用逻辑而非情绪**去交易的武器。

**现在，让我们去写代码吧。**