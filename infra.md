# Personal OS - Root Infrastructure & Initialization Spec V1

## 1. Tech Stack Requirements
- **Frontend:** React (Vite), Tailwind CSS (strict Brutalist configuration), React Router for navigation.
- **Backend:** FastAPI (Python) or Node.js/Express. (FastAPI recommended for easier integration with local AI/n8n later).
- **Database:** SQLite (Local file-based, e.g., `personal_os.db`).
- **Deployment:** Docker & `docker-compose` for isolated, one-click local running.

## 2. Global Application Shell (The UI Wrapper)
The frontend must implement a persistent layout shell wrapping all the modules.

### A. The Sidebar (Left, Fixed, 240px)
- Must remember collapse state.
- Navigation links mapping to the modules:
  - Dashboard
  - Study / Skills
  - Projects
  - Scrum Board
  - Health & Fitness
  - Goals & Achievements
  - Knowledge Base
  - Finance
  - Hobbies
  - Settings (Bottom)

### B. The Settings & Data Backup Module
Because this is a local-first application using SQLite, data security is entirely on the user. The application MUST include a basic Settings page with:
1. **Database Export:** A button that triggers a download of the raw SQLite `.db` file.
2. **Theme Toggles:** (Optional) Light/Dark mode, though Brutalist defaults to Light/Surface.

## 3. Directory Structure
Instruct the AI to initialize the project with this mono-repo structure:

```text
personal-os/
├── backend/
│   ├── main.py (or index.js)
│   ├── database.py (SQLite connection)
│   ├── models/ (Schema definitions)
│   ├── routers/ (API endpoints for each module)
│   ├── uploads/ (Ignored in git - for PDFs/PPTs)
│   ├── requirements.txt
│   └── .env (Ignored in git)
├── frontend/
│   ├── index.html
│   ├── src/
│   │   ├── components/ (PaperCard, BrutalistCheckbox, etc.)
│   │   ├── pages/ (Dashboard, Study, Projects, etc.)
│   │   ├── App.jsx
│   │   └── tailwind.config.js (Contains Concrete Pigments)
│   ├── package.json
│   └── vite.config.js
├── docker-compose.yml
├── .gitignore
└── README.md
```

## 4. The Golden Rules for the AI Coder
1. **DO NOT over-engineer.** Do not add Redis, Celery, or complex caching. 
2. **DO NOT use external cloud dependencies.** No Firebase, no AWS S3, no Clerk auth. Everything must read/write to the local SQLite file and local `/uploads` folder.
3. **DO NOT gamify.** Stick strictly to the Architectural Brutalism design system (1px borders, #f9fafb backgrounds, solid concrete pigment colors).