
---

# Agent Behavior Specification v1.1: The Creator Pipeline (Code-Ready)

**Context:** Defining the dual-mode logic for the Creator Agent: "The Interviewer" (Build Time) and "The Editor" (Run Time).

**Target:** `backend/app/agents/creator/`

---

## 1. Build Time: "The Interviewer"

**Context:** User is in the **[ 💬 Console ]** Tab.

**Trigger:** User types `/hire creator` or clicks "New Pipeline".

**Goal:** Extract a structured `PipelineConfig` JSON to initialize a `PipelineTask`.

### A. System Prompt

Plaintext

```
ROLE: You are the "Chief Editor" of Nexus-AGI. Your job is to interview the user to set up a new Content Pipeline.

OBJECTIVE:
Gather the following information to configure a `PipelineTask`:
1. **Topic/Niche**: What specific domain to monitor? (e.g., "Tesla Stock", "AI News")
2. **Frequency**: Real-time or Daily Digest?
3. **Tone/Persona**: (e.g., "Professional Analyst", "Viral Sarcastic", "Tech Explainer")
4. **Platform**: (e.g., "RedNote" (Vertical), "Twitter" (Text-heavy), "LinkedIn")

BEHAVIOR:
- Ask ONE question at a time. Do not overwhelm the user.
- Suggest options if the user is vague (e.g., "For AI news, do you want deep tech dives or market gossip?").
- Once you have all 4 fields, output the Final Configuration Block.

OUTPUT FORMAT (Final Step Only):
:::pipeline_config
{
  "name": "AI Tech Watch",
  "keywords": ["AI", "LLM", "DeepSeek", "OpenAI"],
  "schedule": "realtime",
  "tone": "viral_tech_bro",
  "target_platform": "rednote"
}
:::
```

### B. Logic Hook (Backend)

- **Watcher:** The backend must parse the stream for `:::pipeline_config`.
    
- **Action:** When detected, automatically create a `PipelineTask` in DB and notify frontend to refresh Sidebar.
    

---

## 2. Run Time: "The Editor" (Background Worker)

**Context:** User is in the **[ 📱 App ]** Tab (or offline).

**Trigger:** Cron Job (Scout) OR User Click (Draft).

**Mode:** Headless (No chat interaction).

### Phase A: The Scout (News -> Proposal)

**Input:** `PipelineTask.config` (Keywords).

**Tools:** `JinaReader`, `GoogleSearch`.

**Logic:**

1. Search for news in the last 24h.
    
2. Filter duplicates.
    
3. **LLM Call (Scoring):** "Rate this news event (0-100) based on 'Virality' and 'Relevance' to {keywords}."
    
4. **Output:** Create `MediaItem` with `stage='PROPOSED'` and `heat_score`.
    

### Phase B: The Drafter (Proposal -> Draft)

**Input:** `MediaItem.source_data` + `PipelineTask.config.tone`.

**Trigger:** User clicks `[Approve]` in UI.

**System Prompt:**

Plaintext

```
ROLE: You are an expert Social Media Content Creator.
TASK: Convert the provided News Summary into a Viral Carousel (Multi-slide).

CONTEXT:
- Tone: {tone} (e.g. "Sarcastic")
- Platform: {platform} (e.g. "RedNote")

OUTPUT SCHEMA:
You MUST output a valid JSON artifact fitting the `paged_content` schema.

RULES:
1. **Slide 1 (Hook)**: Must have a clickbait title. High contrast.
2. **Slide 2-4 (Value)**: Break down the news into 3 bullet points per slide. No walls of text.
3. **Slide 5 (CTA)**: "Follow for more".
4. **Visuals**: Suggest an `image_prompt` for each slide (our renderer will handle generation later).

JSON OUTPUT:
:::artifact
{
  "type": "paged_content",
  "meta": { "theme": "black_gold", "aspect_ratio": "9:16" },
  "slides": [
    {
      "id": "s1",
      "layout": "cover_modern",
      "elements": { "title": "DeepSeek Just Killed OpenAI?", "subtitle": "The V4 Benchmark is insane." }
    },
    ...
  ]
}
:::
```

---

## 3. Human-in-the-Loop (HITL) Protocol

**Context:** User edits text in the **Studio Panel**.

### A. Direct Manipulation (Fast)

- **User Action:** User types in the iPhone frame.
    
- **Frontend:** Optimistically updates React State.
    
- **Backend:** Sends `PATCH /media_items/{id}` with `draft_content` update. **No LLM involved.**
    

### B. AI Refinement (Slow)

- **User Action:** User clicks "Regenerate" on Slide 3.
    
- **Frontend:** Sends `POST /media_items/{id}/slides/s3/refine` with instruction: "Make it punchier".
    
- **Backend:**
    
    1. Loads _only_ Slide 3 context.
        
    2. Calls LLM: "Rewrite this slide content to be 'punchier'. Keep JSON structure."
        
    3. Returns updated JSON fragment.
        

---

## 4. Implementation Checklist for Antigravity

1. **Create Prompts**: Add `prompts/creator_interviewer.py` and `prompts/creator_drafter.py`.
    
2. **Background Job**: Implement a `Celery` or `FastAPI BackgroundTasks` worker for the "Scout" phase (Phase A).
    
3. **API Endpoints**:
    
    - `POST /api/v1/pipelines`: Accepts `pipeline_config` JSON.
        
    - `POST /api/v1/media_items/{id}/approve`: Triggers "The Drafter" (Phase B).
        
4. **Parser**: Ensure the `ArtifactParser` in `chat_stream` can handle the `paged_content` type.
    

---