
# 📖 Product Requirement: The Creator Pipeline (v1.1)

| **文档属性** | **详情**                                                   |
| -------- | -------------------------------------------------------- |
| **版本**   | v1.1.0 (Draft)                                           |
| **代号**   | "The Newsroom" (编辑部)                                     |
| **核心目标** | 将 Chatbot 升级为“常驻型 AI 媒体工作台”，实现从情报监控到爆款内容的**全自动闭环**。      |
| **用户画像** | **Alex (独立创作者)**：需要每天发小红书/推特，但痛恨找选题、痛恨排版，只希望做“主编”而非“美工”。 |

---

## 🧩 Epic 1: Pipeline Construction (构建与初始化)

_用户不再是每次都在“新建对话”，而是“雇佣”一个长期的数字员工。_

### US-1.1: The Hiring Interview (雇佣面试)

- **As a** 用户 (Alex),
    
- **I want to** 通过自然对话配置我的媒体管道,
    
- **So that** 我不需要填写复杂的表单就能建立个性化的监控任务。
    
- **Acceptance Criteria (AC):**
    
    1. 用户在 Console 输入 `/hire` 或点击 "New Pipeline" 后，Agent 进入**面试模式**。
        
    2. Agent 必须依次询问 4 个关键要素：**Topic** (关注什么), **Frequency** (频率), **Tone** (毒舌/专业), **Platform** (小红书/推特)。
        
    3. 面试结束后，Agent 输出一个 `Pipeline Configuration Card` 供用户确认。
        
    4. 用户点击 "Confirm" 后，侧边栏 "My Pipelines" 区域出现该任务。
        

### US-1.2: The Dedicated Workspace (专属工作台)

- **As a** 用户,
    
- **I want to** 点击侧边栏的任务后打开一个独立的 Tab,
    
- **So that** 我可以在一个全屏的、沉浸式的环境里工作，而不被其他聊天打扰。
    
- **AC:**
    
    1. 点击 Sidebar 里的 "AI Tech Watch" 任务。
        
    2. 顶部 Tab 栏新增并切换到 `[ 📱 AI Tech Watch ]`。
        
    3. 界面布局从“聊天流”切换为 **"Newsroom (左) + Studio (右)"** 的双栏布局。
        
    4. 该 Tab 的状态需要持久化（刷新页面后 Tab 依然存在）。
        

---

## 🧩 Epic 2: The Newsroom (情报与提案)

_系统在后台默默工作，用户只看结果。_

### US-2.1: Passive Monitoring & Notification (被动监控)

- **As a** 用户,
    
- **I want to** 在我不操作时系统自动抓取新闻,
    
- **So that** 我每次打开应用都有新选题在等我。
    
- **AC:**
    
    1. 后台 Job 每 4 小时根据关键词 (e.g. "DeepSeek") 抓取一次 Jina/Google。
        
    2. 当发现高热度新闻（Heat Score > 80）时，自动生成 `Proposal`。
        
    3. 侧边栏的任务图标上显示**红色数字徽标** (e.g., `🔴 2`)。
        

### US-2.2: Proposal Inbox (提案收件箱)

- **As a** 用户,
    
- **I want to** 在左侧列表快速浏览 AI 推荐的选题,
    
- **So that** 我能像刷邮件一样快速决定“做”还是“不做”。
    
- **AC:**
    
    1. 左侧 Newsroom 面板按时间倒序展示 Proposal Cards。
        
    2. **Type A (待处理)** 卡片显示：
        
        - 🔥 热度分 (颜色区分：红>80, 黄>50)。
            
        - 新闻标题 + 来源摘要。
            
        - `[Approve]` (批准) 和 `[Reject]` (拒绝) 按钮。
            
    3. 点击 `Reject`，卡片带动画消失。
        
    4. 点击 `Approve`，卡片进入 "Drafting..." 状态（转圈动画），触发 Agent 生成内容。
        

---

## 🧩 Epic 3: The Studio (生产与编辑)

_这是核心魔法时刻。所见即所得。_

### US-3.1: The Mobile Preview (真机预览)

- **As a** 用户,
    
