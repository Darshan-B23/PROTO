# Personal OS - Study Module & Sprint Architecture V1

## 1. Global Design System (Architectural Brutalism)
The UI must strictly adhere to the following design constraints. Do NOT use Tailwind components that conflict with this.
- **Theme:** Architectural Brutalism (Tactile, paper-and-ink, focused, no glassmorphism, no heavy shadows).
- **Colors:**
  - Background: `#ffffff`
  - Surface: `#f9fafb`
  - Text Primary: `#111827`, Secondary: `#9ca3af`
  - Borders: `#f3f4f6`
  - Study/Learning Accent: `Concrete Purple (#8d7a8b)`
- **Typography:** System default sans-serif (Inter, SF Pro). No decorative fonts.
- **Components:**
  - Cards: White bg, 1px border (`#f3f4f6`), no shadow, 12px radius, 24px padding.
  - Checkboxes (Critical): Unchecked is a thin border. Checked fills with `Concrete Purple` and gets a hard shadow: `2px 2px 0px rgba(0,0,0,1)`.
  - Tables: Row borders only, no vertical grid lines, generous padding.
  - Progress Bars: 6px height, rounded, full width, filled with accent color.
- **Rules:** Tasks before analytics. Structure before color. Function before customization.

---

## 2. Study Module Requirements (Core Architecture)
The Study module handles deliberate learning via a strict hierarchy: **Subject -> Unit -> Topic**.

### Functional Specs:
1. **Hierarchy & Organization:**
   - Subjects group multiple Units.
   - Units act as chapters/milestones.
   - Topics are actionable items with statuses (`todo`, `in_progress`, `completed`).
2. **Topic Lifecycle:**
   - Automatically track timestamp state changes: `created_at` (default), `started_at` (on status change to in_progress), `completed_at` (on status change to completed).
   - Support a user-assigned `deadline` (Date).
3. **Daily Queue Integration:**
   - If a Topic's deadline is approaching or is today, it automatically surfaces in the global "Today's Queue" on the dashboard. Checking it off in the queue completes the Topic record.
4. **Material Storage (PDF/PPT):**
   - Topics can have attached materials stored securely on the local backend (not cloud).
   - `PDF` files should be rendered natively inline using a standard browser `<iframe>` or `<embed>`.
   - `PPT/PPTX` files cannot be rendered natively. Provide a direct "Download/Open Externally" link, or implement a backend conversion to PDF.

---

## 3. Sprint Module (Decoupled Architecture)
*Note to AI Coder: Build the Sprint UI/Logic as a completely separate object/feature flag, but ensure the database schema relationships exist from Day 1.*

- **Concept:** A minimalist time-box (e.g., "Sprint 1: Oct 12 - Oct 25").
- **Function:** Group high-priority Topics (and later, Projects) across various Subjects into a single focused dashboard view.
- **Rule:** NO complex Scrum overhead (no story points, no velocity charts). It is strictly a container with a start date and end date to filter the active backlog.

---

## 4. SQLite Database Schema

```sql
-- SPRINTS (Independent Module, but relational)
CREATE TABLE sprints (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status TEXT CHECK(status IN ('active', 'completed')) DEFAULT 'active'
);

-- STUDY CORE
CREATE TABLE subjects (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    status TEXT CHECK(status IN ('active', 'archived')) DEFAULT 'active',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE units (
    id TEXT PRIMARY KEY,
    subject_id TEXT REFERENCES subjects(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    order_index INTEGER NOT NULL
);

CREATE TABLE topics (
    id TEXT PRIMARY KEY,
    unit_id TEXT REFERENCES units(id) ON DELETE CASCADE,
    sprint_id TEXT REFERENCES sprints(id) ON DELETE SET NULL,
    title TEXT NOT NULL,
    status TEXT CHECK(status IN ('todo', 'in_progress', 'completed')) DEFAULT 'todo',
    deadline DATE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    started_at DATETIME,
    completed_at DATETIME
);

CREATE TABLE topic_materials (
    id TEXT PRIMARY KEY,
    topic_id TEXT REFERENCES topics(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    file_path TEXT NOT NULL, -- Local path, e.g., 'uploads/os/unit2.pdf'
    file_type TEXT CHECK(file_type IN ('pdf', 'pptx', 'note')),
    uploaded_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

5. UI Layout Specifications
A. Subject Detail View
Max-width: 900px - 1100px.

Minimal text tables for Units and Topics.

Display a 6px progress bar for each Subject based on completed vs. total Topics.

B. Topic Focus View (The Workspace)
[BREADCRUMB: Subject Name / Unit Name / Topic Title]
---------------------------------------------------------------------------------
STATUS: [Dropdown: In Progress]    DEADLINE: [Date Picker]    SPRINT: [Dropdown]
DATES: Started [Date] | Completed [Date]

[MATERIALS & RESOURCES SECTION]
- File Name.pdf       [View Inline Button]
- Presentation.pptx   [Download Button]

+-------------------------------------------------------------------------------+
| INLINE DOCUMENT VIEWER (Embedded PDF)                                         |
| +---------------------------------------------------------------------------+ |
| | <embed src="/uploads/..." width="100%" height="600px" />                  | |
| +---------------------------------------------------------------------------+ |
+-------------------------------------------------------------------------------+

[TOPIC NOTES / TEXT EDITOR AREA]
(Markdown editor or simple textarea for quick notes linked to this topic)