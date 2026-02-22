
## 一、核心原理：MCP 是什么？

你不需要记住 MCP 的全称，只需要理解它的比喻：**一个「万能翻译官」或「通用遥控器」**。

- **过去**：Obsidian, Notion, Outlook... 每个 App 都说自己的“方言”，互相无法沟通。
- **现在**：通过 MCP 这个“翻译官”，AI 可以将你在 Obsidian 中下达的指令，精准地“翻译”成各个 App 能听懂的命令，并让它们去执行。

我们使用的 `Smart Composer` 插件，就是把这个“万能翻译官”直接内置到了 Obsidian 里，让 Obsidian 从一个笔记孤岛，变成了能够号令万物的指挥部。

---

## 二、准备工作：安装与配置 Smart Composer

### Step 1: 安装插件

在 Obsidian 的第三方插件市场中，搜索 `Smart Composer`，点击安装并启用。

### Step 2: 配置 AI 大脑 (API Key)

插件本身只是一个“躯壳”，我们需要为它装上一个 AI “大脑”，才能让它思考。

1.  **添加 AI 供应商 (Provider)**
    - 打开 `Smart Composer` 插件设置。
    - 找到 `Providers` 选项，点击 `Add custom provider`。
    - 填写以下信息 (以智谱 GLM 为例):
        - **ID**: `GLM` (可自定义)
        - **Provider Type**: `OpenAI Compatible` (兼容 OpenAI 协议)
        - **Base URL**: `https://open.bigmodel.cn/api/paas/v4/`
        - **API Key**: 填入你自己的智谱 API Key。
    - 点击 `Add` 保存。

2.  **添加 AI 模型 (Model)**
    - 向下找到 `Models` 选项，点击 `Add custom model`。
    - 填写以下信息:
        - **ID**: `glm-4.5` (可自定义)
        - **Provider ID**: 选择上一步创建的 `GLM`。
        - **Model Name**: `glm-4.5-flash` (或其他你需要的模型，如 `glm-4.6`)
    - 点击 `Add` 保存。

3.  **启用模型**
    - 回到设置顶部，将 `Chat model` 和 `Apply model` 都选择为我们刚刚创建的 `glm-4.5`。

4.  **测试**
    - 在 Obsidian 左侧菜单栏点击 Smart Composer 图标，打开对话框。
    - 输入“你好”，如果 AI 正常回答，则说明配置成功。

---

## 三、核心配置：连接你的外部应用 (MCP Servers)

这是最激动人心的一步，我们将为 Obsidian 连接上 Notion 和 Microsoft 365。

### 1. 连接 Notion

1.  回到 `Smart Composer` 设置，拉到最下方找到 `MCP (Meta Control Protocol)` 部分。
2.  点击 `Add MCP Server`。
3.  填写以下信息：
    - **Name**: `Notion`
    - **Parameters**: 粘贴下面的 JSON 代码。

```
{
  "command": "npx",
  "args": [
    "-y",
    "@notionhq/notion-mcp-server"
  ],
  "env": {
    "OPENAPI_MCP_HEADERS": "{\"Authorization\": \"Bearer YOUR_NOTION_API_KEY\", \"Notion-Version\": \"2022-06-28\"}"
  }
}
```


**参数说明:**
- `YOUR_NOTION_API_KEY`: 替换为你自己的 Notion Internal Integration Token (Secret)。

配置完成后，点击 `Save` 保存。

**测试指令**:
> 读取我的 notion，列出我 notion 中所有数据库和页面的名称。

### 2. 连接 Microsoft 365 (Outlook, Calendar 等)

1.  再次点击 `Add MCP Server`。
2.  填写以下信息：
    - **Name**: `MS-365`
    - **Parameters**: 粘贴下面的 JSON 代码。

```
{
  "command": "npx",
  "args": [
    "-y",
    "@softeria/ms-365-mcp-server"
  ],
  "env": {}
}
```

