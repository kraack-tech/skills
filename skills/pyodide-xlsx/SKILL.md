---
name: pyodide-xlsx
description: Use this skill whenever a spreadsheet or tabular file is the primary input or output in the Pyodide code interpreter. Trigger for .xlsx, .xls, .xlsb, .ods, .csv, or .tsv files; spreadsheet analysis; workbook creation; cleaning tabular files; exporting data to Excel-like formats; or updating spreadsheet templates in-browser with pandas, openpyxl, python-calamine, pyxlsb, xlrd, odfpy, or xlsxwriter. Do not use this skill for Word-first, PDF-first, or native Excel automation tasks.
license: Proprietary. Local workspace adaptation for Open WebUI Pyodide.
compatibility: Pyodide browser runtime only; uses pandas, openpyxl, xlsxwriter, python-calamine, pyxlsb, xlrd, odfpy, tablib, pyarrow.
---

# Spreadsheet Processing in Pyodide

## Overview

This skill covers practical spreadsheet workflows that work inside the Open WebUI Pyodide runtime.

Use these libraries:
- `pandas` for analysis, cleaning, reshaping, CSV/Excel import-export
- `openpyxl` for `.xlsx` formatting, formulas, and workbook editing
- `xlsxwriter` for writing polished new `.xlsx` files
- `python-calamine` for fast reading of `.xlsx`, `.xls`, `.xlsb`, `.ods`
- `pyxlsb` for direct `.xlsb` reading
- `xlrd` for legacy `.xls` reading
- `odfpy` for OpenDocument formats
- `tablib` for easy tabular interchange
- `pyarrow` for columnar data and Parquet-like workflows

Hard limits:
- No native Excel, LibreOffice recalculation, or macros
- No shell commands or package installation
- Formula behavior should be validated logically, not with native office recalc

## Library Selection Rules

Use these installed spreadsheet libraries deliberately. Do not default to one tool for every file type.

- Use `pandas` for analysis, reshaping, cleaning, joins, summaries, and straightforward export
- Use `openpyxl` for editing existing `.xlsx` files when formulas, formatting, sheet structure, cell styles, or workbook details matter
- Use `xlsxwriter` for creating new polished `.xlsx` outputs from structured data
- Use `python-calamine` first for broad read-only spreadsheet compatibility across `.xlsx`, `.xls`, `.xlsb`, and `.ods`
- Use `pyxlsb` when `.xlsb` parsing needs a dedicated fallback
- Use `xlrd` for legacy `.xls`
- Use `odfpy` for OpenDocument-specific cases
- Use `tablib` for lightweight interchange tasks when full `pandas` is unnecessary
- Use `pyarrow` for columnar pipelines, parquet-style workflows, or large typed tables

## Preferred Workflow

When a spreadsheet or tabular file is involved, follow this order:

1. Identify the real file type and whether the task is analysis, editing, or export
2. Load the file with the most appropriate installed reader
3. Use high-level table operations first
4. Switch to workbook-oriented tools only when formatting or formulas matter
5. Write outputs to `/mnt/uploads/`
6. Re-open key sheets and inspect important values or formulas

Default choices:
- Analyze workbook data: `pandas`
- Read mixed spreadsheet types quickly: `python-calamine`
- Edit existing `.xlsx` template: `openpyxl`
- Create a new professional workbook: `xlsxwriter` or `openpyxl`
- Large tabular export or interchange: `pandas` or `pyarrow`
- Native recalculation or macros: clearly state this is outside Pyodide

## Requirements for Outputs

### All spreadsheet outputs
- Preserve existing template formatting when editing a template file
- Keep headers clear and stable
- Avoid silently changing data types that matter, such as IDs with leading zeros
- Prefer formulas over hardcoded derived values when the workbook is meant to stay editable

### Code style
- Write concise Python
- Avoid unnecessary prints
- Keep transformations explicit and inspectable

## Quick Reference

