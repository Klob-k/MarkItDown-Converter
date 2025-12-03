# MarkItDown Converter

PDF to Markdown converter using MarkItDown with intelligent Claude Vision fallback. Features automatic large PDF chunking, cost estimation, and multi-layer quality validation.

## Overview

A cost-efficient hybrid pipeline for converting PDFs to clean, well-structured Markdown. The converter uses a multi-stage approach to ensure high-quality output while minimizing API costs:

1. **PDF Validation** – Analyzes file size and page count, determines if chunking is needed
2. **MarkItDown** – Fast, free initial extraction using Microsoft's open-source tool
3. **Heuristic Quality Checks** – Instant validation of extraction quality (no API calls)
4. **Claude Quality Assessment** – AI-powered quality scoring when heuristics pass
5. **Claude Vision Fallback** – Direct PDF analysis for complex documents (with automatic chunking for large files)

### Claude API PDF Limits

| Limit | Value |
|-------|-------|
| Max file size | 32 MB per request |
| Max pages | 100 pages per request |

Large PDFs exceeding these limits are automatically chunked and processed in parts.

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                         Input PDF                               │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 0: PDF Validation                                         │
│  - Check file size (≤32MB for Claude Vision)                    │
│  - Get actual page count                                        │
│  - Determine if chunking needed (>100 pages or >32MB)           │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 1: MarkItDown Extraction (free, local)                    │
└───────────┬─────────────────────────────────────┬───────────────┘
            │ Success                             │ Fails
            ▼                                     │
┌───────────────────────────────┐                 │
│  Step 2: Heuristic Quality    │                 │
│  - Word density per page      │                 │
│  - Whitespace ratio           │                 │
│  - Header detection           │                 │
│  - Table integrity            │                 │
│  - Artifact detection         │                 │
└───────────┬───────────────────┘                 │
            │ Pass              │ Fail            │
            ▼                   │                 │
┌───────────────────────────────┐                 │
│  Step 3: Claude Quality       │                 │
│  Assessment (score 1-10)      │                 │
└───────────┬───────────────────┘                 │
            │                   │                 │
   Score ≥9 │    Score 7-8     │ Score <7        │
            │         │         │                 │
            ▼         ▼         └────────┬────────┘
┌───────────────┐ ┌───────────────┐      │
│ Skip Cleanup  │ │ Step 4: Claude│      │
│ (excellent)   │ │ Cleanup       │      │
└───────┬───────┘ └───────┬───────┘      │
        │                 │              │
        └────────┬────────┘              │
                 │                       ▼
                 │         ┌─────────────────────────────────────┐
                 │         │  Claude Vision Fallback             │
                 │         │  ┌─────────────────────────────┐    │
                 │         │  │ Large PDF? Split into chunks│    │
                 │         │  │ (≤95 pages, ≤30MB each)     │    │
                 │         │  └──────────────┬──────────────┘    │
                 │         │                 ▼                   │
                 │         │  ┌─────────────────────────────┐    │
                 │         │  │ Process each chunk          │    │
                 │         │  │ (with progress bar)         │    │
                 │         │  └──────────────┬──────────────┘    │
                 │         │                 ▼                   │
                 │         │  ┌─────────────────────────────┐    │
                 │         │  │ Combine chunk results       │    │
                 │         │  └──────────────┬──────────────┘    │
                 │         └─────────────────┼───────────────────┘
                 │                           │
                 └─────────────┬─────────────┘
                               ▼
                    ┌─────────────────────┐
                    │   Output Markdown   │
                    └─────────────────────┘
```

## Features

- **Large PDF Support** – Automatic chunking for PDFs exceeding API limits (>100 pages or >32MB)
- **Cost-Efficient** – Uses free MarkItDown extraction first, only calls Claude API when necessary
- **Cost Estimation** – Preview estimated costs before processing
- **Quality Validation** – Multi-layer quality checks ensure reliable output
- **Automatic Fallback** – Seamlessly switches to Claude Vision for complex documents
- **API Retry Logic** – Exponential backoff for rate limits and transient errors
- **Batch Processing** – Process entire directories with progress bars
- **Environment Flexible** – Works in Google Colab and local environments
- **Detailed Diagnostics** – Full visibility into processing decisions and metrics

## Installation

```bash
pip install 'markitdown[all]' anthropic pypdf tqdm
```

### Dependencies

| Package | Purpose |
|---------|---------|
| `markitdown[all]` | PDF text extraction (includes pdfminer.six) |
| `anthropic` | Claude API client |
| `pypdf` | PDF splitting for large files |
| `tqdm` | Progress bars (optional but recommended) |

## Configuration

The notebook auto-detects your environment and configures paths accordingly.

### API Key Setup

**Google Colab:**
```python
# Store in Colab Secrets as 'Claude_Colab'
from google.colab import userdata
ANTHROPIC_API_KEY = userdata.get('Claude_Colab')
```

**Local Environment:**
```bash
export ANTHROPIC_API_KEY="your-api-key"
```

### Full Configuration Options

```python
# Model Configuration
MODEL = "claude-haiku-4-5-20251001"
MAX_TOKENS = 16384  # Increased for longer page content

# API Retry Configuration
API_RETRY = {
    "max_retries": 3,
    "base_delay": 2,  # seconds
    "retryable_errors": ["rate", "overloaded", "timeout", "connection"],
}

# Claude API PDF Limits
PDF_LIMITS = {
    "max_file_size_mb": 32,         # Maximum file size for Claude Vision
    "max_pages_per_request": 100,   # Maximum pages per API request
    "chunk_size_pages": 95,         # Pages per chunk (with buffer)
    "chunk_size_mb": 30,            # Target chunk size in MB (with buffer)
}