配置完成后，点击 `Save` 保存。

**首次使用授权**:
当你第一次使用 MS-365 的 MCP 时，AI 会返回一个网址和一个设备代码。在浏览器中打开该网址，输入代码并登录你的微软账户，即可完成授权。这是一次性操作。

**测试指令**:
> 读取我 outlook 收件箱中最近的 3 封邮件标题。

---

## 四、实战演练：一个项目管理案例

现在，万事俱备。让我们用一个真实案例来感受它的威力。

### Step 1: 准备项目笔记

在 Obsidian 中创建一篇笔记，粘贴以下内容：

```markdown
# 项目：活力咖啡线上新品发布会

## 1. 项目概览
- **客户**: 活力咖啡 (Vitality Coffee)
- **项目目标**: 成功策划并执行一场线上发布会。
- **关键联系人**: 客户方市场经理，王杰森 (jason.wang@vitalitycoffee.com)
- **项目周期**: 2025年10月15日 - 2025年11月15日

## 2. 核心任务分解
- **策划阶段**: 敲定发布会创意主题和流程。
- **宣发阶段**: 在社交媒体发布预热内容。
- **执行阶段**: 正式举办线上发布会。

## 3. 重要会议安排
- **会议主题**: 项目启动会
- **会议时间**: 下周三下午两点
```

### Step 2: 在 Smart Composer 中下达指令

打开 Smart Composer 对话框，确保 AI 能够读取当前笔记的上下文，然后依次下达以下指令：

**指令一：创建 Notion 任务**
> 读取这篇笔记，把项目计划中的三个核心任务（策划、宣发、执行），插入到我 Notion 的“项目管理数据库”中。

**指令二：发送 Outlook 邮件**
> 根据项目概览里的信息，起草一封项目启动邮件，发送给客户经理王杰森，邮件内容要显得专业正式。

**指令三：创建日历会议**
> 根据项目概览和会议安排，在我的 outlook calendar 里，创建一个项目启动会议，时间定在下周三下午两点。

观察 AI 逐一完成这些操作，感受从笔记到行动的无缝衔接。

## 五、总结

通过 `Smart Composer` + `MCP`，你的 Obsidian 不再仅仅是知识的容器，更是行动的起点。它让你能够 100% 专注于思考和创造，将所有繁琐的执行工作，都交给你的 AI 管家。

欢迎来到真正的 AI Native 工作流。


## 附录

