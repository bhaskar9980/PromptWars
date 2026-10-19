# 🤝 TeamSync AI — Pitch Document

> **Competition:** PromptWars Chennai  
> **Live URL:** https://teamsync-ai-551224894530.asia-south1.run.app/  
> **GitHub:** *(public repo — share link at submission)*  
> **Stack:** Python 3.12 · Streamlit · Gemini 2.5 Flash · Cloud Firestore · Cloud Run · Cloud Logging

---

## 1. The Problem (5 seconds)

> *"Engineering teams lose 17+ hours per person per week to coordination overhead — and the biggest killer isn't meetings, it's the blockers nobody sees."*

### The Coordination Tax

Picture a 5-person engineering team on a Monday morning. A critical payment bug is blocked because one developer needs prod log access — but no one knows this until Wednesday's standup. By then, it's cascaded: the security audit depends on the login mockup that's waiting on brand guidelines, and the sprint demo is at risk. **Three days lost to invisible dependencies.**

This isn't rare — it's the default:

| Pain Point | The Real Cost |
|------------|---------------|
| **Invisible blockers** | A blocker discovered at a standup has already wasted 24 hours. Multiply by frequency — teams lose days per sprint to blockers that could've been surfaced instantly. |
| **Status meeting overhead** | A 30-minute daily standup × 5 people = **12.5 hours/week** of engineering time spent *talking about work* instead of *doing work*. Most of it could be a generated summary. |
| **Workload blindness** | Managers allocate tasks based on gut feel. One person has 6 tasks, another has 2 — but no one sees the imbalance until someone burns out or misses a deadline. |
| **Context-switching friction** | Creating a task means opening a tool, filling 5 fields, picking a priority, assigning someone. It breaks flow. So people don't bother — and tasks fall through cracks. |

**The core insight:** The information to solve all of these already exists in the task data. What's missing is an **intelligent layer** that reads the data, surfaces what matters, and acts on it — without waiting for a human to ask the right question at the right time.

---

## 2. The Solution (10 seconds)

> *"We built TeamSync AI — an AI-powered team collaboration platform using Gemini 2.5 Flash + Cloud Firestore + Cloud Run that automates standups, resolves blockers, and gives real-time workload visibility."*

TeamSync AI replaces time-consuming manual standups with **AI-generated summaries**, turns blockers into **actionable resolution steps**, and lets teams create tasks using **plain English** — all backed by **persistent Firestore storage** and **real-time analytics dashboards**.

---

## 3. Features — What It Does

### Feature 1: 📊 Team Dashboard (Analytics)
**What it does:** A real-time dashboard with interactive Plotly charts showing team KPIs at a glance.

**Key elements:**
| Metric | What it shows |
|--------|---------------|
| Total Tasks | Count of all tasks in the system |
| Completion Rate | % of tasks marked "done" |
| Blocked Count | Number of tasks with active blockers |
| Overdue Count | Tasks past their due date that aren't done |

**Charts:**
- **Task Status Donut** — Todo (gray), In Progress (blue), Done (green)
- **Priority Bar Chart** — Critical (red) → High (orange) → Medium (yellow) → Low (green)
- **Workload per Member** — Horizontal bar chart showing task distribution per person

**Why it matters:** Managers see imbalances instantly. No more guessing who's overloaded.

---

### Feature 2: 📋 Smart Task Board (Kanban + AI Task Creation)
**What it does:** A Kanban-style task board with 3 columns (To Do → In Progress → Done) and **natural-language task creation powered by Gemini**.

**How it works:**
1. User types: *"Assign a high-priority bug to Meena about payment timeouts"*
2. Gemini parses it → extracts title, assignee, priority, tags
3. Task appears on the board with the right column, priority badge, and assignee

**Also includes:**
- Inline edit via expanders (change status, priority, assignee)
- Manual task creation form (for structured input)
- Priority badges: ⚠ Critical (red) / High (orange dot) / Medium (yellow dot) / Low (gray)
- Blocker indicators on task cards (red `🚫` text)
- All changes persist to **Firestore** — team members see the same data

