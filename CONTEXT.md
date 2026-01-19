# Project Context Document

## RAG Knowledge Base with Chat Interface

**Date**: 2024-01-19
**Project Type**: Greenfield (New Project)

---

## Existing Constraints

### Infrastructure Constraints
- **Deployment Target**: Self-hosted or cloud VPS (AWS/GCP/Azure)
- **Database**: PostgreSQL 15+ with pgvector extension (or Qdrant as alternative)
- **File Storage**: Local filesystem with configurable S3-compatible backend
- **Container Orchestration**: Docker Compose for development, Kubernetes for production (optional)
- **Reverse Proxy**: Nginx or Caddy for production deployments

### Technical Constraints
- **Backend**: Python 3.11+
- **Frontend**: React 18+ with TypeScript
- **API Communication**: REST API with WebSocket for streaming chat responses
- **Authentication**: JWT-based auth with HTTP-only cookies
- **Rate Limiting**: 10 queries/minute per user (configurable)
- **File Size Limit**: 50MB per file (configurable)
- **Supported File Formats**: 15+ types (PDF, DOCX, TXT, MD, CSV, JSON, HTML, XML, code files, images with OCR)

### Budget Constraints
- **Embedding API**: OpenAI text-embedding-3-small (~$0.02/1M tokens)
- **LLM API**: OpenAI GPT-4o or compatible model for chat responses
- **Hosting**: Target <$50/month for small-scale deployment (10 users, 1000 documents)
- **Storage**: Estimate 100GB for documents + embeddings

### Timeline Constraints
- **MVP Target**: 4-6 weeks to deployable prototype
- **Phase 1**: File upload, ingestion, basic search (2 weeks)
- **Phase 2**: Chat interface with streaming (1 week)
- **Phase 3**: User auth, document management (1 week)
- **Phase 4**: Polish, testing, deployment (1-2 weeks)

### Team Constraints
- **Development**: Single developer initially
- **Code Review**: Self-review with AI assistance
- **Testing**: Automated tests required (80% coverage backend, 75% frontend)
- **Documentation**: README with setup instructions required

### Legal/Compliance Constraints
- **Data Privacy**: User documents stored encrypted at rest
- **Data Ownership**: Users maintain full ownership of uploaded content
- **GDPR/CCPA**: Support data export and deletion requests
- **No Training Data**: User documents NOT used to train external models

---

## Tech Stack

### Backend Stack
| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **API Framework** | FastAPI | 0.104+ | High-performance async API |
| **Python Version** | Python | 3.11+ | Runtime environment |
| **Database** | PostgreSQL | 15+ with pgvector | Relational data + vector search |
| **ORM** | SQLAlchemy | 2.0+ | Database abstraction |
| **Migrations** | Alembic | 1.12+ | Database schema management |
| **Task Queue** | Celery | 5.3+ | Background job processing |
| **Message Broker** | Redis | 7.0+ | Task queue broker + caching |
| **Embeddings** | OpenAI API | text-embedding-3-small | Vector embeddings |
| **LLM** | OpenAI API | GPT-4o | Chat responses |
| **File Parsing** | PyPDF2, python-docx, beautifulsoup4 | Latest | Multi-format text extraction |
| **OCR** | pytesseract | Latest | Image-to-text extraction |
| **Authentication** | python-jose | Latest | JWT token handling |
| **Password Hashing** | bcrypt | 4.1+ | Secure password storage |
| **Validation** | Pydantic | 2.0+ | Request/response validation |
| **Testing** | pytest, pytest-asyncio | Latest | Unit and integration tests |
| **HTTP Client** | httpx | Latest | Async HTTP requests |

### Frontend Stack
| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Framework** | React | 18.2+ | UI framework |
| **Language** | TypeScript | 5.0+ | Type-safe JavaScript |
| **Build Tool** | Vite | 5.0+ | Fast dev server and builds |
| **State Management** | Zustand | 4.4+ | Lightweight state management |
| **HTTP Client** | axios | 1.6+ | API communication |
| **Routing** | React Router | 6.20+ | Client-side routing |
| **UI Components** | Tailwind CSS + shadcn/ui | Latest | Styled components |
| **Forms** | react-hook-form | 7.48+ | Form handling |
| **File Upload** | react-dropzone | 14.0+ | Drag-drop upload |
| **Markdown Rendering** | react-markdown | 9.0+ | Chat response formatting |
| **Syntax Highlighting** | prism-react-renderer | 2.3+ | Code block formatting |
| **Testing** | Vitest, React Testing Library | Latest | Unit and component tests |
| **E2E Testing** | Playwright | Latest | End-to-end tests |

