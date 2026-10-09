# Personal OS - Goals & Achievements Module Architecture V1

## 1. Global Design System Reminder (Architectural Brutalism)
- **Goals Accent Color:** `Concrete Yellow (#c8a97e)` (used for active goal progress and achievement timestamps).
- **Layout:** Strictly chronological and list-based. No digital trophies, badges, or gamified confetti. Achievements are logged like a professional ledger or a minimalist resume.
- **Rules:** Content before decoration. Progress before metrics. Function before customization.

---

## 2. Module Requirements (Core Architecture)

### A. Active Goals (The Future)
- High-level objectives that span across pillars (e.g., "Publish Audio DSP Plugin", "Maintain 8.7+ CGPA", "Hit 80kg Bench Press").
- Goals have a `target_date` and a `category` (Study, Project, Health, Career, Finance).

### B. Achievements Ledger (The Past)
- A permanent, reverse-chronological log of completed goals and standalone milestones (e.g., "Led team in college hackathon", "Deployed secure browser-in-VM prototype").
- **Integration:** When a Project is marked `completed` in the Projects module, or a Subject is finished in the Study module, the system should offer a one-click prompt to log it as a permanent Achievement.

---

## 3. SQLite Database Schema

```sql
-- ACTIVE GOALS
CREATE TABLE goals (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT,
    category TEXT CHECK(category IN ('study', 'project', 'health', 'career', 'finance', 'personal')) DEFAULT 'personal',
    status TEXT CHECK(status IN ('active', 'completed', 'abandoned')) DEFAULT 'active',
    target_date DATE,             -- Expected completion date
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    completed_at DATETIME         -- Stamped when status changes to 'completed'
);

-- ACHIEVEMENTS LOG (The Brag Document)
CREATE TABLE achievements (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT,             -- Impact or takeaway (e.g., "Built using React & Tailwind")
    category TEXT,                -- Maps to the same categories as above
    achieved_on DATE NOT NULL,    -- The date the win happened
    reference_id TEXT,            -- Optional FK to a project_id or subject_id for linking
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## 4. UI Layout Specifications

### A. Goals & Milestones Dashboard View
- **Max-width:** 900px.
- **Top Section:** Active Goals (sorted by nearest `target_date`).
- **Bottom Section:** The Achievement Ledger (Grouped by Year/Month).

```text
  GOALS & TARGETS
  +-------------------------------------------------------------------------------+
  | [PaperCard] Active Objectives                                [+ Create Goal]  |
  |                                                                               |
  | CAREER & PROJECTS                                                             |
  | [ ] Deploy Web-Based Video Editor (Lumina)        Target: Nov 15              |
  |                                                                               |
  | STUDY & ACADEMICS                                                             |
  | [ ] Master n8n AI Automation Workflows            Target: Dec 01              |
  |                                                                               |
  | HEALTH                                                                        |
  | [ ] Hit 80kg Bench Press                          Target: Dec 31              |
  +-------------------------------------------------------------------------------+

  ACHIEVEMENT LEDGER
  +-------------------------------------------------------------------------------+
  | 2026                                                         [+ Log Manual]   |
  |                                                                               |
  | OCT 26    Built secure browser-in-VM prototype (Rust, Crosvm)   [Project]     |
  | AUG 26    Published Digital Guitar Pedal Interface to Web       [Project]     |
  | AUG 26    Achieved 8.7 CGPA in 3rd Year CSE-AIML                [Study]       |
  | JUL 26    Led Student Team in Semester Hackathon                [Career]      |
  |                                                                               |
  | 2025                                                                          |
  | DEC 25    Built Single-File Voxel Sandbox Game (Three.js)       [Project]     |
  +-------------------------------------------------------------------------------+
```

### B. Global Dashboard Widget Integration
Active goals don't need to clutter the "Today's Queue" unless the `target_date` is this week. Instead, they sit in a passive widget to keep you aligned.

```text
CURRENT FOCUS (Dashboard Widget)
---------------------------------------------------------------------------------
[Goal] Deploy Web-Based Video Editor (Lumina) - 38 days left
[Goal] Master n8n AI Automation Workflows - 53 days left
```

---

## 5. React Component Guidance for the AI
- **Form Handling:** The `<AddAchievementForm/>` should include an optional dropdown to auto-fill details from completed Projects or Study subjects via the `reference_id`.
- **Grouping Logic:** For the Achievement Ledger, fetch all achievements, sort them by `achieved_on` descending, and write a utility function in JavaScript to group them by `YYYY` so you can render the year headers cleanly without repeating them.
- **Typography:** The Year headers in the ledger should use the Section Title spec: `text-sm uppercase tracking-widest font-medium text-text-secondary`.