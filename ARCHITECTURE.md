# TBS Portal - Technical Architecture

## System Overview

TBS Portal is a full-stack web application built with Next.js (frontend) and FastAPI (backend), deployed on Azure App Service with PostgreSQL + pgvector for vector search.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    FRONTEND (Next.js 16)                     │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   Student    │  │   Admin      │  │   AI Assistant   │  │
│  │   Pages      │  │   Dashboard  │  │   Chat UI        │  │
│  │              │  │              │  │                  │  │
│  │ • Dashboard  │  │ • Knowledge  │  │ • ChatModal      │  │
│  │ • Timetables │  │ • Q&A        │  │ • ChatMessage    │  │
│  │ • Grades     │  │ • Feedback   │  │ • ChatInput      │  │
│  │ • Calendar   │  │ • Analytics  │  │ • SourceCitation │  │
│  │ • Clubs      │  │              │  │ • ConfidenceBadge│  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                              │
│  State: React Context (AuthContext, ChatContext)            │
│  HTTP: Axios + fetch (SSE streaming)                        │
─────────────────────────────────────────────────────────────┘
                              │
                    HTTPS / SSE Stream
                              │
┌─────────────────────────────────────────────────────────────┐
│                   BACKEND (FastAPI)                          │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   REST API   │  │   RAG        │  │   Ingestion      │  │
│  │   Endpoints  │  │   Pipeline   │  │   Service        │  │
│  │              │  │              │  │                  │  │
│  │ • Auth       │  │ • Retrieve   │  │ • PDF Parse      │  │
│  │ • Timetables │  │ • Rerank     │  │ • Chunk          │  │
│  │ • Calendar   │  │ • Generate   │  │ • Embed          │  │
│  │ • Clubs      │  │ • Cache      │  │ • Store          │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                              │
│  Auth: JWT (45min) + bcrypt                                 │
│  Rate Limit: SlowAPI (5 req/min login)                      │
└─────────────────────────────────────────────────────────────┘
                              │
                    PostgreSQL + pgvector
                              │
┌─────────────────────────────────────────────────────────────┐
│                  AZURE SERVICES                              │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   App        │  │   OpenAI     │  │   Blob Storage   │  │
│  │   Service    │  │              │  │                  │  │
│  │              │  │ • GPT-4o     │  │ • PDF Storage    │  │
│  │ • Frontend   │  │ • Embeddings │  │ • SAS Tokens     │  │
│  │ • Backend    │  │ • API Key    │  │                  │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Database Schema

### Core Tables

```sql
-- Users & Authentication
users (id, email, password_hash, role, is_active)
password_reset_tokens (id, user_id, token, expires_at, used)

-- Academic Data
timetables (id, uploader_id, file_url, semester, major, minor, level, group)
timetable_slots (id, timetable_id, day, start_time, end_time, course_name, teacher_name, room)
subjects (id, name, code, credits, level, semester, major)
exam_schedules (id, timetable_id, date, room, course_name)

-- Calendar & Events
calendar_events (id, title, date_display, start_date, end_date, semester, category)
restaurant_menus (id, date, description, image_url, user_id)

-- Clubs
clubs (id, name, purpose, category, leader_user_id, status)
club_members (id, club_id, name, email, status)
club_board (id, club_id, role, name, email)
club_requests (id, club_id, request_type, event_name, status, ...)

-- Content
news (id, title, content, category, date, link_url, file_url)
media (id, title, featured_text, image_url, category, status)
documents (id, title, description, category, file_url)
guide_chapters (id, title, position, icon)
guide_pages (id, chapter_id, position, heading, body, bullet_points)

-- Teachers
teachers (id, name, email, department)

-- Audit & Notifications
audit_logs (id, actor_user_id, action, method, path, status, ip_address)
notifications (id, user_id, title, message, type, is_read)
```

### AI Assistant Tables

```sql
-- Knowledge Sources
knowledge_sources (
    id UUID PRIMARY KEY,
    name VARCHAR(255),
    source_type VARCHAR(50),  -- 'handbook', 'guide', 'regulation', 'graduate_study'
    file_path VARCHAR(500),
    status VARCHAR(20),       -- 'pending', 'processing', 'ready', 'error'
    chunk_count INTEGER,
    page_count INTEGER,
    created_at TIMESTAMPTZ
)

-- Document Chunks (Vector Store)
document_chunks (
    id UUID PRIMARY KEY,
    source_id UUID FK → knowledge_sources,
    content TEXT,
    embedding VECTOR(1536),   -- pgvector
    chunk_index INTEGER,
    metadata JSONB,           -- {pages: [], sections: [], source_type: ''}
    created_at TIMESTAMPTZ
)

-- Chat Sessions & Messages
chat_sessions (id UUID, user_id UUID, title VARCHAR, message_count INTEGER)
chat_messages (
    id UUID,
    session_id UUID,
    role VARCHAR,             -- 'user', 'assistant'
    content TEXT,
    sources JSONB,            -- [{page: 23, excerpt: '...', similarity: 0.89}]
    confidence VARCHAR,       -- 'high', 'medium', 'low'
    created_at TIMESTAMPTZ
)

-- Question Cache & Feedback
question_cache (id UUID, question_hash VARCHAR UNIQUE, question TEXT, answer TEXT, hit_count INTEGER)
user_feedback (id UUID, user_id UUID, message_id UUID, rating VARCHAR)
unanswered_questions (id UUID, question TEXT, count INTEGER, resolved BOOLEAN, resolution TEXT)

-- Manual Q&A
knowledge_qa (
    id UUID,
    question TEXT,
    answer TEXT,
    embedding VECTOR(1536),
    enabled BOOLEAN,
    created_by UUID
)
```

