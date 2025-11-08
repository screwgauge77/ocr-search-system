# OCR Search System

A full-stack document ingestion, OCR, and search platform. Users upload PDFs or images; the backend extracts pages, performs OCR per page, stores structured text with metadata, and exposes keyword search plus real-time job progress updates via WebSockets.

## Key Features
- Upload PDFs and common image formats (PNG, JPG, WEBP) up to 100MB.
- Automatic PDF page rasterization to images (`pdfjs-dist` + `canvas`).
- Per-page OCR using `tesseract.js` with optional preprocessing (deskew, binarize, denoise).
- Real-time job progress over WebSocket (`/ws`).
- Duplicate detection via SHA-256 file hash.
- Thumbnail generation for PDFs and images.
- Basic keyword search with contextual snippets & highlights.
- Low-confidence page flagging (threshold 60%).
- Reprocess a single page with different OCR options.
- Pluggable storage layer (Postgres via Drizzle ORM; MongoDB driver available).
- Modular pipeline: upload → page extraction → OCR queue → searchable text.

## Tech Stack
- Backend: Node.js 20, Express, WebSocket (`ws`), Drizzle ORM (Postgres) / MongoDB driver.
- OCR & PDF/Image: `tesseract.js`, `pdfjs-dist`, `pdf-parse`, `sharp`, `canvas`.
- Frontend: React 18, Vite, Tailwind CSS, Radix UI components.
- Validation: Zod schemas (`shared/schema.ts`).
- Build: Vite (client) + esbuild (server bundling) + TypeScript.

## Repository Structure
```
server/          # Backend source (routes, processors, storage adapters)
client/          # Frontend SPA (React + Vite)
shared/          # Shared types and Zod/Drizzle schemas
uploads/         # Stored originals, page images, thumbnails
start-dev.bat    # Quick dev start (Mongo default)
start-server.bat # Alternative server start script
README.md        # Project documentation
```

## Architecture Overview
1. Upload endpoint (`POST /api/upload`) stores the original file, computes hash, creates a document row, extracts PDF pages or registers single image, queues OCR job.
2. `JobQueue` (`server/queue.ts`) polls storage for queued jobs and hands them to `OCRProcessor`.
3. `OCRProcessor` (`server/ocr.ts`) iterates pages, optionally preprocesses images, runs Tesseract, saves text + confidence.
4. WebSocket channel (`/ws`) streams progress updates (status, page counts, percentages).
5. Search endpoint (`GET /api/search`) composes document + page results with highlighted snippets.
6. Reprocess endpoint allows targeted improvement for problematic pages.

## ER Diagram (Mermaid)
```mermaid
erDiagram
  USER ||--o{ DOCUMENT : uploads
  USER ||--o{ JOB : submits
  DOCUMENT ||--|{ PAGE : contains
  DOCUMENT ||--o{ JOB : processedBy
  JOB {
    string id PK
    string documentId FK
    string userId FK OPTIONAL
    string language
    string status
    int progressPercent
    int processedPages
    int totalPages
    string errorMessage OPTIONAL
  }
  PAGE {
    string id PK
    string documentId FK
    int pageNumber UNIQUE (within document)
    string imagePath
    text text
    int confidence
    bool lowConfidenceFlag OPTIONAL
  }
  DOCUMENT {
    string id PK
    string title
    string filename
    string fileHash UNIQUE
    int pageCount
    string mimeType
    string language
    string status
  }
```

