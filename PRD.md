# Product Requirements Document (PRD)
## RAG Knowledge Base with Chat Interface

**Project**: RAG (Retrieval-Augmented Generation) Database
**Tech Stack**: Python + FastAPI + React
**Document Version**: 1.0
**Status**: DRAFT

---

## 1. Core Logic & Objective

### Problem Statement
Users possess large volumes of unstructured data across various file formats (PDF, DOCX, TXT, MD, CSV, JSON, images, code files) and cannot efficiently query, search, or extract insights from this collective knowledge. Existing solutions either: (1) handle limited file types, (2) lack intelligent semantic search, or (3) require complex setup without an intuitive interface.

### Success State
A deployed web application where users can:
1. Drag-and-drop files of any type into a web interface
2. Watch real-time progress as files are ingested, chunked, and embedded
3. Ask natural language questions via a chat interface
4. Receive accurate, source-attributed answers grounded in their uploaded documents
5. Reference the specific document chunks used to generate each answer

### Measurable Acceptance Criteria
- **Ingestion**: 100MB file uploads complete within 30 seconds on standard broadband
- **Search**: Semantic queries return relevant results within 2 seconds
- **Accuracy**: Chat responses include source citations with 100% of answers
- **Uptime**: API maintains 99.5% uptime during active usage
- **File Support**: System accepts 15+ file formats without errors
- **Concurrent Users**: System supports 10 simultaneous users without degradation

---

## 2. Story Trifecta

### 2.1 User Stories (The Experience)

#### US-001: File Upload
**As a** Knowledge Worker
**I want** to upload multiple documents at once through a drag-and-drop interface
**So that** I can quickly build my knowledge base without tedious individual uploads

| State | Description |
|-------|-------------|
| **Loading** | Progress bar shows upload percentage (0-100%) with file names listed below. Spinner animation active. |
| **Success** | Green checkmark appears. Success message: "3 files uploaded successfully." Files appear in "My Documents" list with timestamp and file size. |
| **Error** | Red banner: "Upload failed: file_01.pdf exceeds 50MB limit." Retry button shown. Failed files remain in queue for retry. |

#### US-002: Ask Question via Chat
**As a** Knowledge Worker
**I want** to ask questions in natural language through a chat interface
**So that** I can get answers without reading through entire documents

| State | Description |
|-------|-------------|
| **Loading** | "Searching your documents..." message with animated dots. User can see their message in chat (right-aligned, blue). |
| **Success** | Answer appears (left-aligned, gray) with response text + "Sources" section showing 2-5 document excerpts with clickable links to original files. |
| **Error** | "No relevant information found in your documents. Try rephrasing or upload more files." Suggestion to upload more documents shown. |

#### US-003: View Document History
**As a** Knowledge Worker
**I want** to see all uploaded documents with metadata
**So that** I can manage my knowledge base and remove outdated files

| State | Description |
|-------|-------------|
| **Loading** | Skeleton loader shows 5 placeholder rows with shimmer effect. |
| **Success** | Table shows: filename, upload date, file size, document type (badge), and "Delete" button. Pagination controls at bottom (10 per page). |
| **Error** | "Failed to load documents. Refresh to retry." Refresh button shown. |

### 2.2 System Stories (The Backend)

#### SS-001: File Ingestion Pipeline
**When** a file is uploaded via POST /api/files/upload
**The system SHALL**:
1. Validate file size ≤ 50MB and type is supported
2. Store original file in /data/documents/{uuid}.{ext}
3. Extract text content using format-specific parser (PDF→PyPDF2, DOCX→python-docx, etc.)
4. Split text into chunks of 1000 tokens with 200-token overlap
5. Generate embeddings for each chunk using OpenAI text-embedding-3-small
6. Store chunks in vector database (pgvector/Qdrant) with metadata: source_file_id, chunk_index, created_at
7. Update documents table: status = "ingested", chunk_count = N
8. Emit event: file_ingested {file_id, chunk_count, timestamp}

