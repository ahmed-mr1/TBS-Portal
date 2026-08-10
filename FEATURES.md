# TBS Portal - Feature Documentation

## 🤖 AI Assistant

### Overview

The TBS AI Assistant is a Retrieval-Augmented Generation (RAG) system that provides accurate, citation-backed answers to student questions using official TBS documents.

### Key Features

#### 1. Multi-Source Knowledge Base

| Source | Pages | Chunks | Type |
|--------|-------|--------|------|
| TBS Student Handbook | 54 | 39 | Official Policy |
| TBS Student Guide | 86 | 50 | Student Resources |
| Graduate Career Study 2023 | 5 | 5 | Survey Data |
| Manual Q&A | N/A | N/A | Admin-Curated |

#### 2. Intelligent Retrieval

- **Vector Search:** pgvector HNSW index (1536 dimensions)
- **Semantic Similarity:** Azure OpenAI text-embedding-3-small
- **Query Expansion:** 37 synonym groups (work→career, job, employment)
- **Source Priority:** Official docs boosted 1.15x over surveys
- **Fallback:** Live DB queries for teachers, subjects, calendar, clubs

#### 3. Anti-Hallucination Safeguards

- Strict system prompt: "You have NO prior knowledge about TBS"
- Post-generation validation
- Confidence scoring (High/Medium/Low)
- Explicit refusal when information not found
- Temporal awareness (current year: 2026)

#### 4. Conversation Features

- Multi-turn context (last 6 messages)
- Session persistence
- Streaming responses (SSE)
- Source citations with document names
- Feedback mechanism (👍/👎)

#### 5. Admin Dashboard

- Knowledge source management (upload, monitor status)
- Manual Q&A curation (embedded instantly)
- Unanswered question tracking
- Feedback analytics
- Auto-resolve when Q&A added

### Example Interactions

**Q:** "How many credits to graduate?"  
**A:** "You need 120 credits to graduate from TBS. This includes required courses, electives, and internship credits. (Source: TBS Student Handbook, page 42)"

**Q:** "What happens if I fail a course?"  
**A:** "If you fail a course, you must retake it. TBS allows a maximum of 2 failed courses per semester. Failing more than 4 courses total may result in academic probation. (Source: TBS Student Handbook, page 28)"

**Q:** "Hi"  
**A:** "Hello! How can I help you with TBS today?"

**Q:** "What was the average salary for IT graduates?"  
**A:** "According to the 2023 TBS Junior Enterprise graduate study, the reported average first salary for IT graduates in Tunisia was 1,510 TND. Note: This is 2023 data and may not reflect current 2026 salaries."

---

## 📋 Student Services

### Timetable Management

- **View Timetables:** Search by major, minor, level, group
- **Custom Timetables:** Build personal schedules
- **OCR Extraction:** Upload PDF/PNG timetables → automatic slot extraction
- **Conflict Detection:** Warns about overlapping courses
- **Teacher Contacts:** Integrated directory with email

### Grade Tracking

- **GPA Calculator:** Semester and cumulative GPA
- **Subject Catalog:** Course codes, credits, prerequisites

### Teacher Directory

- **Search:** By name, department, email
- **Contact Info:** Email addresses, office locations
- **Department Filter:** Organize by faculty

### Club Management

- **Browse Clubs:** Active student organizations
- **Join Requests:** Apply to join clubs
- **Board Management:** President, VP, Secretary, Treasurer roles
- **Event Requests:** Submit official requests with PDF generation
- **Approval Workflow:** Admin review process

### Academic Calendar

- **Event Categories:** Holidays, exams, administrative deadlines
- **Semester View:** S1, S2, Annual events
- **Date Parsing:** Automatic date extraction from uploads

### Restaurant Menus

- **Daily Menus:** Lunch options posted daily
- **Image Upload:** Photo of menu board
- **Date Tracking:** Historical menu archive

### Document Center

- **Official Forms:** Download administrative forms
- **Categories:** Organized by type (registration, financial, academic)
- **File Storage:** Azure Blob Storage with SAS tokens

### Exam Schedules

- **OCR Extraction:** Upload exam timetables → structured data
- **Search:** By course, date, room
- **Conflict Check:** Detect overlapping exams

---

## 👨‍💼 Admin Features

### Knowledge Management

- **Upload PDFs:** Automatic ingestion (PyMuPDF + embeddings)
- **Monitor Status:** pending → processing → ready
- **Delete Sources:** Remove outdated documents
- **View Chunks:** Inspect extracted content

### Manual Q&A

- **Add Q&A:** Question + answer pairs
- **Auto-Embed:** Instantly searchable via vector search
- **Auto-Resolve:** Marks matching unanswered questions as resolved
- **Priority:** Ranked below official documents

### Unanswered Questions

- **Track Gaps:** Questions with no answer or low confidence
- **Frequency Count:** How often each question is asked
- **Resolve:** Add Q&A or mark as out-of-scope
- **Filter:** Active vs. Resolved

### Feedback Analytics

- **Thumbs Up/Down:** Per-answer feedback
- **Satisfaction Rate:** Percentage of positive feedback
- **Trend Analysis:** Track quality over time

### User Management

- **View Users:** All registered students
- **Role Assignment:** student, tbsadmin, admin, super_admin
- **Password Reset:** Admin-initiated password resets
- **Audit Logs:** Track all admin actions with IP logging

### Content Management

- **News:** Post announcements with categories
- **Media Gallery:** Upload photos with approval workflow
- **Guide Editor:** New student guide chapters and pages
- **Subjects:** Manage course catalog

---

## 🔐 Authentication & Authorization

### Login Flow

