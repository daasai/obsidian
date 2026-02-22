
### 1. 关键架构决策回应 (Architectural Decisions)

在进入最终列表前，我先回应你在备注中提出的几个核心技术问题：

#### Q1: 关于 INF-02 的 Schema 完整性与命名

> **决策**:
> 
> - **命名**: 数据库表名统一使用 **`agent`** (在代码/UI中对应 "Pilot/智能助手")，废弃 `plugin` 以避免歧义。
>     
> - **字段补全**: 针对你提到的 `Connect`, `Command`, `Plan`, `Context`, `Memory` 等实体，我已在下方的 **Addendum: 完整 Schema 定义** 中列出了核心字段。
>     
> - **迁移策略**: **重建 (Rebuild)**。
>     
>     - _理由_: v0.5 是 SQLite 原型，且 Ontology 与 v1.0 差异巨大。写迁移脚本成本高于新建。我们将使用 Docker 部署全新的 Postgres，并在启动时通过 `bootstrap` 脚本初始化 System Agents。
>         

#### Q2: 关于 Module 6 (结晶器) 的逻辑细节

> **决策**: 结晶器 (Crystallizer) 将作为一个独立的 **Meta-Agent (元智能体)** 运行。
> 
> - **提取方法**: 不依靠规则正则，而是将 `TaskEvent` (历史对话 + 工具调用链) 投喂给 LLM，Prompt 为 _"Analyze this session and extract a generic workflow template."_
>     
> - **审核机制**: 结晶不直接入库，而是生成一个 `Draft Skill JSON`，在前端弹窗中以**表单**形式呈现，让用户修改/确认。
>     
> - **归属**: 新 Skill 默认归属于 **User** (Private)，但继承自 `Parent Agent` (e.g., Creator)。触发方式为 `/run {skill_name}` 或在 Sidebar 点击。
>     

#### Q3: 关于 Module 4 (Chat Stream)

> **决策**: 我已补全 Module 4，重点在于 **"过程的可视化"**。左侧的 Stream 不仅要吐字，还要能优雅地展示 "Tool Calls" (折叠的思考过程)，避免满屏的 JSON 吓跑用户。

---

### 🚀 Nexus-AGI v1.0 Feature List (Final Lock - v1.0.1)

这是基于你反馈修正后的**最终执行版**。

#### Module 1: 基础设施与协议 (The Kernel)

_系统骨架，开发顺序 P0 -> P1_

|**ID**|**特性名称**|**优先级**|**详细描述 / 验收标准 (AC)**|**涉及端**|
|---|---|---|---|---|
|**INF-01**|**Nexus Protocol v1.0 (SSE)**|**P0**|实现通用 SSE 协议。支持 `stream_message` (文本流), `tool_state` (工具状态), `render_instruction` (UI指令) 多路复用。<br><br>  <br><br>**AC**: 协议结构需兼容后续 Analyst/Investor 场景，支持全量/增量更新。|BE/FE|
|**INF-02**|**Postgres Docker & Schema**|**P0**|Docker 部署 PG。实现 `agent` (原Plugin), `skill`, `task`, `artifact` 等全量表结构 (详见附录)。<br><br>  <br><br>**AC**: 采用 **Rebuild** 策略，系统启动自动初始化 Schema。|BE|
|**INF-03**|**Agent/Pilot 注册机制**|**P0**|实现 `manifest.json` 加载。系统启动时自动注册 "The Creator" Agent。<br><br>  <br><br>**AC**: DB 中 `agent` 表包含 `creator_v1` 及其默认 Skills/Connects。|BE|
|**INF-04**|**Artifact 存储**|**P0**|实现 Local FS 存储。<br><br>  <br><br>**AC**: 生成的文件存入 `/static/artifacts/{task_id}/`，URL 可访问。|BE|

#### Module 2: 交互界面 (The Interface)

_保持 ChatGPT 极简风格，隐形 OS_

|**ID**|**特性名称**|**优先级**|**详细描述 / 验收标准 (AC)**|**涉及端**|
|---|---|---|---|---|
|**UI-01**|**Sidebar: Task List**|**P0**|**严格参照 v1.0 规格书**。<br><br>  <br><br>**AC**: Recent Tasks (8条), Hover 显示 Pin/Archive, Archive 后移入 History。|FE|
|**UI-02**|**Sidebar: Artifacts**|**P0**|**严格参照 v1.0 规格书**。<br><br>  <br><br>**AC**: 仅显示 Active Task 的资产。联动逻辑：Task 归档 -> Artifact 列表对应条目消失。|FE|
|**UI-03**|**Home: OS Dashboard**|**P0**|**严格参照 v1.0 规格书**。<br><br>  <br><br>**AC**: 2x2 Grid (Creator, Analyst, Investor, Legal)。点击卡片触发 `/run {agent}`。|FE|
|**UI-04**|**Settings Modal**|**P1**|**严格参照 v1.0 规格书**。<br><br>  <br><br>**AC**: 能够查看 Agent 详情及其 Skills/Connects 状态。|FE|

#### Module 3: 智能体逻辑 (The Creator Agent)

_业务大脑_

|**ID**|**特性名称**|**优先级**|**详细描述 / 验收标准 (AC)**|**涉及端**|
|---|---|---|---|---|
|**AGT-01**|**Basic Reading Connects**|**P0**|实现基础读取能力。<br><br>  <br><br>**AC**: 1. `requests/bs4` 抓取 URL 文本; 2. 集成 YouTube Transcript API; 3. NotebookLM (Mock 或 API)。**不引入重型爬虫**。|BE|
|**AGT-02**|**Logic Reasoning Skill**|**P0**|实现逻辑推演 Chain。<br><br>  <br><br>**AC**: 给定文本，输出 `Event -> Impact` 结构化数据。|BE|
|**AGT-03**|**Nexus JSON Emitter**|**P0**|核心 Prompt。<br><br>  <br><br>**AC**: 输出严格符合 **INF-01** 定义的 Protocol JSON，包含 `branding` 水印。|BE|
|**AGT-04**|**Style Transfer**|**P1**|风格迁移。<br><br>  <br><br>**AC**: 支持 User 指令修改文案语气 (Tone)。|BE|

