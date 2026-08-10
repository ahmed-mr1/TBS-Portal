# TBS Portal - Project Structure

## Repository Organization

```
TBS-System/
── frontend/                    # Next.js Frontend Application
│   ├── frontend/
│   │   ├── app/                 # App Router pages
│   │   │   ├── (auth)/          # Auth pages (login, signup)
│   │   │   ├── admin/           # Admin dashboard pages
│   │   │   ├── assistant/       # AI Assistant page
│   │   │   ├── dashboard/       # Student dashboard
│   │   │   └── page.tsx         # Landing page
│   │   │
│   │   ├── components/          # React components
│   │   │   ├── Assistant/       # AI Chat components
│   │   │   ├── Navbar.tsx       # Navigation bar
│   │   │   └── TimetableGrid.tsx
│   │   │
│   │   ├── context/             # React Context providers
│   │   │   ├── AuthContext.tsx  # Authentication state
│   │   │   ├── ChatContext.tsx  # AI Chat state
│   │   │   └── ToastContext.tsx # Notification toasts
│   │   │
│   │   ├── lib/                 # API clients & utilities
│   │   │   ├── api.ts           # Axios instance
│   │   │   ├── assistantService.ts  # AI API client
│   │   │   └── teacherService.ts
│   │   │
│   │   ├── public/              # Static assets
│   │   │   ├── tbslogo.png      # TBS Logo
│   │   │   └── tbsbg.jpg        # Background image
│   │   │
│   │   ├── package.json         # Frontend dependencies
│   │   ├── tailwind.config.ts   # Tailwind CSS config
│   │   └── next.config.ts       # Next.js config
│   │
│   └── package.json             # Root frontend package
│
├── backend/                     # FastAPI Backend Application
│   ├── app/
│   │   ├── api/                 # API route handlers
│   │   │   ├── auth.py          # Authentication endpoints
│   │   │   ├── assistant.py     # AI Assistant endpoints
│   │   │   ├── admin/           # Admin endpoints
│   │   │   └── deps.py          # Dependency injection
│   │   │
│   │   ├── core/                # Core configuration
│   │   │   ├── config.py        # Environment variables
│   │   │   ├── database.py      # Database connection
│   │   │   └── security.py      # JWT & password hashing
│   │   │
│   │   ├── models/              # SQLAlchemy ORM models
│   │   │   ├── __init__.py      # All models
│   │   │   ├── knowledge.py     # AI knowledge models
│   │   │   └── assistant.py     # Chat session models
│   │   │
│   │   ├── services/            # Business logic
│   │   │   ├── assistant_service.py  # RAG pipeline
│   │   │   ├── ingestion_service.py  # PDF ingestion
│   │   │   ├── embedding_service.py  # Vector embeddings
│   │   │   └── llm_service.py   # LLM API calls
│   │   │
│   │   ├── schemas.py           # Pydantic request/response schemas
│   │   └── main.py              # FastAPI application entry
│   │
│   ├── alembic/                 # Database migrations
│   │   ├── versions/            # Migration files (35+)
│   │   └── env.py               # Alembic configuration
│   │
│   ├── tests/                   # Backend tests
│   │   ├── test_ingestion.py
│   │   ├── test_retrieval.py
│   │   └── conftest.py
│   │
│   ├── requirements.txt         # Python dependencies
│   └── alembic.ini              # Alembic config
│
├── scripts/                     # Utility scripts
│   ├── ingest_local.py          # Local PDF ingestion
│   ├── upload_document.py       # Document upload helper
│   ├── test_pipeline_local.py   # Local RAG testing
│   └── debug_chat.py            # Chat debugging
│
├── docs/                        # Documentation & Knowledge Sources
│   ├── ai-assistant-architecture.md  # AI system design
│   ├── TBS_Handbook 2023.pdf    # Student Handbook
│   ├── TBS Guide.pdf            # Student Guide
│   └── TBS Graduates Study.pdf  # Graduate Career Study
│
├── showcase/                    # Public Documentation (This Repo)
│   ├── README.md                # Project overview
│   ├── ARCHITECTURE.md          # Technical architecture
│   ├── FEATURES.md              # Feature documentation
│   ├── ROADMAP.md               # Development roadmap
│   └── .gitignore
│
├── .github/
│   └── workflows/               # CI/CD pipelines
│       ├── deploy.yml           # Backend deployment
│       └── deploy-frontend.yml  # Frontend deployment
│
├── docker-compose.yml           # Docker development stack
── .gitignore                   # Git ignore rules
└── package.json                 # Root package.json
```

## Key Directories

### `/frontend/frontend/app/` - Pages

Contains all Next.js pages using the App Router:

