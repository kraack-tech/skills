---
name: pyodide-docx
description: Use this skill whenever the user wants to create, read, analyze, extract from, or lightly transform Word documents in the Pyodide code interpreter. Trigger for .docx files, Word templates, letters, reports, memos, contracts, form-like documents, or when the user wants DOCX output or DOCX content extraction. Use it especially when the work must happen in-browser with python-docx or mammoth. Do not use this skill for PDF-only tasks, spreadsheet-first tasks, or for DOCX to PDF conversion that requires native binaries.
license: Proprietary. Local workspace adaptation for Open WebUI Pyodide.
compatibility: Pyodide browser runtime only; uses python-docx, mammoth, lxml, pillow, pyyaml.
---

# DOCX Processing in Pyodide

## Overview

This skill covers practical DOCX workflows that work inside the Open WebUI Pyodide runtime.

Use these libraries:
- `python-docx` for creating and editing `.docx`
- `mammoth` for extracting HTML or raw text from `.docx`
- `lxml` for XML parsing when a document embeds XML-like payloads or needs structured inspection
- `pillow` for working with extracted or generated images before inserting them

Hard limits:
- Do not use `pip install`, `micropip.install()`, `subprocess`, or shell commands
- Do not assume `pandoc`, `LibreOffice`, `soffice`, or OCR tools exist
- Do not promise faithful DOCX -> PDF conversion in Pyodide

## Library Selection Rules

Use these rules aggressively. Do not default to saying a task is unsupported if one of the installed libraries can do it.

- Use `python-docx` for creation, editing, placeholder replacement, table insertion, headings, lists, and image insertion
- Use `mammoth` for reading, extraction, HTML conversion, and quick semantic inspection of an existing `.docx`
- Use `mammoth.extract_raw_text` when the user wants content analysis, summarization, indexing, or search prep rather than a styled output
- Use `lxml` only for structured XML inspection or targeted low-level fixes; do not start with XML editing unless higher-level APIs fail
- Use `pillow` only to prepare or resize images before placing them into documents

## Preferred Workflow

When a `.docx` file is involved, follow this order:

1. Identify whether the task is extraction, editing, or generation
2. Inspect the existing file first if one was provided
3. Choose the highest-level installed library that solves the task
4. Write outputs to `/mnt/uploads/`
5. Re-open or re-read key parts of the output to confirm the result is usable

Default choices:
- Extraction or analysis: `mammoth`
- Create new DOCX: `python-docx`
- Edit existing DOCX without redesign: `python-docx`
- Complex layout-preserving conversion to PDF: explicitly state this is outside Pyodide

## Quick Reference

| Task | Best tool |
|------|-----------|
| Create a new `.docx` | `python-docx` |
| Edit simple content in existing `.docx` | `python-docx` |
| Extract clean HTML from `.docx` | `mammoth` |
| Extract rough text from `.docx` | `mammoth.extract_raw_text` |
| Insert tables/images/headings | `python-docx` |
| Fill simple template placeholders | `python-docx` |

## File Handling

Always read inputs from `/mnt/uploads/` when the user refers to uploaded files.

Always write final outputs back to `/mnt/uploads/`.

If the user gives a template document:
1. Inspect structure first
2. Preserve the existing formatting unless the user asked for redesign
3. Make the smallest possible content changes
4. Save as a new file unless the user explicitly wants overwrite behavior

## Common Workflows

### Read or extract content

Use `mammoth` when the user wants readable content or HTML.

```python
import mammoth

with open('/mnt/uploads/input.docx', 'rb') as docx_file:
    result = mammoth.convert_to_html(docx_file)
    html = result.value
    messages = result.messages

print(html[:2000])
print(messages)
```

For raw text:

```python
import mammoth

with open('/mnt/uploads/input.docx', 'rb') as docx_file:
    result = mammoth.extract_raw_text(docx_file)
    text = result.value

print(text)
```

### Create a new document

Use `python-docx` for generated reports, letters, summaries, and forms.

```python
from docx import Document

output_path = '/mnt/uploads/report.docx'

document = Document()
document.add_heading('Report Title', level=1)
document.add_paragraph('Executive summary goes here.')

table = document.add_table(rows=1, cols=2)
header = table.rows[0].cells
header[0].text = 'Metric'
header[1].text = 'Value'

for metric, value in [('Revenue', '$120,000'), ('Growth', '12%')]:
    row = table.add_row().cells
    row[0].text = metric
    row[1].text = value

document.save(output_path)
print(output_path)
```

### Fill placeholder-style templates

Use plain string replacement only when placeholders are simple and isolated. Prefer replacing paragraph text, table cells, or runs carefully rather than rebuilding the whole file.

```python
from docx import Document

replacements = {
    '{{CLIENT_NAME}}': 'Acme Corp',
    '{{DATE}}': '2026-04-23',
}

doc = Document('/mnt/uploads/template.docx')

for paragraph in doc.paragraphs:
    for old, new in replacements.items():
        if old in paragraph.text:
            for run in paragraph.runs:
                run.text = run.text.replace(old, new)

for table in doc.tables:
    for row in table.rows:
        for cell in row.cells:
            for old, new in replacements.items():
                if old in cell.text:
                    for paragraph in cell.paragraphs:
                        for run in paragraph.runs:
                            run.text = run.text.replace(old, new)

output_path = '/mnt/uploads/filled-template.docx'
doc.save(output_path)
print(output_path)
```

## Best Practices

- Preserve template layout when editing an existing document
- Prefer headings, tables, and paragraph styles over manual spacing hacks
- Keep edits localized when modifying an existing file
- If extraction quality matters more than exact styling, prefer `mammoth`
- If final deliverable must remain editable in Word, prefer `.docx` output over HTML or PDF
- Do not apologize for missing `pandoc` or `soffice`; choose `mammoth` or `python-docx` when they cover the task
- Prefer producing a correct `.docx` over attempting unsupported conversion steps

## Limits and Fallbacks

- DOCX -> PDF is not a Pyodide capability
- Complex tracked-changes workflows are not reliable in this environment
- Legacy `.doc` is not supported directly; ask for `.docx` if possible
- For exact visual reproduction, keep the output as `.docx`

## Output Checklist

Before finishing:
1. Confirm the output file exists in `/mnt/uploads/`
2. If editing a template, verify placeholders were replaced
3. If extracting content, inspect the first chunk of output for obvious corruption
4. State clearly if any limitation prevented exact formatting parity
