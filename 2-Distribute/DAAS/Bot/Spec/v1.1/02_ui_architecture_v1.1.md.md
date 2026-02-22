
---

# UI Architecture v1.1: The Tabbed Workspace (Code-Ready)

**Context:** Implementing the "OS-like" Tabbed Interface and the Creator App.

**Tech Stack:** React, Tailwind CSS, Zustand, Lucide React, Framer Motion.

---

## 1. Global Layout Refactor (`src/layouts`)

We are moving from a static "Sidebar + Chat" layout to a dynamic "App Launcher + Tab Container" layout.

代码段

```
graph TD
    Root[RootLayout] --> Sidebar[GlobalSidebar]
    Root --> Main[MainArea]
    Main --> TabBar[WorkspaceTabBar]
    Main --> Content[TabContentContainer]
    Content --> Console[ConsoleView (Chat)]
    Content --> Creator[CreatorApp (New!)]
```

### Component Specifications

#### `GlobalSidebar.tsx` (Updated)

- **Role:** App Launcher & Status Monitor.
    
- **Props:** None (Connects to `useTaskStore`).
    
- **Changes:**
    
    - **Section 1: "My Pipelines"** (New): Lists active `PipelineTasks`.
        
        - Renders `PipelineItem` component: Icon + Name + Notification Badge (Red Dot).
            
        - _On Click:_ Calls `workspaceStore.openTab(taskId)`.
            
    - **Section 2: "Automations"**: Existing chat history (Legacy).
        

#### `WorkspaceTabBar.tsx` (New)

- **Role:** Navigation between open contexts.
    
- **State:** `useWorkspaceStore` (`activeTabId`, `openTabs`).
    
- **UI:** Horizontal list of tabs.
    
    - **Fixed Tab:** `[ 💬 Console ]` (Always present, cannot close).
        
    - **Closable Tabs:** `[ 📱 AI Tech Watch (x) ]`.
        
    - _Visuals:_ Active tab has `border-b-2 border-primary` and `bg-zinc-50`.
        

---

## 2. The Creator App (`src/apps/creator`)

This is the "Light App" rendered when a Creator Tab is active. It uses a **Master-Detail** layout.

### Component Tree (ASCII)

Plaintext

```
CreatorApp.tsx (Provider: CreatorStoreContext)
├── Header.tsx (App-specific control)
│   ├── AppMeta (Icon + Status: "Monitoring")
│   └── ViewToggle (Switch: "Config Chat" vs "App Manager")
│
└── Body.tsx (Grid: 30% Left | 70% Right)
    ├── NewsroomPanel.tsx (The Inbox)
    │   ├── FilterBar (Search input, "Unread" toggle)
    │   └── ProposalList (Scrollable)
    │       ├── ProposalCard (Type: PROPOSED)
    │       │   ├── HeatBadge (e.g. "🔥 98")
    │       │   ├── SourceInfo
    │       │   └── ActionButtons (Approve / Reject)
    │       └── DraftCard (Type: DRAFTING | READY)
    │           ├── StatusIndicator (Spinner or Check)
    │           └── DraftPreview (Thumbnail)
    │
    └── StudioPanel.tsx (The Workspace)
        ├── EmptyState (If no draft selected)
        └── EditorWorkspace (If draft active)
            ├── StudioToolbar (Top)
            │   ├── ThemeSelector (Dropdown)
            │   └── DeviceToggle (iPhone/iPad)
            │
            ├── CanvasStage (Center - Dark bg)
            │   └── DeviceFrame (CSS Container: iPhone 16 Pro)
            │       └── CarouselRenderer
            │           ├── SlideView (Swiper/Framer)
            │           └── EditableText (ContentEditable)
            │
            └── ActionFooter (Bottom Floating)
                ├── RegenButton (Regenerate current slide)
                └── ExportButton (Download ZIP)
```

---

## 3. State Management (`src/stores`)

We separate the **Global Workspace State** from the **Local App State**.

### A. `useWorkspaceStore.ts` (Global)

Controls "Which tab is open?"

TypeScript

```
interface WorkspaceStore {
  activeTabId: string; // 'console' | uuid
  openTabs: Array<{ id: string; type: 'console' | 'pipeline'; title: string }>;
  
  openTab: (task: PipelineTask) => void;
  closeTab: (taskId: string) => void;
  switchToConsole: () => void;
}
```

### B. `useCreatorStore.ts` (Local per App)

Controls the data inside the Creator App.

_Note: This should likely be a Context-based store provider so multiple Creator tabs don't share state._

TypeScript

```
interface CreatorState {
  // Data
  pipelineId: string;
  items: MediaItem[]; // Fetched from backend
  
  // Selection
  selectedItemId: string | null;
  
  // Editor State
  isEditing: boolean;
  currentSlideIndex: number;
  
  // Actions
  fetchItems: () => Promise<void>;
  approveProposal: (id: string) => Promise<void>; // Triggers Agent
  updateDraftContent: (id: string, path: string, value: any) => void; // Optimistic update
}
```

---

## 4. CSS / Styling Strategy (Tailwind)

### The "Device Frame" (Crucial for immersion)

We need a reusable `DeviceFrame` component that isolates styles.

TypeScript

```
// src/components/ui/DeviceFrame.tsx
export const DeviceFrame = ({ children }) => (
  <div className="relative mx-auto border-gray-800 dark:border-gray-800 bg-gray-800 border-[14px] rounded-[2.5rem] h-[600px] w-[300px] shadow-xl">
    {/* Dynamic Island */}
    <div className="w-[148px] h-[18px] bg-black top-0 rounded-b-[1rem] left-1/2 -translate-x-1/2 absolute"></div>
    
    {/* Screen Content */}
    <div className="h-[32px] w-[3px] bg-gray-800 absolute -left-[17px] top-[72px] rounded-l-lg"></div>
    <div className="rounded-[2rem] overflow-hidden w-full h-full bg-white dark:bg-black">
      {children}
    </div>
  </div>
);
```

---

## 5. Implementation Steps for Antigravity

1. **Refactor Layout**: Update `MainLayout` to support the Tab system first. Ensure the "Console" tab works exactly like the current chat.
    
2. **Scaffold Creator**: Create the `src/apps/creator` folder and the basic `CreatorApp` shell.
    
3. **Build Newsroom**: Implement `NewsroomPanel` fetching data from the mock API (defined in Doc 1).
    
4. **Build Studio**: Implement the `DeviceFrame` and `CarouselRenderer`.
    
5. **Wire Logic**: Connect `useCreatorStore` to handle the "Approve -> Drafting -> Preview" flow.
    

---