| Task | Best tool |
|------|-----------|
| Analyze tabular data | `pandas` |
| Read `.xlsx` | `pandas`, `openpyxl`, `python-calamine` |
| Read `.xlsb` | `python-calamine` or `pyxlsb` |
| Read `.xls` | `xlrd` or `python-calamine` |
| Edit existing `.xlsx` with formatting/formulas | `openpyxl` |
| Create polished new `.xlsx` | `xlsxwriter` or `openpyxl` |
| Read `.ods` | `python-calamine` or `odfpy` |
| Export between tabular formats | `pandas` or `tablib` |

## File Handling

Read uploads from `/mnt/uploads/`.

Write outputs back to `/mnt/uploads/`.

If the user gives a workbook template:
1. Inspect sheets first
2. Preserve structure and formatting
3. Change only requested regions when possible

## Common Workflows

### Analyze an Excel file

```python
import pandas as pd

path = '/mnt/uploads/input.xlsx'
all_sheets = pd.read_excel(path, sheet_name=None)

for name, df in all_sheets.items():
    print(f'--- {name} ---')
    print(df.head())
    print(df.dtypes)
```

### Read mixed spreadsheet formats quickly

```python
from python_calamine import CalamineWorkbook

wb = CalamineWorkbook.from_path('/mnt/uploads/input.xlsb')
print(wb.sheet_names)
for sheet_name in wb.sheet_names:
    sheet = wb.get_sheet_by_name(sheet_name)
    print(sheet.to_python()[:5])
```

### Create a new workbook with formulas

```python
from openpyxl import Workbook
from openpyxl.styles import Font

wb = Workbook()
ws = wb.active
ws.title = 'Summary'

ws['A1'] = 'Month'
ws['B1'] = 'Revenue'
ws['C1'] = 'Tax'
ws['D1'] = 'Net'

for cell in ws[1]:
    cell.font = Font(bold=True)

rows = [
    ('Jan', 12000, 1800),
    ('Feb', 15000, 2200),
    ('Mar', 17000, 2500),
]

for index, (month, revenue, tax) in enumerate(rows, start=2):
    ws[f'A{index}'] = month
    ws[f'B{index}'] = revenue
    ws[f'C{index}'] = tax
    ws[f'D{index}'] = f'=B{index}-C{index}'

output_path = '/mnt/uploads/financial-summary.xlsx'
wb.save(output_path)
print(output_path)
```

### Edit an existing workbook while preserving formulas/styles

```python
from openpyxl import load_workbook

wb = load_workbook('/mnt/uploads/template.xlsx')
ws = wb['Sheet1']
ws['B2'] = 42
ws['B3'] = 'Updated value'

output_path = '/mnt/uploads/template-updated.xlsx'
wb.save(output_path)
print(output_path)
```

### Export cleaned data

```python
import pandas as pd

csv_path = '/mnt/uploads/raw.csv'
df = pd.read_csv(csv_path)

df.columns = [column.strip().lower().replace(' ', '_') for column in df.columns]
df = df.dropna(how='all')

output_path = '/mnt/uploads/cleaned.xlsx'
df.to_excel(output_path, index=False)
print(output_path)
```

## Best Practices

- Use `pandas` for analysis and reshaping
- Use `openpyxl` when workbook structure, formatting, or formulas matter
- Use `python-calamine` first for broad spreadsheet format coverage
- Preserve string-like IDs with explicit dtype handling
- Validate formulas logically by inspecting references and expected values
- Do not tell the user spreadsheets are unsupported when the task fits `pandas`, `openpyxl`, `python-calamine`, or `xlsxwriter`
- Prefer the simplest installed tool that preserves correctness for the requested output

## Formula Guidance

Prefer formulas when the user expects the workbook to stay editable.

Check these before finishing:
- Referenced cells exist
- Sheet names are correct
- Divisions handle zero safely when needed
- IDs and dates are not accidentally converted to the wrong type

## Limits and Fallbacks

- Native formula recalculation is not available
- Macros and VBA are out of scope
- Very advanced Excel features may not round-trip perfectly
- If fidelity matters more than analysis, keep edits minimal and preserve the workbook shape

## Output Checklist

Before finishing:
1. Confirm output files exist in `/mnt/uploads/`
2. Re-open workbook if possible and inspect key cells/sheets
3. If formulas were added, print a few formula strings and expected results
4. Call out any recalculation or macro limitation that affects confidence
