

---

**核心目标**: 跑通 "URL -> Deep Reading -> Viral Social Content -> Crystallization" 的完整闭环。

---

## Module 1: 基础设施与协议 (The Kernel)

_这是系统的骨架，必须最先完成，否则 UI 无法渲染。_

| **ID**     | **特性名称**                              | **优先级** | **详细描述 / 验收标准 (AC)**                                                                                                                                  | **涉及端** | 备注                                                                                                                                                                                                                                                              |
| ---------- | ------------------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **INF-01** | **Nexus Protocol v1.0 (SSE Upgrade)** | **P0**  | 升级 v0.5 的流式接口。后端需支持发送 `stream_message` (增量文本) 和 `render_instruction` (JSON 指令) 双通道数据。<br><br>  <br><br>**AC**: 前端能解析 SSE 事件，区分左侧对话和右侧 Canvas 指令。      | BE/FE   | 关键点：前后端的协议模型，必须支持v1.x的全部Agent/Pilot功能，且具备良好的拓展性                                                                                                                                                                                                                 |
| **INF-02** | **PostgreSQL 迁移**                     | **P0**  | 将 v0.5 的 SQLite (`daas_kernel.db`) 迁移至 Postgres。实现 `Task`, `TaskEvent`, `Artifact`, `Plugin`, `Skill` 5 张核心表。<br><br>  <br>**AC**: 数据持久化成功，重启服务数据不丢失。 | BE      | 1. Postgres 开发环境采用docker部署；<br>2.Plugin的名称容易被混淆， 更名为Pilot或者Agent，中文：智能助手/助手<br>3. 其他主体Connect、Command 、Plan、Context、Task Memory 、<br>User、User Profile、Preferences & Principles、Domain Knowledge、Trigger是否需要构建，以及所有这些实体的属性（字段）确认<br>4. v0.5的最优迁移方式，迁移 or 重建<br> |
| **INF-03** | **Plugin 注册机制**                       | **P0**  | 实现 `plugin_manifest.json` 加载逻辑。系统启动时自动写入 "The Creator" 的定义到 DB。<br><br>  <br><br>**AC**: 数据库 `plugin` 表中有 `agent_creator_v1` 记录。                      | BE      | —                                                                                                                                                                                                                                                               |
| **INF-04** | **Artifact 存储服务**                     | **P0**  | 实现简单的文件存储接口 (Local FS for v1.0)。<br><br>  <br><br>**AC**: 生成的图片/PDF 能被保存到 `/static/artifacts/` 并通过 URL 访问。                                            | BE      | —                                                                                                                                                                                                                                                               |

---

## Module 2: 交互界面 (The Interface)

_基于 v0.5 改造，实现 "Invisible OS" 设计语言。_
注意：始终保持项目当前的设计风格（ChatGPT，极简）

| **ID**    | **特性名称**                             | **优先级** | **详细描述 / 验收标准 (AC)**                                                                                                                                          | **涉及端** | 备注                                                                   |
| --------- | ------------------------------------ | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | -------------------------------------------------------------------- |
| **UI-01** | **Sidebar 重构 (Task List)**           | **P0**  | 实现 v1.0 定义的 Sidebar。<br><br>  <br><br>**AC**:<br><br>1. 默认展示 8 条 Recent Tasks。<br>2. 鼠标 Hover 显示 `Pin` / `Archive` 按钮。<br>3. 点击 `Archive`，任务从列表消失，进入 History。 | FE      | 参照：<br>[[Nexus-AGI (DAAS Bot) v1.0 产品技术规格说明书 (20260211)]]<br>核对每个细节。 |
| **UI-02** | **Artifacts & History 折叠面板**         | **P0**  | 实现 Sidebar 中部的折叠区。<br><br>  <br><br>**AC**:<br><br>  <br><br>1. `Artifacts` 仅显示**当前未归档任务**的产出物。<br><br>  <br><br>2. 点击 Artifact 条目，右侧 Canvas 滑出并加载该文件。        | FE      | 参照：<br>[[Nexus-AGI (DAAS Bot) v1.0 产品技术规格说明书 (20260211)]]<br>核对每个细节。 |
| **UI-03** | **Home Screen (OS Dashboard)**       | **P0**  | 实现 2x2 引导网格 (Creator, Analyst, Investor, Legal)。<br><br>  <br><br>**AC**: 点击 "Creator" 卡片或输入 `/run creator`，自动创建新 Task 并唤起对应 Agent。                           | FE      | 参照：<br>[[Nexus-AGI (DAAS Bot) v1.0 产品技术规格说明书 (20260211)]]<br>核对每个细节。 |
| **UI-04** | **Settings Modal (The Engine Room)** | **P1**  | 实现 Sidebar 底部的设置弹窗。<br><br>  <br><br>**AC**: 展示 "The Creator" 详情页，列出其包含的 Skills (`News_Scan`, `Viral_Gen`) 和 Connects状态。                                      | FE      | 参照：<br>[[Nexus-AGI (DAAS Bot) v1.0 产品技术规格说明书 (20260211)]]<br>核对每个细节。 |

