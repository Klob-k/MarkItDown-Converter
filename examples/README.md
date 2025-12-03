# Example PDFs

This folder contains example PDF files for testing and demonstrating the MarkItDown Converter.

## Test Files

| File | Size | Difficulty | Description |
|------|------|------------|-------------|
| `white-house-supply-chain-review-2024.pdf` | 6.4 MB | **Hard** | White House Quadrennial Supply Chain Review (2021-2024). Dense government report with complex tables, multi-column layouts, policy matrices, and detailed charts. |
| `dat-freight-focus-2025.pdf` | 2.4 MB | **Medium** | DAT Freight Focus 2025. Industry report with rate charts, trend graphs, and structured sections. |
| `freight-broker-training-guide.pdf` | 3.4 MB | **Easy** | JW Surety Bonds Freight Broker Training Guide. Text-heavy guide with simple formatting, minimal tables/charts. |
| `how-to-start-freight-brokerage.pdf` | 1.3 MB | **Easy** | DAT "How to Start a Freight Brokerage" whitepaper. Clean formatting, straightforward content. |

## Expected Processing Paths

Based on document complexity, here's what to expect:

| File | Expected Path | Reason |
|------|---------------|--------|
| Easy files | `MarkItDown + Claude Cleanup` or `MarkItDown (no cleanup)` | Simple formatting extracts well |
| Medium files | `MarkItDown + Claude Cleanup` or `Claude Vision` | Charts/graphs may trigger fallback |
| Hard files | `Claude Vision` | Complex layouts need visual processing |

## Running the Demo

### Single File (Hard Example)

```python
# In the notebook, update INPUT_PDF:
INPUT_PDF = "/content/examples/pdfs/white-house-supply-chain-review-2024.pdf"

# Then run Cell 6 to process
result = process_pdf(INPUT_PDF, verbose=True)
```

### Batch Processing (All Examples)

```python
# Process all example PDFs
results = batch_process(
    input_dir="/content/examples/pdfs",
    output_dir="/content/examples/output",
    verbose=False,
    show_progress=True
)
```

### Expected Output

```
Found 4 PDF files to process

Estimating costs...
Estimated max cost: $0.XXXX (if all use Claude Vision)

Processing PDFs: 100%|██████████| 4/4 [01:23<00:00, 20.8s/file]

============================================================
BATCH SUMMARY
============================================================
Processed: 4 files
Successful: 4
Failed: 0
Total tokens: XX,XXX
Estimated cost: $0.XXXX
Output directory: /content/examples/output
```

## Cost Estimates

Approximate costs using Claude Haiku:

| File | Est. Tokens | Est. Cost |
|------|-------------|-----------|
| Easy files | 500-2,000 | $0.001-0.005 |
| Medium files | 2,000-10,000 | $0.005-0.02 |
| Hard files | 10,000-50,000 | $0.02-0.10 |

*Actual costs depend on processing path taken and document page count.*

## Sources

- **White House Supply Chain Review**: [bidenwhitehouse.archives.gov](https://bidenwhitehouse.archives.gov/wp-content/uploads/2024/12/20212024-Quadrennial-Supply-Chain-Review.pdf)
- **DAT Freight Focus 2025**: [dat.com](https://www.dat.com/wp-content/uploads/2024/12/DAT-Freight-Focus-2025.pdf)
- **Freight Broker Training Guide**: [jwsuretybonds.com](https://www.jwsuretybonds.com/license-bonds/freight-broker-bonds/freight-broker-ebook.pdf)
- **How to Start a Freight Brokerage**: [dat.com](https://www.dat.com/resources/wp-content/uploads/resource-assets/whitepaper/how-to-start-a-freight-brokerage.pdf)