## Installation (Windows, PowerShell)
Prerequisites:
- Node.js 20+ (https://nodejs.org)
- Git
- MongoDB 6+ or Postgres 14+ (choose one)
- (If `canvas` fails to install) Visual Studio Build Tools with C++ workload + Python 3.10.

Clone and install:
```powershell
git clone <repo-url> DocumentSearchEngine
cd DocumentSearchEngine
npm install
```

### Environment Variables
Create a `.env` (or set in shell):
```
DB_DRIVER=mongo              # mongo | postgres
MONGO_URL=mongodb://127.0.0.1:27017
MONGO_DB=document_search
POSTGRES_URL=postgres://user:pass@host:5432/dbname   # if using Postgres
PORT=5000
SESSION_SECRET=change_me_long_random
OCR_LANG_PATH=./eng.traineddata         # optional local traineddata path
OCR_CACHE_PATH=./.ocr-cache             # optional cache dir
```

### Quick Start (Mongo)
```powershell
./start-dev.bat
# or
npm run dev:mongo
```
Visit: http://localhost:5000

### Using Postgres
1. Set `DB_DRIVER=postgres` and `POSTGRES_URL`.
2. Run migrations:
```powershell
npm run db:push
npm run dev
```

## Core NPM Scripts
- `npm run dev` – Start server (default Postgres; overridden by env).
- `npm run dev:mongo` – Start with Mongo driver env variables.
- `npm run build` – Bundle server + build client.
- `npm run start` – Run built server from `dist/`.
- `npm run check` – TypeScript type checking.
- `npm run db:push` – Apply Drizzle migrations.

## API Endpoints (Summary)
| Method | Path | Description |
|--------|------|-------------|
| POST | /api/upload | Upload PDF/image, create document & OCR job |
| GET | /api/documents | List documents (pagination) |
| GET | /api/documents/:id | Document details + pages |
| GET | /api/documents/:id/download | Download original file |
| DELETE | /api/documents/:id | Delete document (cascade pages/jobs) |
| GET | /api/search | Keyword search with filters & sort |
| GET | /api/jobs | List jobs (optionally by status) |
| PATCH | /api/jobs/:id/cancel | Cancel an OCR job |
| POST | /api/jobs/:id/retry | Retry a job |
| DELETE | /api/jobs/:id | Delete job |
| POST | /api/pages/:id/reprocess | Re-OCR a page with options |
| GET | /api/stats | Basic system stats |

WebSocket: `ws://localhost:5000/ws` – send `{ "type": "subscribe", "jobId": "..." }` to receive progress events.

## Search Behavior
- Simple substring match; highlights built by scanning extracted page text.
- Basic relevance (currently score field from storage; may be primitive).
- Future improvement: full-text indexing (Postgres FTS or Mongo Atlas Search), fuzzy match, semantic ranking.

## Limitations
- OCR accuracy declines with low-quality scans, handwriting, complex layouts.
- CPU-bound OCR (WASM); no parallel job scheduler or priority handling yet.
- Limited language handling (manual selection, no auto-detect).
- Basic keyword search only; no fuzzy or semantic capabilities.
- Storage cleanup & lifecycle not automated; `uploads/` may grow indefinitely.
- Minimal metrics/observability.
- No integrated virus scanning or PII redaction.

## Future Scope
- Redis/BullMQ queue with concurrency, retries, prioritization.
- Advanced preprocessing (deskew, contrast curve, adaptive thresholds) exposed via queryable settings.
- Multi-language auto-detection & custom dictionaries.
- Vector/semantic search (embeddings + pgvector / external vector DB).
- Full-text indexing & relevance scoring.
- Page region segmentation (tables, columns) & searchable reconstructed PDF/A output.
- Lifecycle management: archival, dedup pruning, tiered storage (S3/MinIO).
- Observability: Prometheus metrics, structured logs, tracing.
- Security: antivirus scan, MIME validation enhancements, configurable file size caps per role.

## Performance Tips
- Use local traineddata (`OCR_LANG_PATH`) to avoid repeated network fetches.
- Adjust page render scale (currently 1.5 in `pdfProcessor.ts`) for speed vs accuracy trade-off.
- Batch large uploads and throttle concurrent OCR jobs.
- Prefer Postgres for advanced indexing/analytics; ensure proper indexes on status/createdAt fields.

## Development Notes (Windows)
If `canvas` install errors appear:
1. Install Visual Studio Build Tools (C++ workload).
2. Ensure `npm config set msvs_version <year>` if needed.
3. Re-run `npm install`.

## Testing Ideas (Not yet implemented)
- Upload & OCR: small PDF (2 pages). Assert page count & OCR text presence.
- Duplicate detection: same file twice returns 409.
- Reprocess endpoint improves confidence for synthetic noisy page.
- Search returns snippet with highlight.

## Contributing
1. Fork & branch (`feat/xyz`).
2. Write/update tests (to be added under `server/__tests__`).
3. Ensure `npm run check` passes.
4. Open PR with description & screenshots for UI changes.

## Security
- Validate MIME & size at upload.
- Consider adding rate limiting & session hardening for production.
- Keep dependencies updated; run `npm audit` regularly.

## License
MIT License — see `package.json`.

## ERD Prompt (For AI/Diagram Tools)
"Generate an ERD for OCR Search System with entities: User, Document, Page, Job. Show cardinalities: User 1:M Document, User 1:M Job, Document 1:M Page, Document 1:M Job. Include attributes: Document(id,title,filename,fileHash,pageCount,mimeType,language,status), Page(id,documentId,pageNumber,imagePath,text,confidence,status), Job(id,documentId,userId,status,progress,currentPage,totalPages,language,errorMessage), User(id,email,passwordHash,role,createdAt). Enforce unique (documentId,pageNumber)."

---
Need help implementing queues, semantic search, or tests? Open an issue or start a discussion.
