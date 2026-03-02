
# 📄 《注意力量化套利系统 (AQAS) 产品设计与架构白皮书 V3.1》

**—— 记忆、归因与回测中枢 (Memory, Attribution & Backtest Hub)**

**文档状态：** 🟢 架构定稿 (Architecture Finalized & Ready for Dev)

**核心模块：** `MemoryBank` (存储底座), `FactorEngine` (归因引擎), `BacktestOracle` (回测判官)

**设计思想：** 领域驱动设计 (DDD) + 三重存储架构 + 人机协同 (Human-in-the-loop) + 历史胜率回测

---

## 1. 宏观产品哲学与系统定位 (Product Philosophy)

AQAS 并非市面上常见的“AI 爆款文案生成器”，而是一个**在完全零和博弈的社交媒体环境中，以“极低成本获取高质量人类注意力”为唯一目标的自主化高频套利机器**。

- **核心范式**：从 Instruction-following（大模型写什么我发什么）跃迁至 **Goal-oriented Autonomy（目标导向的自主进化）**。
    
- **商业护城河**：系统的核心资产不是生成的单篇爆款内容，而是沉淀在多维数据库中、随着时间推移在真实市场中通过“实弹射击”被数学验证过的**“模因特征集（MemeGene）”**与动态更新的**“胜率模型”**。
    

---

## 2. 全局领域实体模型 (Global Domain Entity Model)

系统基于领域驱动设计（DDD），构建了连接宏观环境、物理内容与高维逻辑的四层实体网状结构。实体命名严格遵循代码级规范。

### 2.1 宏观环境层 (Meta-Environment)

定义了套利的“赌场”和基本面。

- **`Track` (赛道)**
    
    - **核心属性**：`track_id` (如: `money_making`), `name` (如: 搞钱副业), `description`。
        
    - **动态指标**：`baseline_metrics` (动态更新的大盘平均点赞量 $\mu$ 与标准差 $\sigma$、赛道实时赞藏比及平台风控严苛度)。
        

### 2.2 作战单元层 (Players & Agents)

分为靶机（敌军）与舰队（我方）。

- **`Account` (账号基类)**
    
    - **通用属性**：`account_id` (平台 UID), `sec_uid`, `nickname`, `avatar`, `desc` (简介), `tags`, `followers_count`。
        
    - **`BenchmarkAccount` (对标靶机)**：外挂起号史、生命周期（起号/爆发/洗粉变现）及核心爆款率，用于提取**纵向 Alpha 因子**。
        
    - **`MatrixAccount` (己方舰队)**：外挂账号健康度（限流状态）、历史违规记录、当前分发的受众画像。
        

### 2.3 物理弹药层 (Physical Assets)

这是所有量化因果关系的最终结算凭证。`Post` 作为聚合根（Aggregate Root），下辖动态数据与评论。

- **`Post` (内容聚合根)**
    
    - **基础元数据**：`post_id`, `author_id` (关联 Account), `track_id` (所属赛道), `publish_timestamp`, `post_type` (图文/视频)。
        
    - **内容原矿**：`title`, `raw_desc` (正文原文本), `cover_image_url`, `ocr_text` (封面提取的大字报文本)。
        
    - **`MetricsSnapshot` (指标快照)**：`fetch_timestamp`, `likes`, `collects` (收藏), `shares`。用于计算真实的动态赞藏比。
        
- **`Comment` (评论子实体 - 语义归因核燃料)**
    
    - **核心属性**：`comment_id`, `post_id` (归属笔记), `content`, `like_count`, `publish_timestamp`。
        
    - **战术作用**：提取 Top 50 高赞评论，投喂给大模型进行情绪聚类，完成因子的语义归因。
        

### 2.4 高维智慧层 (Intelligence & Logic)

从物理世界抽象出的“量化因子”与“交易策略”。

- **`MemeGene` (模因基因 / 注意力因子)**
    
    - **核心属性**：`gene_id`, `track_id` (**强绑定**：基因具有赛道隔离性), `win_rate` (历史胜率), `attention_factors` (视觉/情绪/身份特征字典)。
        
    - **特性**：若需跨界变异（如职场因子平移至美妆），系统将在目标赛道克隆生成全新基因，重新计算胜率。
        
- **`Strategy` (执行策略)**
    
    - **核心属性**：`strategy_id`, `target_track_id`, `used_genes` (引用的基因 ID 列表), `generated_content` (生成的 HTML 模板与文本配置)。
        

---

## 3. 三重存储架构 (Triple-Engine Architecture)

为解决复杂关系映射与高维语义检索的冲突，系统采用“三位一体”底层存储方案：

1. **L1 关系与算力引擎 (SQLite / PostgreSQL)**：
    
    - **职责**：存储 `Track`, `Account`, `Post` 实体间的硬性关联和标量数值。
        
    - **场景**：执行冷酷的统计学计算。例如：毫秒级算出某对标账号近 30 天的 $\mu + 1.5\sigma$ 爆款绝对线。
        
