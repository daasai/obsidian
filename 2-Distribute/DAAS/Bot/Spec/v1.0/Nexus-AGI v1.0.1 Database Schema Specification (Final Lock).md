

---

**版本**: v1.0.1

**数据库**: PostgreSQL 15+

**ORM**: SQLModel (SQLAlchemy Core)

---

## 1. 实体关系总览 (ER Logic)

我们将数据分为三层：**用户层 (User Space)**、**本体层 (Ontology Space)** 和 **运行时层 (Runtime Space)**。

- **User Space**: `User` -> `UserProfile` (含 `Preferences`) -> `UserConnection` (Connect 实例)。
    
- **Ontology Space**: `Agent` (原 Plugin) -> 包含 `Skill` -> 触发于 `Command`。
    
- **Runtime Space**: `Task` (核心) -> 产生 `TaskEvent` (流)、`Artifact` (资产)、`TaskPlan` (动态规划)、`TaskContext` (短期记忆)、`TaskMemory` (长期总结)。
    

---

## 2. 详细 Schema 定义代码

### Module A: 用户与权限 (User Space)

Python

```
from sqlmodel import SQLModel, Field, Relationship
from typing import List, Optional, Dict
from datetime import datetime
from sqlalchemy import Column, JSON, Text, Boolean, ForeignKey

class User(SQLModel, table=True):
    """
    基础用户表 (Auth)
    """
    __tablename__ = "user"
    
    id: str = Field(primary_key=True)  # UUID
    email: str = Field(unique=True, index=True)
    hashed_password: str
    is_active: bool = Field(default=True)
    created_at: datetime = Field(default_factory=datetime.utcnow)

    # 关联
    profile: Optional["UserProfile"] = Relationship(back_populates="user")
    tasks: List["Task"] = Relationship(back_populates="user")
    connections: List["UserConnection"] = Relationship(back_populates="user")

class UserProfile(SQLModel, table=True):
    """
    用户画像与偏好 (User Profile & Preferences)
    """
    __tablename__ = "user_profile"
    
    user_id: str = Field(foreign_key="user.id", primary_key=True)
    nickname: str
    avatar_url: Optional[str] = None
    
    # 核心：用户偏好与原则 (JSONB)
    # e.g., { "theme": "dark", "language": "zh", "risk_level": "medium", "output_style": "concise" }
    preferences: Dict = Field(default={}, sa_column=Column(JSON))
    
    # 核心：协作原则 (Collaborative Principles)
    # e.g., ["Always cite sources", "No emojis in reports"]
    principles: List[str] = Field(default=[], sa_column=Column(JSON))

    user: Optional[User] = Relationship(back_populates="profile")
```

### Module B: 本体论定义 (Ontology Space)

Python