**Why it matters:** Creating tasks feels like talking, not filling out forms. Reduces friction.

---

### Feature 3: 🤖 AI Standup Generator + Blocker Resolver
**What it does:** One click generates a complete daily standup from all current task data.

**Standup output includes:**
- **Team Summary** — one paragraph on overall team status
- **Per-Member Updates** — what each person is working on, their blockers, next steps
- **Blockers Alert** — highlighted blocked tasks with resolution suggestions

**Blocker Resolution:**
- Click "Analyze Blockers" → Gemini reads all blocked tasks
- Returns **specific, actionable suggestions** per blocker (e.g., *"Request emergency access from DevOps lead. Alternatively, use staging logs with anonymized data."*)

**Why it matters:** Replaces a 30-minute standup meeting with a 10-second button click.

---

## 4. Architecture — How It's Built

```
┌─────────────────┐     ┌──────────────┐     ┌───────────────────┐
│   Streamlit      │────▶│   core.py    │────▶│ Gemini 2.5 Flash  │
│   (app.py)       │     │  (business   │     │ (AI generation)   │
│   UI Layer       │     │   logic)     │     └───────────────────┘
└──────┬───────────┘     └──────┬───────┘
       │                        │
       ▼                        ▼
┌─────────────────┐     ┌───────────────────┐
│   config.py      │     │ firestore_client  │
│   (env vars,     │     │ (Firestore CRUD)  │
│    constants)    │     └───────────────────┘
└─────────────────┘

Deployed on: Google Cloud Run (asia-south1, 512Mi)
Persistence: Google Cloud Firestore (NoSQL)
Audit trail: Google Cloud Logging (structured JSON from stdout)
```

### Why this architecture?

| Layer | File | Why separated? |
|-------|------|---------------|
| **UI** | `app.py` | Streamlit-specific code. All rendering here. |
| **Logic** | `core.py` | No UI imports possible in tests. Pure business logic. |
| **Config** | `config.py` | Single source of truth for all constants + env vars. |
| **Persistence** | `firestore_client.py` | Isolates Firestore so tests can mock it easily. |

**Key design pattern — Lazy Init:**
- `_get_model()` — Gemini client created only when needed (not at import time)
- `_get_firestore_client()` — Cached via `@st.cache_resource` (singleton per app lifecycle)
- This prevents import crashes in tests and cold start failures on Cloud Run

---

## 5. Technical Highlights — Mapped to Evaluation Criteria

### 🏗️ Code Quality
- **Type hints** on every function signature (`-> dict`, `-> bool`, `-> str`)
- **Docstrings** on all public functions
- **Single-responsibility** — every function does one thing, max ~25 lines
- **Named constants** — `MAX_INPUT_LENGTH`, `COLOR_PRIMARY`, `GEMINI_CACHE_TTL` (no magic numbers)
- **Clean separation** — 4 modules with clear boundaries (UI / Logic / Config / Persistence)

### 🔒 Security
- All secrets in `.env`, loaded via `python-dotenv` — **never hardcoded**
- `sanitize_text()` filters 4 prompt injection patterns (regex-based):
  - "ignore previous instructions"
  - "system:"
  - "you are now"
  - "forget everything"
- `validate_input()` + `validate_task()` — validates all user data before processing
- Gemini's native `system_instruction` API — real instruction-data separation (not XML tags that can be escaped)
- HTTPS-only for all external API calls
- No sensitive data in logs or error responses

### ⚡ Efficiency
- `@st.cache_data(ttl=300)` on read-only `get_team_metrics()` — prevents redundant computation
- `@st.cache_resource` on Firestore client — singleton prevents connection storms
- Lazy model init via `_get_model()` — no module-level `genai.configure()` calls
- AI generation calls intentionally **not** cached (prevents stale data)

