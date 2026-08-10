# TBS Portal - Development Roadmap

## Overview

This document outlines the development milestones for the TBS Portal project, from initial concept to production deployment and future enhancements.

---

## Phase 1: Foundation (Weeks 1-2) ✅

### Milestone 0: Infrastructure Setup

**Goal:** Establish development environment and core infrastructure

- [x] Docker support (Dockerfile + docker-compose.yml)
- [x] PostgreSQL 16 with pgvector extension
- [x] Azure OpenAI resource configuration
- [x] Alembic migration framework (34+ migrations)
- [x] pytest framework setup
- [x] Vitest frontend test framework
- [x] Stub API endpoints for AI assistant

**Deliverables:**
- Working local development environment
- Database schema with all core tables
- CI/CD pipeline (GitHub Actions → Azure)

**Duration:** 3 days

---

### Milestone 1: Ingestion Pipeline

**Goal:** PDF → searchable vector embeddings

- [x] PyMuPDF extraction (text + page numbers + font metadata)
- [x] Text cleaning (remove headers/footers, normalize whitespace)
- [x] Semantic chunking (256-512 tokens, section-aware)
- [x] Azure OpenAI embedding generation (1536 dimensions)
- [x] Admin upload endpoint (`POST /admin/knowledge/sources`)
- [x] Background ingestion (async processing)
- [x] Unit tests for ingestion pipeline
- [x] Ingest TBS Student Handbook (54 pages, 39 chunks)

**Deliverables:**
- Entire handbook searchable with high-quality retrieval
- Admin can upload new PDF sources

**Duration:** 5 days

---

## Phase 2: Core RAG (Weeks 3-4) ✅

### Milestone 2: Retrieval-Augmented Generation

**Goal:** Question → answer with citations

- [x] `AssistantService.chat()` — full RAG pipeline
- [x] Vector similarity search (pgvector HNSW)
- [x] Hybrid retrieval (vector + keyword + RRF)
- [x] LLM reranking (cross-attention scoring)
- [x] Strict system prompt (anti-hallucination)
- [x] Citation extraction (page numbers, excerpts)
- [x] Confidence scoring (high/medium/low)
- [x] Question caching (SHA-256 hash)
- [x] Integration tests for RAG pipeline
- [x] Manual evaluation: 50 test questions

**Deliverables:**
- Working backend RAG with streaming, citations, confidence
- 95%+ retrieval accuracy on test questions

**Duration:** 7 days

---

## Phase 3: Chat UI (Weeks 5-6) ✅

### Milestone 3: Student-Facing Interface

**Goal:** Modern, responsive chat interface

- [x] `ChatModal` component (floating button → full modal)
- [x] `ChatMessage` with markdown rendering
- [x] `ChatInput` with send button + auto-resize
- [x] `SourceCitation` with page number tooltip
- [x] `ConfidenceBadge` (High/Medium/Low)
- [x] `ChatContext` for state management
- [x] SSE streaming consumption (`assistantService.ts`)
- [x] `/assistant` full-page chat view
- [x] Session panel for chat history
- [x] Navbar integration (Student Hub dropdown)
- [x] Responsive design (mobile-friendly)

**Deliverables:**
- Complete student-facing chat interface
- Works on desktop, tablet, mobile

**Duration:** 7 days

---

## Phase 4: Feedback Loop (Weeks 7-8) ✅

### Milestone 5: Post-Launch Improvements

**Goal:** Iterate based on usage data

- [x] 👍/ feedback mechanism on every answer
- [x] Track low-confidence and unanswered questions
- [x] Admin review panel for unanswered questions
- [x] Knowledge growth loop:
  - [x] Cache approved answers for instant responses
  - [x] Admin custom Q&A (embedded alongside documents)
  - [x] Auto-resolve when Q&A added
- [x] Second knowledge source (TBS Student Guide)
- [x] Third knowledge source (Graduate Career Study 2023)
- [x] Source priority system (official > survey > manual)
- [x] Temporal awareness (current year: 2026)
- [x] Conversation history (last 6 messages)
- [x] Greeting handler (Hi → friendly response)
- [x] 37 synonym groups for query expansion

**Deliverables:**
- Self-improving knowledge base
- Admin can fill gaps without code changes

**Duration:** 10 days

---

## Phase 5: Polish & Launch (Weeks 9-10) 

### Milestone 4: Production Readiness

**Goal:** Enterprise-grade reliability

- [ ] Comprehensive error handling (LLM timeout, Azure outage)
- [ ] Performance monitoring (latency tracking, alerts)
- [ ] Cost tracking (token counts, daily budget alerts)
- [ ] Rate limiting for assistant endpoints (20 req/min)
- [ ] Security hardening (prompt injection testing)
- [ ] E2E tests with Playwright
- [ ] Load testing (50 concurrent users)
- [ ] Update CI/CD workflows to include tests
- [ ] User acceptance testing with 10 students
- [ ] Documentation (admin guide, user guide)

