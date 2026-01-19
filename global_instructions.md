# Global Instructions & Coding Constitution
## RAG Knowledge Base with Chat Interface

**Version**: 1.0
**Status**: IMMUTABLE (Changes require unanimous approval)
**Last Updated**: 2024-01-19

---

## 1. Immutable Laws

These laws are NEVER violated. Any agent action violating these laws is INVALID.

### Law 1: Simplicity is King
**Always choose the simplest implementation.**
- If a library isn't strictly necessary, do not add it
- Prefer standard library over external packages
- Delete code rather than commenting it out
- When in doubt: less code > more code

### Law 2: Small Files
**Many small, single-purpose files over large complex ones.**
- Maximum 50 lines per file (Python), 150 lines (TypeScript)
- Maximum 20 lines per function
- One export per module (TypeScript)
- If file grows beyond limit: split it

### Law 3: Atomic Scope
**Never mix refactoring with feature work. Never mix debugging with new generation.**
- Feature work: add/edit code for ONE user story only
- Refactoring: separate PR, separate task
- Bug fix: write test first, fix second, one bug per commit

### Law 4: No Magic
**Code must be explicit. No clever one-liners that are hard to read.**
- Name every variable meaningfully
- Extract complex logic to named functions
- Avoid nested ternary operators
- Comments explain WHY, not WHAT

### Law 5: Test Sanctity
**Tests are the source of truth. Implementation serves tests, not vice versa.**
- Tests define correct behavior
- Implementation is wrong if tests fail
- Never modify tests to pass bad implementation
- Tests become READ-ONLY after Phase 7.3

---

## 2. Architectural Pattern

### 2.1 Project Structure (Exact)

```
RAG/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   ├── deps.py          # FastAPI dependencies
│   │   │   └── routes/
│   │   │       ├── __init__.py
│   │   │       ├── auth.py      # Auth endpoints (≤50 lines)
│   │   │       ├── documents.py # Document endpoints (≤50 lines)
│   │   │       └── chat.py      # Chat endpoints (≤50 lines)
│   │   ├── core/
│   │   │   ├── config.py        # Settings (pydantic-settings)
│   │   │   └── security.py      # JWT, password hashing (≤50 lines)
│   │   ├── db/
│   │   │   ├── session.py       # Database session (≤30 lines)
│   │   │   └── base.py          # Base model (≤20 lines)
│   │   ├── models/
│   │   │   ├── user.py          # User ORM (≤50 lines)
│   │   │   ├── document.py      # Document ORM (≤50 lines)
│   │   │   └── chunk.py         # Chunk ORM (≤30 lines)
│   │   ├── schemas/
│   │   │   ├── user.py          # User schemas (≤50 lines)
│   │   │   ├── document.py      # Document schemas (≤50 lines)
│   │   │   └── chat.py          # Chat schemas (≤50 lines)
│   │   ├── services/
│   │   │   ├── embedding.py     # Embedding generation (≤50 lines)
│   │   │   ├── parser.py        # File parsing (≤50 lines)
│   │   │   └── chat.py          # Chat orchestration (≤50 lines)
│   │   └── tasks/
│   │       └── ingestion.py     # Celery tasks (≤50 lines)
│   ├── tests/
│   │   ├── unit/
│   │   │   ├── services/
│   │   │   └── routes/
│   │   ├── integration/
│   │   │   └── api/
│   │   └── conftest.py          # Pytest fixtures
│   ├── alembic/
│   │   └── versions/
│   ├── Dockerfile
│   └── requirements.txt
│
├── frontend/
│   ├── src/
│   │   ├── features/
│   │   │   ├── auth/
│   │   │   │   ├── components/
│   │   │   │   │   ├── LoginForm.tsx      # (≤150 lines)
│   │   │   │   │   └── RegisterForm.tsx   # (≤150 lines)
│   │   │   │   ├── hooks/
│   │   │   │   │   └── useAuth.ts         # (≤50 lines)
│   │   │   │   ├── services/
│   │   │   │   │   └── authApi.ts         # (≤50 lines)
│   │   │   │   └── types/
│   │   │   │       └── auth.ts            # (≤30 lines)
│   │   │   ├── upload/
│   │   │   │   ├── components/
│   │   │   │   │   ├── Dropzone.tsx       # (≤150 lines)
│   │   │   │   │   └── FileList.tsx       # (≤150 lines)
│   │   │   │   ├── hooks/
│   │   │   │   │   └── useUpload.ts       # (≤50 lines)
│   │   │   │   └── types/
│   │   │   │       └── upload.ts          # (≤30 lines)
│   │   │   └── chat/
│   │   │       ├── components/
│   │   │       │   ├── ChatInterface.tsx  # (≤150 lines)
│   │   │       │   └── MessageList.tsx    # (≤150 lines)
│   │   │       ├── hooks/
│   │   │       │   └── useChat.ts         # (≤50 lines)
│   │   │       └── types/
│   │   │           └── chat.ts            # (≤30 lines)
│   │   ├── shared/
│   │   │   ├── lib/
│   │   │   │   └── api.ts                # Axios wrapper (≤50 lines)
│   │   │   ├── types/
│   │   │   │   └── common.ts             # Shared types (≤50 lines)
│   │   │   └── utils/
│   │   │       └── format.ts             # Formatters (≤50 lines)
│   │   └── main.tsx
│   ├── tests/
│   │   ├── unit/
│   │   └── e2e/
│   ├── Dockerfile
│   └── package.json
│
├── docker-compose.yml
├── .env.example
├── PRD.md
├── CONTEXT.md
├── OBJECTIVE_KERNEL.yaml
└── global_instructions.md
```