### 🧪 Testing — 29 Tests, 9 Test Classes
| Test Class | What It Tests | Count |
|------------|---------------|-------|
| `TestValidateInput` | Empty, whitespace, valid, too-long inputs | 4 |
| `TestValidateTask` | Missing title, invalid status/priority, valid task | 4 |
| `TestSanitizeText` | Injection patterns stripped, normal text preserved | 2 |
| `TestGenerateStandup` | Structured response, API failure fallback, empty tasks | 3 |
| `TestAnalyzeBlockers` | No blockers = empty, API failure = sample data | 2 |
| `TestCreateTaskFromText` | Empty input error, API failure fallback | 2 |
| `TestGetTeamMetrics` | Empty tasks, sample data metrics | 2 |
| `TestParseJsonResponse` | Valid JSON, markdown fences, invalid fallback | 3 |
| `TestCountOverdue` | Past due, done task, no date, future date | 3 |
| `TestBoundaryEdgeCases` | At-max + over-max for blocker, title, input | 4 |
| `TestFirestoreClient` | All CRUD ops with null client, ID generation | 8 |
| `TestAuditLogging` | Logging doesn't crash, empty details | 2 |
| `TestConfigValidation` | Missing key raises, valid key passes | 2 |
| `TestSanitizationEdgeCases` | system:, "you are now", "forget everything" | 4 |

**All API calls mocked** — `@patch("core._get_model")` — never calls real Gemini in tests.

### ♿ Accessibility
- Heading hierarchy: `st.title` → `st.header`/`st.subheader` → `st.caption`
- `st.caption()` on **every chart** describing what colors mean
- `st.spinner` on **all AI operations** (standup, task creation, blocker analysis)
- Text labels with color badges (e.g., "⚠ Critical" — not just a red dot)
- Keyboard-navigable (Streamlit native)
- Clear `st.error()` / `st.warning()` messages (never raw stack traces)

### ☁️ Google Services — 4 Services Used
| Service | How It's Used |
|---------|--------------|
| **Gemini 2.5 Flash** | 3 AI features: standup generation, blocker analysis, NL task creation |
| **Cloud Run** | Production deployment (asia-south1, 512Mi, 300s timeout) |
| **Cloud Firestore** | Task persistence + team member storage (NoSQL) |
| **Cloud Logging** | Structured JSON audit trail via `_audit_log()` — auto-ingested from stdout |

---

## 6. Submission Description (Under 1024 Characters)

> Copy-paste this into the hack2skill form:

```
TeamSync AI solves team coordination pain — invisible blockers, wasted standups, no workload visibility.

Features:
1. AI Standup Generator: One-click Gemini-powered standup with per-member updates and blocker alerts
2. Smart Task Board: Kanban board with natural-language task creation via Gemini + inline editing
3. Team Dashboard: Interactive Plotly charts showing status, priority, and workload distribution

Technical Highlights:
- AI: Gemini 2.5 Flash for standup generation, blocker analysis, NL task parsing (3 distinct features)
- Google Services: Cloud Run + Firestore + Cloud Logging (4 services total)
- Testing: 29 unit tests across 9 test classes with mocked APIs (pytest)
- Security: Input validation, prompt injection sanitization, env-based secrets, system_instruction API
- Efficiency: @st.cache_data, @st.cache_resource, lazy model init
- Accessibility: Heading hierarchy, chart captions, spinners, text+color badges
- Architecture: app.py (UI) → core.py (logic) → Gemini API. Clean separation, type hints, docstrings.

Deployed on Cloud Run (asia-south1).
```

*Character count: ~920 — under 1024 ✅*

---

## 7. 45-Second Verbal Pitch Script

> Practice this out loud 2-3 times before presenting.

**[0-5s] Problem:**  
*"Every team wastes time in status meetings, blockers stay invisible, and managers have no real-time visibility into workload."*

**[5-15s] Solution:**  
*"We built TeamSync AI — an AI-powered team platform using Gemini 2.5 Flash, Cloud Firestore, and Cloud Run. It automates standups, resolves blockers with AI suggestions, and gives teams a real-time analytics dashboard."*

