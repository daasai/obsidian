# Domain Model v1.1: The Creator Pipeline (Code-Ready)

**Context:** Transitioning Nexus-AGI from a generic Chatbot to a Tabbed App OS.

**Scope:** Backend Database Schema (Postgres/SQLModel) & Frontend Type Definitions (TypeScript).

---

## 1. Backend Schema (Python / SQLModel)

> **File:** `backend/app/models/creator_pipeline.py`
> 
> **Directives:**
> 
> 1. Use `sqlalchemy.dialects.postgresql.JSONB` for all dict fields.
>     
> 2. Ensure `foreign_key` constraints are explicit.
>     

Python

```
from typing import Optional, List, Dict, Any
from datetime import datetime
from uuid import UUID, uuid4
from sqlmodel import SQLModel, Field, Relationship, Column
from sqlalchemy.dialects.postgresql import JSONB

# Enums
class PipelineStatus(str, Enum):
    ACTIVE = "active"
    PAUSED = "paused"
    ARCHIVED = "archived"

class MediaItemStage(str, Enum):
    SCOUTED = "scouted"     # Raw signal found
    PROPOSED = "proposed"   # Angle generated (In Inbox)
    DRAFTING = "drafting"   # Agent is writing...
    READY = "ready"         # Ready for review (In Studio)
    PUBLISHED = "published" # Exported/Done

# --- 1. Pipeline Task (The "App" Instance) ---
# Extends the generic Task concept into a long-running application
class PipelineTask(SQLModel, table=True):
    __tablename__ = "pipeline_task"
    
    id: UUID = Field(default_factory=uuid4, primary_key=True)
    user_id: UUID = Field(index=True)
    
    # Identify the specific App Logic (e.g., "creator_v1", "analyst_v1")
    agent_id: str = Field(index=True) 
    
    # Display Name (e.g., "AI Tech Watch")
    name: str 
    
    # Configuration from "Build Time" (The Interview)
    # Schema: { 
    #   "keywords": ["OpenAI", "DeepSeek"], 
    #   "frequency": "realtime", 
    #   "tone": "viral", 
    #   "platform": "rednote" 
    # }
    config: Dict[str, Any] = Field(default={}, sa_column=Column(JSONB))
    
    status: PipelineStatus = Field(default=PipelineStatus.ACTIVE)
    created_at: datetime = Field(default_factory=datetime.utcnow)
    
    # Relationships
    media_items: List["MediaItem"] = Relationship(back_populates="pipeline")

# --- 2. Media Item (The Unit of Work) ---
# Represents one piece of content flowing through the factory
class MediaItem(SQLModel, table=True):
    __tablename__ = "media_item"
    
    id: UUID = Field(default_factory=uuid4, primary_key=True)
    pipeline_id: UUID = Field(foreign_key="pipeline_task.id", index=True)
    
    stage: MediaItemStage = Field(default=MediaItemStage.SCOUTED)
    
    # 1. The Scout Result (From Jina/Cron)
    # Schema: { "url": "...", "raw_title": "...", "heat_score": 98, "summary": "..." }
    source_metadata: Dict[str, Any] = Field(default={}, sa_column=Column(JSONB))
    
    # 2. The Proposal (The Angles)
    # Schema: { "angle": "Tech Deep Dive", "reasoning": "...", "target_audience": "Devs" }
    proposal_data: Dict[str, Any] = Field(default={}, sa_column=Column(JSONB))
    
    # 3. The Draft (The Artifact JSON)
    # This holds the "Paged Content" for the Canvas
    draft_content: Dict[str, Any] = Field(default={}, sa_column=Column(JSONB))
    
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)
    
    pipeline: Optional[PipelineTask] = Relationship(back_populates="media_items")
```

---

## 2. Frontend Type Definitions (TypeScript)

> **File:** `frontend/src/types/creator.ts`
> 
> **Directives:** Strict typing for the "Paged Content" protocol.

TypeScript

```
// --- 1. The Protocol (Canvas Content) ---

export type DeviceFrame = 'iphone_16_pro' | 'macbook_air' | 'ipad_pro';
export type SlideTheme = 'black_gold' | 'vitality_pop' | 'minimalist_white';

// The root object stored in MediaItem.draft_content
export interface PagedArtifact {
  type: 'paged_content';
  meta: {
    device_frame: DeviceFrame;
    theme: SlideTheme;
    aspect_ratio: '9:16' | '16:9' | '3:4';
  };
  slides: Slide[];
}

export interface Slide {
  id: string;
  layout: 'cover_modern' | 'text_heavy' | 'chart_focus' | 'cta_end';
  // Atomic elements for HITL editing
  elements: {
    title?: string;
    subtitle?: string;
    body_text?: string;
    image_url?: string;
    chart_config?: any; // Recharts config
    highlight_tag?: string;
  };
}

// --- 2. Workspace State (Zustand Store) ---

export interface CreatorStore {
  // Navigation
  activePipelineId: string | null;
  viewMode: 'newsroom' | 'studio'; // Mobile responsive switch
  
  // Data (Synced via React Query / SSE)
  proposals: MediaItemSummary[]; 
  
  // Studio State
  currentDraftId: string | null;
  draftContent: PagedArtifact | null;
  
  // Interaction
  isEditing: boolean;
  isGenerating: boolean; // Spinner state
  
  // Actions
  setTheme: (theme: SlideTheme) => void;
  updateSlideText: (slideId: string, key: string, value: string) => void;
}

// --- 3. API DTOs (Data Transfer Objects) ---

export interface MediaItemSummary {
  id: string;
  stage: 'proposed' | 'drafting' | 'ready' | 'published';
  heat_score: number;
  title: string;
  source: string;
  created_at: string;
}
```

---

## 3. Migration Strategy (Instructions for Antigravity)

1. **Generate Migration**: Run `alembic revision --autogenerate -m "add_creator_pipeline"` to create the tables.
    
2. **Seed Data**: Create a file `backend/scripts/seed_creator_v1.py`.
    
    - Create 1 `PipelineTask`: "AI Tech Watch".
        
    - Create 1 `MediaItem` (Stage: `PROPOSED`): "DeepSeek V4 News".
        
    - Create 1 `MediaItem` (Stage: `READY`): "OpenAI Sora Update" (Populate `draft_content` with dummy `slides`).
        
3. **API Endpoints**: Scaffold `backend/app/api/v1/creator.py`:
    
    - `GET /pipelines/{id}/inbox` -> Returns list of `MediaItems`.
        
    - `GET /items/{id}` -> Returns full details (inc. draft).
        
    - `POST /items/{id}/draft` -> Triggers Agent to generate draft from proposal.
        
    - `PATCH /items/{id}/content` -> Saves user edits (HITL).
        

---