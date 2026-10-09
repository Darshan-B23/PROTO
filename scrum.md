# Personal OS - Scrum / Active Sprint Module Architecture V1

## 1. Global Design System Reminder (Architectural Brutalism)
- **Scrum Accent Color:** `Concrete Black (#31353c)` (High-focus, neutral utility).
- **Layout:** A brutalist Kanban alternative. Instead of heavy, floating drag-and-drop cards with shadows, use structured vertical lists grouped by status (Todo, In Progress, Done) with minimal padding and raw text.
- **Rules:** Tasks before analytics. Structure before color. Function before customization. No complex Jira mechanics (no story points, no velocity charts).

---

## 2. Scrum Module Requirements (Core Architecture)

### Functional Specs:
1. **The Aggregator:** The Scrum board is a unified view. It does not store its own tasks. Instead, it queries the `project_tasks` (from Projects) and `topics` (from Study) tables where the `sprint_id` matches the currently active Sprint.
2. **Sprint Lifecycle:** Sprints have three states: `planning`, `active`, and `completed`.
3. **Sprint Planning (Backlog):** A dedicated view to create a new Sprint, assign start/end dates, and select tasks/topics from your active projects and subjects to attach to the Sprint.
4. **Active Board:** A 3-column workspace showing what you are building and studying *right now*.

---

## 3. SQLite Database Schema

*Note to AI Coder: The `topics` and `project_tasks` tables already have a `sprint_id` foreign key. This is just the table for the Sprints themselves.*

```sql
CREATE TABLE sprints (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,           -- e.g., "Sprint 3: Core Infrastructure"
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status TEXT CHECK(status IN ('planning', 'active', 'completed')) DEFAULT 'planning',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## 4. UI Layout Specifications

### A. Active Scrum Board View
- **Max-width:** 1200px (Wide to accommodate 3 columns).
- **Top Section:** Sprint metadata and a 6px `Concrete Black` progress bar.
- **Grid:** Use a 3-column CSS Grid. Use simple bottom-borders to separate items inside the columns. No card shadows.

```text
  ACTIVE SPRINT: Sprint 3 (Oct 12 - Oct 25)                   [Complete Sprint]
  Progress: [==========------] 60%                            5 Days Remaining

  +-------------------------+-------------------------+-------------------------+
  | TODO                    | IN PROGRESS             | DONE                    |
  | ----------------------- | ----------------------- | ----------------------- |
  | [Project: Personal OS]  | [Project: Personal OS]  | [Study: Audio DSP]      |
  | Add Finance Schema      | Build Brutalist Checkbox| Ring Buffers Concept    |
  |                         | Started: Oct 08         | Completed: Oct 07       |
  | ----------------------- | ----------------------- | ----------------------- |
  | [Study: OS]             | [Project: Video Editor] |                         |
  | Memory Paging           | Fix Timeline State      |                         |
  |                         | Started: Oct 09         |                         |
  +-------------------------+-------------------------+-------------------------+
```

---

## 5. React Component Guidance for the AI
- **Unified Backend Endpoint:** Create a specific backend endpoint (e.g., `GET /api/sprints/active/board`). This endpoint should query both `project_tasks` and `topics` by `sprint_id`, map them to a standardized object format (e.g., `{ id, type: 'project' | 'study', parent_name, title, status, timestamp }`), and return a unified array.
- **Status Updates:** Since the items belong to different parent tables, ensure the frontend sends the `type` back in the PUT request so the backend knows whether to update the `status` column in `project_tasks` or `topics`.
- **Drag and Drop:** If implementing drag-and-drop, use a lightweight library like `@hello-pangea/dnd` or `dnd-kit`. When a card is dropped into a new column, instantly fire the PUT request to update its status. If skipping DnD to keep it strictly brutalist, render a simple `<select>` dropdown inside each item to change its status.