# Quality Thresholds
QUALITY_THRESHOLDS = {
    "min_words_per_page": 50,       # Minimum words expected per page
    "max_whitespace_ratio": 0.40,   # Maximum whitespace allowed
    "min_quality_score": 7,         # Minimum Claude quality score (1-10)
    "skip_cleanup_score": 9,        # Skip cleanup if score >= this
    "min_headers_for_long_doc": 1,  # Minimum headers for docs > 500 words
    "quality_sample_chars": 8000,   # Characters to sample for quality check
}

# Cost Estimation (Claude Haiku pricing)
COST_ESTIMATES = {
    "input_cost_per_1k": 0.00025,   # $ per 1K input tokens
    "output_cost_per_1k": 0.00125,  # $ per 1K output tokens
    "avg_tokens_per_page": 1500,    # Approximate tokens per PDF page
}
```

## Usage

### Single File Processing

```python
result = process_pdf("/path/to/document.pdf", verbose=True)

if result["success"]:
    with open("output.md", "w") as f:
        f.write(result["markdown"])
    print(f"Path taken: {result['path_taken']}")
    print(f"Tokens used: {result['total_tokens']:,}")
```

### Batch Processing

```python
results = batch_process(
    input_dir="/path/to/pdfs",
    output_dir="/path/to/markdown_output",
    verbose=False,        # Detailed logs per file
    show_progress=True    # Show tqdm progress bar
)
```

**Batch output example:**
```
Found 10 PDF files to process

Estimating costs...
Estimated max cost: $0.1234 (if all use Claude Vision)

Processing PDFs: 100%|██████████| 10/10 [02:34<00:00, 15.4s/file]

============================================================
BATCH SUMMARY
============================================================
Processed: 10 files
Successful: 9
Failed: 1
Total tokens: 45,230
Estimated cost: $0.0892
Output directory: /path/to/markdown_output

Failed files:
  - corrupted_file.pdf
```

### Cost Estimation

```python
# Get PDF info and estimate cost before processing
pdf_info = get_pdf_info("/path/to/large_document.pdf")

print(f"Pages: {pdf_info['page_count']}")
print(f"Size: {pdf_info['file_size_mb']:.1f} MB")
print(f"Needs chunking: {pdf_info['needs_chunking']}")

if pdf_info['needs_chunking']:
    print(f"Estimated chunks: {pdf_info['estimated_chunks']}")

# Estimate cost for Claude Vision path
cost = estimate_cost(pdf_info, path="vision")
print(f"Estimated cost: ${cost['estimated_cost_usd']:.4f}")
```

## Quality Thresholds Explained

| Parameter | Default | Description |
|-----------|---------|-------------|
| `min_words_per_page` | 50 | Flags documents with suspiciously low text extraction |
| `max_whitespace_ratio` | 0.40 | Detects extraction issues causing excessive whitespace |
| `min_quality_score` | 7 | Claude's quality rating threshold (1-10 scale) |
| `skip_cleanup_score` | 9 | Skip cleanup step for excellent extractions |
| `min_headers_for_long_doc` | 1 | Ensures structure is preserved in longer documents |

## Heuristic Checks

The heuristic layer performs instant validation without API calls:

- **Word density** – Ensures adequate text was extracted per page
- **Whitespace ratio** – Flags documents with excessive whitespace (optimized calculation)
- **Header detection** – Verifies document structure is preserved
- **Table integrity** – Checks for consistent table formatting
- **Artifact detection** – Identifies OCR noise and extraction errors

## Processing Paths

The converter can take several paths depending on document quality:

| Path | When | Cost |
|------|------|------|
| `MarkItDown (no cleanup needed)` | Quality score ≥ 9 | ~500 tokens (quality check only) |
| `MarkItDown + Claude Cleanup` | Quality score 7-8 | ~1,000-5,000 tokens |
| `Claude Vision` | Heuristics or quality fail | ~1,500 tokens/page |
| `Claude Vision [chunked]` | Large PDF + quality fail | ~1,500 tokens/page × chunks |

## Output

The `process_pdf()` function returns a dictionary with:

```python
{
    "success": True,
    "markdown": "# Document Title\n\nContent...",
    "path_taken": "Claude Vision (Heuristic failed) [chunked]",
    "total_tokens": 45230,
    "diagnostics": {
        "pdf_info": {
            "page_count": 150,
            "file_size_mb": 25.4,
            "needs_chunking": True,
            "estimated_chunks": 2
        },
        "markitdown_extraction": {...},
        "heuristic_check": {...},
        "claude_quality_check": {...},
        "final_conversion": {
            "method": "claude_vision_fallback",
            "reason": "heuristic_check_failed",
            "tokens": 45230,
            "chunked": True,
            "chunks_processed": 2
        }
    }
}
```

## Error Handling

The converter includes robust error handling:

- **API Retry** – Automatic retry with exponential backoff (2s, 4s, 8s) for rate limits and transient errors
- **Chunk Failures** – If one chunk fails, processing continues with remaining chunks
- **Batch Resilience** – Individual file failures don't stop batch processing
- **Graceful Degradation** – Falls back to simpler methods if advanced features fail

## Requirements

- Python 3.8+
- `markitdown[all]` (includes pdfminer.six)
- `anthropic`
- `pypdf`
- `tqdm` (optional, for progress bars)

## Environment Support

| Environment | API Key Source | Default Paths |
|-------------|----------------|---------------|
| Google Colab | Colab Secrets (`Claude_Colab`) | `/content/` |
| Local | `ANTHROPIC_API_KEY` env var | Current directory |

## License

MIT

## Acknowledgments

- [MarkItDown](https://github.com/microsoft/markitdown) by Microsoft
- [Anthropic Claude API](https://www.anthropic.com/)
- [pypdf](https://github.com/py-pdf/pypdf) for PDF manipulation
