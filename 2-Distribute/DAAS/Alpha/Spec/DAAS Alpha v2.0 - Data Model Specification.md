

这份文档将作为后端开发（DB Schema 设计）和前端状态管理的核心依据。它整合了我们讨论过的 **IAM 体系**、**量化核心**、**AI 交互**以及 **v2.0 特有的叙事商店与归因能力**。

---

# DAAS v2.0 Data Model Specification

**Version:** 2.0.0

**Status:** Approved for Implementation

**Design Philosophy:** Domain-Driven Design (DDD), High Cohesion, Event Sourcing (Partial).

---

## 1. 核心域设计图谱 (Domain Map)

系统划分为四大核心领域，通过外键和 JSONB 弱引用进行连接。

|**领域 (Domain)**|**职责 (Responsibility)**|**核心实体 (Entities)**|
|---|---|---|
|**IAM & SaaS**|租户管理、权限控制、商业化配额|`User`, `UserQuota`, `Subscription`|
|**Quant Core**|策略定义、因子管理、回测执行|`Narrative`, `Factor`, `BacktestExecution`|
|**Interaction**|AI 对话状态、上下文记忆、GenUI|`ChatSession`, `ChatMessage`|
|**Market Data**|基础行情元数据、搜索索引|`StockMeta`|

---

## 2. 实体详细定义 (Entity Definitions)

### A. IAM & SaaS Domain (身份与商业化)

#### 1. `User` (用户)

- **Type**: Aggregate Root
    
- **Description**: 系统的核心租户。
    
- **Fields**:
    
    - `id`: `Integer` (PK)
        
    - `email`: `String` (Unique, Indexed)
        
    - `tier`: `String` (Enum: `free`, `pro`, `institutional`) - 决定配额和功能开关。
        
    - `api_key`: `String` (Unique) - 允许程序化访问。
        
    - `preferences`: `JSONB` - 用户偏好 (e.g., `{"theme": "dark", "color_scheme": "red_up_green_down", "default_risk": "low"}`).
        
    - `created_at`: `DateTime`
        

#### 2. `UserQuota` (配额)

- **Type**: Entity
    
- **Description**: 限制用户对昂贵资源（LLM, Backtest）的消耗。
    
- **Fields**:
    
    - `user_id`: `Integer` (FK -> User.id, PK)
        
    - `daily_narrative_generations`: `Integer` - 当日已生成叙事次数。
        
    - `daily_backtest_runs`: `Integer` - 当日已运行回测次数。
        
    - `last_reset_at`: `DateTime` - 上次重置时间。
        

---

### B. Quant Core Domain (量化核心)

#### 3. `Narrative` (叙事/策略)

- **Type**: Aggregate Root
    
- **Description**: 投资逻辑的容器。支持“官方模版”和“用户实例”的双层结构。
    
- **Fields**:
    
    - `id`: `Integer` (PK)
        
    - `user_id`: `Integer` (FK -> User.id, Nullable for Official Templates)
        
    - `type`: `Enum` (`template`, `instance`) - **v2.0 Feature**: 区分商店里的商品和用户手里的实例。
        
    - `forked_from_id`: `Integer` (FK -> Narrative.id) - **v2.0 Feature**: 记录血缘关系 (Lineage)。
        
    - `title`: `String`
        
    - `description`: `Text` - 自然语言描述。
        
    - `status`: `Enum` (`draft`, `active`, `archived`)
        
    - `strategy_config`: `JSONB` - **Core Logic**: 存储策略的具体参数 (e.g., `{"universe": ["AI"], "factors": [{"name": "rsi", "period": 14}]}`).
        

#### 4. `Factor` (因子)

- **Type**: Entity
    
- **Description**: System 1 (感知层) 的产出物，连接非结构化信息与量化引擎。
    
- **Fields**:
    
    - `id`: `Integer` (PK)
        
    - `name`: `String` (Unique) - e.g., `low_altitude_policy_intensity`.
        
    - `source_type`: `String` (`news`, `financial`, `tech`) - 因子来源。
        
    - `implementation_path`: `String` - 指向计算逻辑代码或数据列名。
        

#### 5. `NarrativeFactorAssociation` (叙事-因子关联)

- **Type**: Link Table (Many-to-Many)
    
- **Description**: 定义一个叙事由哪些因子组成，及其权重。
    