```
class Agent(SQLModel, table=True):
    """
    数字员工/智能助手 (原 Plugin) - 静态定义
    """
    __tablename__ = "agent"
    
    id: str = Field(primary_key=True)  # e.g., "agent_creator_v1"
    name: str
    description: str
    avatar_url: str
    
    # 系统提示词模板 (The Brain's Persona)
    role_prompt: str = Field(sa_column=Column(Text))
    
    # 能力元数据快照 (JSONB)
    # 用于前端快速渲染 Settings 页面，无需连表查询
    capabilities_snapshot: Dict = Field(default={}, sa_column=Column(JSON))
    
    # 关联
    skills: List["Skill"] = Relationship(back_populates="agent")
    commands: List["Command"] = Relationship(back_populates="agent")

class Skill(SQLModel, table=True):
    """
    原子技能 (Skill) - 含系统预置与用户结晶
    """
    __tablename__ = "skill"
    
    id: str = Field(primary_key=True)
    agent_id: str = Field(foreign_key="agent.id", index=True)
    
    name: str
    description: Optional[str] = None
    
    # 核心：技能定义 (Input Schema + Workflow Graph)
    # e.g., { "inputs": [{"name": "url", "type": "string"}], "steps": [...] }
    definition: Dict = Field(default={}, sa_column=Column(JSON))
    
    # 结晶属性
    created_by_user_id: Optional[str] = Field(default=None, index=True) # None=System, Value=User
    is_public: bool = Field(default=False)

    agent: Optional[Agent] = Relationship(back_populates="skills")
    commands: List["Command"] = Relationship(back_populates="skill")

class Command(SQLModel, table=True):
    """
    指令映射 (Command) - 触发入口
    """
    __tablename__ = "command"
    
    trigger: str = Field(primary_key=True) # e.g., "/scan"
    agent_id: str = Field(foreign_key="agent.id")
    target_skill_id: str = Field(foreign_key="skill.id")
    description: str
    
    agent: Optional[Agent] = Relationship(back_populates="commands")
    skill: Optional[Skill] = Relationship(back_populates="commands")

class UserConnection(SQLModel, table=True):
    """
    连接/权限 (Connect) - 用户实例
    存储用户授权给 Agent 的 Key/Token
    """
    __tablename__ = "user_connection"
    
    id: str = Field(primary_key=True) # UUID
    user_id: str = Field(foreign_key="user.id", index=True)
    agent_id: str = Field(foreign_key="agent.id")
    
    provider: str # e.g., "youtube", "notion_oauth"
    display_name: str # e.g., "Shawn's Notion"
    
    # 敏感字段 (需加密存储)
    encrypted_credentials: str = Field(sa_column=Column(Text))
    
    status: str = Field(default="active") # active, expired, revoked
    last_verified_at: datetime = Field(default_factory=datetime.utcnow)

    user: Optional[User] = Relationship(back_populates="connections")
```

### Module C: 运行时与状态 (Runtime Space)

Python

