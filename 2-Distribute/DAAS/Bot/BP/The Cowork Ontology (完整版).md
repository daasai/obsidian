

#### Layer 1: 基础设施层 (Infrastructure & Environment)

_定义系统的物理边界与触手。_

1. **Environment (环境沙箱)**
    
    - **定义：** Agent 运行的物理空间。
        
    - **特性：** **本地化 (Local-First)**。Cowork 的核心突破在于挂载了用户的本地文件系统（Folder）。
        
    - **权限：** Read/Write/Execute。Agent 可以直接修改本地文件，而非仅在云端生成文本。
        
2. **Connector / MCP (连接器)**
    
    - **定义：** 赋予 Agent “手和脚”的标准协议（Model Context Protocol）。
        
    - **类型：**
        
        - _Built-in:_ 官方内置 (e.g., Google Drive, Slack)。
            
        - _Local:_ 本地运行的工具 (e.g., SQLite Client, Local Python Runtime)。
            
        - _Browser:_ 浏览器代理，用于访问无 API 的网页。
            

#### Layer 2: 能力资产层 (Capability Assets)

_定义系统的静态能力，即“可复用的微型应用”。_

3. **Skill (技能 / 微型智能体)**
    
    - **定义：** 封装好的、针对特定领域问题的解决方案包。**它不是一个简单的指令，而是一个“数字员工”的配置文件。**
        
    - **构成公式：**
        
        $$Skill = \text{System Prompt} + \text{Knowledge Base} + \text{Tools Config} + \text{SOP}$$
        
    - **生命周期：**
        
        - _Demonstration:_ 用户演示操作流程。
            
        - _Encapsulation:_ 系统自动打包为 `.md` 或 `.json`。
            
        - _Mounting:_ 在会话中加载 (Load Skill)。
            
    - **关键修正：** 1 个 Skill 包含 N 种能力（Capabilities），可响应多种 Intent。
        
4. **Plugin (插件包)**
    
    - **定义：** 一组相关 Skill 的集合容器（如“营销套件”包含“SEO 技能”、“文案技能”、“数据分析技能”）。
        

#### Layer 3: 执行运行时 (Execution Runtime)

_定义系统如何动态处理任务，这是 Cowork 的“大脑”。_

5. **Task (任务对象)**
    
    - **定义：** 一个**持久化**的目标单元。
        
    - **属性：** 拥有状态 (Pending/In Progress/Done)、拥有所有者、拥有关联文件。
        
    - **区别：** 聊天 (Chat) 是流式的，任务 (Task) 是资产式的。
        
6. **Plan (动态规划图)**
    
    - **定义：** 在 Skill 执行前生成的**有向无环图 (DAG)**。
        
    - **组成：** `Steps` (步骤节点)。
        
    - **交互：** 用户可以预视 (Preview)、修正 (Edit) 或打断 (Interrupt) 这个规划。
        
7. **Context (语境 / 工作台)**
    
    - **定义：** 任务运行时的**易失性内存 (RAM)**。
        
    - **内容：** 当前挂载的 Skills + 当前活跃的 Files + 上一步的 Output。
        
8. **Memory (记忆 / 档案)**
    
    - **定义：** 任务运行的**持久化存储 (Hard Drive)**。
        
    - **实现：** 基于文件的记忆（如 `TASKS.md` 或 `.project/memory.json`），记录偏好和历史决策。
        

#### Layer 4: 交互交付层 (Interaction & Delivery)

_定义用户如何操作以及获得什么。_

9. **Trigger / Intent (触发意图)**
    
    - **定义：** 用户唤起 Skill 的方式。
        
    - **形式：**
        
        - _Natural Language:_ "帮我把这个视频转成 Newsletter"（模糊触发）。
            
        - _Explicit Command:_ `/video-to-post`（精确触发）。
            
10. **Artifact (产出物)**
    
    - **定义：** 任务的高价值交付结果。
        
    - **特性：** 独立于对话流存在，通常渲染在右侧 Canvas 或直接写入本地文件。
        
    - **类型：** 代码库、文档报告、图表、甚至是一个自动化的工作流脚本。
        

---

### 💡 对 DAAS 产品的核心启示 (PM Remarks)

基于这个完整的本体论，我们的 DAAS Bot 需要在以下 **3 个关键点** 上建立优势：

1. **"Skill Dock" (技能坞) 的可视化设计**
    
    - Cowork 目前加载 Skill 还需要打字或去设置里找。
        
    - **DAAS 机会：** 我们做一个像 Mac Dock 一样的底栏，用户把“SEO 专家”、“Python 分析师”拖进当前的 Chat 窗口，实现**多专家协同 (Multi-Agent Orchestration)** 的直观操作。
        
2. **"Demonstration as Code" (演示即编程)**
    
    - 视频中展示的 _“我做一遍，你存为 Skill”_ 是杀手级功能。
        
    - **DAAS 机会：** 我们必须内置 **Workflow Recorder**。用户操作一次，DAAS 自动生成背后的 JSON Manifest，让普通人也能由“使用者”变成“开发者”。
        
3. **可编辑的 Plan (Editable Plan)**
    
    - Cowork 的 Plan 主要是展示。
        
    - **DAAS 机会：** 让 Plan 变成**看板 (Kanban)**。用户可以拖拽步骤顺序，或者插入一个 "Human Approval" (人工审批) 节点，让 AI 的黑盒变白盒。