### 2.2 File Placement Rules

| Rule | Description | Examples |
|------|-------------|----------|
| **New feature** | `src/features/{feature-name}/` | `upload`, `chat`, `auth` |
| **Shared (2+ features)** | `src/shared/` | API client, types, utils |
| **Test file** | Mirror path in `tests/` | `tests/unit/services/embedding.test.ts` |
| **Max file size** | 50 lines (Python), 150 lines (TS) | Split when exceeded |
| **Max function size** | 20 lines | Extract if exceeded |
| **One export per module** | TypeScript only | Prefer file-per-component |

---

## 3. Coding Standards

### 3.1 Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| **Python Files** | snake_case | `user_service.py`, `chat_api.py` |
| **TypeScript Files** | kebab-case | `user-service.ts`, `chat-interface.tsx` |
| **Python Classes** | PascalCase | `UserService`, `DocumentParser` |
| **Python Functions** | snake_case | `validate_email`, `get_document_chunks` |
| **TS Functions** | camelCase | `validateEmail`, `getDocumentChunks` |
| **Constants** | SCREAMING_SNAKE | `MAX_FILE_SIZE`, `DEFAULT_TIMEOUT` |
| **Private (Python)** | leading underscore | `_internal_method` |
| **Private (TS)** | No prefix (use export control) | `export const helper` |
| **Types/Interfaces** | PascalCase | `IUserService`, `TUserInput` |
| **Test Files** | `*_test.py` or `*.test.ts` | `user_service_test.py` |
| **Fixtures** | `conftest.py` or `*.fixture.ts` | `document_fixture.py` |

### 3.2 Error Handling Strategy

**Python (Result Type Pattern)**:
```python
from typing import TypeVar, Generic
from dataclasses import dataclass

T = TypeVar('T')
E = TypeVar('E')

@dataclass
class Ok(Generic[T]):
    value: T

@dataclass
class Err(Generic[E]):
    error: E

Result = Ok[T] | Err[E]

# Usage
def parse_document(file_path: str) -> Result[str, ParseError]:
    try:
        content = _read_file(file_path)
        return Ok(content)
    except IOException as e:
        return Err(ParseError(f"Failed to read: {e}"))
```

**TypeScript (Result Type Pattern)**:
```typescript
type Result<T, E> =
  | { ok: true; value: T }
  | { ok: false; error: E };

// Usage
function parseDocument(filePath: string): Result<string, ParseError> {
  try {
    const content = readFile(filePath);
    return { ok: true, value: content };
  } catch (e) {
    return { ok: false, error: new ParseError(`Failed: ${e}`) };
  }
}
```

**Rules**:
- NEVER throw raw exceptions in business logic
- ALWAYS use typed Result objects
- Exceptions allowed ONLY at system boundaries (HTTP handlers)
- Every error type defined in `types/errors.ts` or `core/errors.py`

### 3.3 Async/Await Rules

