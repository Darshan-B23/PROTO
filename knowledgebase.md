# Personal OS - Knowledge, Input Queue & Hobbies Architecture V1

## 1. Global Design System Reminder (Architectural Brutalism)
- **Accent Colors:**
  - Knowledge Base: `Concrete Black (#31353c)` (Neutral, text-heavy, raw information).
  - Hobbies (Creative/Leisure): `Concrete Pink (#d69695)`.
- **Layout:** High text density. The Knowledge Base should look like a raw physical notebook or technical documentation. No WYSIWYG rich text editor bloat—pure Markdown.
- **Rules:** Content before decoration. Space before borders. Function before customization.

---

## 2. Module Requirements (Core Architecture)

### A. Knowledge Base (Second Brain)
- A repository for standalone concepts, code snippets, and system configurations that don't belong to a specific active project.
- **Tag-Based:** Uses a comma-separated tag system instead of rigid folders (e.g., tagging a note with `arch-linux, hyprland, config`).

### B. Input Queue (Consumption Buffer)
- A holding area for articles, videos, GitHub repositories, or tutorials you discover while working. 
- Prevents context-switching. You drop the URL here and return to your active work.

### C. Hobbies & Leisure Tracker
- A low-friction logger for unstructured activities (e.g., playing PS5, practicing guitar, tinkering with local LLMs).
- Unlike "Study" or "Projects," there are no deadlines or statuses. You just log a session and add optional notes.

---

## 3. SQLite Database Schema

```sql
-- 1. KNOWLEDGE BASE
CREATE TABLE knowledge_notes (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    content TEXT,                 -- Stored as raw Markdown
    tags TEXT,                    -- Comma-separated string: "dsp,c++,audio"
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 2. INPUT QUEUE
CREATE TABLE input_queue (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    url TEXT NOT NULL,
    content_type TEXT CHECK(content_type IN ('article', 'video', 'repo', 'book', 'other')) DEFAULT 'article',
    status TEXT CHECK(status IN ('queued', 'consumed', 'archived')) DEFAULT 'queued',
    added_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    consumed_at DATETIME
);

-- 3. HOBBIES
CREATE TABLE hobbies (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,           -- e.g., "Electric Guitar", "Gaming"
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE hobby_sessions (
    id TEXT PRIMARY KEY,
    hobby_id TEXT REFERENCES hobbies(id) ON DELETE CASCADE,
    duration_minutes INTEGER,
    notes TEXT,                   -- e.g., "Practiced sweep picking", "Chiaki-ng stream setup"
    logged_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## 4. UI Layout Specifications

### A. Knowledge Base View (Two-Pane Workspace)
- **Max-width:** 1200px (wider to accommodate the dual-pane layout).
- **Left Pane (300px):** Search bar and a flat list of recent notes.
- **Right Pane (Flex):** The Brutalist Markdown Editor.

```text
  KNOWLEDGE BASE
  +---------------------------+---------------------------------------------------+
  | SEARCH: [ hyprland_ ]     | TITLE: Hyprland Waybar Workspace Bind Fix         |
  |                           | TAGS: [linux] [wm] [scripting]                    |
  | RECENT NOTES              |                                                   |
  | - Audio DSP Ring Buffers  | ------------------------------------------------- |
  | - n8n Local Workflow Spec | [Markdown Textarea - borderless, surface bg]      |
  | - Crosvm Network Config   |                                                   |
  | - Hyprland Waybar Bind..  | The issue with the workspace bind messing up      |
  | - FastAPI Auth Boilerplate| the .lua config is caused by...                   |
  |                           |                                                   |
  |                           | ```bash                                           |
  |                           | bind = $mainMod, 1, workspace, 1                  |
  |                           | ```                                               |
  +---------------------------+---------------------------------------------------+
```

### B. Input Queue View (The Buffer)
- **Max-width:** 900px.
- A flat, chronological list sorted by `added_at` ascending (oldest first).

```text
  INPUT QUEUE (UNREAD)                                       [+ Add URL]
  +-------------------------------------------------------------------------------+
  | [PaperCard] Articles & Repositories                                           |
  |                                                                               |
  | [ ] Building a Non-Linear Video Editor in React (Article)       [Open Link]   |
  |     Added: Oct 02                                                             |
  |                                                                               |
  | [ ] llama.cpp ROCm optimization guide (Repo)                    [Open Link]   |
  |     Added: Oct 05                                                             |
  +-------------------------------------------------------------------------------+
```

### C. Hobbies View (The Leisure Log)
- **Max-width:** 900px.

```text
  HOBBIES & LEISURE
  +-------------------------------------------------------------------------------+
  | [PaperCard] Log a Session                                                     |
  | SELECT: [Electric Guitar | v]   DURATION: [ 60 ] min                          |
  | NOTES:  [ Worked on digital pedal interface aesthetics ]         [ LOG IT ]   |
  +-------------------------------------------------------------------------------+
  
  RECENT SESSIONS
  Oct 08 - PlayStation 5 (90 min) - Configured Chiaki-ng for remote play on Linux
  Oct 06 - Electric Guitar (45 min) - Audio DSP basics in C++
```

---

## 5. React Component Guidance for the AI
- **Markdown Rendering:** In the Knowledge Base, use a lightweight parser like `react-markdown`. Style the output using Tailwind Typography (`prose prose-sm prose-neutral`) but override blockquotes and code blocks to use the Brutalist `#f9fafb` surface color and `#f3f4f6` borders. No rounded corners larger than `4px` on code blocks.
- **Tag Input:** The Knowledge Base tags input should be a simple comma-separated text string before saving to the DB. Don't build a complex tokenized input field; keep it raw text.
- **Input Queue Global Add:** The `<TopBar/>` of the main application layout MUST contain a "Quick Add to Queue" button so the user can dump a URL into the database from anywhere in the app without navigating to the Queue page.