---

## Module 3: 智能体逻辑 (The Creator Agent)

_这是 "The Brain"，负责业务逻辑。_

| **ID**     | **特性名称**                  | **优先级** | **详细描述 / 验收标准 (AC)**                                                                                                                                 | **涉及端** | 备注                                                          |
| ---------- | ------------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | ----------------------------------------------------------- |
| **AGT-01** | **Deep Reading MCP**      | **P0**  | 集成爬虫工具 (Firecrawl/Jina)。<br><br>  <br><br>**AC**: 输入 URL，Agent 能提取 Title, Summary, Key Arguments (Markdown格式)。                                       | BE      | 不必集成爬虫，基础已经开源的Skill/MCP即可，暂定实现：<br>URL抓取、Youtube、NotebookLM |
| **AGT-02** | **Logic Reasoning Skill** | **P0**  | 实现“逻辑推演”Prompt 链。<br><br>  <br><br>**AC**: 给定新闻，能输出 `Event -> Impact -> Tickers` 的节点关系数据。                                                            | BE      |                                                             |
| **AGT-03** | **Nexus JSON Emitter**    | **P0**  | 核心 Prompt 工程。强制 Agent 输出符合 `mobile_previewer` 组件规范的 JSON。<br><br>  <br><br>**AC**: 输出包含 `slides`, `branding: "Powered by DAAS Bot"`, `bg_style` 等字段。 | BE      | 需要确认，输出符合**INF-01**的定义                                      |
| **AGT-04** | **Tone Adaptation**       | **P1**  | 风格迁移能力。<br><br>  <br><br>**AC**: 用户指令“更犀利一点”，Agent 能重写文案并保持 JSON 结构不变。                                                                               | BE      |                                                             |

---

## 注意：新增 Module 4: 会话流的升级（Chat Stream ）

## Module 5: 画布与渲染 (The Canvas)

_这是 "The Creator" 的独有体验，v1.0 的高光时刻。_

| **ID**     | **特性名称**                     | **优先级** | **详细描述 / 验收标准 (AC)**                                                                                  | **涉及端** |
| ---------- | ---------------------------- | ------- | ----------------------------------------------------------------------------------------------------- | ------- |
| **CVS-01** | **Mobile Previewer 组件**      | **P0**  | 实现 iPhone 16 Pro 外壳容器。<br><br>  <br><br>**AC**: 支持深色/浅色模式，居中悬浮在 Canvas 区域。                            | FE      |
| **CVS-02** | **Slide 1: Dynamic Cover**   | **P0**  | 动态封面渲染器。<br><br>  <br><br>**AC**: 根据 JSON 中的 `bg_style` (e.g., cyberpunk) 切换 CSS 背景，渲染大标题。            | FE      |
| **CVS-03** | **Slide 2: Logic Chart**     | **P1**  | 基于 Mermaid 或 ReactFlow 渲染简单的逻辑传导图。<br><br>  <br><br>**AC**: 显示 `Node A -> Node B` 的连线。                | FE      |
| **CVS-04** | **Human-in-the-loop Editor** | **P0**  | 文案编辑与回写。<br><br>  <br><br>**AC**:<br>1. 用户直接修改手机里的文字。<br>2. 停止输入 500ms 后，前端自动 Patch 更新后端 Artifact 数据。 | FE/BE   |
| **CVS-05** | **Download Assets**          | **P1**  | 前端生成图片。<br><br>  <br><br>**AC**: 使用 `html2canvas` 将 React 组件截图并下载为 PNG。                               | FE      |

---

## Module 6: 结晶器 (The Crystallizer)

_这是我们的核心护城河。_
注意：这部分的设计需要单独讨论，有但不限于以下细节：
1. 从Task 历史中提取Skill的方法，我们要提取哪些内容，并最终形成怎样的prompt（包含哪些关键项）
2. 新Skill的草稿如何让用户审核、确认
3. 新Skill应该归属于某个Pilot/Agent，然后如何通过command触发？

| **ID**     | **特性名称**                  | **优先级** | **详细描述 / 验收标准 (AC)**                                                                                                | **涉及端** | 备注  |
| ---------- | ------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------- | ------- | --- |
| **CRZ-01** | **"Save as Skill" 按钮**    | **P0**  | Canvas 顶部或底部的操作入口。<br><br>  <br><br>**AC**: 点击后弹出 "Create Skill" 模态框。                                               | FE      |     |
| **CRZ-02** | **Skill Synthesis Logic** | **P1**  | 后端根据当前 Task History 生成 Prompt 模板。<br><br>  <br><br>**AC**: 提取用户刚才用的 URL 作为变量 `{input_url}`，生成一个新的 Skill JSON 存入 DB。 | BE      |     |
| **CRZ-03** | **My Skills List**        | **P1**  | Sidebar 或 Home Screen 显示用户创建的 Skill。<br><br>  <br><br>**AC**: 点击后可直接运行该 Skill (跳过 Prompting 阶段)。                    | FE      |     |