**Python**:
```python
# ALWAYS use async/await
async def fetch_document(doc_id: str) -> Document:
    return await db.query(doc_id)

# ALWAYS handle errors
async def process_upload(file_id: str):
    try:
        await ingest_file(file_id)
    except IngestionError as e:
        logger.error(f"Ingestion failed: {e}")
        raise
```

**TypeScript**:
```typescript
// ALWAYS use async/await (never .then chains)
async function fetchDocument(docId: string): Promise<Document> {
  return await db.query(docId);
}

// ALWAYS handle errors
async function processUpload(fileId: string): Promise<void> {
  try {
    await ingestFile(fileId);
  } catch (e) {
    logger.error(`Ingestion failed: ${e}`);
    throw e;
  }
}
```

**Rules**:
- ALWAYS use async/await over .then() chains
- ALWAYS handle errors with try/catch at call site
- NEVER fire-and-forget async operations
- ALWAYS add timeout to external calls

### 3.4 Import Rules

**Python**:
```python
# Group 1: Standard library
import os
from pathlib import Path
from typing import Optional

# Group 2: Third-party
from fastapi import Depends
from sqlalchemy.orm import Session

# Group 3: Local application
from app.models.document import Document
from app.services.embedding import generate_embedding
```

**TypeScript**:
```typescript
// Group 1: External (node_modules)
import { useState, useEffect } from 'react';
import axios from 'axios';

// Group 2: Shared (@/ alias)
import { useAuthStore } from '@/shared/stores/auth';
import { api } from '@/shared/lib/api';

// Group 3: Feature-local
import { ChatMessage } from '../types/chat';
import { useChat } from '../hooks/useChat';
```

**Rules**:
- Absolute imports for cross-feature (use @/ alias in TS)
- Relative imports within same feature
- No circular dependencies (enforced by linter)
- Sort imports alphabetically within groups

### 3.5 Type Safety Rules

**Python**:
```python
from typing import List, Optional, Dict
from pydantic import BaseModel

# ALWAYS specify return types
def get_document(doc_id: str) -> Optional[Document]:
    ...

# ALWAYS use Pydantic for request/response
class DocumentCreate(BaseModel):
    filename: str
    content: str
```

**TypeScript**:
```typescript
// ALWAYS use explicit types (no 'any')
interface Document {
  id: string;
  filename: string;
  content: string;
}

function getDocument(docId: string): Document | null {
  return db.find(docId) ?? null;
}
```

**Rules**:
- Python: strict mode with mypy
- TypeScript: strict mode in tsconfig.json
- No `any` types (use `unknown` if truly unknown)
- All function parameters typed

---

## 4. AI Behavior Protocol

### 4.1 Test Sanctity (CRITICAL - IMMUTABLE)

```
WHEN IMPLEMENTING CODE (PHASE 8+):
- You MUST NOT modify any file in tests/
- You MUST NOT delete test assertions
- You MUST NOT comment out failing tests
- You MUST NOT add @skip or @pytest.skip decorators
- You MUST NOT change expected values to match actual

IF A TEST FAILS:
- The IMPLEMENTATION is wrong
- Fix the implementation, not the test
- If test seems wrong: ESCALATE to human review
- Test hashes are verified at Phase 9 gates
```

### 4.2 Bug Fixing Protocol

```
WHEN FIXING A BUG:
1. FIRST: Write a failing test that reproduces the bug
2. THEN: Fix the implementation
3. VERIFY: All tests pass (old and new)
4. NEVER: Fix bug without test proving it existed

BUG FIX EXAMPLE:
# Step 1: Write failing test
def test_upload_duplicate_file_rejected():
    result = upload_file(test_pdf)
    assert result.status == 409

# Step 2: Fix implementation
def upload_file(file):
    if is_duplicate(file):
        return Response(status=409)

# Step 3: Verify all tests pass
# pytest passes
```

### 4.3 Clarification Protocol

```
IF A REQUIREMENT IS VAGUE:
- STOP immediately
- ASK for clarification via AskUserQuestion tool
- DO NOT assume or guess
- DO NOT proceed with "reasonable" defaults

EXAMPLE VAGUE REQUIREMENTS:
- "Make it fast" → Ask: "What is the target latency?"
- "User-friendly" → Ask: "What specific UX pattern?"
- "Handle errors" → Ask: "Which errors? How?"
```

