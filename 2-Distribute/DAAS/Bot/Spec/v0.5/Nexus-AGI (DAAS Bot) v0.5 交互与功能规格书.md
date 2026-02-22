
---
**模块**: UX Architecture & Core Mechanics

**版本**: v1.0 (The Studio Edition)

**日期**: 2026-02-10

**核心理念**: **Chat is Programming, Canvas is Product.** (对话即编程，画布即产品)

---

## 1. 交互架构：双流设计 (The Dual-Stream Architecture)

我们将屏幕空间在逻辑上和视觉上划分为两个平行的世界，彻底解耦“过程”与“结果”。

### 1.1 左侧：The Stream (心流/过程)

- **定义**: 自然语言编程环境 (Natural Language IDE) 与 调试控制台。
    
- **职能**:
    
    - **沟通 (Communication)**: 用户输入指令，表达意图。
        
    - **编排 (Orchestration)**: 展示 Agent 的思考链 (CoT)、工具调用日志、错误重试。
        
    - **迭代 (Refinement)**: 对右侧产品的修改指令（如“换个颜色”、“语气再夸张点”）。
        
- **数据特性**: **过程性 (Procedural)**。它是构建产品的“脚手架”，虽然会被保存为历史记录，但其核心价值在于“生成当前的结果”。
    

### 1.2 右侧：The Canvas (画布/产品)

- **定义**: 多态的交付容器 (Polymorphic Delivery Container)。
    
- **职能**:
    
    - **呈现 (Presentation)**: 渲染最终的知识资产（Artifact）。
        
    - **交互 (Interaction)**: 提供针对特定内容的原生交互（如缩放图表、复制卡片、编辑文档）。
        
    - **状态 (State)**: **持久性 (Persistent)**。它是这一轮对话产出的“实体”，拥有独立的版本控制。
        
- **多态性 (Polymorphism)**: Canvas 根据 Artifact 的类型自动切换渲染引擎：
    
    - `Type: SocialCard` -> **手机预览模拟器** (小红书/推特样式)。
        
    - `Type: Report` -> **富文本编辑器** (Notion 样式)。
        
    - `Type: Dashboard` -> **BI 仪表盘** (可交互图表)。
        

---

## 2. 核心机制：技能结晶 (Skill Crystallization)

这是将“一次性对话”转化为“可复用资产”的关键机制，也是 Nexus-AGI 的**“Zero-Code”**核心。

### 2.1 概念定义

用户与 Agent 的每一次成功协作（例如：调试好了一个“小红书爆款文案生成”流程），都可以被系统“结晶”为一个标准化的 **Skill**。

- **Chat** = 编写代码的过程。
    
- **Crystallize** = 编译 (Compile)。
    
- **Skill** = 可执行程序 (.exe)。
    

### 2.2 用户旅程 (The User Journey)

1. **原型阶段 (Prototype)**: 用户在 Stream 中通过多轮对话，调整好了满意的结果（如：“把这个URL转成小红书卡片，要这种语气”）。
    
2. **结晶触发 (Trigger)**: 点击界面上的 ✨ **"Save as Skill"** 按钮。
    
3. **自动编译 (Auto-Synthesis)**:
    
    - 系统后台的 `Architect Agent` 分析对话历史。
        
    - **变量提取**: 识别出 URL 是变量 (`{input_url}`)，语气是参数 (`{tone}`)。
        
    - **逻辑固化**: 生成一段标准的 Markdown 流程描述。
        
4. **技能生成 (Deployment)**: 侧边栏 "My Skills" 出现新项目 "小红书生成器"。
    
5. **复用 (Re-use)**: 下次点击该 Skill，Stream 界面变为**表单模式 (Form Mode)**，只需填入 URL 即可运行。
    

---

## 3. 功能详细规格 (Functional Specs)

### 3.1 消息协议升级 (Protocol Upgrade)

后端 API 的响应不再是单一的文本流，而是结构化的“双通道”数据。

JSON

```
// Backend Response Schema
{
  "stream_content": "正在为您读取 TechCrunch 的文章... 分析完成。", // 显示在左侧
  "artifact": { // 显示在右侧 (Optional)
    "id": "art_20240210_001",
    "type": "social_card_xhs",
    "version": 1,
    "payload": {
      "title": "DeepSeek V3 深度解析",
      "tags": ["#AI", "#Tech"],
      "body": "...",
      "theme": "red_alert"
    }
  },
  "crystallization_hint": true // 提示前端显示“保存为技能”按钮
}
```

### 3.2 技能文件标准 (Skill Artifact Standard)

系统自动生成的 `.md` 文件结构：

Markdown

```
---
name: "Viral Tech Card Generator"
description: "Convert any tech article URL into a Xiaohongshu-style viral card."
author: "User_01"
created_at: "2026-02-10"
---

## Inputs
- `target_url`: string (Description: The link to the article)
- `tone`: string (Default: "Exciting", Options: ["Exciting", "Professional", "Sarcastic"])

## Workflow
1. **Read**: Call `DeepReader.read_url({target_url})`.
2. **Analyze**: Extract 3 key bullet points. Calculate virality score.
3. **Draft**: Rewrite summary using `{tone}` style. Add 5 emojis.
4. **Render**: Output `SocialCardData` for Canvas rendering.

## Prompt Template
"You are a social media expert. The user wants a {tone} summary of..."
```

---

## 4. v1.0 实施路线图 (Implementation Plan)

### Phase 1: 基础双流架构 (The Split)

- **前端**:
    
    - 实现 `SplitLayout` 组件 (Left: Chat / Right: Canvas)。
        
    - 使用 Zustand 建立 `artifactStore`，监听后端 WebSocket/SSE 消息。
        
    - 实现第一个 Canvas 组件: `SocialCardPreview` (手机壳样式)。
        
- **后端**:
    
    - 升级 `Agent.run()` 返回结构，支持 `artifact` 字段注入。
        

### Phase 2: 交互闭环 (The Loop)

- **前端**:
    
    - 实现 **“局部引用更新”**：当用户在左侧说“改个标题”时，后端返回新的 `artifact` 版本，右侧无刷新更新内容 (Hot Reload)。
        
- **后端**:
    
    - 优化 Prompt，让 Agent 理解“修改现有 Artifact”的指令。
        

### Phase 3: 结晶器 (The Crystallizer)

- **后端**:
    
    - 开发 `SkillSynthesizer` (元认知 Agent)。
        
    - 实现将生成的 Markdown 保存到 `backend/skills/custom/` 目录。
        
- **前端**:
    
    - 开发 "My Skills" 侧边栏面板。
        
    - 开发 **通用表单渲染器** (根据 Skill 的 Inputs 定义自动生成输入框)。
        

---

## 5. 结语：产品形态的终极差异

|**特性**|**传统 Chatbot (ChatGPT/Claude)**|**Nexus-AGI (v1.0)**|
|---|---|---|
|**交互核心**|文本流 (Text Stream)|**画布 (Canvas) + 工作流 (Workflow)**|
|**产出物**|聊天记录 (History)|**数字资产 (Artifacts)**|
|**复用性**|每次都要重新 Prompt|**一键保存为 Skill (SOP)**|
|**用户心智**|咨询顾问|**数字员工 / 生产力工厂**|

这份文档将作为我们 v1.0 开发的**法律**。它定义了 Nexus-AGI 如何从众多 AI 玩具中脱颖而出，成为真正的生产力工具。