| 工具名称 (MCP Server) | 主要功能与应用场景 | 所属分类 | 配置地址 / 文档 |
| :--- | :--- | :--- | :--- |
| Playwright (Official) | 提供浏览器自动化功能，使 LLM 能够通过结构化数据与网页交互，进行点击、导航、输入和表单填充等操作。 | 个人效率 / 日常办公 | [github.com/microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp) |
| Time | 提供时间和时区转换功能。 | 日常办公 / 个人效率 | [github.com/modelcontextprotocol/servers/.../time](https://github.com/modelcontextprotocol/servers/.../time) |
| Memory | 基于知识图谱的持久化内存系统，用于维护上下文。 | 个人学习与知识管理 | [github.com/modelcontextprotocol/servers/.../memory](https://github.com/modelcontextprotocol/servers/.../memory) |
| Sequential Thinking | 通过思维序列实现动态和反思性的问题解决。 | 个人效率 | [github.com/modelcontextprotocol/servers/.../sequentialthinking](https://github.com/modelcontextprotocol/servers/.../sequentialthinking) |
| Adfin | 支付平台，提供发票和会计核对功能。 | 日常办公 | [github.com/Adfin-Engineering/mcp-server-adfin](https://github.com/Adfin-Engineering/mcp-server-adfin) |
| Agile Luminary | 简化项目管理，可直接向 IDE 发送 Agile Luminary 故事。 | 日常办公 | [github.com/AgileLuminary/mcp-agile-luminary](https://github.com/AgileLuminary/mcp-agile-luminary) |
| Airtable | 对 Airtable 数据库的读写访问。 | 日常办公 | [github.com/domdomegg/airtable-mcp-server](https://github.com/domdomegg/airtable-mcp-server) |
| AnkiConnect | 用于通过 AnkiConnect 与 Anki（抽认卡）交互。 | 个人学习与知识管理 | [github.com/spacholski1225/anki-connect-mcp](https://github.com/spacholski1225/anki-connect-mcp) |
| Apple Notes | 与你的 Apple Notes 对话，或读取本地 Apple Notes 数据库（仅限 macOS）。 | 个人学习与知识管理 | [github.com/RafalWilinski/mcp-apple-notes](https://github.com/RafalWilinski/mcp-apple-notes) |
| Apple Shortcuts | 与 Apple Shortcuts 的集成。 | 个人效率 | [github.com/recursechat/mcp-server-apple-shortcuts](https://github.com/recursechat/mcp-server-apple-shortcuts) |
| Basecamp | 与 Basecamp 项目管理平台的集成。 | 日常办公 | [github.com/georgeantonopoulos/Basecamp-MCP-Server](https://github.com/georgeantonopoulos/Basecamp-MCP-Server) |
| Baserow | 提供对 Baserow 表格的读写访问。 | 日常办公 | [baserow.io/user-docs/mcp-server](https://baserow.io/user-docs/mcp-server) |
| Box | 与智能内容管理平台 Box 进行交互、文件访问和搜索。 | 日常办公 / 知识管理 | [github.com/box-community/mcp-server-box](https://github.com/box-community/mcp-server-box) |
| Calculator | 使 LLM 能够使用计算器进行精确的数值计算。 | 个人效率 | [github.com/githejie/mcp-server-calculator](https://github.com/githejie/mcp-server-calculator) |
| CalDAV MCP | 暴露日历操作作为 AI 助手的工具，支持 Nextcloud Calendar 等。 | 日常办公 / 个人效率 | [github.com/dominik1001/caldav-mcp](https://github.com/dominik1001/caldav-mcp) |
| Chroma Package Search | 帮助编码代理理解和更好地使用数千个依赖项。 | 个人学习与知识管理 | [trychroma.com/package-search](https://trychroma.com/package-search) |
| Context 7 | 为任何 Cursor 提示提供最新的文档。 | 个人学习与知识管理 | [github.com/upstash/context7-mcp](https://github.com/upstash/context7-mcp) |
| context-awesome | 查询 8,500 多个精选 awesome lists，并获取最佳资源。 | 个人学习与知识管理 | [github.com/bh-rat/context-awesome](https://github.com/bh-rat/context-awesome) |
| Cal.com | 管理 Cal.com 的事件类型、创建预订和访问排程数据。 | 日常办公 / 个人效率 | [github.com/Danielpeter-99/calcom-mcp](https://github.com/Danielpeter-99/calcom-mcp) |
| Dart | 允许与 AI 原生项目管理工具 Dart 中的任务、文档和项目数据进行交互。 | 日常办公 | [github.com/its-dart/dart-mcp-server](https://github.com/its-dart/dart-mcp-server) |
| Data Exploration | 对基于 .csv 的数据集进行自主数据探索，提供智能见解。 | 日常办公 | [github.com/reading-plus-ai/mcp-server-data-exploration](https://github.com/reading-plus-ai/mcp-server-data-exploration) |
| DeepResearch | 闪电般快速、高准确度的深度研究代理。 | 个人学习与知识管理 | [github.com/OctagonAI/octagon-deep-research-mcp](https://github.com/OctagonAI/octagon-deep-research-mcp) |
| DeepWiki by Devin | 提供 AI 驱动的代码库上下文和答案。 | 个人学习与知识管理 | [docs.devin.ai/work-with-devin/deepwiki-mcp](https://docs.devin.ai/work-with-devin/deepwiki-mcp) |
| DifyWorkflow | 查询和执行 Dify AI 平台上的自定义工作流。 | 个人效率 | [github.com/gotoolkits/mcp-difyworkflow-server](https://github.com/gotoolkits/mcp-difyworkflow-server) |
| EduBase | 与电子学习平台 EduBase 交互，提供高级测验、考试管理和内容组织功能。 | 个人学习与知识管理 | [github.com/EduBase/MCP](https://github.com/EduBase/MCP) |
| Email Send MCP | 通过各种提供商发送电子邮件，并支持附件。 | 日常办公 | [github.com/YUHAI0/email-send-mcp](https://github.com/YUHAI0/email-send-mcp) |
| eSignatures | 用于起草、审查和发送具有约束力的合同的合同和模板管理。 | 日常办公 | [github.com/esignaturescom/mcp-server-esignatures](https://github.com/esignaturescom/mcp-server-esignatures) |
| Everything Search | 使用 Everything SDK 进行快速 Windows 文件搜索。 | 日常办公 / 个人效率 | [github.com/mamertofabian/mcp-everything-search](https://github.com/mamertofabian/mcp-everything-search) |
| Excel | Excel 操作，包括数据读写、工作表管理、格式化、图表和数据透视表。 | 日常办公 | [github.com/haris-musa/excel-mcp-server](https://github.com/haris-musa/excel-mcp-server) |
| Fibery | 在您的 Fibery 工作区中执行查询和实体操作。 | 日常办公 | [github.com/Fibery-inc/fibery-mcp-server](https://github.com/Fibery-inc/fibery-mcp-server) |
| Google Keep | 读取、创建、更新和删除 Google Keep 笔记。 | 个人学习与知识管理 / 个人效率 | [github.com/feuerdev/keep-mcp](https://github.com/feuerdev/keep-mcp) |
| Graphlit | 可将 Slack、Gmail 和播客源等内容提取到可搜索的 Graphlit 项目中。 | 个人学习与知识管理 | [github.com/graphlit/graphlit-mcp-server](https://github.com/graphlit/graphlit-mcp-server) |
| GrowthBook | 创建和读取功能标志、审查实验、搜索文档等。 | 日常办公 | [github.com/growthbook/growthbook-mcp](https://github.com/growthbook/growthbook-mcp) |
| HackMD | 与 HackMD 笔记平台集成，允许 AI 模型与 HackMD 交互。 | 个人学习与知识管理 | [github.com/yuna0x0/hackmd-mcp](https://github.com/yuna0x0/hackmd-mcp) |
| GitLab & Jira | GitLab 和 Jira 的统一 MCP 服务器，通过 AI 代理管理项目和票证。 | 日常办公 | [github.com/HainanZhao/mcp-gitlab-jira](https://github.com/HainanZhao/mcp-gitlab-jira) |
| Inbox Zero | 电子邮件的 AI 个人助理。 | 日常办公 / 个人效率 | [github.com/elie222/inbox-zero](https://github.com/elie222/inbox-zero) |
| Inkeep | 在您的内容上进行 RAG 搜索。 | 个人学习与知识管理 | [github.com/inkeep/mcp-server-python](https://github.com/inkeep/mcp-server-python) |
| Jina Reader | 以 Markdown 格式获取远程 URL 的内容。 | 个人学习与知识管理 | [github.com/wong2/mcp-jina-reader](https://github.com/wong2/mcp-jina-reader) |
| Markmap | 将 Markdown 转换为交互式的思维导图。 | 个人学习与知识管理 | [github.com/jinzcdev/markmap-mcp-server](https://github.com/jinzcdev/markmap-mcp-server) |
| Read Website Fast | 快速、令牌高效的网页内容提取，可将网站转换为干净的 Markdown。 | 个人学习与知识管理 | [github.com/just-every/mcp-read-website-fast](https://github.com/just-every/mcp-read-website-fast) |
| Zotero | 用于操作 Zotero Cloud 上的文献集合和资源的连接器。 | 个人学习与知识管理 | [github.com/kaliaboi/mcp-zotero](https://github.com/kaliaboi/mcp-zotero) |
| Kontent.ai | 使用自然语言在任何 MCP 兼容的 AI 工具中创建、管理和探索内容和内容模型。 | 个人学习与知识管理 | [github.com/kontent-ai/mcp-server](https://github.com/kontent-ai/mcp-server) |
| Lazy Toggl | 通过 Toggl API 跟踪时间。 | 个人效率 | [github.com/movstox/lazy-toggl-mcp](https://github.com/movstox/lazy-toggl-mcp) |
| Lingo.dev | 使您的 AI 代理能够使用 Lingo.dev 本地化引擎“说”地球上的每种语言。 | 日常办公 / 个人学习 | [github.com/lingodotdev/lingo.dev](https://github.com/lingodotdev/lingo.dev) |
| Linear | 与 Linear 项目管理系统集成。 | 日常办公 | [github.com/tacticlaunch/mcp-linear](https://github.com/tacticlaunch/mcp-linear) |
| Local History | 访问 VS Code/Cursor 的本地历史记录。 | 个人效率 | [github.com/xxczaki/local-history-mcp](https://github.com/xxczaki/local-history-mcp) |
| Make | 将您的 Make 场景转换为 AI 助手的可调用工具。 | 个人效率 | [github.com/integromat/make-mcp-server](https://github.com/integromat/make-mcp-server) |
| Mastra Docs | 提供对 Mastra.ai 完整知识库的直接访问。 | 个人学习与知识管理 | [github.com/mastra-ai/mastra/.../mcp-docs-server](https://github.com/mastra-ai/mastra/.../mcp-docs-server) |
| Mem0 | 帮助管理编码偏好和模式，用于存储、检索和语义处理代码实现、最佳实践和技术文件。 | 个人学习与知识管理 | [github.com/mem0ai/mem0-mcp](https://github.com/mem0ai/mem0-mcp) |
| Microsoft 365 | 连接到整个 Microsoft 365 套件（Office, Outlook, Excel, 日历）使用 Graph API。 | 日常办公 / 个人效率 | [github.com/softeria/ms-365-mcp-server](https://github.com/softeria/ms-365-mcp-server) |
| MintMCP | Google Calendar, Gmail, Outlook Calendar 和 Outlook 的 MCP 服务器。 | 日常办公 / 个人效率 | [github.com/mintmcp/servers](https://github.com/mintmcp/servers) |
| Miro | 访问 MIRO 白板，批量创建和读取项目。 | 日常办公 / 个人学习 | [github.com/k-jarzyna/mcp-miro](https://github.com/k-jarzyna/mcp-miro) |
| Google Chat | 连接 AI 助手到 Google Chat。 | 日常办公 | [github.com/siva010928/multi-chat-mcp-server](https://github.com/siva010928/multi-chat-mcp-server) |
| Needle | 生产级 RAG，可即时搜索和检索来自您自己文档的数据。 | 个人学习与知识管理 | [github.com/needle-ai/needle-mcp](https://github.com/needle-ai/needle-mcp) |
| Notion (Official) | Notion 官方 MCP 服务器。 | 日常办公 / 个人学习 | [github.com/makenotion/notion-mcp-server](https://github.com/makenotion/notion-mcp-server) |
| Obsidian | 通过 REST API 与 Obsidian 交互，或读写 Markdown 笔记目录。 | 个人学习与知识管理 | [github.com/MarkusPfundstein/mcp-obsidian](https://github.com/MarkusPfundstein/mcp-obsidian) |
| Odoo | 将 AI 助手连接到 Odoo ERP 系统，进行业务数据访问和工作流自动化。 | 日常办公 | [github.com/ivnvxd/mcp-server-odoo](https://github.com/ivnvxd/mcp-server-odoo) |
| Pandoc | 用于无缝文档格式转换，支持 Markdown、HTML、PDF、csv 和 docx 等格式。 | 日常办公 / 个人学习 | [github.com/vivekVells/mcp-pandoc](https://github.com/vivekVells/mcp-pandoc) |
| Paperless-NGX | 与 Paperless-NGX API 服务器交互，用于文档、标签和通信管理。 | 日常办公 / 知识管理 | [github.com/baruchiro/paperless-mcp](https://github.com/baruchiro/paperless-mcp) |
| Perplexity | 连接 Perplexity 的 Sonar API，支持实时、全网范围的研究。 | 个人学习与知识管理 | [github.com/ppl-ai/modelcontextprotocol](https://github.com/ppl-ai/modelcontextprotocol) |
| Plane | 官方 MCP 服务器，实现 Plane 项目、工作项和周期的全面 AI 自动化。 | 日常办公 | [github.com/makeplane/plane-mcp-server](https://github.com/makeplane/plane-mcp-server) |
| Project Manager | 分層任務管理（想法 → 史詩 → 任務）。 | 日常办公 / 个人效率 | [github.com/croffasia/mcp-project-manager](https://github.com/croffasia/mcp-project-manager) |
| Ragie | 从您的 Ragie (RAG) 知识库中检索上下文，可连接至 Google Drive、Notion、JIRA 等。 | 个人学习与知识管理 | [github.com/ragieai/mcp-server](https://github.com/ragieai/mcp-server) |
| Reminder | 通过 Slack 或 Telegram 安排和触发提醒。 | 个人效率 | [github.com/arifszn/reminder-mcp](https://github.com/arifszn/reminder-mcp) |
| Rember | 在 Rember 中创建间隔重复抽认卡，以记住您在聊天中学习的任何内容。 | 个人学习与知识管理 | [github.com/rember/rember-mcp](https://github.com/rember/rember-mcp) |
| Routine | 与 Routine（日历、任务、笔记等）进行交互。 | 日常办公 / 个人效率 | [github.com/routineco/mcp-server](https://github.com/routineco/mcp-server) |
| Rube | 连接到 500 多个应用（如 Gmail、Slack、GitHub、Notion），执行“发送电子邮件”或“创建任务”等实际操作。 | 日常办公 / 个人效率 | [rube.composio.dev](https://rube.composio.dev) |
| Slite | 搜索和检索笔记，浏览 Slite 工作区中的笔记层次结构。 | 个人学习与知识管理 | [github.com/fajarmf/slite-mcp](https://github.com/fajarmf/slite-mcp) |
| Spotify Player | 控制 Spotify 播放、队列、音量和播放列表。 | 个人效率 | [github.com/vsaez/mcp-spotify-player](https://github.com/vsaez/mcp-spotify-player) |
| Task Orchestrator | AI 驱动的任务编排和工作流自动化。 | 日常办公 / 个人效率 | [github.com/EchoingVesper/mcp-task-orchestrator](https://github.com/EchoingVesper/mcp-task-orchestrator) |
| Taskade (Official) | 通过统一的工作区和 API 实时访问任务、项目、工作流和 AI 代理。 | 日常办公 / 个人效率 | [github.com/taskade/mcp](https://github.com/taskade/mcp) |
| Taskeract | 集成您的 Taskeract 项目任务并加载任务上下文。 | 日常办公 / 个人效率 | [github.com/Acqusys/taskeract-mcp](https://github.com/Acqusys/taskeract-mcp) |
| Tasks | 高效的任务管理器，支持多种文件格式。 | 日常办公 / 个人效率 | [github.com/flesler/mcp-tasks](https://github.com/flesler/mcp-tasks) |
| Tavily | 专为 AI 代理设计的搜索引擎（搜索 + 提取）。 | 个人学习与知识管理 | [github.com/tavily-ai/tavily-mcp](https://github.com/tavily-ai/tavily-mcp) |
| Tldv | 将您的 AI 代理连接到 Google-Meet、Zoom 和 Microsoft Teams。 | 日常办公 | [gitlab.com/tldv/tldv-mcp-server](https://gitlab.com/tldv/tldv-mcp-server) |
| Todoist | Todoist Rest API 的完整实现。 | 日常办公 / 个人效率 | [github.com/stanislavlysenko0912/todoist-mcp-server](https://github.com/stanislavlysenko0912/todoist-mcp-server) |
| Trello | 处理 Trello 看板、列表和卡片。 | 日常办公 | [github.com/m0xai/trello-mcp-server](https://github.com/m0xai/trello-mcp-server) |
| Text Editor | 面向行的文本文件编辑器，针对 LLM 工具进行了优化，以高效访问文件部分。 | 个人效率 / 知识管理 | [github.com/tumf/mcp-text-editor](https://github.com/tumf/mcp-text-editor) |
| Unstructured | 设置和交互您的非结构化数据处理工作流。 | 个人学习与知识管理 | [github.com/Unstructured-IO/UNS-MCP](https://github.com/Unstructured-IO/UNS-MCP) |
| Vectorize | 高级检索、私人深度研究、Anything-to-Markdown 文件提取和文本分块。 | 个人学习与知识管理 | [github.com/vectorize-io/vectorize-mcp-server](https://github.com/vectorize-io/vectorize-mcp-server) |
| WayStation | 连接到 Notion、Monday、AirTable 等流行的生产力工具。 | 日常办公 / 个人效率 | [waystation.ai/connect/mcp-server](https://waystation.ai/connect/mcp-server) |
| Xero (Official) | 与您的业务会计数据进行交互。 | 日常办公 | [github.com/XeroAPI/xero-mcp-server](https://github.com/XeroAPI/xero-mcp-server) |
| Yuga Planner | 使用 LLamaIndex 和 Timefold 进行 AI 任务排程计划。 | 日常办公 / 个人效率 | [github.com/blackopsrepl/yuga-planner](https://github.com/blackopsrepl/yuga-planner) |
| Mindmap | 用于生成漂亮交互式心智图（mindmap）的 MCP 伺服器。 | 个人学习与知识管理 | [github.com/YuChenSSR/mindmap-mcp-server](https://github.com/YuChenSSR/mindmap-mcp-server) |
| Zapier (Official) | 立即将您的 AI 代理连接到 8,000 个应用程序。 | 个人效率 | [zapier.com/mcp](https://zapier.com/mcp) |
| Markdownify | 将几乎任何文件或网络内容转换为 Markdown。 | 个人学习与知识管理 | [github.com/zcaceres/markdownify-mcp](https://github.com/zcaceres/markdownify-mcp) |
| Google Drive | Google Drive 集成，用于列出、阅读和搜索文件。 | 日常办公 / 知识管理 | [github.com/modelcontextprotocol/servers/.../gdrive](https://github.com/modelcontextprotocol/servers/.../gdrive) |
| Exa (Official) | 官方 MCP，为 AI 设计的搜索引擎。 | 个人学习 / 个人效率 | [github.com/exa-labs/exa-mcp-server](https://github.com/exa-labs/exa-mcp-server) |
| Filesystem | 官方参考服务器，用于安全地操作本地文件，可配置访问权限。 | 个人效率 / 知识管理 | [github.com/modelcontextprotocol/servers/.../filesystem](https://github.com/modelcontextprotocol/servers/.../filesystem) |
| Slack | 功能强大的社区版 Slack 服务器，支持收发消息等工作空间操作。 | 日常办公 | [github.com/korotovsky/slack-mcp-server](https://github.com/korotovsky/slack-mcp-server) |