### 4.4 Scope Discipline

```
WHEN ASKED TO IMPLEMENT FEATURE X:
- Implement ONLY Feature X
- Do NOT refactor adjacent code
- Do NOT add "improvements"
- Do NOT fix unrelated issues
- Do NOT add features not in PRD

VIOLATION EXAMPLES:
❌ "Added extra error handling while fixing button"
❌ "Refactored service layer while adding endpoint"
❌ "Fixed typos in comments while implementing chat"

CORRECT EXAMPLES:
✅ "Implemented document upload endpoint"
✅ "Added JWT authentication"
✅ "Created chat interface component"
```

### 4.5 File Modification Rules

```
BEFORE MODIFYING ANY FILE:
1. Check if file is in writable_paths (from task manifest)
2. If NOT in writable_paths → REFUSE and log
3. If in read_only_paths → REFUSE and log
4. Log every file modification with reason

READ-ONLY PATHS (PHASE 8+):
- tests/**/* (ALL test files)
- PRD.md
- CONTEXT.md
- OBJECTIVE_KERNEL.yaml
- global_instructions.md (this file)
- .phase_*/*.LOCK files
```

### 4.6 Code Generation Rules

```
WHEN GENERATING CODE:
1. READ global_instructions.md FIRST
2. READ relevant PRD sections
3. READ task manifest for scope
4. THEN generate code complying with all standards
5. VERIFY against T-weights in OBJECTIVE_KERNEL.yaml

IF STANDARDS CONFLICT:
- T1 (Correctness) > T2 (Simplicity) > T3 (Reliability) > T4 (Extensibility)
- When in doubt: prefer explicit over clever
- When in doubt: prefer simple over complex
- When in doubt: ask human
```

---

## 5. Testing Standards

### 5.1 Test Structure

```
tests/
├── unit/                    # Isolated function tests
│   ├── services/
│   │   ├── embedding_test.py
│   │   └── parser_test.py
│   └── routes/
│       └── documents_test.py
├── integration/             # Service interaction tests
│   └── api/
│       ├── upload_test.py
│       └── chat_test.py
├── e2e/                     # Full user flow tests
│   ├── upload_flow_test.py
│   └── chat_flow_test.py
└── fixtures/                # Test data factories
    ├── document_fixtures.py
    └── user_fixtures.py
```

### 5.2 Test Naming

**Python (pytest)**:
```python
def test_{unit}_{scenario}_{expected outcome}

# Examples
def test_validate_email_empty_string_returns_error():
    ...

def test_upload_file_pdf_successfully_ingests():
    ...

def test_chat_query_no_results_returns_empty_sources():
    ...
```

**TypeScript (Vitest)**:
```typescript
describe('Feature', () => {
  test('unit + scenario + expected outcome', () => {
    // ...
  });
});

// Example
describe('ChatInterface', () => {
  test('sending message with valid query returns response', () => {
    // ...
  });
});
```

### 5.3 Test Independence

**Rules**:
- Each test MUST be runnable in isolation
- No shared mutable state between tests
- No test order dependencies
- Setup/teardown in fixtures, not inline

**Python (pytest fixtures)**:
```python
@pytest.fixture
def clean_db():
    # Setup
    db.create_all()
    yield
    # Teardown
    db.drop_all()

def test_upload_succeeds(clean_db):
    # Use clean_db
    ...
```

**TypeScript (beforeEach/afterEach)**:
```typescript
let store: ChatStore;

beforeEach(() => {
  store = createChatStore();
});

afterEach(() => {
  store.cleanup();
});

test('sends message', () => {
  // Use fresh store
});
```

### 5.4 Coverage Requirements

| Metric | Backend (Python) | Frontend (TypeScript) |
|--------|-----------------|----------------------|
| **Line Coverage** | ≥ 80% | ≥ 75% |
| **Branch Coverage** | ≥ 70% | ≥ 65% |
| **Critical Path** | 100% | 100% |
| **Edge Cases** | 100% | 100% |

**Critical Path Tests (Required)**:
- File upload → Ingestion → Search
- Chat query → Response with sources
- User authentication
- Document deletion

