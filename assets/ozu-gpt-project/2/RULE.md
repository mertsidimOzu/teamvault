# Project: OzU-GPT

**Last Updated:** 2026-02-19

## Overview

AI assistant platform for Ozyegin University (Özyeğin Üniversitesi). Multi-GPT platform with RAG, GraphRAG, voice chat, Moodle/SharePoint integrations, and multi-provider LLM support.

## Technology Stack

- **Backend:** Python 3.11, FastAPI, SQLAlchemy 2.x, Alembic, Pydantic v2
- **Frontend:** React 18, React Router v6, TailwindCSS, CRA (react-scripts)
- **Database:** PostgreSQL 16 (primary), Qdrant (vector DB for RAG/semantic search)
- **LLM Adapters:** Ollama, OpenAI, Anthropic, Azure OpenAI, OpenRouter, AWS Bedrock
- **Auth:** Azure AD OAuth2/OIDC (MSAL), JWT via httpOnly cookies
- **Infra:** Docker Compose, Nginx (frontend), GitLab CI/CD
- **Testing:** pytest, pytest-asyncio, pytest-cov

## Directory Structure

```
ozu-gpt-main/
├── backend/
│   ├── api/v1/            # Route registration
│   ├── routers/           # FastAPI endpoint handlers
│   ├── use_cases/         # Business logic orchestration
│   ├── services/          # Domain services (chat, rag, voice, etc.)
│   ├── core/              # Middleware, responses, logging, security
│   ├── config/            # Pydantic BaseSettings config
│   ├── alembic/           # DB migrations
│   ├── scripts/           # One-off utility scripts
│   └── main.py            # FastAPI app entry point
├── frontend/
│   └── src/               # React app
├── voice-service/         # Separate Python voice STT service
├── moodle-local_ozugpt/   # Moodle plugin (PHP)
├── docker-compose.yml     # Production
└── docker-compose.dev.yml # Development (no sensitive data)
```

## Key Files

- **Config:** `backend/config/settings.py` (Pydantic BaseSettings), `.env.example`
- **API responses:** `backend/core/responses.py` — use `ApiResponse[T]`, `PaginatedResponse[T]`, `ErrorResponse`
- **Entry point:** `backend/main.py`
- **DB models:** `backend/database.py` (re-exports from `backend/models/`)
- **Migrations:** `backend/alembic/versions/`

## Development Commands

**Backend:**
```bash
pip install -r backend/requirements.txt     # Install deps
pip install -r backend/requirements-dev.txt # + test deps
cd backend && uvicorn main:app --reload --port 8001
alembic upgrade head                         # Run migrations
pytest -q                                    # Run tests
```

**Frontend:**
```bash
cd frontend && npm install
npm start                 # Dev server :3000
npm run build             # Production build
```

**Docker (preferred):**
```bash
docker compose up -d                                    # Production
docker compose -f docker-compose.dev.yml up -d          # Development
```

## Service Ports

| Service    | Production | Development |
|------------|-----------|-------------|
| Backend    | 8001      | 8002        |
| Frontend   | 3000/80   | 3001        |
| PostgreSQL | 5433      | 5434        |
| Qdrant     | 6333      | 6334        |

## API Response Pattern

All endpoints use standardized responses from `core/responses.py`:

```python
from core.responses import ApiResponse, PaginatedResponse, MessageResponse, DeleteResponse

# Success
return ApiResponse.ok(data=result)

# Error
return ApiResponse.fail(code="NOT_FOUND", message="Resource not found")

# Paginated
return PaginatedResponse(data=items, total=100, page=1, page_size=20)
```

## Architecture

Clean layered architecture: `routers` → `use_cases` → `services` → `models/database`

- **Routers:** HTTP concerns only (request parsing, auth, response wrapping)
- **Use cases:** Orchestrate business flows (`use_cases/chat/send_message_use_case.py`)
- **Services:** Domain logic (`services/rag/`, `services/chat/`, `services/vector_db/`)
- **Core:** Cross-cutting concerns (auth, middleware, logging, CSRF, rate limiting)

## RAG Pipeline

Qdrant-based with BM25 + semantic search + RRF reranking. Turkish tokenizer included (`services/rag/turkish_tokenizer.py`). Key components in `services/rag/`: query expander, reranker, smart chunker, citation verifier.

## Security Notes

- JWT_SECRET_KEY must be 64+ chars
- CORS_ORIGINS must never use `"*"` in production
- Dev environment has no sensitive data (no SharePoint sync, no production DB)
- CSRF protection, rate limiting, security headers via `core/middleware.py`