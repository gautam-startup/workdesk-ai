# WorkDesk AI

> **"One affordable AI workspace for everyday work."**

WorkDesk AI is a modular, multi-agent SaaS application designed for students, freelancers, creators, professionals, and small businesses. Instead of functioning as a disconnected collection of chatbot widgets, WorkDesk AI routes everyday tasks to specialized agents (Supervisor, Document, Data, Writer) backed by deterministic programmatic tools and cost-optimized LLM routing.

---

## Phase 1: Project Foundation & Architecture

Phase 1 provides the production-grade foundation with clean separation of concerns:
- **`frontend/`**: React 19, Vite, TypeScript, Tailwind CSS v4, typed API client abstraction, centralized error boundary, and responsive SaaS layout.
- **`backend/`**: Python FastAPI REST API, Pydantic settings & schema validation, SQLAlchemy 2.0 async and sync engines, Alembic migration scaffold, Docker Compose setup, structured logging, request ID tracing, and CORS security.

---

## Directory Structure

```
workdesk-ai/
├── .env.example                  # Environment configuration template
├── README.md                     # Setup instructions & developer guide
├── package.json                  # Root workspace script runner
├── index.html                    # Single-page application entrypoint
├── tsconfig.json                 # TypeScript compiler options
│
├── frontend/                     # Modern React + Vite Frontend
│   ├── package.json              # Frontend dependencies & scripts
│   └── src/
│       ├── api/
│       │   ├── client.ts         # Base API client with request IDs & error normalization
│       │   └── health.ts         # Health check service (GET /api/v1/health)
│       ├── components/
│       │   ├── common/
│       │   │   ├── ErrorBoundary.tsx   # React error boundary
│       │   │   ├── Header.tsx          # Top navigation & backend connectivity pill
│       │   │   ├── HealthWidget.tsx    # Live API latency & status inspector
│       │   │   └── PhaseChecklist.tsx  # Interactive milestone roadmap
│       │   └── layout/
│       │       └── AppLayout.tsx       # Responsive SaaS layout with sidebar
│       ├── pages/
│       │   └── OverviewPage.tsx        # Phase 1 diagnostic & interactive REST tester
│       ├── types/
│       │   ├── api.ts            # Normalized API responses & error contracts
│       │   └── index.ts          # Core domain models
│       └── App.tsx               # Root frontend component
│
├── backend/                      # Production FastAPI REST Backend
│   ├── requirements.txt          # Python dependencies
│   ├── Dockerfile                # Multi-stage production container image
│   ├── docker-compose.yml        # Local PostgreSQL 16 + FastAPI stack
│   ├── alembic.ini               # Database migration configuration
│   ├── alembic/
│   │   ├── env.py                # Alembic runtime environment
│   │   ├── script.py.mako        # Migration revision template
│   │   └── versions/             # Migration revisions (.gitkeep)
│   ├── app/
│   │   ├── main.py               # FastAPI application with CORS, middlewares & handlers
│   │   ├── core/
│   │   │   ├── config.py         # Pydantic Settings (.env loader)
│   │   │   ├── database.py       # SQLAlchemy engine (asyncpg + psycopg2)
│   │   │   ├── logging.py        # Structured logging configuration
│   │   │   └── exceptions.py     # Custom exception hierarchy & standard errors
│   │   ├── schemas/
│   │   │   └── health.py         # Pydantic health check response model
│   │   └── api/
│   │       └── v1/
│   │           ├── router.py     # Aggregated v1 API router
│   │           └── health.py     # GET /api/v1/health probe endpoint
│   └── tests/
│       ├── test_config.py        # Settings & CORS origin parser tests
│       └── test_health.py        # Health schema & exception tests
```

---

## Local Setup & Quickstart

### 1. Prerequisites
- **Node.js**: >= 18.0.0
- **Python**: >= 3.10
- **PostgreSQL**: >= 16 (or use the provided Docker Compose)
- **Docker & Docker Compose** (optional, recommended for DB)

### 2. Environment Configuration
Copy the configuration template:
```bash
cp .env.example .env
```

---

### 3. Backend Setup