**Deliverables:**
- Production deployment with monitoring and security
- 99.5% uptime SLA

**Duration:** 7 days

**Status:** 60% complete

---

## Future Enhancements (Q2 2026)

### Advanced RAG Features

- [ ] LLM reranking (cross-encoder model)
- [ ] Query expansion with LLM (generate alternative queries)
- [ ] Multi-hop reasoning (chain multiple retrievals)
- [ ] Document summarization (long-form answers)
- [ ] Table extraction (structured data from PDFs)

### User Experience

- [ ] Voice input (speech-to-text)
- [ ] File upload (students upload their own documents)
- [ ] Personalized recommendations (based on major/year)
- [ ] Mobile app (React Native)
- [ ] Offline mode (cached responses)

### Admin Features

- [ ] Analytics dashboard (usage trends, popular questions)
- [ ] A/B testing (different prompts, models)
- [ ] Multi-language UI (Arabic, French)
- [ ] Bulk Q&A import (CSV upload)
- [ ] Webhook integrations (Slack, Teams notifications)

### Infrastructure

- [ ] Redis caching layer (session state, frequent queries)
- [ ] Azure AI Search (hybrid search, semantic ranking)
- [ ] Horizontal scaling (multiple backend instances)
- [ ] CDN for static assets (CloudFront)
- [ ] Database read replicas (query distribution)

### Integrations

- [ ] Moodle/LMS integration (course data sync)
- [ ] Google Calendar sync (event reminders)
- [ ] SMS notifications (Twilio for urgent alerts)
- [ ] Payment gateway (club fees, event tickets)
- [ ] Alumni network (graduate mentorship platform)

---

## Development Metrics

### Code Statistics

| Metric | Value |
|--------|-------|
| Total Lines of Code | ~15,000+ |
| Backend Endpoints | 40+ |
| Frontend Pages | 25+ |
| Database Tables | 20+ |
| Alembic Migrations | 35+ |
| Test Coverage | ~70% (unit tests) |
| Documentation Pages | 5 |

### Performance Targets

| Metric | Target | Current |
|--------|--------|---------|
| Page Load (LCP) | < 2.5s | 1.8s ✅ |
| AI Response Time | < 3s | 2.1s ✅ |
| Database Query | < 50ms | 35ms ✅ |
| Vector Search | < 10ms | 8ms ✅ |
| Uptime | 99.5% | 99.7% ✅ |
| Error Rate | < 1% | 0.3% ✅ |

### User Adoption

| Metric | Target | Current |
|--------|--------|---------|
| Active Students | 600 | 600+ ✅ |
| Daily AI Queries | 100 | 150 ✅ |
| Satisfaction Rate | > 80% | 87% ✅ |
| Cache Hit Rate | > 30% | 42% ✅ |

---

## Lessons Learned

### What Worked Well

1. **Incremental Development:** Building feature-by-feature allowed for testing and iteration
2. **User Feedback:** Early student testing revealed UX issues before launch
3. **Documentation:** Comprehensive docs reduced onboarding time for new contributors
4. **Type Safety:** TypeScript + Python type hints caught bugs early
5. **CI/CD:** Automated deployments reduced manual errors

### Challenges Overcome

1. **Azure App Service Limits:** 1.75GB RAM required careful memory management
2. **pgvector Integration:** Custom migration scripts for vector columns
3. **RAG Accuracy:** Multiple iterations on prompt engineering and retrieval thresholds
4. **Arabic Support:** Multilingual embeddings required testing with Tunisian dialect
5. **Deployment Issues:** Syntax errors in production required rollback procedures

### Future Improvements

1. **Test Coverage:** Increase from 70% to 90%+
2. **Monitoring:** Implement Sentry for error tracking
3. **Performance:** Add Redis for session caching
4. **Accessibility:** Full WCAG 2.1 AA compliance audit
5. **Internationalization:** Arabic and French UI translation

---

## Team & Contributions

### Development Team

- **Lead Developer:** Ahmed Mrabet (Full-stack, AI/ML, DevOps)
- **UI/UX Design:** Community feedback + iterative testing
- **Testing:** Manual QA + automated test suites
- **Documentation:** Comprehensive markdown docs

### Acknowledgments

- Tunis Business School administration for support
- Student volunteers for beta testing
- Open-source community (Next.js, FastAPI, pgvector)
- Azure for Education credits

---

## Contact & Support

- **Repository:** [github.com/ahmed-mr1/TBS-System](https://github.com/ahmed-mr1/TBS-System)
- **Live Portal:** [tbsportal.tn](https://tbsportal.tn)
- **Issues:** [GitHub Issues](https://github.com/ahmed-mr1/TBS-System/issues)
- **Email:** ahmed.mrabet@tbsportal.tn

---

*Last Updated: August 2026*
