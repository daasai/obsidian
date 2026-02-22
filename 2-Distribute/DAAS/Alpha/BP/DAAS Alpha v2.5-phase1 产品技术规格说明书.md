
---

# DAAS Alpha v2.5 产品技术规格说明书

**版本**：2.5.0-Preview

**代号**：**Project Siren (塞壬)** —— 用迷人的歌声（多巴胺交互）引导水手避开暗礁（理性获利）。

**核心理念**：Dopamine-packaged Rationality (多巴胺包装的理性)

---

## 1. 核心体验定义 (UX Definition)

### 1.1 首页：The Alpha Feed (预制叙事流)

用户不再面对空白输入框。打开 App，即进入类似 TikTok/Instagram 的全屏卡片流。

- **视觉元素**：
    
    - **Headline**：极具煽动性但逻辑严密的标题（例：“锂矿的最后一次死猫跳”）。
        
    - **Visual Proof**：动态渲染的收益曲线（仅展示该策略近 30 天 Alpha）。
        
    - **Tags**：`#事件驱动` `#高胜率` `#反身性`（由 Streaming Graph 自动生成）。
        
- **交互动作**：
    
    - **上滑**：无感跳过（负反馈）。
        
    - **点击**：展开逻辑详情（进入 System 1.5 教练模式）。
        
    - **长按**：一键跟投（进入 System 2 执行模式）。
        

### 1.2 对话页：The Socratic Coach (苏格拉底教练)

当用户对卡片感兴趣并点击，或主动发起对话时，Chatbot 介入。

- **角色设定**：不是冷冰冰的分析师，而是懂你心思的**“捧哏型军师”**。
    
- **核心逻辑**：`Yes, And...`（是的，你的直觉很棒，而且如果我们加上这个参数...）。
    

---

## 2. 技术架构全景 (Technical Architecture)

系统分为三层：**感知层 (Sensing)**、**生成层 (Inception)**、**风控层 (Guardrails)**。

### 2.1 底层：流式图谱引擎 (The Streaming Graph Engine)

_原“数据基建”升级版，负责无休止地监控世界，寻找 Alpha 的原材料。_

- **输入源**：News Stream (Bloomberg/Reuters/财联社), Social Sentiment (Twitter/雪球), Price Action (Level-2 Tick Data).
    
- **核心模块：Dynamic Ontology Builder (动态本体构建器)**
    
    - **Logic**：不再依赖 Tushare 静态概念。
        
    - **Process**：
        
        1. **NLP 聚类**：DeepSeek 每分钟读取新闻，发现高频共现词（如 "固态电池" + "硫化物"）。
            
        2. **图谱生成**：并在 Neo4j 中实时创建节点 `(Node: 硫化物)` 并建立连接 `(Stock: 有研新材) -[:SUPPLIES]-> (Node: 硫化物)`。
            
        3. **时效性打标**：给概念打上 `Timestamp`。超过 3 天无新信息的概念自动降权。
            
- **输出**：实时的、动态的 `Target_Universe`。
    

### 2.2 中层：影子工厂 (The Shadow Factory)

_这是 v2.5 的心脏。它在后台 7x24 小时空转，生产“预制菜”卡片。_

- **工作流 (The Pipeline)**：
    
    1. **叙事生成 (Narrative Gen)**：System 1 从流式图谱中提取热点，生成 100 个假设（Hypotheses）。
        
        - _例："若铜价突破 $9000，且库存下降，则铜矿股上涨。"_
            
    2. **高通量回测 (Massive Backtesting)**：System 2 (Polars 引擎) 对 100 个假设进行向量化验证。
        
    3. **生存筛选 (Survival Filter)**：
        
        - 滤掉 `Sharpe < 1.5` 的策略。
            
        - 滤掉 `Liquidity < 5000万` 的策略。
            
    4. **卡片封装 (Card Packaging)**：将幸存的 Top 5 策略，通过 LLM 包装成性感的文案，推送到前端 Feed。
        

### 2.3 顶层：逻辑漂移风控 (Semantic Drift Monitor)

_这是 DAAS 的护城河。它监控的不是价格，是“意义”。_