**[15-35s] Demo (show the live app):**  
1. Open the Cloud Run URL *(proves it's deployed)*
2. Show the **Dashboard** tab — KPI cards, donut chart, workload bars
3. Switch to **Task Board** — type *"High-priority bug for Meena about payment timeouts"* → click Create → task appears
4. Switch to **AI Standup** → click Generate → show per-member summaries + blocker alerts

**[35-45s] Technical:**  
*"Clean architecture with separated concerns — 4 Python modules, 29 tests with mocked APIs, 4 Google services, input sanitization, and type hints on every function. Secure, efficient, accessible by design."*

---

## 8. Demo Flow (Step by Step)

| Step | Action | What to Point Out |
|------|--------|------------------|
| 1 | Open the Cloud Run URL | *"This is live on Google Cloud Run"* |
| 2 | Show Dashboard tab (already open) | *"Real-time KPIs — 8 tasks, 12.5% completion, 3 blocked"* |
| 3 | Point to the charts | *"Donut chart for status, bar chart for priority, workload per member"* |
| 4 | Switch to Task Board tab | *"Kanban columns — To Do, In Progress, Done"* |
| 5 | Type a task in NL input: *"Critical bug for Ravi: API rate limits hitting production"* | *"Natural language → Gemini parses it into a structured task"* |
| 6 | Click "🪄 Create Task" | *"AI extracted the title, assignee, priority, and tags"* |
| 7 | Show task appeared with Critical badge | *"Persisted to Firestore — the whole team sees it"* |
| 8 | Switch to AI Standup tab | *"Instead of a 30-min meeting..."* |
| 9 | Click "Generate Today's Standup" | *"Gemini analyzes all tasks and gives per-member updates"* |
| 10 | Point to Blockers section | *"AI detected 3 blockers and suggests specific resolutions"* |

> **Timing:** Steps 1-4 take ~5 seconds. Steps 5-7 take ~8 seconds (AI processing). Steps 8-10 take ~7 seconds.

---

## 9. Pre-Submission Checklist

| Check | Status |
|-------|--------|
| ✅ GitHub repo is **PUBLIC** | Verify before submitting |
| ✅ Repo size < 10 MB | Should be ~50 KB (no large files) |
| ✅ Single branch only (`main`) | Check: `git branch` |
| ✅ Cloud Run URL is **live and accessible** | https://teamsync-ai-551224894530.asia-south1.run.app/ |
| ✅ No `.env` or API keys in repo | `.gitignore` protects `.env` |
| ✅ `test_app.py` in repo | 29 tests, all passing |
| ✅ `README.md` explains setup + architecture | Includes criteria coverage table |
| ✅ `requirements.txt` has all dependencies | 6 pinned production deps |
| ✅ `Dockerfile` is present | Cloud Run config |
| ✅ `.env.example` is committed | Shows required env vars |

---

## 10. Judge Q&A — Quick Answers

### 🎤 Basics

| Question | Answer |
|----------|--------|
| What does your app do? | AI scrum master — automates standups, creates tasks from plain English, shows who's blocked and overloaded. |
| Who is this for? | Engineering team leads managing 4-10 person dev teams. |
| How is it different from Jira? | Jira is passive — you put data in. TeamSync is proactive — AI reads your tasks and tells you what's wrong. |

### 🤝 Collaboration (matches the challenge!)

| Question | Answer |
|----------|--------|
| How does this enable collaboration? | 3 levels: **Shared data** (Firestore — everyone sees same tasks), **Shared awareness** (AI standup shows what everyone's doing), **Shared action** (blocker resolver suggests who should do what). |
| How do multiple people use it? | Same Cloud Run URL. All tasks saved in Firestore — one person creates a task, everyone sees it. |
| Real-time updates? | Currently refreshes on interaction. With more time → Firestore real-time listeners for instant sync. |

### 🏗️ Architecture

| Question | Answer |
|----------|--------|
| Walk me through the architecture | 4 files: `app.py` (UI) → `core.py` (logic) → Gemini API. `config.py` (env vars), `firestore_client.py` (database). |
| Why Streamlit? | Accessible by default + deploys cleanly to Cloud Run. Best speed-quality tradeoff for a 4-hour sprint. |
| Why Firestore not session state? | Session state resets on refresh. Firestore persists — it's a real shared workspace. |
| Why lazy init `_get_model()`? | Global `genai.configure()` crashes tests and Cloud Run cold starts. Lazy = created only when needed. |

### 🔒 Security

| Question | Answer |
|----------|--------|
| How do you prevent prompt injection? | Two layers: `sanitize_text()` regex filter + Gemini's native `system_instruction` API (real separation, not string concat). |
| Where are API keys? | `.env` file (gitignored) + `python-dotenv`. On Cloud Run → `--set-env-vars`. Never hardcoded. |
| Malicious input? | `validate_input()` (length), `sanitize_text()` (injection), `validate_task()` (allowed values only). |

### 🧪 Testing

| Question | Answer |
|----------|--------|
| How many tests? | 29 tests, 14 classes. All Gemini calls mocked. Covers validation, AI features, edge cases, boundaries. |
| Show a test? | Open `test_app.py` → `TestGenerateStandup`: mocks `_get_model()`, checks success + API failure fallback. |

### ☁️ Google Services

| Question | Answer |
|----------|--------|
| Which services? | **Gemini 2.5 Flash** (3 AI features), **Cloud Run** (hosting), **Firestore** (database), **Cloud Logging** (audit trail). |
| Why not Vertex AI? | Vertex adds IAM complexity. Google AI Studio SDK with API key is simpler. Same model underneath. |
| Why not cache Gemini? | Standups must reflect latest tasks. Caching = stale data. Only cache read-only metrics. |

### ⚡ Efficiency

| Question | Answer |
|----------|--------|
| Caching strategy? | `@st.cache_data` on metrics (5 min TTL). `@st.cache_resource` on Firestore client (singleton). AI calls NOT cached. |
| Gemini failures? | Graceful degradation — try/except returns sample data. App never crashes. Fallbacks are logged. |

### 🤖 How I Used AI to Build This

| Question | Answer |
|----------|--------|
| How did you build this in 3 hours? | AI as pair programmer. I prepared `AGENTS.md` rules first (type hints, tests, security). AI generated code, I made all architecture decisions. |
| Did AI write everything? | AI typed fast. I decided what to build, how to structure it, what to test, and which security patterns. Quality = my rules, speed = AI. |
| AI's role vs yours? | AI = fast junior developer. I = architect. AI wrote boilerplate, I reviewed every function and designed the system. |

### 🎲 Curveballs

| Question | Answer |
|----------|--------|
| What would you improve? | Role-based access, Firestore real-time listeners, sprint retrospective generation. |
| Weakest part? | Real-time collab — last-write-wins on simultaneous edits. Fix: Firestore listeners + optimistic locking. |
| Can it scale? | Cloud Run auto-scales. Firestore handles millions of docs. Bottleneck = Gemini rate limits → solve with queuing. |

## Quick Reference Card

```
╔═══════════════════════════════════════════════════════╗
║  TeamSync AI — Quick Stats                            ║
╠═══════════════════════════════════════════════════════╣
║  Files:      6 Python files + Dockerfile + README     ║
║  Tests:      29 tests across 9 classes (all mocked)   ║
║  AI Model:   Gemini 2.5 Flash (3 features)            ║
║  Google:     Cloud Run + Firestore + Cloud Logging    ║
║  Security:   sanitize + validate + system_instruction  ║
║  Caching:    cache_data (metrics) + cache_resource     ║
║  Region:     asia-south1                               ║
║  Live URL:   teamsync-ai-551224894530.asia-south1...   ║
╚═══════════════════════════════════════════════════════╝
```