#### Module 4: 会话流升级 (Chat Stream Upgrade) - _New_

_左侧 Stream 的体验优化_

|**ID**|**特性名称**|**优先级**|**详细描述 / 验收标准 (AC)**|**涉及端**|
|---|---|---|---|---|
|**STM-01**|**Markdown Streaming**|**P0**|支持流式 Markdown 渲染。<br><br>  <br><br>**AC**: 表格、粗体、代码块正常渲染，无闪烁。|FE|
|**STM-02**|**Collapsible Tool Logs**|**P0**|工具调用可视化。<br><br>  <br><br>**AC**: 当 Agent 调用 `Reading` 时，显示折叠的 `[⚡️ Reading URL...]`，点击可查看原始 Log。不要直接把 JSON 甩在对话流里。|FE|
|**STM-03**|**Thinking State**|**P1**|思考状态动画。<br><br>  <br><br>**AC**: 类似于 DeepSeek 的 "Thinking..." 展开/收起效果。|FE|

#### Module 5: 画布与渲染 (The Canvas)

_右侧高光时刻_

|**ID**|**特性名称**|**优先级**|**详细描述 / 验收标准 (AC)**|**涉及端**|
|---|---|---|---|---|
|**CVS-01**|**Mobile Previewer**|**P0**|iPhone 16 Pro 容器。<br><br>  <br><br>**AC**: 居中悬浮，支持 Dark Mode。|FE|
|**CVS-02**|**Dynamic Cover**|**P0**|动态封面渲染。<br><br>  <br><br>**AC**: 基于 HTML/CSS 渲染封面，支持 `bg_style` 切换。|FE|
|**CVS-03**|**Logic Chart**|**P1**|逻辑图渲染。<br><br>  <br><br>**AC**: 使用 `Mermaid` 或 `ReactFlow` 渲染简单的节点关系图。|FE|
|**CVS-04**|**HITL Editor**|**P0**|人工介入编辑。<br><br>  <br><br>**AC**: 修改手机内文字 -> 500ms Debounce -> Patch Artifact API。|FE/BE|
|**CVS-05**|**Download as Image**|**P1**|导出图片。<br><br>  <br><br>**AC**: `html2canvas` 截图下载。|FE|

#### Module 6: 结晶器 (The Crystallizer) - _Needs Design Session_

_核心护城河_

|**ID**|**特性名称**|**优先级**|**详细描述 / 验收标准 (AC)**|**涉及端**|
|---|---|---|---|---|
|**CRZ-01**|**Trigger UI**|**P0**|入口。<br><br>  <br><br>**AC**: Canvas 顶部或底部的 "✨ Save as Skill" 按钮。|FE|
|**CRZ-02**|**Skill Extraction**|**P1**|提取逻辑。<br><br>  <br><br>**AC**: LLM 读取 History，输出 `Draft Skill JSON` (含 Inputs, Workflow)。|BE|
|**CRZ-03**|**Draft Review Modal**|**P1**|**审核交互**。<br><br>  <br><br>**AC**: 弹窗显示提取出的 Prompt 和变量，允许用户修改 Skill 名称和默认参数。|FE|

---

### Addendum: 补充 Schema 定义 (For INF-02)

为了响应你的 **INF-02 备注 3**，这是我们需要构建的完整实体字段定义（简版）：

SQL

```
-- 1. Agent (原 Plugin)
CREATE TABLE agent (
    id VARCHAR PRIMARY KEY, -- "creator_v1"
    name VARCHAR,
    role_prompt TEXT, -- System Prompt Template
    avatar VARCHAR
);

-- 2. Connect (连接/权限)
CREATE TABLE connect (
    id VARCHAR PRIMARY KEY,
    agent_id VARCHAR REFERENCES agent(id),
    type VARCHAR, -- "api_key", "oauth", "file_system"
    provider VARCHAR, -- "youtube", "notion"
    config_schema JSONB -- 定义连接需要什么字段 (e.g. key)
);

-- 3. Command (指令映射)
CREATE TABLE command (
    trigger VARCHAR PRIMARY KEY, -- "/scan"
    agent_id VARCHAR REFERENCES agent(id),
    target_skill_id VARCHAR,
    description VARCHAR
);

-- 4. User Profile & Preferences
CREATE TABLE user_profile (
    user_id VARCHAR PRIMARY KEY,
    nickname VARCHAR,
    preferences JSONB -- { "language": "zh", "theme": "dark" }
);

-- 5. Task Memory (任务记忆)
-- 实际上集成在 task 表和 task_event 表中，通过 context_window 动态构建
-- 但我们可以加一个表存储 Long-term Summary
CREATE TABLE task_summary (
    task_id VARCHAR REFERENCES task(id),
    summary_text TEXT, -- 对过去对话的总结
    key_entities JSONB -- { "extracted_urls": [...] }
);

-- 6. Trigger (自动化)
CREATE TABLE trigger (
    id VARCHAR PRIMARY KEY,
    user_id VARCHAR,
    skill_id VARCHAR,
    cron VARCHAR, -- "0 8 * * *"
    params JSONB, -- { "url": "..." }
    is_active BOOLEAN
);
```

