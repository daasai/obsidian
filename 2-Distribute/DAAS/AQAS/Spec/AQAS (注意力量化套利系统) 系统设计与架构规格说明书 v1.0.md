
---

# AQAS (注意力量化套利系统) 系统设计与架构规格说明书 v1.0

## 1. 系统概述与设计哲学 (System Overview & Philosophy)

### 1.1 产品定调

AQAS 并非传统的“AI 辅助内容生成工具”，而是一个在社交媒体（如小红书、抖音）中运行的**高频自主套利机器**。系统通过实时抓取环境数据、逆向工程爆款特征、生成带有对抗性变异的内容，以获取高质量的人类注意力（流量池中的套利空间）。

### 1.2 核心工程原则

- **目标导向自主 (Goal-oriented Autonomy):** 研发团队严禁将系统设计为“指令-执行”模式。系统由且仅由数学化的奖励函数驱动，自主决定策略。
    
- **隔离与解耦 (Isolation & Decoupling):** IO 阻塞任务（如爬虫）与核心决策中枢必须物理隔离；执行域（渲染/排版）与决策域必须沙盒隔离。
    
- **代码即动作 (Code-as-Action):** 放弃用自然语言描述复杂操作。Agent 必须通过输出并在沙盒中执行 Python/TypeScript 脚本来完成文件修改、视觉渲染和参数调整。
    
- **拒绝人工调参 (No Manual Tuning):** 所有的阈值、变异率、Prompt 规则，最终都必须交由系统的“元认知”模块通过强化学习自主更新。
    

---

## 2. 核心算法引擎：奖励闭环与变异计算 (The Core Engine)

系统的演进方向由多阶复合奖励函数决定，这是整个业务的“北极星指标”。

### 2.1 奖励函数数学模型 (Reward Function)

系统生成每一组内容策略的最终得失 $R_{total}$ 计算如下：

$$R_{total} = \alpha \cdot R_{proxy} + \beta \cdot R_{hook} + \gamma \cdot R_{deep} - \lambda \cdot P_{toxicity}$$

- **$R_{proxy}$ (预审代理奖励):** 策略在 Planner 与 Monitor 博弈时的沙盒预打分。评估其“认知失调感”与“钩子强度”。
    
- **$R_{hook}$ (T+15m 破冰奖励):** 内容发布后前 15 分钟的真实点击率与秒级停留率。这是套利的第一道关卡。
    
- **$R_{deep}$ (T+24h 价值奖励):** 发布 24 小时后的“赞藏比”及长尾分发斜率，验证商业转化深度。
    
- **$P_{toxicity}$ (毒性惩罚):** 触发平台风控、限流或极端负面评论的惩罚项。权重 $\lambda$ 为一票否决级。
    

### 2.2 对抗性变异公式 (Adversarial Mutation)

为防止生成同质化内容，系统在期望公式中强制引入变异因子 $\epsilon$：

$$E(Arbitrage) = \frac{\Delta(Attention\_Hook)}{P(Toxicity)} \times \epsilon(Mutation\_Rate)$$

- **工程实现：** Monitor 监控历史收益方差。若发现策略收益收敛（用户免疫），Monitor 将通过编程式工具调用，动态调高 $\epsilon$（如提升 LLM 的 $Temperature$ 参数或在 Prompt 中注入反常识指令），强制系统进行高熵探索。若触发 $P_{toxicity}$，则通过梯度下降回调该参数。
    

---

## 3. 总体架构拓扑 (Architecture Topology)

系统采用异步事件驱动架构，彻底解耦数据感知与策略计算。

### 3.1 异步数据感知层 (h2A Queue)

- **职责：** 作为后台常驻进程，执行“差异化抓取 (Differential Scraping)”。
    
- **逻辑：** 仅锁定目标矩阵内偏离历史均值 $+3\sigma$（爆款）和 $-2\sigma$（扑街）的异常动能数据。
    
- **通信：** 剔除冗余 HTML 节点和噪音后，将结构化 JSON 推入 Redis 消息队列。严禁直接调用主推理模型。
    