```
class Task(SQLModel, table=True):
    """
    任务 (Task) - 核心状态机
    """
    __tablename__ = "task"
    
    id: str = Field(primary_key=True)
    user_id: str = Field(foreign_key="user.id", index=True)
    agent_id: str = Field(foreign_key="agent.id")
    
    # 来源追踪 (Traceability)
    trigger_source: str # "chat_manual", "slash_command", "cron_trigger"
    origin_skill_id: Optional[str] = Field(foreign_key="skill.id", nullable=True)
    
    title: str
    status: str = Field(index=True) # running, waiting_input, finished, failed
    
    # Sidebar 交互状态
    is_pinned: bool = Field(default=False)
    is_archived: bool = Field(default=False)
    
    created_at: datetime = Field(default_factory=datetime.utcnow)
    last_active_at: datetime = Field(default_factory=datetime.utcnow)

    # 关联
    events: List["TaskEvent"] = Relationship(back_populates="task")
    artifacts: List["Artifact"] = Relationship(back_populates="task")
    plan: Optional["TaskPlan"] = Relationship(back_populates="task")
    context: Optional["TaskContext"] = Relationship(back_populates="task")
    memory: Optional["TaskMemory"] = Relationship(back_populates="task")
    user: Optional[User] = Relationship(back_populates="tasks")

class TaskEvent(SQLModel, table=True):
    """
    任务事件流 (Chat Stream & Protocol)
    """
    __tablename__ = "task_event"
    
    id: int = Field(primary_key=True) # Auto-increment
    task_id: str = Field(foreign_key="task.id", index=True)
    sequence_no: int
    
    role: str # user, assistant, system, tool
    type: str # message (文本), nexus_render (UI指令), tool_use (工具调用), tool_result (工具返回)
    
    # 内容载荷
    content_text: Optional[str] = Field(sa_column=Column(Text))
    
    # 核心：UI 渲染指令 (JSONB)
    # e.g. { "target_zone": "canvas", "component": "mobile_previewer", ... }
    render_payload: Optional[Dict] = Field(default=None, sa_column=Column(JSON))
    
    created_at: datetime = Field(default_factory=datetime.utcnow)
    
    task: Optional[Task] = Relationship(back_populates="events")

class TaskPlan(SQLModel, table=True):
    """
    动态规划 (Plan)
    记录 Agent 生成的步骤图 (DAG)
    """
    __tablename__ = "task_plan"
    
    task_id: str = Field(foreign_key="task.id", primary_key=True)
    
    # 步骤列表 (JSONB)
    # e.g., [{ "step_id": 1, "action": "read_url", "status": "done" }, ...]
    steps: List[Dict] = Field(default=[], sa_column=Column(JSON))
    
    current_step_index: int = 0
    updated_at: datetime = Field(default_factory=datetime.utcnow)
    
    task: Optional[Task] = Relationship(back_populates="plan")

class TaskContext(SQLModel, table=True):
    """
    任务上下文 (Context) - 短期工作台
    """
    __tablename__ = "task_context"
    
    task_id: str = Field(foreign_key="task.id", primary_key=True)
    
    # 变量表 (JSONB) - 存储提取出的实体、临时变量
    # e.g., { "target_company": "Tesla", "extracted_revenue": "25B" }
    variables: Dict = Field(default={}, sa_column=Column(JSON))
    
    # 文件引用 (JSONB) - 当前 Task 正在使用的文件列表
    active_files: List[str] = Field(default=[], sa_column=Column(JSON))
    
    task: Optional[Task] = Relationship(back_populates="context")

class TaskMemory(SQLModel, table=True):
    """
    任务记忆 (Task Memory) - 长期总结
    用于归档后的回顾或 RAG 检索
    """
    __tablename__ = "task_memory"
    
    task_id: str = Field(foreign_key="task.id", primary_key=True)
    
    summary_text: str = Field(sa_column=Column(Text)) # LLM 生成的总结
    key_takeaways: List[str] = Field(default=[], sa_column=Column(JSON))
    
    # 向量索引 ID (指向 Vector DB)
    vector_doc_id: Optional[str] = None
    
    task: Optional[Task] = Relationship(back_populates="memory")

class Artifact(SQLModel, table=True):
    """
    数字资产 (Artifact)
    """
    __tablename__ = "artifact"
    
    id: str = Field(primary_key=True)
    task_id: str = Field(foreign_key="task.id", index=True)
    user_id: str = Field(index=True)
    
    type: str # image, pdf, chart_data
    display_name: str
    storage_uri: str
    
    # 预览元数据 (JSONB) - 用于 Sidebar 缩略图
    meta_data: Dict = Field(default={}, sa_column=Column(JSON))
    
    is_pinned: bool = Field(default=False)
    # 注意：is_archived 逻辑上由 Task 决定，但这里保留字段以便独立归档
    is_archived: bool = Field(default=False)
    
    created_at: datetime = Field(default_factory=datetime.utcnow)
    
    task: Optional[Task] = Relationship(back_populates="artifacts")
```

---

## 3. 关键设计说明 (Key Design Notes)

1. **JSONB 的大量使用**: 为了应对 v1.0 到 v1.x 的快速迭代，我们在 `UserProfile.preferences`, `Skill.definition`, `TaskContext.variables` 中大量使用了 Postgres 的 JSONB 类型。这允许我们在不修改表结构的情况下增加新的偏好设置或技能参数。
    
2. **Task 状态拆分**: 我将 Task 的状态拆分为了三个表：
    
    - `TaskPlan`: 存**将来**要做什么（步骤）。
        
    - `TaskContext`: 存**现在**手头有什么（变量、文件）。
        
    - `TaskMemory`: 存**过去**发生了什么（总结）。
        
    - 这种设计（Separation of Concerns）非常利于 Agent 的推理和上下文管理。
        
3. **Agent vs Plugin**: 表名已统一为 `agent`，代码中的外键引用也全部更新为 `agent_id`。
    
4. **Nexus Protocol 支持**: `TaskEvent.render_payload` 是专门为 INF-01 协议设计的字段，它直接存储发往前端的 JSON 指令，实现了“UI 渲染历史”的持久化。
    

这份 Schema 定义已经包含了你列出的所有 13 个实体，且逻辑闭环。你可以直接将其交给 Antigravity 进行代码生成。