**Edge Case Tests (All Required)**:
- Network timeout during upload
- Malicious file upload (path traversal)
- Embedding generation timeout
- Concurrent document deletion
- Empty search results
- Database connection exhaustion
- Duplicate file upload

### 5.5 Test Quality Standards

**DO**:
- Test behavior, not implementation
- Use descriptive test names
- Test one thing per test
- Use fixtures for test data
- Mock external dependencies (API, DB)

**DON'T**:
- Test private methods
- Write tests that always pass
- Use random test data
- Test third-party libraries
- Share state between tests

---

## 6. Documentation Standards

### 6.1 Code Comments

**WHEN TO COMMENT**:
- ✅ Non-obvious business logic (cite PRD section)
- ✅ Workarounds for external bugs
- ✅ Complex algorithm explanations
- ✅ Security-sensitive operations

**WHEN NOT TO COMMENT**:
- ❌ Obvious code (`x = x + 1  # increment x`)
- ❌ Function signatures (type hints suffice)
- ❌ Public API docs (use docstrings/JSDoc instead)

### 6.2 Docstrings (Python)

```python
def ingest_document(file_path: str, user_id: str) -> Result[Document, IngestionError]:
    """
    Parse, chunk, and embed a document for semantic search.

    PRD Reference: SS-001 File Ingestion Pipeline

    Args:
        file_path: Absolute path to uploaded file
        user_id: UUID of user uploading the document

    Returns:
        Ok(Document) if ingestion succeeds
        Err(IngestionError) if parsing or embedding fails

    Example:
        >>> result = ingest_document("/data/file.pdf", "user-123")
        >>> if isinstance(result, Ok):
        ...     print(f"Ingested {result.value.chunk_count} chunks")
    """
```

### 6.3 JSDoc (TypeScript)

```typescript
/**
 * Ingest a document for semantic search
 * @reference PRD SS-001 File Ingestion Pipeline
 * @param filePath - Absolute path to uploaded file
 * @param userId - UUID of user uploading
 * @returns Result with Document or IngestionError
 * @example
 * const result = await ingestDocument("/data/file.pdf", "user-123");
 * if (result.ok) {
 *   console.log(`Ingested ${result.value.chunkCount} chunks`);
 * }
 */
async function ingestDocument(
  filePath: string,
  userId: string
): Promise<Result<Document, IngestionError>>
```

### 6.4 README Standards

**Feature README (one paragraph max)**:
```markdown
# Document Upload

Allows users to drag-and-drop files for ingestion into the vector database.
Supports 15+ file formats including PDF, DOCX, and code files.

**Key Files**: `Dropzone.tsx`, `upload_api.ts`
**Related**: PRD US-001
```

---

## 7. Dependency Rules

### 7.1 Python Dependencies

**Requirements for adding dependency**:
1. Must justify in commit message
2. Prefer standard library
3. Minimum 1000 GitHub stars OR widely adopted
4. Pin exact versions (no ^)

**requirements.txt format**:
```txt
# Core
fastapi==0.104.1
sqlalchemy==2.0.23
pydantic==2.5.0

# External (justify each)
openai==1.6.1  # Embeddings and LLM
celery==5.3.4  # Background tasks
redis==5.0.1   # Task broker

# Dev
pytest==7.4.3
pytest-asyncio==0.21.1
```

### 7.2 TypeScript Dependencies

**Requirements for adding dependency**:
1. Must justify in commit message
2. Prefer React ecosystem standards
3. Minimum 1000 weekly npm downloads
4. Pin exact versions in package.json

**package.json format**:
```json
{
  "dependencies": {
    "react": "^18.2.0",
    "zustand": "^4.4.7",
    "axios": "^1.6.2"
  },
  "devDependencies": {
    "vitest": "^1.0.4",
    "typescript": "^5.3.3"
  }
}
```

### 7.3 Prohibited Dependencies

**AVOID** (use alternatives):
- ❌ heavy UI frameworks (use shadcn/ui components instead)
- ❌ complex state management (use Zustand, not Redux)
- ❌ authentication libraries (implement JWT directly)
- ❌ ORMs with magic (use SQLAlchemy, not Django ORM)

---

## 8. Security Standards

### 8.1 Secrets Management

**RULES**:
- NEVER commit secrets to git
- ALWAYS use environment variables
- Provide .env.example with all required variables
- Use git-secrets or trufflehog for scanning

