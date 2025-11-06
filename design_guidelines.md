# Design Guidelines: Document Search System with OCR

## Design Approach
**Selected System:** Material Design with Carbon Design influences
**Justification:** This utility-focused, data-intensive application requires robust component patterns for complex workflows (file uploads, job queues, search results). Material Design provides excellent data visualization patterns while Carbon's enterprise focus guides our job management and monitoring interfaces.

**Key Principles:**
- Clarity over decoration - users need to monitor OCR jobs and search efficiently
- Progressive disclosure - complex features revealed contextually
- Status visibility - always show system state (processing, queued, errors)

## Typography System
- **Primary Font:** Inter (Google Fonts) for UI and data
- **Monospace Font:** JetBrains Mono for file names, technical details, confidence scores

**Hierarchy:**
- Page Headers: 2xl/3xl, semibold (Dashboard, Search Results, Job Queue)
- Section Headers: xl, semibold (Active Jobs, Recent Uploads)
- Card Titles: lg, medium (Document names, job titles)
- Body Text: base, regular (Search snippets, metadata)
- Caption/Meta: sm, regular (File sizes, timestamps, page counts)
- Technical Data: sm, mono (OCR confidence %, processing times)

## Layout System
**Spacing Primitives:** Tailwind units of 2, 4, 6, and 8 (p-2, m-4, gap-6, py-8)
**Container Strategy:**
- Dashboard: max-w-7xl with 3-column grid (sidebar + main + details panel)
- Search Results: max-w-6xl, single column with side filters
- Document Viewer: Full-width with fixed sidebar for page navigation

## Core Components

### Navigation & Structure
**Top Navigation Bar:**
- Logo + app name (left)
- Global search input (center, always visible)
- Upload button (prominent, primary color, right side)
- Job queue indicator with badge count (right side)
- User profile menu (right)

**Sidebar (Persistent, 240px):**
- Dashboard
- My Documents
- Search
- Job Queue
- Settings
Clear active state with background fill and left border accent

### Upload Interface
**Drag-and-Drop Zone:**
- Large dashed border area (min-h-64)
- Upload icon (cloud with up arrow, 48px)
- Primary text: "Drop PDFs or images here"
- Secondary text: "or click to browse" with file type/size limits
- Active drag state: solid border, background tint

**Upload Progress Cards:**
- File name with icon (PDF/image indicator)
- Progress bar with percentage
- Status text ("Validating...", "Extracting page 3/12", "Indexing...")
- Cancel button (icon only, ghost style)
- Stack vertically in upload modal/panel

### Job Queue Dashboard
**Job Cards (Horizontal layout):**
- Left: Document thumbnail/icon (80px square)
- Center: Title, file size, page count, language selection
- Status badge: color-coded (blue=processing, green=complete, red=failed, gray=queued)
- Progress bar for active jobs with page-by-page counter
- Right: Action menu (retry, cancel, delete)

**Queue Filters:** Tabs for All/Processing/Completed/Failed with counts

### Search Interface
**Search Bar (Prominent):**
- Large input with search icon
- Advanced filters toggle button (right side)
- Voice input option (icon)

**Filter Panel (Collapsible sidebar, 280px):**
- Document Type (checkboxes: PDF, JPG, PNG)
- Date Range (calendar picker)
- Tags (multi-select chips)
- Language (dropdown)
- Sort options (relevance, date, title)
Apply button at bottom

**Search Results:**
- Result card per document (horizontal layout)
- Left: thumbnail preview (120px)
- Title with match highlighting (yellow background)
- Snippet with highlighted keywords (2-3 lines max)
- Metadata row: file type, pages, upload date, confidence score
- Action buttons: View, Download (ghost style)
- Pagination at bottom (1 2 3 ... 10 Next)

### Document Viewer
**Two-Panel Layout:**
- Left sidebar (280px): Page thumbnails with extracted text preview, scroll to select
- Main area: Full page view (PDF render or image) with text overlay option toggle
- Bottom toolbar: Page navigation, zoom controls, download, share

**Text Extraction View:**
- Per-page text blocks
- Confidence indicators (progress bar or color-coded background per block)
- Edit icon for manual corrections
- Re-OCR button for low-confidence pages

### Status & Notifications
**Toast Notifications (top-right):**
- Success: "Document processed successfully" with view link
- Error: "OCR failed on page 5" with retry action
- Warning: "Low confidence detected" with review prompt
Auto-dismiss in 5s, closable

**Job Status Indicator (Nav bar):**
- Pulsing dot for active jobs
- Number badge for queue length
- Dropdown showing 3 most recent jobs with quick actions

## Interaction Patterns
- Hover states: subtle background change (5% opacity shift)
- Active jobs: gentle pulse animation on progress bars
- Page loading: skeleton screens for search results and document list
- Drag-and-drop: smooth transition on drag enter/leave
- Search: debounced input (300ms), loading spinner in search bar

## Responsive Strategy
**Desktop (≥1024px):** Full 3-column layouts, sidebar always visible
**Tablet (768-1023px):** Collapsible sidebar, 2-column search results
**Mobile (<768px):** Stack to single column, bottom navigation, hamburger menu

## Images
**No hero image required** - this is a utility dashboard, not a marketing page.
**Thumbnails:** Generate from uploaded PDFs/images for document cards and page navigation (lazy-loaded).