**Database mutations**:
```sql
INSERT INTO documents (id, filename, file_size, file_type, status, uploaded_at)
VALUES (:uuid, :filename, :size, :type, 'processing', NOW());

INSERT INTO document_chunks (id, document_id, chunk_index, content, embedding, created_at)
VALUES (:chunk_uuid, :doc_uuid, :index, :text, :vector, NOW());

UPDATE documents SET status = 'ingested', chunk_count = :count WHERE id = :doc_uuid;
```

**State transitions**: `uploaded` → `processing` → `ingested` OR `failed`

**Side effects**:
- Background job: re-index related documents (semantic similarity)
- Cache invalidation: clear /documents/list cache

**Idempotency**: Re-uploading same file (SHA256 checksum) replaces existing; old chunks marked deleted, new chunks created.

#### SS-002: Semantic Search Query
**When** a query is submitted via POST /api/chat
**The system SHALL**:
1. Generate embedding for user query using same model
2. Execute vector similarity search over document_chunks table with threshold = 0.7 similarity
3. Retrieve top 5 most relevant chunks
4. Format chunks as context for LLM prompt
5. Stream response from GPT-4o with source citations
6. Log query and response for analytics

**Database mutations**:
```sql
INSERT INTO chat_history (id, session_id, user_query, bot_response, sources_used, created_at)
VALUES (:uuid, :session_id, :query, :response, :source_ids, NOW());
```

**State transitions**: None (read operation with audit log write)

**Side effects**:
- Analytics: track query latency, relevance feedback
- Rate limiting: enforce 10 queries/minute per user

**Idempotency**: Same query within 60 seconds returns cached response (if parameters identical).

#### SS-003: Document Deletion
**When** DELETE /api/files/{file_id} is called
**The system SHALL**:
1. Verify file belongs to authenticated user
2. Mark document_chunks records as deleted (soft delete)
3. Mark document record as deleted (soft delete)
4. Schedule file cleanup job (T+24 hours)
5. Emit event: document_deleted {file_id, timestamp}

**Database mutations**:
```sql
UPDATE document_chunks SET deleted_at = NOW() WHERE document_id = :file_id;
UPDATE documents SET deleted_at = NOW() WHERE id = :file_id;
```

**State transitions**: `active` → `deleted`

**Side effects**:
- Background job: delete physical file from disk after 24h
- Cache invalidation: clear search results containing deleted chunks

**Idempotency**: Calling delete on already-deleted file returns 404 (idempotent).

### 2.3 Developer Stories (The Tooling)

#### DS-001: Logging and Observability
**As a developer**, I need structured logging with correlation IDs
**So that** I can trace requests across services and debug production issues

**Requirements**:
- All logs include: timestamp, level, correlation_id, service, event_type
- Events to log at INFO: file_uploaded, file_ingested, query_executed, document_deleted
- Events to log at ERROR: ingestion_failed, query_timeout, authentication_failed
- Logs output to stdout (JSON format) for container log aggregation
- Integration with Sentry for error tracking

**Example log**:
```json
{
  "timestamp": "2024-01-19T10:30:45Z",
  "level": "INFO",
  "correlation_id": "req_abc123",
  "service": "ingestion-service",
  "event_type": "file_ingested",
  "file_id": "uuid-123",
  "chunk_count": 15,
  "duration_ms": 2340
}
```

#### DS-002: Local Development Setup
**As a developer**, I need a complete local development environment
**So that** I can test features end-to-end without cloud dependencies

**Requirements**:
- Docker Compose setup with: PostgreSQL (with pgvector), Redis, backend API, frontend
- Seed data script: loads 3 sample documents with varied content
- Health check endpoint: GET /health returns {status: "ok", services: {db: "ok", redis: "ok"}}
- Hot reload for both backend (uvicorn --reload) and frontend (Vite dev server)
- Local .env file template with required variables

**Seed data**:
- 1 PDF with technical documentation
- 1 DOCX with meeting notes
- 1 TXT with project requirements