- **Fields**:
    
    - `narrative_id`: `Integer` (FK)
        
    - `factor_id`: `Integer` (FK)
        
    - `weight`: `Float` - 权重 (Tweakable by user).
        
    - `parameters`: `JSONB` - 特定于该因子的参数 (e.g., `{"window": 30}`).
        

#### 6. `BacktestExecution` (回测执行)

- **Type**: Entity (Immutable)
    
- **Description**: 叙事的一次具体运行快照。每次“Run Simulation”生成一条新记录。
    
- **Fields**:
    
    - `id`: `Integer` (PK)
        
    - `narrative_id`: `Integer` (FK)
        
    - `run_at`: `DateTime`
        
    - `scenario_name`: `String` (Default: `standard`) - **v2.0 Feature**: 支持压力测试场景 (e.g., `2015_crash`).
        
    - `config_snapshot`: `JSONB` - 运行时参数快照 (用于复现)。
        
    - `kpi_result`: `JSONB` - `{"sharpe": 2.4, "return": 0.18}`.
        
    - `equity_curve`: `JSONB` - 资金曲线数据点。
        
    - `attribution_metrics`: `JSONB` - **v2.0 Feature**: 归因分析 (Narrative Purity).
        

---

### C. Interaction Domain (交互域)

#### 7. `ChatSession` (会话)

- **Type**: Aggregate Root
    
- **Fields**:
    
    - `id`: `UUID` (PK)
        
    - `user_id`: `Integer` (FK)
        
    - `context_memory`: `JSONB` - **AI Feature**: 存储 Consultant Agent 对用户的长期记忆 (Intent, Risk Profile).
        

#### 8. `ChatMessage` (消息)

- **Type**: Entity
    
- **Fields**:
    
    - `id`: `Integer` (PK)
        
    - `session_id`: `UUID` (FK)
        
    - `role`: `String` (`user`, `assistant`, `tool`)
        
    - `content`: `Text`
        
    - `meta_data`: `JSONB` - **GenUI Feature**: 控制前端渲染组件 (e.g., `{"artifact_type": "config_ticket", "status": "ready"}`).
        

---

## 3. 关键关系图解 (ER Diagram Logic)

代码段

```
erDiagram
    User ||--o{ Narrative : "owns"
    User ||--o{ ChatSession : "initiates"
    User ||--|| UserQuota : "has"

    Narrative ||--o{ BacktestExecution : "generates"
    Narrative }|--|| Narrative : "forks_from"
    
    Narrative }|--|{ Factor : "composes"
    NarrativeFactorAssociation }|--|| Narrative : "links"
    NarrativeFactorAssociation }|--|| Factor : "links"

    ChatSession ||--o{ ChatMessage : "contains"
    
    %% 弱引用：聊天可以衍生出叙事
    ChatSession |o--o| Narrative : "context_for"
```

---

## 4. 关键设计决策 (Design Rationale)

1. **JSONB for Flexibility**:
    
    - **策略配置 (`strategy_config`)** 和 **回测结果 (`kpi_result`)** 使用 JSONB 存储。
        
    - _理由_：量化策略结构多变（从单因子到复杂的 AI 模型），关系型 Schema 难以适应频繁的字段变更。JSONB 兼顾了灵活性和数据库内的查询能力（PostgreSQL）。
        
2. **Snapshotting (快照机制)**:
    
    - `BacktestExecution` 存储了 `config_snapshot`。
        
    - _理由_：用户可能修改了 Narrative 的参数，但我们需要保证历史的回测记录是**可溯源、可复现**的。
        
3. **Explicit Lineage (显式血缘)**:
    
    - `forked_from_id` 在 `Narrative` 表中。
        
    - _理由_：支撑 v2.0 的“叙事商店”模式。系统需要知道用户的策略是基于哪个官方模版修改的，以便推送模版更新或进行归因分析。
        
4. **Factor as First-Class Citizen (因子即一等公民)**:
    
    - 将因子从代码逻辑中提取为 `Factor` 实体表。
        
    - _理由_：支撑 System 1 的自动化挖掘。未来可以有 Agent 自动往这个表里注册新发现的因子。
        

---

此规格说明书可直接交给后端开发人员进行 ORM (SQLAlchemy) 代码编写和数据库迁移 (Alembic) 配置。