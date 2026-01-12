# BIR Document Scraper - Technical Write-Up

## Approach and Key Decisions

### Overview
I built a unified Python scraper that extracts legal documents from three Bureau of Internal Revenue (Philippines) sources and outputs structured data in XLSX format.

### Architecture
I chose a **single unified script** approach rather than separate scripts for each source because:
- Shared utilities (date parsing, document number normalization)
- Consistent output schema across all sources
- Single entry point for execution
- Easier maintenance and updates

### Technology Stack
- **Python 3** - Primary language
- **requests** - API calls and HTTP requests
- **BeautifulSoup4** - HTML parsing
- **openpyxl** - Excel file generation (prevents date auto-conversion issues)
- **dataclasses** - Clean data modeling

---

## How I Handled Inconsistencies Across Sources

| Source | Data Format | Challenge | Solution |
|--------|-------------|-----------|----------|
| **Source 1** (Legal Rulings) | JSON API with embedded HTML | Ruling links embedded in HTML strings; inconsistent title formats (`BIR-RULING-001-2025.pdf` vs `BIR Ruling No. 001-2025`) | Parse HTML with BeautifulSoup; normalize all titles to `BIR Ruling No. XXX-YYYY` format |
| **Source 2** (RDAO) | JSON API with HTML table | Full HTML table in JSON; dates in various formats (`December 19, 2024`, `November 7, 2024`) | Parse table rows; normalize dates to ISO format (`YYYY-MM-DD`) |
| **Source 3** (PDF) | Direct file URL | No API; scanned document (image-based, not searchable) | Extract metadata from URL only (`document_number`, `title`, `filename`) |

### Document Number Normalization
Different sources used different formats:
- `BIR-RULING-001-2025.pdf` → `001-2025`
- `BIR Ruling No. 18-2025` → `018-2025` (zero-padded)
- `RDAO No. 35-2024` → `35-2024`
- `RA-10963-RRD.pdf` → `10963`

### Excel Date Conversion Issue
Numbers like `01-2024` were being auto-converted to dates (`Jan-24`) in Excel. Solution: Output to XLSX format with explicit text formatting (`cell.number_format = '@'`).

---

## AI Tools Used

### Claude (Anthropic)
- **Code generation** - Initial scraper structure and boilerplate
- **Debugging** - Identifying API endpoints from DevTools screenshots
- **Problem-solving** - Excel date conversion workarounds
- **Refactoring** - Merging three separate scripts into unified solution

### How AI Helped
1. **Faster iteration** - Quickly tested different approaches (apostrophe prefix, formula format, XLSX output)
2. **API discovery** - Analyzed network requests to find `bir-cms-ws.bir.gov.ph` endpoints
3. **Edge case handling** - Identified inconsistent date formats and title patterns
4. **Code quality** - Suggested dataclasses, type hints, and clean architecture

---

## Assumptions Made

1. **Document numbers are unique** within each source/year
2. **PDF URLs are stable** and won't change
3. **API structure is stable** (template IDs 3322, 3708)
4. **Dates not available** for Source 1 (Legal Rulings) - only year is provided
5. **Source 3 PDF is scanned** - no OCR attempted; only URL-derived metadata captured
6. **Empty fields** left blank rather than using "N/A" for cleaner data processing

---

## What I'd Do Differently With More Time

### 1. Add Document Date Extraction (Source 1)
Legal Rulings don't have dates in the API. Would implement:
- Download each PDF
- OCR the scanned images (Tesseract/EasyOCR)
- Use LLM to extract date from OCR text

### 2. Add Summary Field
Use LLM (GPT-4o-mini or Gemini) to generate brief summaries:
- Extract text from PDFs via OCR
- Send to LLM with prompt: "Summarize this BIR ruling in 1-2 sentences"

### 3. Add Ruling Type Classification
For Source 2, extract ruling type prefixes (`OT`, `VAT`, `SH30`, `DT`, `PSH`) from original filenames to categorize rulings.

### 4. Incremental Updates
Add logic to:
- Track previously scraped documents
- Only fetch new/updated documents
- Append to existing output file

### 5. Error Recovery
- Save progress after each source
- Resume from last successful point on failure
- Log failed documents for manual review

---

## Output Schema

| Field | Description | Source 1 | Source 2 | Source 3 |
|-------|-------------|----------|----------|----------|
| `title` | Document title | ✅ | ✅ | ✅ |
| `document_number` | Unique identifier | ✅ | ✅ | ✅ |
| `document_date` | Issue date (ISO format) | ❌ | ✅ | ❌ |
| `year` | Year of document | ✅ | ✅ | ❌ |
| `subject_matter` | Description/summary | ❌ | ✅ | ❌ |
| `document_type` | Type classification | ✅ | ✅ | ❌ |
| `category` | Category grouping | ✅ | ✅ | ❌ |
| `pdf_url` | Direct link to PDF | ✅ | ✅ | ✅ |
| `source_url` | Source page URL | ✅ | ✅ | ❌ |
| `scraped_at` | Timestamp of extraction | ✅ | ✅ | ✅ |

---

## Files Delivered

```
├── bir_scraper.py          # Unified scraper script
├── requirements.txt        # Python dependencies
├── unified_data.xlsx       # Combined output (all sources)
├── legal_rulings.xlsx      # Source 1 output
├── rdao_orders.xlsx        # Source 2 output
├── pdf_documents.xlsx      # Source 3 output
└── README.md               # This write-up
```

---

## How to Run

```bash
# Install dependencies
pip install requests beautifulsoup4 openpyxl

# Run scraper
python bir_scraper.py
```

Output will be saved to `C:\BIR Extracts\` (configurable in script).
