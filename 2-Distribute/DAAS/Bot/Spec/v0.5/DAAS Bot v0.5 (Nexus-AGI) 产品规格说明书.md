
---


**版本**: v1.0 (The Toolbox Edition)

**日期**: 2026-02-09

**定位**: 智能时代的通用专业服务连接器与编排者

**核心理念**: **Intelligence-Driven, Not Flow-Driven** (智能驱动，而非流程驱动)

---

## 1. 产品愿景 (Product Vision)

DAAS Bot (Nexus-AGI) 是存在于 **专业人士 (Experts)**、**基座模型 (LLMs)** 和 **最终用户 (Users)** 之间的连接者。

它不生产大模型，而是**封装人类的专业能力**。通过一套高度抽象的系统部件体系，它让 AI 代理能够像人类专家一样，利用专业工具（数据、算法、内容），以个性化、高信任、可视化的方式交付服务。

**v1.0 的核心目标**：

构建一个 **“全能情报官”**。它不再依赖硬编码的 SOP (标准作业程序)，而是通过提供一组强大的 **原子工具 (MCP Tools)**，利用 LLM 的极限推理能力，自主完成“感知 -> 分析 -> 决策 -> 交付”的闭环。

---

## 2. 核心架构理念 (Core Philosophy)

### 2.1 从“流程管道”到“智能工具箱”

- **旧范式 (Pipeline)**: 程序员写死 `Input -> Step A -> Step B -> Output`。灵活性差，维护成本高。
    
- **新范式 (Toolbox)**:
    
    - 我们不给 LLM 设限。
        
    - 我们提供一个**“武器库” (Arsenal)**：里面有“读取企业宽表的接口”、“搜索新闻的接口”、“生成战报图片的接口”。
        
    - LLM (Master Model) 根据用户模糊的意图，**自主决定**先用哪个工具，后用哪个工具。
        

### 2.2 三位一体身份 (The Trinity Identity)

1. **连接者 (Connector)**: 向上连接用户意图，向下连接企业数据与外部知识。
    
2. **部件集 (Toolkit & GenUI)**: 将后端能力（算法/数据）和前端能力（图表/卡片）原子化封装。
    
3. **编排者 (Orchestrator)**: 动态路由不同模型（DeepSeek 负责计算，Claude 负责创意），实现成本与效果的最优解。
    

---

## 3. 功能模块：原子工具集 (The v1.0 Arsenal)

v1.0 不区分“左脑/右脑”，而是提供三组标准化的 MCP (Model Context Protocol) 工具。

### 🛠️ 工具组 A：企业数据能力 (Enterprise Data MCP)

_目标：安全、准确地获取业务真相，拒绝 Text-to-SQL 的幻觉风险。_

- **设计逻辑**: 对接企业内部数据中间层（已清洗好的宽表/指标层）。
    
- **核心工具 (Tools)**:
    
    - `get_dataset_meta()`: 获取当前可用的数据表定义（如：销售宽表、用户行为表）。
        
    - `query_metrics(dataset_id, dimensions=[], metrics=[], time_range)`:
        
        - _Input_: "sales_daily", ["region"], ["gmv", "dau"], "last_7_days"
            
        - _Output_: 标准 JSON 数据。
            
    - `check_anomaly(data_series)`: 调用统计学算法检测数据异常点。
        

### 🛠️ 工具组 B：全域深度阅读 (Deep Reading MCP)

_目标：打破信息孤岛，获取外部解释性信息。_

- **核心工具 (Tools)**:
    
    - `read_url(url)`: 智能解析网页/公众号，提取核心正文。
        
    - `read_document(doc_id)`: 解析 PDF/Word/Excel，支持长文档摘要。
        
    - `search_internet(query)`: 联网搜索实时新闻（用于归因分析）。
        
    - `fetch_youtube_transcript(video_url)`: 获取视频字幕。
        

### 🛠️ 工具组 C：分发与交互 (Distribution & GenUI MCP)