- **I want to** 在一个逼真的手机框里查看生成的内容,
    
- **So that** 我能确切知道它发到小红书上长什么样。
    
- **AC:**
    
    1. 右侧主舞台居中渲染一个 **iPhone 16 Pro** 的 CSS 容器（带刘海、圆角、阴影）。
        
    2. 容器内渲染 Agent 生成的 `Carousel` (滑动卡片)。
        
    3. 支持点击左右箭头或底部圆点进行**翻页**。
        

### US-3.2: Click-to-Edit (点选编辑)

- **As a** 主编 (Editor),
    
- **I want to** 点击手机屏幕上的任何文字直接修改,
    
- **So that** 我不需要在那跟 AI 废话“把第一页标题改短点”，而是直接上手改。
    
- **AC:**
    
    1. 鼠标悬停在标题或正文上时，显示虚线边框（暗示可编辑）。
        
    2. 点击文字，变为输入框 (Input/Textarea)，且自动获取焦点。
        
    3. 输入修改内容，鼠标点击外部 (Blur) 后，自动保存修改到本地 State。
        
    4. 修改必须是**无损**的（不影响卡片样式）。
        

### US-3.3: Visual Regeneration (图片抽卡)

- **As a** 用户,
    
- **I want to** 在不改变文案的情况下换一张配图,
    
- **So that** 我能找到最符合意境的背景图。
    
- **AC:**
    
    1. 鼠标悬停在图片区域时，右上角浮现 `[🔄 Regen Image]` 按钮。
        
    2. 点击后，图片区域显示 Loading Spinner。
        
    3. 前端调用 API，Agent 根据当前页面的文案重新生成/搜索一张新图并替换。
        

### US-3.4: Theme Switching (一键换肤)

- **As a** 用户,
    
- **I want to** 快速切换整套 PPT 的视觉风格,
    
- **So that** 我能找到最吸睛的配色。
    
- **AC:**
    
    1. Studio 顶部工具栏提供 "Theme" 下拉菜单。
        
    2. 选项包括：`Black Gold` (黑金/高端), `Vitality Pop` (高饱和/年轻), `Minimalist` (极简/知识)。
        
    3. 切换后，所有 Slide 的字体、背景色、装饰元素**毫秒级**实时更新。
        

---

## 🧩 Epic 4: Delivery (交付与反馈)

_完成“最后一公里”。_

### US-4.1: One-Click Export (一键导出)

- **As a** 用户,
    
- **I want to** 把做好的图下载下来,
    
- **So that** 我可以直接上传到媒体平台。
    
- **AC:**
    
    1. Studio 底部悬浮栏显示 `[🚀 Export]` 按钮。
        
    2. 点击后，前端将当前 Canvas 的 6 张 Slide 渲染为高清 PNG (1080x1920)。
        
    3. 打包为 ZIP 下载，文件名为 `{Date}_{Topic}.zip`。
        
    4. 左侧 Newsroom 里的卡片状态变为 `Published` (绿色)。
        

---

## 🧩 Epic 5: Workspace Management (系统级交互)

### US-5.1: Context Switching (上下文切换)

- **As a** 高级用户,
    
- **I want to** 在“修改配置”和“日常使用”之间切换,
    
- **So that** 我既能调整 Agent 的设定，又不影响当前的工作流。
    
- **AC:**
    
    1. 在 App Tab 的右上角提供 `[🛠️ Config Chat]` 切换按钮。
        
    2. 点击后，Tab 内容翻转回传统的 Chat Stream (Console 视图)。
        
    3. 用户在 Chat 里输入“把语气改得更幽默一点”，Agent 确认。
        
    4. 用户切回 `[📱 App Manager]`，新生成的 Draft 将应用新语气。
        

---

### 📝 附录：非功能性需求 (NFR)

1. **响应速度**: 点击 "Approve" 后，Draft 生成时间不应超过 **15秒** (流式输出优先)。
    
2. **数据安全**: 用户的 API Key 和生成的草稿必须按 `user_id` 隔离，严禁越权访问。
    
3. **兼容性**: Studio 的手机预览框必须在 13寸笔记本和 27寸显示器上都能自适应居中。
    

---