#### Option A: Running with Docker Compose (Recommended for Local Dev)
Starts both PostgreSQL 16 and the FastAPI backend service with a single command:
```bash
docker compose -f backend/docker-compose.yml up --build -d
```
The backend will be available at:
- **API Base:** `http://localhost:8000`
- **Health Check:** `http://localhost:8000/api/v1/health`
- **Interactive OpenAPI Docs:** `http://localhost:8000/docs`

To view logs:
```bash
docker compose -f backend/docker-compose.yml logs -f api
```

#### Option B: Running Natively with Python Virtualenv
1. Create and activate a virtual environment:
```bash
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

2. Install backend dependencies:
```bash
pip install -r backend/requirements.txt
```

3. Ensure PostgreSQL is running and update `DATABASE_URL` in `.env` if necessary.

4. Run database migrations:
```bash
cd backend
alembic upgrade head
cd ..
```

5. Start the FastAPI development server:
```bash
uvicorn backend.app.main:app --reload --host 0.0.0.0 --port 8000
```

---

### 4. Running Backend Tests
Execute unit and contract tests using either `pytest` or Python's standard `unittest`:
```bash
# Using pytest
pytest backend/tests

# Or using standard unittest
python3 -m unittest discover backend/tests
```

---

### 5. Frontend Setup

1. Install Node.js dependencies:
```bash
npm install
```

2. Start the Vite development server:
```bash
npm run dev
```

3. Open your browser at:
`http://localhost:3000` (or `http://localhost:5173`)

---

## API Health Probe Verification

You can verify the backend health probe using `curl`:
```bash
curl -X GET http://localhost:8000/api/v1/health
```

Example response:
```json
{
  "status": "healthy",
  "project_name": "WorkDesk AI Backend",
  "environment": "development",
  "version": "1.0.0",
  "database": "connected",
  "timestamp": "2026-10-01T09:22:00.000000",
  "services": {
    "api": "operational",
    "cors_configured": true,
    "database_driver": "asyncpg",
    "phase": "Phase 1 - Foundation & Project Structure"
  }
}
```

---

## Security & Architecture Highlights

1. **CORS Security:** Configured in `backend/app/main.py` using strict whitelist origins (`http://localhost:3000`, `http://localhost:5173`) with allowed methods, headers, and credential support.
2. **Centralized Error Handling:** Standardized error envelope (`error_code`, `message`, `details`, `request_id`) via `AppException` across all HTTP endpoints.
3. **Structured Logging:** Every request is assigned a correlation ID (`X-Request-ID`), tracking route, method, status code, and latency in milliseconds.
4. **Resilient Client Abstraction:** The frontend `apiClient` wraps requests, exposes typed responses, attaches tracing headers, and gracefully surfaces connection diagnostics.

---

## Phase 2: Authentication, Subscriptions & Credit Ledger

Phase 2 implements the complete authentication engine and PostgreSQL schema:
- **Authentication Endpoints:**
  - `POST /api/v1/auth/signup`: Validates email, enforces strong passwords, allocates initial 10 free credits, creates user with UUID PK, and returns a signed HS256 JWT.
  - `POST /api/v1/auth/login`: Authenticates credentials with constant-time password verification and sliding-window rate limiting.
  - `POST /api/v1/auth/logout`: Invalidates session.
  - `GET /api/v1/auth/me`: Protected route returning user profile, role, credit quota, and active subscription.
  - `GET /api/v1/auth/admin/users`: Admin-only RBAC endpoint listing all registered accounts.
- **Normalized PostgreSQL Schema (Alembic 001_initial_schema):**
  - `users`, `sessions`, `plans`, `subscriptions`, `payments`, `credits`, `credit_transactions`, `conversations`, `messages`, `documents`, `agent_runs`, `audit_logs`, `usage_logs`.
- **Security & Authorization:**
  - PBKDF2-HMAC-SHA256 password hashing (100,000 iterations).
  - Bearer JWT token strategy with signature validation & expiration checks.
  - Role-based access control (User vs. Admin) with 403 Forbidden enforcement.
  - In-memory sliding-window rate limiter protecting auth endpoints.
