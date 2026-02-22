
---


**版本:** v1.0 (The Creator Pilot)

**日期:** 2026-02-10

**核心目标:** 构建 Agentic OS 基座，并以 "The Creator" (财经创作者) 为首个落地场景。

---

## 1. UI/UX 交互规格说明 (User Interface Specification)

### 1.1 全局布局 (Global Layout)

采用 **T-Shape + Dual-Stream** 布局架构。

- **容器**: 100vw * 100vh, `overflow: hidden`.
    
- **Sidebar (左侧)**: 宽 260px，固定，深色/浅色自适应。
    
- **Main Stage (右侧)**: 剩余宽度。
    
    - **Stream (对话流)**: 宽 40% - 50% (可拖拽调整)，位于中间。
        
    - **Canvas (画布)**: 剩余宽度，位于右侧。默认隐藏，有 Artifact 生成时滑出。
        

---

### 1.2 侧边栏详解 (Sidebar Specification)

**布局模式**: Flex Column (Top-Middle-Bottom)。

#### A. 顶部区 (Header)

- **Logo Area**: `[Logo] AgenticOS` + `[<]` (收起按钮)。
    
- **Action**: **[+] 新任务 (New Task)** 按钮。
    
    - _点击行为_: 清空右侧 Context，重置为“首页状态”。
        

#### B. 滚动内容区 (Scrollable Content)

- **1. 任务列表 (Tasks)**
    
    - **默认状态**: `Expanded` (展开)。
        
    - **数据源**: 显示 `status="running"` 的任务 + 按 `last_active_at` 排序的最近任务。
        
    - **显示数量**: 初始加载 **8条**。
        
    - **Show More**: 底部显示 "Show More" 按钮。点击一次加载 +5 条，直至显示全部。
        
    - **Item 交互 (Hover State)**:
        
        - _Normal_: `[Icon] 任务名称 ......... 2h` (显示创建距今时长)。
            
        - _Hover_: `[Icon] 任务名称 ......... [📌][🗃️]` (时长被操作按钮替换)。
            
            - `📌 (Pin)`: 置顶该任务到列表顶部。
                
            - `🗃️ (Archive)`: 归档任务。**重要逻辑**: 归档后，该 Task 移入 History，且其关联的 Artifacts 从下方列表隐藏。
                
    - **选中状态**: 高亮背景，右侧 Main Stage 加载该任务上下文。
        
- **2. 数字资产 (Artifacts)**
    
    - **默认状态**: `Collapsed` (折叠)。点击标题展开。
        
    - **定义**: 仅显示 **当前活跃任务 (Active Tasks)** 产出的结果文件 (PDF/Image/Chart)。已归档任务的资产不在此显示。
        
    - **Item 样式**: `[FileIcon] 文件名.ext`.
        
    - **交互**: 单击在右侧 Canvas 预览；Hover 显示 `[📌][🗃️]`。
        
- **3. 历史归档 (History)**
    
    - **默认状态**: `Collapsed` (折叠)。
        
    - **内容**: 所有 `is_archived=True` 的任务。
        
    - **交互**: 点击条目 -> 恢复到主界面 (同时其关联 Artifacts 暂时在 Sidebar 可见)。
        

#### C. 底部固定区 (Footer)

- **1. 自动化 (Triggers)**
    
    - **位置**: 底部区域的最上方，紧贴内容区。
        
    - **默认状态**: `Collapsed` (折叠)。
        
    - **内容**: `[⏰] 每日早报 (08:00) ... [Active]`。
        
- **2. 分割线 (Divider)**: `1px solid border-gray-200`.
    
- **3. 设置 (Settings)**
    
    - **图标**: `⚙️`。
        
    - **交互**: 点击弹出 **Settings Modal** (详见 1.4)。
        
- **4. 用户 (User Profile)**
    
    - **样式**: `[Avatar] UserName`.
        
    - **交互**: 点击跳转用户个人中心。
        

---

### 1.3 首页状态 (Home State)

当没有选中 Task 时显示。

- **Greeting**: "AgenticOS - 今天有什么可以帮您的？"
    
- **Agent Grid (2x2)**:
    
    1. **[⚡️ 财经创作者]**: Command `/run creator`.
        
    2. **[📊 数据分析师]**: Command `/run analyst`.
        
    3. **[🤵 投资顾问]**: Command `/run investor`.
        
    4. **[⚖️ 法律顾问]**: Command `/run legal`.
        
- **Omni-Bar**: 底部输入框，Placeholder: _"Ask anything or type '/' to run an agent..."_
    

---

### 1.4 设置模态框 (Settings Modal)

体现本体论层级关系。

- **左侧导航**: `Agent Store` (Plugin 列表)。
    
- **右侧详情**: 选中某个 Agent (如 The Creator) 后显示：
    
    - **Info**: 名称、版本、描述。
        
    - **Capabilities**:
        
        - **Skills**: 列表 (e.g., `News_Scan`), 支持编辑 Prompt。
            
        - **Commands**: 列表 (e.g., `/scan`), 映射关系。
            
        - **Connects**: 列表 (e.g., `YouTube`), 显示连接状态 (Connected/Disconnected)。
            

---

### 1.5 画布交互 (Canvas Interaction - v1.0 Pilot)

针对 "The Creator" 场景，Canvas 渲染 **Mobile Previewer**。

- **容器**: 模拟 iPhone 16 Pro 机身。
    
- **组件**:
    
    - **Carousel**: 顶部轮播图。支持 `Cover Image` (React 组件渲染) 和 `Logic Chart` (ECharts)。
        
    - **Text Editor**: 底部文案区域，支持直接编辑。
        