#### DS-003: Testing Infrastructure
**As a developer**, I need comprehensive test coverage
**So that** I can refactor with confidence and catch regressions early

**Requirements**:
- Unit tests: pytest for backend, Jest for frontend
- Integration tests: test database operations with testcontainers
- E2E tests: Playwright for critical user flows (upload, chat, delete)
- Coverage threshold: minimum 80% for backend, 75% for frontend
- Test data fixtures: factory_boy for generating test documents

---

## 3. Data & State Requirements

### 3.1 Database Schema

#### Table: documents
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique document identifier |
| user_id | UUID | FOREIGN KEY → users.id, NOT NULL | Owner of the document |
| filename | VARCHAR(255) | NOT NULL | Original filename |
| file_size | BIGINT | NOT NULL, ≥ 0 | Size in bytes |
| file_type | VARCHAR(50) | NOT NULL | MIME type or extension |
| file_hash | VARCHAR(64) | UNIQUE, SHA256 | For duplicate detection |
| storage_path | VARCHAR(500) | NOT NULL | Path to stored file |
| status | ENUM | NOT NULL, DEFAULT 'processing' | uploaded, processing, ingested, failed |
| chunk_count | INTEGER | ≥ 0 | Number of chunks created |
| uploaded_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | Upload timestamp |
| ingested_at | TIMESTAMPTZ | NULLABLE | When ingestion completed |
| deleted_at | TIMESTAMPTZ | NULLABLE | Soft delete timestamp |

**Data lifecycle**:
1. **Created**: INSERT on file upload
2. **Updated**: status changes during ingestion, chunk_count set after chunking
3. **Archived**: deleted_at set (soft delete), physical file deleted after 24h
4. **Deleted**: Hard delete after 30 days (cleanup job)

#### Table: document_chunks
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique chunk identifier |
| document_id | UUID | FOREIGN KEY → documents.id, NOT NULL | Source document |
| chunk_index | INTEGER | NOT NULL, ≥ 0 | Position in document |
| content | TEXT | NOT NULL | Chunk text content |
| embedding | VECTOR(1536) | NOT NULL | OpenAI embedding |
| token_count | INTEGER | NOT NULL, ≥ 0 | Approximate token count |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | Creation timestamp |
| deleted_at | TIMESTAMPTZ | NULLABLE | Soft delete timestamp |

**Relationships**:
- documents → document_chunks: 1:N, CASCADE on hard delete
- Indexes: (document_id, chunk_index), embedding vector HNSW index

#### Table: chat_history
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique message identifier |
| session_id | UUID | NOT NULL | Chat session identifier |
| user_id | UUID | FOREIGN KEY → users.id, NOT NULL | User who sent message |
| user_query | TEXT | NOT NULL | User's question |
| bot_response | TEXT | NOT NULL | AI's answer |
| sources_used | JSONB | NULLABLE | Array of chunk_ids referenced |
| model_version | VARCHAR(50) | NOT NULL | LLM version used |
| latency_ms | INTEGER | ≥ 0 | Response time |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | Timestamp |

#### Table: users
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | User identifier |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Email address |
| password_hash | VARCHAR(255) | NOT NULL | Bcrypt hash |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | Account created |
| last_login | TIMESTAMPTZ | NULLABLE | Last login time |

### 3.2 Validation Rules

#### File Upload
- **Size**: 1 byte ≤ file_size ≤ 50MB (configurable)
- **Type**: Must be in allowed_types list: pdf, docx, txt, md, csv, json, html, xml, py, js, ts, jpg, png (ocr for images)
- **Filename**: 1-255 characters, no path traversal (..)

#### Chat Query
- **Length**: 10 ≤ query_length ≤ 2000 characters
- **Rate limit**: 10 queries per minute per user
- **Content**: No empty or whitespace-only queries

#### Authentication
- **Password**: 8-100 characters, must contain uppercase, lowercase, number
- **Email**: Valid email format (RFC 5322)

---