- **Frontend SaaS Authentication Flow:**
  - Login Page with show/hide password and one-click demo fast-testing.
  - Signup Page with password strength indicators and instant free credit allocation.
  - Protected Dashboard displaying personalized greeting, credit quota, subscription plan, prompt dispatcher, and Admin User Directory (RBAC).

---

## Phase 3: Supervisor Agent & Smart LLM Routing Layer

Phase 3 builds the multi-agent AI engine and resilience pipeline:
- **LLM Provider Abstraction:**
  - `BaseLLMProvider` abstract base class with standardized `LLMResponse`.
  - `GroqProvider` (default: `llama-3.3-70b-versatile`): ultra-fast inference for low-latency tasks.
  - `MistralProvider` (default: `mistral-large-latest`): complex reasoning, analysis, and multilingual tasks.
  - `GeminiProvider` (default: `gemini-2.5-flash`): multimodal documents and long-context processing.
- **Smart AI Router (`AIRouter`):**
  - Routes by complexity (`low` -> Groq, `medium` -> preferred/Groq, `high` -> Mistral, `multimodal` -> Gemini).
  - Implements automatic **provider fallback**: if primary provider times out or fails, automatically retries and falls back to secondary providers.
- **Supervisor Agent:**
  - Classifies user requests into specialized domains (`writer`, `document`, `data`, `research`).
  - Emits safe, transparent execution milestones without exposing internal chain of thought.
- **Specialist Agents:**
  - `WriterAgent`: Synthesizes tone-transformed proposals, emails, and WhatsApp-style messages.
  - `DocumentAgentSkeleton`: Scoped to `document:extract` tools for PDF/DOCX processing.
  - `DataAgentSkeleton`: Scoped to `data:profile` and `data:clean` tools for deterministic dataset profiling.
  - `ResearchAgentSkeleton`: Scoped to `research:search` tools.
- **Tool Registry & Explicit Permissions Gatekeeper:**
  - Enforces explicit agent permission boundaries; prevents unauthorized tool invocation with `403 UNAUTHORIZED_TOOL_ACCESS`.
- **Credit Estimation & Quota Enforcement:**
  - Pre-flight checks before execution; blocks requests with `402 INSUFFICIENT_CREDITS` if balance is inadequate.
- **REST Endpoints:**
  - `POST /api/v1/agent/run`: Dispatches multi-agent workflow, updates remaining credits, and returns safe execution steps.
  - `POST /api/v1/agent/estimate`: Returns credit cost estimate before dispatch.
  - `GET /api/v1/agent/tools`: Lists public registered tools and agent permission boundaries.

---

## Phase 4: Document Intelligence Agent & RAG Pipeline

Phase 4 builds the complete Document Agent with multi-modal format ingestion, deterministic extraction, prompt-injection defense, and semantic vector RAG:
- **Multi-Format Document Support:**
  - PDF: Page-by-page text stream parsing and layout tracking.
  - DOCX: Native OpenXML paragraph extraction.
  - TXT: Character set normalized text parsing.
  - PNG, JPG/JPEG: OCR-ready visual document extraction.
- **Storage & Size Enforcement:**
  - `document_storage`: User-isolated directory paths, SHA-256 integrity checksums, and strict 25MB file size limit.
- **Adversarial Prompt-Injection Defense:**
  - Neutralizes injection triggers (e.g., `ignore previous instructions`, `output system prompt`) before any vector indexing or LLM ingestion.
  - **Privacy:** Document contents are never exposed in application server logs.
- **Semantic Chunking & Vector Storage Abstraction:**
  - Overlapping chunking preserving explicit page references (`page_number`).
  - Dense semantic embeddings with cosine similarity retrieval.
- **RAG & Document Intelligence:**
  - Question & Answer with verifiable page citations (`[Page 1]`, `[Page 2]`).
  - Executive Summaries, Bulleted Key Takeaways, and Action Items with Deadlines.
  - Foundation for cross-document differential comparison.