1. Student enters email (@tbs.u-tunis.tn) and password
2. Backend validates credentials (bcrypt)
3. Returns JWT token (45-minute expiry)
4. Frontend stores in httpOnly cookie
5. All requests include Authorization header

### Role-Based Access

| Feature | Student | TBS Admin | Admin |
|---------|---------|-----------|-------|
| View Timetables | ✅ | ✅ | ✅ |
| Manage Clubs | ✅ | ✅ | ✅ |
| Upload Content |  | ✅ | ✅ |
| AI Dashboard | ❌ | ✅ | ✅ |
| User Management | ❌ | ❌ | ✅ |
| Audit Logs | ❌ | ❌ | ✅ |

### Security Measures

- **Password Hashing:** bcrypt (12 rounds)
- **JWT Expiry:** 45 minutes (auto-logout)
- **Rate Limiting:** 5 login attempts/minute per IP
- **CORS:** Whitelisted origins only (tbsportal.tn)
- **Audit Trail:** All admin actions logged with IP, timestamp, action

---

## 📊 Analytics & Monitoring

### AI Assistant Metrics

- **Daily Active Users:** Unique students using AI
- **Query Volume:** Total questions asked per day
- **Response Time:** p50, p95, p99 latency
- **Cache Hit Rate:** Repeated questions served from cache
- **Satisfaction Rate:**  / (👍 + 👎) percentage
- **Unanswered Rate:** Questions with no answer

### System Metrics

- **Uptime:** 99.5% target
- **Error Rate:** HTTP 5xx < 1%
- **Database Load:** Query count, connection pool usage
- **Token Cost:** Daily OpenAI API cost tracking

### User Engagement

- **Session Duration:** Average time spent per visit
- **Page Views:** Most visited pages
- **Feature Usage:** Which services are most popular
- **Retention:** Daily/weekly/monthly active users

---

##  Design System

### Colors

| Color | Hex | Usage |
|-------|-----|-------|
| TBS Black | #050505 | Primary background |
| TBS Gold | #D4AF37 | Accents, buttons, highlights |
| Gold Light | #F4D06F | Hover states |
| Gold Dark | #AA8C2C | Active states |
| Slate 950 | #020617 | Secondary backgrounds |
| Zinc 900 | #18181b | Cards, panels |

### Typography

- **Primary:** Inter (sans-serif) - UI text, body
- **Serif:** Playfair Display - Headings, branding
- **Mono:** Geist Mono - Code, technical content

### Components

- **Buttons:** Rounded-xl, amber-500 background, black text
- **Cards:** Rounded-2xl, zinc-900 background, white/10 border
- **Inputs:** Rounded-xl, black/40 background, gold focus ring
- **Modals:** Fixed position, backdrop blur, slide-in animation

### Responsive Breakpoints

- **Mobile:** < 640px (single column)
- **Tablet:** 640px - 1024px (2 columns)
- **Desktop:** > 1024px (3-4 columns)

---

##  Performance

### Frontend

- **LCP (Largest Contentful Paint):** < 2.5s
- **FID (First Input Delay):** < 100ms
- **CLS (Cumulative Layout Shift):** < 0.1
- **Bundle Size:** < 200KB (gzipped)

### Backend

- **API Response Time:** < 200ms (non-AI endpoints)
- **AI Response Time:** < 3s (first token)
- **Database Queries:** < 50ms (indexed queries)
- **Vector Search:** < 10ms (HNSW index)

### Infrastructure

- **Concurrent Users:** 50+ supported
- **Database Connections:** 10 (pool size) + 20 (overflow)
- **Storage:** Azure Blob (unlimited)
- **Bandwidth:** Azure App Service (auto-scaling)

---

## 📱 Mobile Experience

### Responsive Design

- **Navigation:** Hamburger menu on mobile
- **Chat:** Full-width modal, keyboard-aware
- **Timetables:** Horizontal scroll, pinch-to-zoom
- **Forms:** Touch-friendly inputs, large tap targets

### Progressive Web App (PWA)

- **Manifest:** installable on home screen
- **Offline:** Service worker for cached pages
- **Push Notifications:** Planned for announcements

---

##  Internationalization

### Current Support

- **English:** Primary language (UI + content)
- **Arabic:** AI responses (RTL support in chat)
- **French:** AI responses (Tunisian French)

### Future Plans

- **UI Translation:** Arabic and French interface
- **RTL Layout:** Full right-to-left support
- **Date Formats:** Localized date display

---

## ♿ Accessibility

### WCAG 2.1 AA Compliance

- **Keyboard Navigation:** All features accessible via keyboard
- **Screen Readers:** ARIA labels, semantic HTML
- **Color Contrast:** 4.5:1 minimum ratio
- **Focus Indicators:** Visible focus rings
- **Alt Text:** Descriptive image alt attributes

### Features

- **Skip Links:** Jump to main content
- **Landmark Regions:** header, main, footer, nav
- **Form Labels:** Associated with inputs
- **Error Messages:** Clear, descriptive validation

---

## 🔄 Integrations

### Azure Services

- **App Service:** Hosting (Frontend + Backend)
- **PostgreSQL:** Database + pgvector
- **OpenAI:** GPT-4o, text-embedding-3-small
- **Blob Storage:** PDF and image storage
- **AI Foundry:** Model deployment and monitoring

### Third-Party

- **Resend:** Transactional emails (password reset)
- **Google Analytics:** User behavior tracking
- **GitHub Actions:** CI/CD pipeline
- **Lucide Icons:** Icon library

### Planned Integrations

- **SMS Gateway:** Twilio for notifications
- **Payment Gateway:** Stripe for club fees
- **Calendar Sync:** Google Calendar integration
- **LMS Integration:** Moodle/Canvas API