2. **L2 语义检索引擎 (ChromaDB)**：
    
    - **职责**：纯本地的向量知识库，存储 `MemeGene` 与 `Strategy` 的深度解析文本。
        
    - **场景**：赋予 Planner RAG 能力。支持自然语言模糊检索：“找出与‘阶层滑落’相关的高分策略”。
        
3. **L3 人机对齐真理之源 (Markdown File System)**：
    
    - **职责**：所有核心因子和策略最终落盘为带有 YAML Meta 头的 `.md` 文件。
        
    - **场景**：实现 Human-in-the-loop。人类可以直接阅读交易日志，手动修正大模型的错误推演。后台系统监听 `.md` 变更并自动同步回 L1 和 L2。
        

---

## 4. 动态归因与市场结算机制 (Dynamic Attribution & Settlement)

解决“信用分配问题 (Credit Assignment Problem)”——精准计算多个因子叠加时的独立贡献度，消除“搭便车效应”。

系统在 T+24h 执行三重归因：

1. **控制变量的正交测试 (Orthogonal A/B Testing)**：Planner 生成时采用单变量控制（如仅换图，不换文）。发布后根据 Delta 差值，100% 奖励或惩罚特定因子。
    
2. **语义降维归因 (Semantic LLM Attribution)**：抓取 `Comment` 实体的 Top 50 热评，交由 LLM 进行情绪焦点聚类。若评论区集中讨论“排版”，则重奖“视觉因子”。
    
3. **多元套索回归 (Lasso Regression)**：当样本量（`sample_size`）足够大时，执行统计学回归：
    
    $$Score = \beta_0 + \beta_1 \cdot G_1 + \beta_2 \cdot G_2 + \dots + \beta_n \cdot G_n$$
    
    无情地将无效的“搭便车因子”权重 $\beta$ 压缩归零。
    

---

## 5. 记忆切片的数据契约 (The MemeGene Schema Contract)

每个存活的注意力因子必须严格遵守以下 `.md` 结构：

**文件路径：** `data/memory_bank/factors/money_making/gene_20260301_001.md`

YAML

```
---
id: "gene_20260301_001"
schema_version: "3.1"
type: "cross_sectional_factor"  
track_id: "money_making"      # 绝对绑定：跨赛道需克隆重算

# 系统免疫状态
lifecycle_stage: "market_validated" # hypothesized / market_validated / blacklisted

# 核心量化指标 (由三重归因引擎动态更新)
performance_metrics:
  win_rate: 0.85               # 历史爆款胜率 (回测生命线)
  avg_save_to_like: 1.45       # 深度价值
  attribution_confidence: 0.92 # 归因置信度 (对抗搭便车效应)
  sample_size: 124             # 真实回测样本量
  volatility: "medium"         # 流量波动率

# 注意力因子拆解
attention_factors:
  identity_anchor: "前大厂精英 vs 街头小贩"
  emotion_trigger: "祛魅与反内卷"
  visual_style: "备忘录高亮 / 粗糙偷拍风"
  hook_structure: "身份跌落 + 意外收获"

# 物理世界溯源 (关联 SQLite 的 Post)
source_refs: 
  - post_id: "post_111"
    match_type: "genesis"      # 提取该基因的“起源靶机贴”
  - post_id: "post_222"
    match_type: "applied"      # 系统套用该基因生成的“己方实弹贴”
timestamp: 1709348444
---

# 核心认知失调 (Core Dissonance)
放弃高薪光环与社会地位，换取看似低贱但拥有绝对身心自由的底层工作。这精准切中了白领群体的“系统性疲惫”。(本段将被 ChromaDB 向量化以供检索)

## 语义归因报告 (Semantic Attribution Log)
- [2026-03-01][post_222]: 抽取 Top 50 评论分析，用户焦点集中于 `emotion_trigger` (72% 提及反内卷)。该因子置信度上调 0.1。
```

---

## 6. 核心业务工作流闭环 (The AQAS Loop)

1. **双轨数据摄取 (Ingestion)**：Timeline 低成本扫荡锁定偏离值 -> Detail 爬虫提取正文与评论。
    
2. **因子挖掘 (Factor Extraction)**：LLM 介入，生成截面/纵向大盘因子，初始化 `MemeGene` 存入记忆库。
    
3. **高熵 RAG 生成 (Planning)**：Planner 基于 ChromaDB 检索高胜率因子，生成携带特定参数的 `Strategy` 图纸。
    
4. **沙盒回测判决 (The Oracle Check)**：Monitor 提取草稿因子，对撞数据库。胜率 > 阈值放行，否则强制斩杀重写。
    
5. **去 AI 化渲染 (De-AI Rendering)**：Actor 根据图纸调取 HTML 模板，进行无头浏览器像素光栅化。
    
6. **市场对齐与结算 (Market Settlement)**：T+24h 抓取真实转评赞藏，触发归因机制，重新计算因子 `win_rate` 并覆写 `.md` 文件，完成自主演进。
    