## 4. Pessimistic Edge Cases

### EC-001: Network Failure During File Upload
**Scenario**: User's connection drops while uploading a 40MB file at 60% progress.

**Trigger**: Client loses internet connectivity or server becomes unreachable during multipart form data transfer.

**REQUIRED Response**:
1. Client detects connection loss (fetch timeout or offline event)
2. Display error: "Upload interrupted. Connection lost."
3. Store partial file in temporary storage with expiration (T+1 hour)
4. Provide "Resume upload" button if server supports chunked uploads
5. If resume not available, require full re-upload
6. Log event: upload_interrupted {file_id, bytes_received, expected_bytes}

**Test Assertion**:
```python
def test_upload_interruption_resume():
    # Upload 40MB file, kill connection at 60%
    # Verify partial file stored with expiration
    # Verify resume option available
    # Verify complete upload after resume
    assert document.status == 'ingested'
    assert document.chunk_count > 0
```

### EC-002: Malicious File Upload (Path Traversal)
**Scenario**: Attacker uploads file with filename "../../etc/passwd" attempting to escape storage directory.

**Trigger**: User uploads file with path traversal sequences in filename or attempts directory manipulation via multipart form.

**REQUIRED Response**:
1. Sanitize filename: remove all path separators, use only basename
2. Reject files with sequences: "../", "./", "\", absolute paths
3. Return 400 Bad Request: "Invalid filename. Please rename your file."
4. Log security event: potential_path_traversal {ip, raw_filename, sanitized}
5. Rate limit IP after 3 rejected uploads

**Test Assertion**:
```python
def test_path_traversal_rejected():
    malicious_files = ["../../etc/passwd", "..\\..\\system32", "/etc/shadow"]
    for filename in malicious_files:
        response = client.upload(file_with_name(filename))
        assert response.status_code == 400
        assert "invalid filename" in response.json()["error"].lower()
    # Verify no files written outside allowed directory
    assert all(Path(storage_dir).rglob("*")) in allowed_subdirs
```

### EC-003: Embedding Generation Timeout
**Scenario**: OpenAI API becomes slow or unresponsive while generating embeddings for a large document (100+ chunks).

**Trigger**: External API latency exceeds 30 seconds or connection times out during embedding batch request.

**REQUIRED Response**:
1. Mark document status as "failed" after 3 retry attempts
2. Store error message: "Embedding generation failed. Please try again."
3. Preserve uploaded file (do not delete)
4. Allow user to retry ingestion via "Retry" button in UI
5. Implement exponential backoff: retry at 1s, 2s, 4s intervals
6. Alert admins if failure rate > 10% (monitoring alert)

**Test Assertion**:
```python
def test_embedding_timeout_recovery():
    with mock_openai_timeout():
        response = client.upload(large_pdf)
    assert response["status"] == "failed"
    assert response["retry_available"] == True

    # Verify retry succeeds
    with mock_openai_success():
        retry_response = client.retry_ingestion(doc_id)
    assert retry_response["status"] == "ingested"
```

### EC-004: Concurrent Document Deletion
**Scenario**: User A deletes a document while User B's chat session is actively using it as a source for answering a question.

**Trigger**: DELETE /api/files/{id} executes simultaneously with POST /api/chat referencing the same document_id.

**REQUIRED Response**:
1. Chat query proceeds with document still available (read wins race)
2. Document marked deleted but chunks remain queryable until existing queries complete
3. Subsequent queries exclude deleted document from results
4. Chat UI shows "Some sources may no longer be available" warning if deleted chunk was cited
5. Background cleanup removes chunks after T+5 minutes (grace period for active sessions)

**Test Assertion**:
```python
async def test_concurrent_delete_query():
    async with TaskGroup() as tg:
        tg.create_task(delete_document(doc_id))  # User A
        tg.create_task(chat_query("summarize doc"))  # User B

    # Both operations complete without error
    # Chat response includes document (read won race)
    # Subsequent queries exclude document
    assert doc_in_chat_response == True
    assert doc_in_next_query == False
```