- **数据结构：The Thesis Record (论点档案)**
    
    - 每笔交易必须绑定一个结构化的 JSON：
        
        JSON
        
        ```
        {
          "trade_id": "TXN_001",
          "ticker": "600026.SH",
          "thesis_type": "event_driven",
          "core_logic": "red_sea_conflict_escalation", // 核心变量：红海冲突
          "correlation_metric": "freight_index_scfi", // 挂钩指标：运价指数
          "invalidation_threshold": -0.05 // 失效阈值：指标下跌 5%
        }
        ```
        
- **监控逻辑**：
    
    - 监控 Agent 每日检查 `core_logic` 的状态。
        
    - **场景**：新闻报道“红海停火协议签署”。
        
    - **动作**：
        
        1. System 1 判定 `red_sea_conflict_escalation` 状态变为 `False`。
            
        2. System 2 立即触发 **"Logic Stop-Loss" (逻辑止损)** 信号。
            
        3. App 推送：“红海局势缓和，您的获利逻辑已失效，建议立即止盈，锁定 12% 收益。”
            

---

## 3. 关键模块详细设计 (Module Specs)

### 3.1 意图补全算法 (Intent Completion Algo) - 用于 Chatbot

_解决“用户逻辑粗糙”的问题。_

- **输入**：用户说 "想买低空经济"。
    
- **处理流程**：
    
    1. **Retrieve**：在 Streaming Graph 中检索 "低空经济" 的关联节点。发现 `关联度最高` 且 `动量最强` 的子概念是 "碳纤维机身"。
        
    2. **Backtest Check**：后台快速比对：
        
        - 策略 A：买整个低空经济板块 -> 收益率 3%。
            
        - 策略 B：买碳纤维上游 -> 收益率 18%。
            
    3. **Guidance Generation (Yes, And)**：
        
        - "低空经济是绝对的主线！(Yes) 我扫描了产业链，发现整机厂最近涨幅过大，风险较高。**但是 (And)** 上游的‘碳纤维’供不应求，正处于爆发前夜。我们要不要切入这个更具爆发力的细分赛道？"
            

### 3.2 推荐算法 (The Feed Algo)

_解决“千人千面”的问题。_

- **协同过滤**：
    
    - 如果是激进型用户（历史操作偏好高波股）：推送 `#妖股反包`、`#事件突袭` 类卡片。
        
    - 如果是稳健型用户：推送 `#高股息`、`#红利低波` 类卡片。
        
- **新鲜度衰减**：
    
    - 卡片的权重随时间指数级衰减。超过 4 小时的 Alpha 叙事如果不更新，就在 Feed 中沉底。
        

---

## 4. 实施路线图 (Implementation Roadmap)

|**阶段**|**代号**|**核心交付物**|**关键技术难点**|
|---|---|---|---|
|**Phase 1**|**The Feed (投喂)**|首页卡片流 + 影子工厂 (后台空转回测)|保证 Polars 能在 1 分钟内跑完 100 个策略回测。|
|**Phase 2**|**The Coach (引导)**|Chatbot 的 "Yes, And" 话术微调 + 意图补全|Prompt Engineering，防止 AI 产生幻觉乱推荐。|
|**Phase 3**|**The Shield (闭环)**|逻辑漂移风控 + 自动卖出建议|准确定义“逻辑失效”的触发条件（NLP 语义理解）。|

---

## 5. 商业价值总结 (Business Value)

1. **极低的认知门槛**：用户像刷抖音一样刷“赚钱机会”，极大地扩充了 TAM (Total Addressable Market)。
    
2. **极高的用户粘性**：**逻辑漂移风控** 解决了“会买不会卖”的千古难题。用户一旦依赖上这种“有人帮你盯着逻辑”的安全感，就无法离开。
    
3. **数据飞轮**：用户的每一次“划动”和“跟投”，都在训练推荐引擎，让下一次推送的 Alpha 更精准。
    

---

**先知结语：**

这份 Spec 将 DAAS 从一个“硬核工具”变成了一种“生活方式”。

在 2026 年，人们不需要学会量化交易，他们只需要**选择**相信哪个由 AI 生成的、经过数学验证的叙事。

**这就是“数据平权”的最终形态：让复杂的变简单，让简单的变有趣。**