- `(auth)/` - Login, signup, password reset
- `admin/` - Admin dashboard (15+ pages)
- `assistant/` - AI Assistant chat interface
- `dashboard/` - Student dashboard
- Public pages: `/`, `/about`, `/calendar`, `/clubs`, etc.

### `/backend/app/api/` - API Endpoints

RESTful API endpoints organized by feature:

- `auth.py` - Authentication (login, signup, JWT)
- `assistant.py` - AI chat endpoints
- `admin/` - Admin-specific endpoints
- Domain-specific: `timetables.py`, `clubs.py`, `calendar.py`, etc.

### `/backend/app/services/` - Business Logic

Core services that implement business logic:

- `assistant_service.py` - RAG pipeline orchestration
- `ingestion_service.py` - PDF processing and chunking
- `embedding_service.py` - Azure OpenAI embeddings
- `llm_service.py` - GPT-4o API calls

### `/backend/app/models/` - Database Models

SQLAlchemy ORM models for all database tables:

- `User`, `UserRole` - Authentication
- `KnowledgeSource`, `DocumentChunk` - AI knowledge base
- `ChatSession`, `ChatMessage` - AI conversations
- `Timetable`, `TimetableSlot` - Academic schedules
- `Club`, `ClubMember`, `ClubBoard` - Student organizations

### `/docs/` - Knowledge Sources

PDF documents used by the AI Assistant:

- TBS Student Handbook (54 pages)
- TBS Student Guide (86 pages)
- Graduate Career Study 2023 (5 pages)

### `/showcase/` - Public Documentation

This repository contains project documentation for portfolio and showcase purposes.

## File Naming Conventions

- **Python:** `snake_case.py` (e.g., `assistant_service.py`)
- **TypeScript/React:** `PascalCase.tsx` for components, `camelCase.ts` for utilities
- **Database migrations:** `revision_id_description.py`
- **Documentation:** `UPPERCASE.md` for major docs

## Environment Structure

```
Development:
├── Local PostgreSQL (pgvector)
├── Local Next.js dev server
└── Local FastAPI server

Production:
├── Azure App Service (Frontend)
├── Azure App Service (Backend)
├── Azure PostgreSQL Flexible Server
── Azure AI Foundry (OpenAI models)
└── Azure Blob Storage (PDFs)
```

## Deployment Flow

```
1. Developer pushes to main branch
    ↓
2. GitHub Actions triggered
    ↓
├─ Frontend: npm install → npm run build → Deploy to Azure
└─ Backend: pip install → Deploy to Azure → Auto-migrate DB
    ↓
3. Azure App Service restarts
    ↓
4. Live at tbsportal.tn
```

## Database Migration Flow

```
1. Developer modifies SQLAlchemy models
    ↓
2. Run: alembic revision --autogenerate -m "description"
    ↓
3. Review generated migration in alembic/versions/
    ↓
4. Commit migration file
    ↓
5. On deploy: alembic upgrade head (auto-run)
```

## Testing Strategy

```
Unit Tests:
├── Backend: pytest (70% coverage)
│   ├── test_ingestion.py
│   ├── test_retrieval.py
│   └── test_assistant_api.py
│
└── Frontend: Vitest
    ├── component tests
    └── service tests

Integration Tests:
└── Manual testing with real Azure services

E2E Tests:
└── Playwright (planned)
```

## CI/CD Pipeline

### Backend Deployment

```yaml
name: Deploy FastAPI backend to Azure App Service
on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - Checkout code
      - Setup Python 3.10
      - Install dependencies (pip install -r requirements.txt)
      - Deploy to Azure App Service
      - Auto-run Alembic migrations on startup
```

### Frontend Deployment

```yaml
name: Deploy Next.js frontend to Azure App Service
on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - Checkout code
      - Setup Node.js 20
      - Install dependencies (npm install)
      - Build (npm run build)
      - Deploy to Azure App Service
```

## Security Architecture

```
─────────────────────────────────────┐
│         Client Browser              │
│  (HTTPS, httpOnly JWT cookie)       │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│      Azure App Service (Frontend)   │
│  (CORS whitelist: tbsportal.tn)     │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│      Azure App Service (Backend)    │
│  (JWT validation, rate limiting)    │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│      Azure PostgreSQL               │
│  (SSL required, private network)    │
└─────────────────────────────────────┘
```

## Monitoring & Analytics

### Key Metrics Tracked

- **User Engagement:** Daily active users, session duration
- **AI Performance:** Query volume, response time, satisfaction rate
- **System Health:** Uptime, error rate, database load
- **Cost Tracking:** Token usage, API costs

### Tools

- **Google Analytics:** User behavior tracking
- **Azure Monitor:** Infrastructure metrics
- **Custom Logging:** Structured JSON logs
- **Audit Logs:** Admin action tracking

---

*This structure represents the complete TBS Portal project organization as of August 2026.*
