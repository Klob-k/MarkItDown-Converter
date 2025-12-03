# MarkItDown Converter

PDF to Markdown converter using MarkItDown with intelligent Claude Vision fallback. Features heuristic quality checks and automatic fallback for complex documents.

## Overview

A cost-efficient hybrid pipeline for converting PDFs to clean, well-structured Markdown. The converter uses a multi-stage approach to ensure high-quality output while minimizing API costs:

1. **MarkItDown** – Fast, free initial extraction using Microsoft's open-source tool
2. **Heuristic Quality Checks** – Instant validation of extraction quality (no API calls)
3. **Claude Quality Assessment** – AI-powered quality scoring when heuristics pass
4. **Claude Vision Fallback** – Direct PDF analysis for complex or poorly-extracted documents

## How It Works

```
┌─────────────────┐
│   Input PDF     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   MarkItDown    │──── Fails ────┐
│   Extraction    │               │
└────────┬────────┘               │
         │ Success                │
         ▼                        │
┌─────────────────┐               │
│   Heuristic     │──── Fails ────┤
│   Quality Check │               │
└────────┬────────┘               │
         │ Pass                   │
         ▼                        │
┌─────────────────┐               │
│  Claude Quality │──── Low ──────┤
│   Assessment    │   Score       │
└────────┬────────┘               │
         │ High Score             │
         ▼                        ▼
┌─────────────────┐     ┌─────────────────┐
│  Claude Cleanup │     │  Claude Vision  │
│   (Optional)    │     │    Fallback     │
└────────┬────────┘     └────────┬────────┘
         │                       │
         └───────────┬───────────┘
                     ▼
              ┌─────────────┐
              │ Output .md  │
              └─────────────┘
```

## Features

- **Cost-Efficient** – Uses free MarkItDown extraction first, only calls Claude API when necessary
- **Quality Validation** – Multi-layer quality checks ensure reliable output
- **Automatic Fallback** – Seamlessly switches to Claude Vision for complex documents
- **Batch Processing** – Process entire directories of PDFs
- **Detailed Diagnostics** – Full visibility into processing decisions and metrics
- **Configurable Thresholds** – Adjust quality parameters to your needs

## Installation

```bash
pip install 'markitdown[all]' anthropic pdfminer.six
```

## Configuration

Set your Anthropic API key and adjust parameters as needed:

```python
# API Setup
ANTHROPIC_API_KEY = "your-api-key"

# Model Configuration
MODEL = "claude-haiku-4-5-20251001"
MAX_TOKENS = 16384

# Quality Thresholds
QUALITY_THRESHOLDS = {
    "min_words_per_page": 50,      # Minimum words expected per page
    "max_whitespace_ratio": 0.40,   # Maximum whitespace allowed
    "min_quality_score": 7,         # Minimum Claude quality score (1-10)
    "min_headers_for_long_doc": 1,  # Minimum headers for docs > 500 words
}

# File Paths
INPUT_PDF = "/content/input.pdf"
OUTPUT_MD = "/content/output.md"
```

## Usage

### Single File Processing

```python
result = process_pdf("/path/to/document.pdf", verbose=True)

if result["success"]:
    with open("output.md", "w") as f:
        f.write(result["markdown"])
    print(f"Tokens used: {result['total_tokens']}")
```

### Batch Processing

```python
results = batch_process(
    input_dir="/path/to/pdfs",
    output_dir="/path/to/markdown_output",
    verbose=False
)
```

## Quality Thresholds Explained

| Parameter | Default | Description |
|-----------|---------|-------------|
| `min_words_per_page` | 50 | Flags documents with suspiciously low text extraction |
| `max_whitespace_ratio` | 0.40 | Detects extraction issues causing excessive whitespace |
| `min_quality_score` | 7 | Claude's quality rating threshold (1-10 scale) |
| `min_headers_for_long_doc` | 1 | Ensures structure is preserved in longer documents |

## Heuristic Checks

The heuristic layer performs instant validation without API calls:

- **Word density** – Ensures adequate text was extracted per page
- **Whitespace ratio** – Flags documents with excessive whitespace
- **Header detection** – Verifies document structure is preserved
- **Table integrity** – Checks for consistent table formatting
- **Artifact detection** – Identifies OCR noise and extraction errors

## Output

The `process_pdf()` function returns a dictionary with:

```python
{
    "success": True,
    "markdown": "# Document Title\n\nContent...",
    "path_taken": "MarkItDown + Claude Cleanup",
    "total_tokens": 1250,
    "diagnostics": {
        "markitdown_extraction": {...},
        "heuristic_check": {...},
        "claude_quality_check": {...},
        "final_conversion": {...}
    }
}
```

## Requirements

- Python 3.8+
- `markitdown[all]`
- `anthropic`
- `pdfminer.six`

## License

MIT

## Acknowledgments

- [MarkItDown](https://github.com/microsoft/markitdown) by Microsoft
- [Anthropic Claude API](https://www.anthropic.com/)