_目标：将冷冰冰的数据转化为可传播、可视化的资产。_

- **核心工具 (Tools)**:
    
    - `render_chart(type, data, title)`: 调用前端 Recharts 生成动态图表。
        
    - `generate_social_card(template, content_json)`: 生成小红书/推特风格的图片卡片（React 组件截图）。
        
    - `send_to_im(platform, target_id, artifact)`: 推送到飞书群/企业微信/Slack。
        

---

## 4. 系统架构设计 (System Architecture)

采用 **前后端分离 (Decoupled)** + **微内核 (Micro-Kernel)** 架构。

### 4.1 技术栈

- **Frontend (The Face)**:
    
    - **React 19 + Vite + TypeScript**: 极致响应速度。
        
    - **Shadcn/UI + Tailwind**: "精致极简" (Cream & Sage) 设计语言。
        
    - **T-Shape Layout**: 顶部状态栏 + 底部指令输入 + 中间多态画布 (Canvas)。
        
- **Backend (The Brain)**:
    
    - **FastAPI**: 高并发异步接口。
        
    - **Kernel (Phidata)**: 负责 LLM 会话管理与 Tool 注册。
        
    - **Enterprise Connectors**: 标准化的 API 适配层。
        

### 4.2 数据流转 (The Loop)

1. **感知**: 用户输入自然语言（"分析下昨天华东区销售为何下跌"）。
    
2. **规划**: Master Model (e.g., Claude 3.5 Sonnet) 分析意图，查看工具箱。
    
3. **行动 (ReAct Loop)**:
    
    - 调用 `Enterprise Data MCP` 获取数据 -> 发现跌幅。
        
    - 自主决定调用 `Deep Reading MCP` 搜新闻 -> 发现台风影响。
        
4. **交付**:
    
    - 调用 `GenUI` 在画布上渲染 "归因分析报告"。
        
    - 调用 `Distribution MCP` 发送到工作群。
        

---

## 5. 用户体验设计 (UX Design)

### 5.1 界面布局：Canvas-First

- **左/右侧边栏**: 历史记录与设置。
    
- **中央画布 (The Stage)**:
    
    - 不仅仅是聊天气泡。
        
    - 根据 LLM 的输出，动态渲染组件：
        
        - _数据场景_: 渲染交互式 Dashboard。
            
        - _阅读场景_: 渲染 Markdown 深度研报。
            
        - _社交场景_: 渲染手机预览模式的卡片。
            

### 5.2 交互模式：流体式 (Fluid)

- **引用与追问**: 用户可以选中 Canvas 中的某一段文字或图表，直接追问 Agent。
    
- **人工介入 (Human-in-the-Loop)**: 在发飞书之前，用户可以点击“编辑”按钮，修改 Agent 生成的文案，点击“Approve”后才发送。
    

---

## 6. 实施路线图 (Implementation Roadmap)

我们不追求大而全，追求**“最小闭环，最大智能”**。

### Phase 1: 骨架构建 (The Skeleton)

- [x] 初始化 React + FastAPI 项目结构。
    
- [x] 实现基础 T 型布局与流式对话。
    
- [ ] **关键任务**: 实现 `Enterprise Data MCP` 的 Mock 接口（模拟宽表数据）。
    

### Phase 2: 工具注入 (The Injection)

- [ ] 实现 `Deep Reading MCP` (集成 Jina/Firecrawl)。
    
- [ ] 实现 `GenUI` 组件库 (社交卡片模版、ECharts 封装)。
    
- [ ] **关键任务**: 调试 System Prompt，让 LLM 学会自主调度这些工具。
    

### Phase 3: 场景验证 (The Verification)

- [ ] 跑通 **"数据异动归因"** 场景 (查数 -> 搜新闻 -> 发群)。
    
- [ ] 跑通 **"研报转推文"** 场景 (读 PDF -> 写文案 -> 生成卡片)。
    

---

## 7. 结语

v1.0 的本质是**“给超级大脑装上双手”**。