**.env.example**:
```bash
# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/ragdb

# Auth
SECRET_KEY=your-secret-key-min-32-chars

# OpenAI
OPENAI_API_KEY=sk-...
```

### 8.2 Input Validation

**Python (Pydantic)**:
```python
class DocumentUpload(BaseModel):
    filename: str = Field(min_length=1, max_length=255)
    content: bytes = Field(max_length=50 * 1024 * 1024)  # 50MB

    @validator('filename')
    def no_path_traversal(cls, v):
        if '..' in v or v.startswith('/'):
            raise ValueError('Invalid filename')
        return v
```

**TypeScript (Zod)**:
```typescript
const documentUploadSchema = z.object({
  filename: z.string().min(1).max(255).refine(
    (s) => !s.includes('..') && !s.startsWith('/'),
    'Invalid filename'
  ),
  content: z.instanceof(File).refine(
    (f) => f.size <= 50 * 1024 * 1024,
    'File too large (max 50MB)'
  )
});
```

### 8.3 Authentication & Authorization

**Rules**:
- All API routes require auth except /health, /auth/login
- JWT stored in HTTP-only cookies
- Passwords hashed with bcrypt (cost factor 12)
- Rate limiting: 10 queries/minute per user

---

## 9. Performance Standards

### 9.1 Response Time Targets

| Endpoint | Target (p95) |
|----------|-------------|
| POST /api/files/upload | 30s (file dependent) |
| GET /api/documents | 100ms |
| POST /api/chat | 2s |
| DELETE /api/files/{id} | 200ms |

### 9.2 Database Query Rules

- ALWAYS use indexed columns in WHERE clauses
- LIMIT results to max 100 items
- Use connection pooling (max 20 connections)
- Query timeout: 5 seconds

### 9.3 Frontend Rendering Rules

- Use React.memo for expensive components
- Virtualize lists > 100 items
- Lazy load routes
- Optimize images (WebP, lazy loading)

---

## 10. Git Commit Standards

### 10.1 Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types**: feat, fix, refactor, test, docs, chore

**Examples**:
```
feat(upload): add drag-and-drop file upload

- Implements PRD US-001
- Supports 15 file formats
- Shows progress bar during upload

Refs: PRD.md#US-001
```

```
fix(chat): handle empty search results gracefully

- Returns 200 with empty sources array
- Displays helpful message to user
- Logs analytics event

Fixes: EC-005
```

---

## 11. Enforcement & Violations

### 11.1 Pre-Commit Hooks

**Required checks**:
- Format: Black (Python), Prettier (TS)
- Lint: Ruff (Python), ESLint (TS)
- Type check: mypy (Python), tsc (TS)
- Secrets scan: git-secrets
- Test coverage: pytest (min 80%)

### 11.2 Violation Handling

| Violation Severity | Action |
|-------------------|--------|
| **Critical** (secrets, test modification) | Reject PR, rebase required |
| **High** (type errors, failing tests) | Block commit, must fix |
| **Medium** (formatting, naming) | Auto-fix if possible |
| **Low** (documentation) | Warning, allow commit |

### 11.3 CI/CD Gates

**Required to pass**:
- All tests pass (unit, integration, e2e)
- Coverage meets minimum (80% backend, 75% frontend)
- No security vulnerabilities (npm audit, pip-audit)
- Type check passes (mypy, tsc)
- No secrets detected (trufflehog)

---

## APPENDIX: Quick Reference

### File Size Limits
- Python: 50 lines max
- TypeScript: 150 lines max
- Functions: 20 lines max

### Import Order
1. Standard library / Built-ins
2. Third-party
3. Local application

### Error Handling
- Business logic: Result types
- Boundaries only: try/catch

### Testing
- Unit: Isolated functions
- Integration: Service interactions
- E2E: Critical user paths

### Coverage
- Backend: ≥80%
- Frontend: ≥75%
- Edge cases: 100%

---

**THIS DOCUMENT IS IMMUTABLE**

Changes to this document require:
1. Discussion with all developers
2. Update to OBJECTIVE_KERNEL.yaml if mission changes
3. Unanimous approval
4. Version number increment

**Generated**: 2024-01-19
**Version**: 1.0
**Project**: RAG Knowledge Base
**Tech Stack**: Python + FastAPI + React