### 3.2 决策主循环 (nO Main Loop)

- **职责：** 核心战略中枢，仅监听 h2A 队列发出的“高频异常动能”事件。
    
- **逻辑：** 类似于量化交易的极速撮合引擎，一旦接收到信号，立即唤醒后端的 Agent 蜂群进行策略生成与任务派发。
    

---

## 4. Agent 蜂群沙盒模型 (Agent Swarm & Sandboxing)

放弃单体大模型处理所有任务的幻想，采用 P-M-A (Planner-Monitor-Actor) 物理隔离机制，防止上下文污染与逻辑降智。

---

## 5. 记忆与上下文防御系统 (Memory & Context Defense)

解决系统长时间跨度运行导致的“灾难性遗忘”与上下文膨胀。

### 5.1 混合检索海马体 (Hybrid Vector DB)

- **存储 Schema：** 摒弃文本级存储。所有被验证的策略均切片为 JSON 格式存入 Qdrant/Milvus。
    
- **向量空间：** 存储核心论点、痛点特征、认知失调点（用于语义泛化）。
    
- **标量载荷：** 存储历史 $R_{hook}$、$R_{deep}$ 分值、发布时间戳（用于硬性条件过滤）。
    

### 5.2 状态持久化 (`MEME_STATE.json`)

- 系统在本地维护全局状态机，记录当前有效的“多巴胺触发器特征”、风控水位线和最优超参数。
    
- 跨日唤醒时，Planner 直接读取该文件恢复业务上下文，无需加载冗长历史对话。
    

### 5.3 wU2 上下文强制压缩

- 当任意 Agent 的 Context Token 利用率达到 90% 水位线时，中间件强制阻断当前对话。
    
- 自动调用轻量级模型对历史博弈进行实体抽取与状态总结，清空执行 Log，仅保留高价值结论。
    

---

## 6. 元认知与自我重写网络 (Meta-Cognition & Self-Modification)

这是 AQAS 建立技术壁垒的核心，要求研发团队实现“让 AI 优化 AI”的链路。

### 6.1 对比特征自发现 (Contrastive Feature Discovery)

- **机制：** Planner 定期从向量库召回同类目下一正（$+3\sigma$）一负（$-1\sigma$）的数据对。
    
- **执行：** 生成探索性 Meta-Prompt，要求大模型计算两者的“正交差异”（如发现“视觉高奢与文字粗鄙的背离率”）。
    
- **结果：** 系统为新发现的特征动态生成 JSON Schema 字段，并热更新至数据库结构中。
    

### 6.2 代码即动作的自我迭代 (Self-Modification)

- 当 Monitor 验证新特征或新参数具有正向 Advantage 时，系统不得等待人工发版。
    
- Planner 将直接生成一段 Python 脚本，覆写本地 `prompts/actor_system_prompt.md` 或 `configs/hyperparameters.yaml`。
    
- **研发要求：** 必须为 Planner 配置具有安全隔离沙盒的本地代码执行环境（类似 Docker 容器内运行）。
    

---

## 7. 物理层渲染管线 (The Last Mile Rendering)

彻底消除“AI 塑料味”，保障内容破冰率。

### 7.1 De-AI 文本渲染管线

- 强制禁用大模型惯用词汇（如“综上所述”、“深刻体现”）。
    
- **Burstiness 注入：** Actor 在生成文本时，强制打碎标点与段落结构，RAG 召回真实爆款人类博主的语料作为 Few-Shot 锚点进行像素级克隆。
    

### 7.2 编程式工业视觉合成 (Programmatic UI)

- 严禁使用多模态大模型直接生成带文字的封面。
    
- **底层出图：** Visual Agent 调用 ComfyUI (配合 SDXL + 垂类 LoRA) 生成纯净的场景底图。
    
- **CSS 注入与截屏：** Visual Agent 生成 HTML/CSS 布局代码，将标题与视觉元素动态注入。在本地启动 Puppeteer 无头浏览器，合并底图与 CSS 渲染层，执行毫秒级截屏，输出工业级海报。
    