### DevOps Stack
| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Containerization** | Docker | Application containers |
| **Orchestration** | Docker Compose | Local development setup |
| **Reverse Proxy** | Nginx | Production routing |
| **Process Management** | Supervisor / systemd | Production process management |
| **Monitoring** | Prometheus + Grafana | Metrics and dashboards |
| **Logging** | Structured JSON logs | Log aggregation |
| **Error Tracking** | Sentry | Error monitoring |
| **CI/CD** | GitHub Actions | Automated testing and deployment |

### Development Tools
| Category | Tools |
|----------|-------|
| **Version Control** | Git + GitHub |
| **Code Quality** | Ruff (Python), ESLint (TypeScript) |
| **Type Checking** | mypy (Python), tsc (TypeScript) |
| **Formatting** | Black (Python), Prettier (TypeScript) |
| **Pre-commit Hooks** | pre-commit |
| **API Documentation** | FastAPI auto-docs (Swagger/ReDoc) |

---

## Environment Variables Required

### Backend (.env)
```bash
# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/ragdb
POSTGRES_USER=rag_user
POSTGRES_PASSWORD=secure_password
POSTGRES_DB=rag_db

# Redis
REDIS_URL=redis://localhost:6379/0

# Authentication
SECRET_KEY=your-secret-key-min-32-chars
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# OpenAI
OPENAI_API_KEY=sk-...
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_CHAT_MODEL=gpt-4o

# File Storage
UPLOAD_DIR=/data/documents
MAX_FILE_SIZE_MB=50
ALLOWED_EXTENSIONS=pdf,docx,txt,md,csv,json,html,xml,py,js,ts,jpg,png

# CORS
FRONTEND_URL=http://localhost:5173
```

### Frontend (.env)
```bash
VITE_API_URL=http://localhost:8000
VITE_WS_URL=ws://localhost:8000
```

---

## Project Structure

```
RAG/
├── backend/
│   ├── app/
│   │   ├── api/              # API routes
│   │   ├── core/             # Security, config, deps
│   │   ├── db/               # Database session, models
│   │   ├── models/           # SQLAlchemy models
│   │   ├── schemas/          # Pydantic schemas
│   │   ├── services/         # Business logic
│   │   │   ├── embedding.py  # Embedding generation
│   │   │   ├── parser.py     # File parsing
│   │   │   └── chat.py       # Chat orchestration
│   │   └── tasks/            # Celery tasks
│   ├── tests/                # Backend tests
│   ├── alembic/              # Database migrations
│   ├── Dockerfile
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/       # React components
│   │   ├── pages/            # Page components
│   │   ├── hooks/            # Custom hooks
│   │   ├── stores/           # Zustand stores
│   │   ├── services/         # API client
│   │   └── types/            # TypeScript types
│   ├── tests/                # Frontend tests
│   ├── Dockerfile
│   └── package.json
├── docker-compose.yml        # Dev orchestration
├── .env.example              # Environment template
├── PRD.md                    # Product requirements
├── CONTEXT.md                # This file
└── README.md                 # Setup instructions
```

---

## Non-Goals (Out of Scope for MVP)

### Phase 1 Exclusions
- Multi-tenancy (organizations, teams)
- Document sharing between users
- Real-time collaborative editing
- Advanced RAG techniques (hybrid search, re-ranking, query expansion)
- Fine-tuned embedding models
- Offline mode
- Mobile apps (iOS/Android)
- WebRTC voice queries
- Document versioning
- Advanced analytics dashboard
- A/B testing framework

### Future Considerations
- Hybrid search (keyword + semantic)
- Query re-ranking with Cross-Encoders
- Document summarization on upload
- Chat with multiple documents in context
- Export chat history to PDF
- Integrations (Notion, Obsidian, Google Drive)
- Local LLM support (Ollama, PrivateGPT)

---

## Success Metrics

### Technical Metrics
- API response time < 200ms (p95)
- File ingestion throughput > 100MB/min
- Search latency < 2 seconds (p95)
- 99.5% uptime target
- Test coverage ≥ 80% backend, ≥ 75% frontend

### User Metrics
- Time to first successful upload < 30 seconds
- Time to first relevant answer < 1 minute
- User retention (Day 7) > 50%
- Average queries per session > 5

### Business Metrics
- Cost per user < $5/month (infrastructure + APIs)
- Time to MVP completion < 6 weeks
- Ready for production deployment with < 1 day setup

---

**END OF CONTEXT DOCUMENT**
