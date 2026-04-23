---
name: pyodide-pdf
description: Use this skill whenever the user wants to read, analyze, extract from, split, merge, create, or lightly transform PDF files in the Pyodide code interpreter. Trigger for .pdf inputs, table extraction, text extraction, page splitting, merging, metadata inspection, or creating a new PDF from data. Use it when the task must run in-browser with pypdf, pdfplumber, reportlab, or PyMuPDF. Do not use this skill for OCR-heavy workflows or any task requiring poppler, qpdf, pdftk, or system binaries.
license: Proprietary. Local workspace adaptation for Open WebUI Pyodide.
compatibility: Pyodide browser runtime only; uses pypdf, pdfplumber, reportlab, PyMuPDF, pdfminer.six, pillow.
---

# PDF Processing in Pyodide

## Overview

This skill covers PDF workflows that are feasible inside the Open WebUI Pyodide runtime.

Use these libraries:
- `pypdf` for merge, split, rotate, metadata, and page-level operations
- `pdfplumber` for text and table extraction
- `PyMuPDF` (`import fitz`) for page rendering, structured extraction, and page images
- `reportlab` for generating new PDFs from scratch

Hard limits:
- No OCR stack: do not assume `pytesseract`, `pdf2image`, or poppler exist
- No native CLI tools: do not assume `qpdf`, `pdftk`, `pdftotext`, or `soffice`
- No shell commands or package installation

## Library Selection Rules

Use the installed PDF libraries before declaring a limitation.

- Use `pdfplumber` first for text extraction with layout awareness and for tables
- Use `pypdf` for structural operations: merge, split, rotate, metadata, watermark-like page composition
- Use `PyMuPDF` (`fitz`) when page geometry, block extraction, page rendering, or image export is needed
- Use `reportlab` only when generating a brand-new PDF from scratch
- Use `pypdf` plus `reportlab` together for simple overlays or generated companion pages

## Preferred Workflow

When a PDF is involved, follow this order:

1. Decide whether the task is extraction, transformation, or generation
2. Check whether the PDF likely contains text or is image-only
3. Use the highest-level installed library that directly fits the task
4. Save outputs to `/mnt/uploads/`
5. Verify by reopening the PDF, checking page count, or sampling extracted text

Default choices:
- Text extraction: `pdfplumber`
- Table extraction: `pdfplumber`
- Merge/split/rotate: `pypdf`
- Render pages to images: `PyMuPDF`
- Create new PDFs: `reportlab`
- OCR or scanned-PDF transcription: clearly state it is not available in Pyodide

## Quick Reference

| Task | Best tool |
|------|-----------|
| Merge or split PDFs | `pypdf` |
| Extract plain text | `pdfplumber` or `PyMuPDF` |
| Extract tables | `pdfplumber` |
| Inspect metadata | `pypdf` |
| Render page to image | `PyMuPDF` |
| Create new PDF | `reportlab` |
| Add simple watermark or overlay | `pypdf` |

## File Handling

Read uploaded files from `/mnt/uploads/`.

Write outputs back to `/mnt/uploads/`.

If the user asks for analysis only, print summaries first and save derived files only if helpful.

## Common Workflows

### Extract text

```python
import pdfplumber

with pdfplumber.open('/mnt/uploads/input.pdf') as pdf:
    text = []
    for page in pdf.pages:
        text.append(page.extract_text() or '')

full_text = '\n\n'.join(text)
print(full_text[:4000])
```

### Extract tables

```python
import pdfplumber

with pdfplumber.open('/mnt/uploads/input.pdf') as pdf:
    for page_number, page in enumerate(pdf.pages, start=1):
        tables = page.extract_tables()
        print(f'Page {page_number}: {len(tables)} tables')
        for table in tables:
            print(table)
```

### Merge PDFs

```python
from pypdf import PdfReader, PdfWriter

writer = PdfWriter()
for path in ['/mnt/uploads/a.pdf', '/mnt/uploads/b.pdf']:
    reader = PdfReader(path)
    for page in reader.pages:
        writer.add_page(page)

output_path = '/mnt/uploads/merged.pdf'
with open(output_path, 'wb') as handle:
    writer.write(handle)

print(output_path)
```

### Split pages

```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader('/mnt/uploads/input.pdf')

for index, page in enumerate(reader.pages, start=1):
    writer = PdfWriter()
    writer.add_page(page)
    output_path = f'/mnt/uploads/page-{index:02d}.pdf'
    with open(output_path, 'wb') as handle:
        writer.write(handle)
    print(output_path)
```

### Render a page to an image

```python
import fitz

pdf = fitz.open('/mnt/uploads/input.pdf')
page = pdf[0]
pix = page.get_pixmap(matrix=fitz.Matrix(2, 2))
output_path = '/mnt/uploads/page-01.png'
pix.save(output_path)
print(output_path)
```

### Create a new PDF

```python
from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer
from reportlab.lib.styles import getSampleStyleSheet

output_path = '/mnt/uploads/report.pdf'
doc = SimpleDocTemplate(output_path, pagesize=letter)
styles = getSampleStyleSheet()
story = [
    Paragraph('Report Title', styles['Title']),
    Spacer(1, 12),
    Paragraph('This PDF was generated inside Pyodide.', styles['BodyText']),
]
doc.build(story)
print(output_path)
```

## Best Practices

- Use `pdfplumber` first for tables
- Use `PyMuPDF` when page geometry or rendering matters
- Use `pypdf` for structural edits like merge/split/rotate
- Use `reportlab` only for brand-new PDFs, not for editing a complex existing layout
- For scanned PDFs, state clearly that OCR is unavailable in this runtime
- Do not claim `qpdf`, `pdftk`, or `pdftotext` support here; use installed Python libraries instead
- Prefer extraction plus regeneration over promising high-fidelity in-place editing of a complex PDF

## Limits and Fallbacks

- Scanned-image PDFs without text layers cannot be reliably converted to text here
- Form-filling is possible only for simple PDFs and should be treated case by case
- High-fidelity PDF editing is limited; prefer extracting content and producing a new PDF when necessary

## Output Checklist

Before finishing:
1. Confirm output files exist in `/mnt/uploads/`
2. If extracting text or tables, inspect a sample of the result
3. If merging or splitting, verify page counts
4. Call out OCR or native-tool limitations when they materially affect the result
