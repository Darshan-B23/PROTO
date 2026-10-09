# Personal OS - Health, Nutrition & Fitness Module Architecture V2

## 1. Global Design System Reminder (Architectural Brutalism)
- **Health Accent Colors:** 
  - General Health/Hygiene: `Concrete Green (#738276)`
  - Fitness/Exercise: `Concrete Red (#c26e60)`
- **Layout:** High-density, minimalist paper ledger. No gamification, no radial progress dials for macros. Use simple horizontal 6px bars or plain text fractions (e.g., `120g / 150g Protein`).
- **Rules:** Tasks before analytics. Structure before color. Function before customization.

---

## 2. Module Requirements (Core Architecture)

### A. General Habits & Hygiene
- Tracks daily metrics (boolean checkboxes or numeric inputs like water intake).

### B. Nutrition & Recipe Engine
- **Recipe Book:** Store meals with base macro values (Calories, Protein, Carbs, Fat).
- **Daily Food Log:** Select a recipe from the book, enter the number of servings, and the system automatically calculates and aggregates daily macros.

### C. Scheduled Fitness
- **Day-Based Workouts:** Assign specific exercises to specific days of the week (e.g., "Monday: Bench Press", "Tuesday: 5km Run"). 
- **Future-Proofing:** Kept simple for now (just checkboxes for the day's exercises), but isolated in its own table so we can add sets, reps, and weight tracking later.

---

## 3. SQLite Database Schema

```sql
-- 1. GENERAL HABITS (Hygiene, Skincare, Water)
CREATE TABLE habit_definitions (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,          
    category TEXT NOT NULL,       
    input_type TEXT CHECK(input_type IN ('boolean', 'numeric')) DEFAULT 'boolean',
    status TEXT CHECK(status IN ('active', 'archived')) DEFAULT 'active',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE habit_logs (
    id TEXT PRIMARY KEY,
    habit_id TEXT REFERENCES habit_definitions(id) ON DELETE CASCADE,
    log_date DATE NOT NULL,       -- YYYY-MM-DD
    value TEXT NOT NULL,          
    logged_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(habit_id, log_date)
);

-- 2. NUTRITION & MACROS
CREATE TABLE recipes (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    calories INTEGER DEFAULT 0,
    protein INTEGER DEFAULT 0,   -- in grams
    carbs INTEGER DEFAULT 0,     -- in grams
    fat INTEGER DEFAULT 0,       -- in grams
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE food_logs (
    id TEXT PRIMARY KEY,
    log_date DATE NOT NULL,      -- YYYY-MM-DD
    recipe_id TEXT REFERENCES recipes(id) ON DELETE CASCADE,
    servings REAL DEFAULT 1.0,   -- Multiplier for macros
    logged_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 3. FITNESS & EXERCISE
CREATE TABLE scheduled_exercises (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,         -- e.g., "Bench Press 3x10", "Yoga"
    day_of_week TEXT NOT NULL,   -- 'monday', 'tuesday', etc. (lowercase)
    status TEXT CHECK(status IN ('active', 'archived')) DEFAULT 'active'
);

CREATE TABLE exercise_logs (
    id TEXT PRIMARY KEY,
    exercise_id TEXT REFERENCES scheduled_exercises(id) ON DELETE CASCADE,
    log_date DATE NOT NULL,      -- YYYY-MM-DD
    completed BOOLEAN DEFAULT true,
    logged_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(exercise_id, log_date)
);
```

---

## 4. UI Layout Specifications

### A. Health & Fitness Dashboard View (The Ledger)
- **Max-width:** 900px.
- **Top Section:** Nutrition Summary (calculated dynamically from `food_logs` joined with `recipes` for today's date).
- **Middle Section:** Today's Workout (filtered by current day of the week).
- **Bottom Section:** General routines.

```text
  HEALTH & FITNESS LEDGER: TODAY
  
  NUTRITION SUMMARY
  +-------------------------------------------------------------------------------+
  | Calories: 1850 / 2500 kcal     Protein: 120g    Carbs: 200g     Fat: 55g      |
  |                                                                               |
  | [PaperCard] Logged Meals                                    [+ Add Meal]      |
  | - Chicken & Rice (x1)               600 kcal | 50g P | 70g C | 10g F          |
  | - Protein Shake (x1.5)              300 kcal | 45g P | 10g C |  5g F          |
  +-------------------------------------------------------------------------------+

  TODAY'S WORKOUT (MONDAY)
  +-------------------------------------------------------------------------------+
  | [PaperCard] Exercises                                   [+ Edit Schedule]     |
  | [X] Bench Press 3x10                                                          |
  | [ ] Incline Dumbbell Press 3x12                                               |
  | [ ] Tricep Pushdowns 4x15                                                     |
  +-------------------------------------------------------------------------------+

  DAILY ROUTINES
  +-------------------------------------------------------------------------------+
  | [PaperCard] Hygiene & Tracking                         [+ Manage Routines]    |
  | [X] Morning Cleanser + Moisturizer                                            |
  | [ ] 2.5L Water Intake                       [ Input: 1.5 ]                    |
  +-------------------------------------------------------------------------------+
```

### B. Daily Queue Integration (Main Dashboard Widget)
How this entire module compresses onto the global "Today's Queue" alongside your Study and Project tasks:

```text
TODAY'S QUEUE
---------------------------------------------------------------------------------
SCHEDULED FOCUS (From Study & Projects)
[ ] 14:00 - Audio DSP Hardware UI (Project)

TODAY'S FITNESS (Monday)
[X] Bench Press 3x10
[ ] Incline Dumbbell Press 3x12

DAILY ROUTINES
[X] Morning Cleanser + Moisturizer
[ ] 2.5L Water Intake: [ Input: ____ ]
```

---

## 5. React Component Guidance for the AI
- **Date Handling:** Ensure all `log_date` queries use local timezone dates formatted exactly as `YYYY-MM-DD` to prevent midnight rollover bugs.
- **Macro Calculation:** When querying `food_logs` for a specific day, calculate total macros in the backend by multiplying the `recipes` columns by the `food_logs.servings` column, returning a single aggregated object to the frontend.
- **Exercise Filtering:** The frontend should determine the current day of the week (e.g., `'monday'`), and request only `scheduled_exercises` where `day_of_week === 'monday'` to populate the workout checklist.