## AI Pipeline Architecture

### Ingestion Pipeline

```
PDF Upload → PyMuPDF Extract → Clean Text → Chunk (256-512 tokens)
    ↓
Azure OpenAI Embeddings (text-embedding-3-small, 1536 dims)
    ↓
Store in PostgreSQL (document_chunks with pgvector)
```

### Retrieval Pipeline (RAG)

```
User Query → Embed Query → Vector Search (pgvector HNSW)
    ↓
Top-K Chunks (similarity > 0.2) + DB Query Fallback
    ↓
Build Context (system prompt + chunks + conversation history)
    ↓
Azure OpenAI GPT-4o (streaming SSE)
    ↓
Response + Citations + Confidence Score
```

### Source Priority System

Documents are boosted based on their authority level:

1. **Priority 1 (1.15x boost):** Official current documents — Handbook, Guide, Decrees, Regulations
2. **Priority 2 (0.85x):** Website crawled content — supplementary information
3. **Priority 3 (1.0x):** Research/Survey documents — 2023 Graduate Study
4. **Priority 4 (1.0x):** Manual Q&A and general knowledge

Context includes type labels (`[TYPE: Official Policy - Highest Authority]`) to help the LLM distinguish sources.

### Live DB Queries

The AI can directly query live portal data for real-time answers:

- **Teachers:** "Who is [name]?" → Returns email, department
- **Clubs:** "President of [club]?" → Returns board members
- **Subjects:** "What courses are offered?" → Returns credits, semester
- **Calendar:** "When is the next holiday?" → Returns events
- **Restaurant:** "What's for lunch?" → Returns daily menu
- **Documents:** "Where to download internship forms?" → Points to publications page

### Anti-Hallucination Measures

- Strict system prompt: "You have NO prior knowledge about TBS"
- Post-generation validation: Check if answer contains claims not in context
- Confidence scoring: Based on retrieval similarity (0.7+ high, 0.5+ medium)
- Fallback: "I couldn't find this information in the official TBS documents"

## Security Architecture

### Authentication Flow

```
Login → POST /auth/login → Validate credentials → Generate JWT (45min)
    ↓
Store token in cookie (httpOnly, secure)
    ↓
Every request → Authorization: Bearer <token> → Validate → Get user
```

### Authorization Levels

- **Student:** Basic access (timetables, grades, calendar)
- **TBS Admin:** Content management (news, media, clubs)
- **Admin:** Full system access (users, audit logs, AI dashboard)

### Security Measures

- bcrypt password hashing (12 rounds)
- JWT with short expiry (45 minutes)
- Rate limiting: 5 login attempts/minute per IP
- CORS whitelisting (tbsportal.tn only)
- Audit logging for all admin actions
- Prompt injection protection (XML delimiters)
- Environment variables (no secrets in code)

## Deployment Architecture

### Azure Infrastructure

```
Azure App Service (Frontend)
    ↓ HTTPS
Azure App Service (Backend)
    ↓
Azure PostgreSQL Flexible Server
    ↓
Azure AI Foundry (OpenAI Models)
    ↓
Azure Blob Storage (PDFs)
```

### CI/CD Pipeline

```
GitHub Push → GitHub Actions
    ↓
├─ Frontend: npm install → npm run build → Deploy to Azure
└─ Backend: pip install → Deploy to Azure → Auto-migrate DB
```

### Environment Variables

```bash
# Backend
DATABASE_URL=postgresql://...
SECRET_KEY=...
AZURE_OPENAI_API_KEY=...
AZURE_OPENAI_ENDPOINT=...
AZURE_OPENAI_CHAT_DEPLOYMENT=gpt-4o
AZURE_OPENAI_EMBEDDING_DEPLOYMENT=text-embedding-3-small
RESEND_API_KEY=...

# Frontend
NEXT_PUBLIC_API_URL=https://app-tbs-backend...azurewebsites.net
```

## Performance Optimization

### Frontend

- Static generation for public pages (/, /about, /calendar)
- Server-side rendering for authenticated pages
- Image optimization (Next.js Image component)
- Code splitting (dynamic imports)
- Tailwind CSS purge (production builds)

### Backend

- Connection pooling (SQLAlchemy, 10 connections)
- Async endpoints (FastAPI + async/await)
- Vector search index (pgvector HNSW, m=16, ef_construction=200)
- Caching: Question cache (SHA-256 hash of normalized question)
- Streaming responses (SSE for AI chat)

### Database

- Indexes on foreign keys and frequently queried columns
- pgvector HNSW index for sub-millisecond ANN search
- JSONB metadata for flexible chunk attributes
- Alembic migrations for schema evolution

## Monitoring & Analytics

### Key Metrics

- Query latency (p50, p95, p99)
- Retrieval relevance (user feedback)
- Cache hit rate (question_cache)
- Daily active users (chat_sessions)
- Token cost tracking (chat_messages.token_count)
- Error rate (HTTP 5xx)

### Logging

- Structured JSON logging (Python logging module)
- Audit logs for admin actions
- Chat interaction logs (user_id, session_id, latency_ms)
- Error tracking (Sentry integration planned)

## Future Enhancements

- [ ] Redis caching layer
- [ ] Azure AI Search for hybrid search
- [ ] User feedback analytics dashboard
- [ ] Mobile app (React Native)
- [ ] Real-time notifications (WebSockets)
- [ ] Multi-language support (Arabic UI)
- [ ] Advanced RAG (reranking, query expansion)