### EC-005: Vector Database Query Returns No Results
**Scenario**: User asks a question about a topic present in uploaded documents, but semantic search returns zero relevant chunks (similarity < 0.7).

**Trigger**: Query embedding has low cosine similarity to all stored chunks due to: vocabulary mismatch, embedding model limitations, or overly strict threshold.

**REQUIRED Response**:
1. Return 200 OK with empty sources array
2. Chat response: "I couldn't find relevant information in your documents. Try rephrasing your question or upload more files."
3. Display suggestion cards: "Try asking about..." with 3 example queries based on available document titles
4. Log analytics event: low_similarity_query {query, max_similarity, threshold}
5. Monitor threshold; if >50% queries return empty, suggest lowering threshold to 0.65

**Test Assertion**:
```python
def test_empty_search_results():
    with mock_vector_search_return_empty():
        response = client.chat("what is the meaning of life?")
    assert response.status_code == 200
    assert "couldn't find" in response["answer"].lower()
    assert response["sources"] == []
    assert len(response["suggestions"]) == 3
```

### EC-006: Database Connection Exhaustion
**Scenario**: Sudden traffic spike causes database connection pool to exhaust all available connections.

**Trigger**: 50+ concurrent requests while pool max_connections = 20, and connections not released promptly due to slow queries.

**REQUIRED Response**:
1. New requests receive 503 Service Unavailable
2. Response includes Retry-After header: 60 seconds
3. Client displays: "Service temporarily unavailable. Please wait a moment."
4. Uncommitted transactions rollback automatically
5. Health check endpoint remains accessible (separate connection pool)
6. Alert triggered: connection_pool_exhausted {active, waiting, max}

**Test Assertion**:
```python
def test_connection_pool_exhaustion():
    # Use pool of 2 connections for testing
    with DatabaseConfig(max_connections=2):
        # Start 5 concurrent slow queries
        tasks = [slow_query() for _ in range(5)]
        results = await asyncio.gather(*tasks, return_exceptions=True)

    # 2 succeed, 3 get 503
    success_count = sum(1 for r in results if r.status == 200)
    error_count = sum(1 for r in results if r.status == 503)
    assert success_count == 2
    assert error_count == 3
```

### EC-007: Duplicate File Upload (Same Content)
**Scenario**: User uploads the exact same file twice (same SHA256 hash) at different times.

**Trigger**: File with existing file_hash already present in documents table for same user_id.

**REQUIRED Response**:
1. Detect duplicate via file_hash comparison
2. Return 409 Conflict: "This file was already uploaded on {date}."
3. Provide option: "View existing document" link to previous upload
4. Do NOT create duplicate document record
5. Do NOT re-process embeddings (save costs)
6. Log analytics: duplicate_upload_attempt {file_id, original_upload_date}

**Test Assertion**:
```python
def test_duplicate_file_prevented():
    # Upload file first time
    response1 = client.upload(test_pdf)
    assert response1.status_code == 201

    # Upload same file again
    response2 = client.upload(test_pdf)
    assert response2.status_code == 409
    assert "already uploaded" in response2.json()["detail"].lower()
    assert response2.json()["existing_document_id"] == response1.json()["id"]
```

---

## 5. Explicit Constraints (What System MUST NOT Do)

