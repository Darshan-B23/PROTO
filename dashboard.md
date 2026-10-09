# Personal OS - Global Dashboard Architecture V1

## 1. Global Design System Reminder (Architectural Brutalism)
- **Dashboard Philosophy:** The dashboard is a landing pad, not a playground. It must answer one question instantly: *"What needs to be done today?"*
- **Layout Priority:** 
  1. Today's Queue (Must be visible without scrolling).
  2. Current Focus (Active Sprint).
  3. Quick Stats & Input Queue.
- **Rules:** Tasks before analytics. Structure before color. Function before customization.

---

## 2. Dashboard Module Requirements (Core Architecture)

### Functional Specs:
1. **The Aggregator:** The Dashboard does not have its own database tables. It relies on a unified backend payload that queries `project_tasks`, `topics`, `habit_logs`, `scheduled_exercises`, and `sprints`.
2. **Today's Queue:** The most critical component. It lists:
   - Study Topics and Project Tasks with a `deadline` of today.
   - Active Health routines (Hygiene, Nutrition).
   - Scheduled Fitness exercises matching the current day of the week.
3. **Active Sprint Overview:** A highly compressed view of the current Scrum board so you can see sprint progress without navigating away.
4. **Global Quick Add:** A persistent UI element to instantly drop a URL into the Input Queue or jot down a Knowledge Base note without breaking workflow.

---

## 3. SQLite Database Schema
*Note to AI Coder: No new schema is required for this module. The backend should construct a unified response by querying the existing tables based on `CURRENT_DATE`, the current day of the week, and `sprint.status = 'active'`.*

---

## 4. UI Layout Specifications

- **Max-width:** 1100px.
- **Structure:** A single-column vertical feed. Do not use complex masonry grids.

```text
  [TopBar]                                   Search... | [+ Quick Add to Queue]

  TODAY'S QUEUE
  ---------------------------------------------------------------------------------
  SCHEDULED FOCUS (Deadlines)
  [ ] 14:00 - Audio DSP Hardware UI                   (Project Task)
  [ ] 16:00 - Virtual Memory                          (Study Topic)

  TODAY'S FITNESS (FRIDAY)
  [X] 5km Run                                         (Exercise)
  [ ] Core Routine                                    (Exercise)

  DAILY ROUTINES
  [X] Morning Cleanser + Moisturizer                  (Boolean Habit)
  [ ] 2.5L Water Intake                               (Numeric Habit) [ Input: 1.5 ]
  
  CURRENT FOCUS (ACTIVE SPRINT)
  ---------------------------------------------------------------------------------
  Sprint 3: Core Infrastructure      Progress: [==========------] 60%
  5 Days Remaining
  
  In Progress: 2 Items
  - Build Brutalist Checkbox                          (Project Task)
  - Fix Timeline State                                (Project Task)
                                                      [Go to Full Scrum Board]

  UNREAD INPUT QUEUE
  ---------------------------------------------------------------------------------
  [ ] React Compiler Deep Dive (Article)                     [Open Link]
  [ ] llama.cpp ROCm optimization guide (Repo)               [Open Link]
```

---

## 5. React Component Guidance for the AI
- **Unified Payload:** Do not make the React frontend send 5 different API requests. Create one backend endpoint (`GET /api/dashboard/today`) that returns a single, structured JSON payload containing `tasks`, `habits`, `fitness`, `sprint_summary`, and `queue`.
- **Component Modularity:** Break the dashboard into distinct React components (`<TodayQueue />`, `<SprintWidget />`, `<InputQueueWidget />`) so they can independently handle their own `onChange` events (like checking off a habit) while hitting their respective API endpoints to update the database.
- **Brutalist Checkbox Sync:** When a user checks a box in the Dashboard, instantly update the UI (optimistic UI rendering) and fire the database update in the background to ensure it feels tactile and fast.