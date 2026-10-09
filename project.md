# Personal OS - Projects Module Architecture V1

## 1. Global Design System Reminder (Architectural Brutalism)
- **Projects Accent Color:** `Concrete Blue (#6e8ca0)` (used for active project progress bars and checked task states).
- **Layout:** Strictly list/table based. Do NOT build a drag-and-drop Trello/Kanban board. Kanban creates UI bloat. We use structured, brutalist lists (Row borders only, minimal padding) to maintain the paper-and-ink aesthetic.
- **Rules:** Tasks before analytics. Structure before color. Function before customization.

---

## 2. Projects Module Requirements (Core Architecture)
The Projects module handles active building, coding, and creating. It follows a strict hierarchy: **Project -> Milestone -> Task**.

### Functional Specs:
1. **Hierarchy & Organization:**
   - **Projects:** The high-level container (e.g., "Personal OS App", "Audio DSP Plugin"). Contains global metadata like GitHub repo links, local directory paths, and live URLs.
   - **Milestones:** Act as phases or version releases (e.g., "V1 Database Schema", "Beta Launch").
   - **Tasks:** Actionable items with binary states (`todo`, `in_progress`, `completed`).
2. **Task Lifecycle:**
   - Automatically track timestamp state changes: `created_at`, `started_at` (when shifted to in_progress), `completed_at` (when checked off).
3. **Daily Queue Integration:**
   - Tasks can be assigned a `deadline`. 
   - Tasks with a deadline of today automatically surface in the global "Today's Queue" on the dashboard. Checking a project task off in the global queue updates this module's database record instantly.

---

## 3. Sprint Module Integration
- Project Tasks can optionally be linked to a `sprint_id`.
- If a Task is assigned to the *currently active Sprint*, it will automatically bypass the backlog and show up in the Sprint Dashboard view alongside active Study topics.
- **Rule:** Keep it simple. No story points or velocity tracking—just a relational link to time-box the work.

---

## 4. SQLite Database Schema

```sql
-- PROJECTS CORE
CREATE TABLE projects (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT,
    status TEXT CHECK(status IN ('active', 'backlog', 'paused', 'completed')) DEFAULT 'backlog',
    repo_url TEXT,            -- e.g., GitHub link
    local_path TEXT,          -- e.g., '~/dev/personal-os'
    live_url TEXT,            -- e.g., Vercel/Render deployment link
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    completed_at DATETIME
);

CREATE TABLE project_milestones (
    id TEXT PRIMARY KEY,
    project_id TEXT REFERENCES projects(id) ON DELETE CASCADE,
    title TEXT NOT NULL,      -- e.g., "Phase 1: Backend Setup"
    status TEXT CHECK(status IN ('todo', 'active', 'completed')) DEFAULT 'todo',
    order_index INTEGER NOT NULL
);

CREATE TABLE project_tasks (
    id TEXT PRIMARY KEY,
    milestone_id TEXT REFERENCES project_milestones(id) ON DELETE CASCADE,
    sprint_id TEXT REFERENCES sprints(id) ON DELETE SET NULL,
    title TEXT NOT NULL,
    status TEXT CHECK(status IN ('todo', 'in_progress', 'completed')) DEFAULT 'todo',
    deadline DATE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    started_at DATETIME,
    completed_at DATETIME
);
```

---

## 5. UI Layout Specifications

### A. Projects Dashboard View (The Portfolio)
- **Max-width:** 1100px.
- Display a minimalist table of all `active` projects.
- Include a 6px `Concrete Blue` progress bar for each project, calculated by `(completed_tasks / total_tasks) * 100`.

```text
  ACTIVE PROJECTS (Table - Row borders only)
  -----------------------------------------------------------------------------
  Project Name          Milestone           Progress (6px Bar)      Links
  Personal OS App       Backend Setup       [======----------]      [Repo]
  Audio DSP Pedal       Hardware UI         [==========------]      [Local]
```

### B. Project Detail View (The Workspace)
- **Max-width:** 900px.
- **Header:** Displays Project Title, Status Dropdown, and quick links (Repo, Local Path) formatted as secondary text (`#9ca3af`).
- **Body:** Renders each Milestone as a section header (uppercase, tracking-widest). Underneath, a flat list of Tasks using the Brutalist Checkbox.

```text
[BREADCRUMB: Projects / Personal OS App]
---------------------------------------------------------------------------------
STATUS: [Active]    REPO: [github.com/user/os](https://github.com/user/os)    LOCAL: ~/dev/personal-os

MILESTONE: V1 DATABASE SCHEMA (Active)
+-------------------------------------------------------------------------------+
| [PaperCard] Tasks                                                             |
| [X] Design SQLite schema for Study module              Completed (Oct 08)     |
| [ ] Design SQLite schema for Projects module           In Progress [Sprint 1] |
| [ ] Write database migration script                    Queued                 |
+-------------------------------------------------------------------------------+

MILESTONE: FRONTEND DASHBOARD (Todo)
+-------------------------------------------------------------------------------+
| [PaperCard] Tasks                                                             |
| [ ] Setup Tailwind with Brutalist config               Queued                 |
| [ ] Build custom Brutalist Checkbox component          Queued                 |
+-------------------------------------------------------------------------------+
```