### Security Constraints
1. **MUST NOT** store plain-text passwords. All passwords hashed with bcrypt (cost factor 12).
2. **MUST NOT** expose database credentials in error messages. Sanitize all errors before client response.
3. **MUST NOT** allow file uploads without authentication. All /api/files/* endpoints require valid JWT.
4. **MUST NOT** execute arbitrary code from uploaded files. Code files (.py, .js) treated as text only.
5. **MUST NOT** log user queries or document content in production logs. Log only metadata (query length, document ID).

### Privacy Constraints
1. **MUST NOT** share documents between users. Data isolation enforced at query level (WHERE user_id = current_user).
2. **MUST NOT** include PII in analytics. Hash emails before sending to analytics services.
3. **MUST NOT** retain deleted documents beyond 30 days. Hard delete enforced by cleanup job.

### Performance Constraints
1. **MUST NOT** block UI thread during file upload > 100MB. Use chunked upload with progress callbacks.
2. **MUST NOT** perform embedding generation synchronously. Use background job queue (Celery/RQ).
3. **MUST NOT** load entire document into memory for files > 10MB. Stream content in chunks.
4. **MUST NOT** return more than 10 chunks per query unless explicitly requested (pagination).

### Data Constraints
1. **MUST NOT** delete documents without soft-delete first. Always set deleted_at before physical removal.
2. **MUST NOT** allow orphaned chunks. All chunks must have valid document_id (FK constraint).
3. **MUST NOT** modify user-uploaded content. Store original file exactly as uploaded.

---

## 6. Test Requirements Summary

### User Story Tests
- US-001 File Upload: 4 tests (success, loading state, file too large, invalid type)
- US-002 Chat Query: 5 tests (success with sources, no results, loading state, rate limit, long query)
- US-003 View Documents: 4 tests (success list, empty state, pagination, delete from list)

**User Story Tests: 13**

### System Story Tests
- SS-001 Ingestion Pipeline: 6 tests (full pipeline, pdf parsing, docx parsing, embedding generation, database writes, failed ingestion)
- SS-002 Semantic Search: 5 tests (vector search, context formatting, streaming response, logging, caching)
- SS-003 Document Deletion: 4 tests (soft delete, cascade to chunks, file cleanup, idempotency)

**System Story Tests: 15**

### Edge Case Tests
- EC-001 Upload Interruption: 3 tests (timeout, resume, partial cleanup)
- EC-002 Path Traversal: 3 tests (path rejection, filename sanitization, security log)
- EC-003 Embedding Timeout: 3 tests (timeout handling, retry logic, failed state)
- EC-004 Concurrent Operations: 4 tests (delete during query, delete during upload, race condition, grace period)
- EC-005 Empty Search Results: 3 tests (no results, suggestions, analytics logging)
- EC-006 Connection Pool: 3 tests (pool exhaustion, retry-after, health check isolation)
- EC-007 Duplicate Upload: 3 tests (exact duplicate detection, conflict response, existing doc link)

**Edge Case Tests: 22**

### Constraint Violation Tests
- Security: 5 tests (password hashing, auth required, code file safety, error sanitization, query logging)
- Privacy: 3 tests (user isolation, PII hashing, deletion retention)
- Performance: 4 tests (chunked upload, background job, streaming, large files)
- Data: 3 tests (soft delete first, orphan prevention, content immutability)

**Constraint Tests: 15**

---

### Total Minimum Tests: 65

**Test Categories**:
- Unit Tests: ~35 (individual functions, database operations, parsing logic)
- Integration Tests: ~20 (API endpoints, database interactions, background jobs)
- E2E Tests: ~10 (complete user flows with Playwright)

**Coverage Requirements**:
- Backend Code Coverage: ≥ 80%
- Frontend Code Coverage: ≥ 75%
- Critical Path Coverage: 100% (upload, ingest, chat, delete flows must have E2E tests)

---

## Appendix: File Types Supported

| Category | Extensions | Parser |
|----------|------------|--------|
| Documents | pdf, docx, doc, odt | PyPDF2, python-docx |
| Text | txt, md, rst, log | Built-in |
| Data | csv, json, xml, yaml | Pandas, built-in |
| Code | py, js, ts, java, cpp, go, rs | Syntax-aware chunker |
| Web | html, html | BeautifulSoup |
| Images | jpg, png, gif (OCR) | pytesseract |

**Total Supported Formats: 15+**

---

**END OF PRD**

This document is TEST-READY. Every requirement can be verified through automated testing. Ambiguity is the enemy of testability; this PRD minimizes ambiguity through specific states, measurable criteria, and explicit REQUIRED responses.