- **Actions**:
    
    - `Regenerate Cover`: 重新生成封面。
        
    - `Download All`: 下载资源包。
        
    - `✨ Save as Skill`: 触发结晶流程。
        

---

## 2. 数据架构设计 (Data Architecture)

采用 PostgreSQL + JSONB。

### 2.1 静态本体 (Ontology)

SQL

```
-- 插件/数字员工
CREATE TABLE plugin (
    id VARCHAR PRIMARY KEY, -- "agent_creator_v1"
    name VARCHAR NOT NULL,
    capabilities JSONB -- { "connects": [...], "skills": [...] }
);

-- 技能 (含用户结晶技能)
CREATE TABLE skill (
    id VARCHAR PRIMARY KEY,
    plugin_id VARCHAR REFERENCES plugin(id),
    name VARCHAR NOT NULL,
    definition JSONB, -- { "inputs": [...], "workflow": [...] }
    created_by_user_id VARCHAR, -- NULL for system skills
    is_public BOOLEAN DEFAULT FALSE
);
```

### 2.2 运行时 (Runtime)

SQL

```
-- 任务实例
CREATE TABLE task (
    id VARCHAR PRIMARY KEY,
    user_id VARCHAR INDEX,
    plugin_id VARCHAR REFERENCES plugin(id),
    title VARCHAR,
    status VARCHAR, -- "running", "finished", "archived"
    
    -- Sidebar 状态
    is_pinned BOOLEAN DEFAULT FALSE,
    is_archived BOOLEAN DEFAULT FALSE,
    
    created_at TIMESTAMP,
    last_active_at TIMESTAMP
);

-- 任务事件 (核心：Stream Persistence)
CREATE TABLE task_event (
    id SERIAL PRIMARY KEY,
    task_id VARCHAR REFERENCES task(id),
    sequence_no INT,
    role VARCHAR, -- "user", "assistant", "system"
    type VARCHAR, -- "message", "nexus_render"
    
    content_text TEXT, -- 聊天文本
    render_payload JSONB, -- Nexus Protocol UI指令
    
    created_at TIMESTAMP
);
```

### 2.3 资产与自动化

SQL

```
-- 数字资产
CREATE TABLE artifact (
    id VARCHAR PRIMARY KEY,
    task_id VARCHAR REFERENCES task(id),
    type VARCHAR, -- "image", "pdf"
    display_name VARCHAR,
    storage_uri VARCHAR,
    
    -- 状态
    is_pinned BOOLEAN DEFAULT FALSE,
    is_archived BOOLEAN DEFAULT FALSE -- 随 Task 联动
);

-- 自动化触发器
CREATE TABLE trigger (
    id VARCHAR PRIMARY KEY,
    user_id VARCHAR,
    target_skill_id VARCHAR REFERENCES skill(id),
    cron_expression VARCHAR, -- "0 8 * * *"
    is_active BOOLEAN
);
```

---

## 3. Nexus Protocol v1.0 定义 (Communication Protocol)

后端 (FastAPI) 通过 SSE (Server-Sent Events) 推送给前端 (Zustand Store) 的标准 JSON 结构。

### 3.1 响应结构 (Response Schema)

JSON

```
{
  "trace_id": "uuid-v4",
  "task_id": "task_123",
  "agent_state": "thinking", // thinking | acting | finished
  
  // Channel 1: 对话流 (左侧)
  "stream_message": {
    "delta": "正在分析马斯克的推特...", // 增量文本
    "tool_call": { "name": "deep_reader", "status": "running" } // 工具状态条
  },

  // Channel 2: 画布渲染 (右侧)
  "render_instruction": {
    "target_zone": "canvas",
    "component_type": "mobile_previewer",
    "mode": "overwrite", // overwrite | append | update
    
    "payload": {
      "platform": "xiaohongshu",
      "branding": "Powered by DAAS Bot",
      "slides": [
        { 
          "type": "cover_image", 
          "data": { "title": "太空算力爆发", "style": "cyberpunk" } 
        }
      ],
      "copywriting": "..."
    },
    
    "actions": [
      { "label": "✨ Save as Skill", "trigger_intent": "crystallize" }
    ]
  }
}
```

---

## 4. 初始化配置 (Bootstrapping)

### 4.1 "The Creator" Plugin Manifest

这是系统初始化时写入数据库的第一个 Agent。

JSON

```
{
  "plugin_id": "agent_creator_v1",
  "name": "The Financial Creator",
  "capabilities": {
    "connects": [
      { "id": "youtube", "auth_type": "api_key" },
      { "id": "notebooklm", "auth_type": "oauth" }
    ],
    "skills": [
      {
        "id": "news_scan",
        "name": "Market News Scanner",
        "description": "Scan Twitter/News for specific keywords."
      },
      {
        "id": "viral_gen",
        "name": "Viral Content Generator",
        "description": "Generate RedNote/Twitter content with images."
      }
    ]
  }
}
```

---

## 5. 开发实施注意事项

1. **Sidebar 状态联动**: 前端必须实现 `useTaskStore`。当用户点击 Archive 时，不仅要更新 DB，还要立刻过滤 `state.artifacts` 列表，确保视觉一致性。
    
2. **Stream 恢复**: 当用户点击历史 Task 时，前端需调用 `/api/tasks/{id}/events`。渲染时，遇到 `type="nexus_render"` 的事件，必须无声地恢复 Canvas 的状态，但不重复播放动画。
    
3. **水印强制**: 后端 `render_instruction` 中的 `branding` 字段必须硬编码写入，不可由 LLM 生成，防止 Agent 遗漏。
    