- **REST Endpoints:**
  - `POST /api/v1/documents/upload`: Uploads and indexes documents with user ownership.
  - `GET /api/v1/documents`: Lists documents belonging exclusively to the requesting user.
  - `GET /api/v1/documents/{id}`: Returns document details, pages, and chunk counts.
  - `DELETE /api/v1/documents/{id}`: Deletes document file and purges vector index (ownership checked).
  - `POST /api/v1/documents/{id}/ask`: RAG query endpoint with citations and relevance scores.
- **Frontend Document Studio (`DocumentsPage.tsx`):**
  - Drag-and-drop file upload with format validation.
  - Document inventory list with page count and chunk metrics.
  - Tabbed inspector: Interactive Q&A chat, Executive Briefing & Action Items, and Text Preview.

---

## Phase 5: Data Analysis Agent (Deterministic Pandas Engine)

Phase 5 builds the complete Data Agent with tabular format parsing, schema detection, health scoring, safe operations, and report generation:
- **Supported Formats:** CSV, XLSX (Excel spreadsheets).
- **Zero Arbitrary Code Execution Sandbox:**
  - Strictly prevents LLM-generated arbitrary Python string execution or `eval()`.
  - Enforces closed, parameter-validated operation dispatch (`SafeOperationDispatcher`).
- **Predefined Safe Deterministic Operations:**
  - `remove_duplicates`: Deduplicates rows by all columns or subset.
  - `fill_missing`: Imputes nulls using `mean`, `median`, `mode`, or `constant` value.
  - `rename_column`: Replaces column headers cleanly.
  - `filter_rows`: Compares column values with safe comparison operators (`==`, `!=`, `>`, `<`, `>=`, `<=`, `contains`).
  - `sort_rows`: Sorts rows ascending/descending with null values placed safely at the end.
  - `group_by`: Group by columns and aggregates numeric target metrics (`sum`, `mean`, `count`, `min`, `max`).
  - `describe`: Computes statistical summaries (count, mean, std, min, 25%, 50%, 75%, max).
  - `correlation`: Pearson correlation matrix for numeric columns.
  - `time_series_summary`: Aggregated chronological trends.
- **Profiling & Automatic Chart Generation:**
  - Schema detection: Numeric, Text, Datetime, Boolean.
  - Dataset health & quality scoring (0-100%).
  - Detected issues list with one-click auto-cleaning.
  - Frontend-ready chart configs (Bar charts, Line charts).
- **REST Endpoints:**
  - `POST /api/v1/data/upload`: Uploads and profiles CSV or XLSX files.
  - `GET /api/v1/data`: Lists user's uploaded datasets.
  - `GET /api/v1/data/{id}`: Retrieves dataset details, column types, statistics, and preview rows.
  - `POST /api/v1/data/{id}/transform`: Executes validated predefined operations.
  - `POST /api/v1/data/{id}/report`: Synthesizes AI natural-language business briefings.
  - `GET /api/v1/data/{id}/download`: Exports cleaned dataset as CSV.
  - `DELETE /api/v1/data/{id}`: Deletes dataset.
- **Frontend Data Studio (`DataPage.tsx`):**
  - Interactive tabular preview with column data type badges.
  - Health score gauge and detected issues alerts.
  - One-click auto clean & custom transformation toolbars.
  - Visual distribution charts.
  - Cleaned dataset download action.

---

## Roadmap

- [x] **Phase 1:** Foundation, Shared Contracts & Unified Frontend Workspace Shell
- [x] **Phase 2:** Authentication, Subscriptions & Credit Ledger (JWT, PostgreSQL schema, RBAC)
- [x] **Phase 3:** Supervisor Agent & Smart LLM Routing Layer (Mistral, Gemini, Groq, Tool Registry)
- [x] **Phase 4:** Document Intelligence Agent & RAG Pipeline (PDF, DOCX, TXT, Images, Vector RAG)
- [x] **Phase 5:** Data Analysis Agent (Deterministic Pandas profiling, Safe Transforms, Charts)
- [ ] **Phase 6:** Writer Agent & Multi-Tone Synthesis
- [ ] **Phase 7:** Admin Dashboard, Telemetry & Production Hardening
