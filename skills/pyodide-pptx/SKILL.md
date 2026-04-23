---
name: pyodide-pptx
description: Use this skill whenever the user wants to create, read, analyze, or update PowerPoint files in the Pyodide code interpreter. Trigger for .pptx files, slide decks, presentations, speaker materials, template-based updates, slide text extraction, or new deck generation when the work must happen in-browser with python-pptx and pillow. Do not use this skill for native PowerPoint rendering, LibreOffice conversion, or pixel-perfect slide-to-image workflows that require system binaries.
license: Proprietary. Local workspace adaptation for Open WebUI Pyodide.
compatibility: Pyodide browser runtime only; uses python-pptx, pillow, lxml.
---

# PPTX Processing in Pyodide

## Overview

This skill covers practical PowerPoint workflows that work inside the Open WebUI Pyodide runtime.

Use these libraries:
- `python-pptx` for creating and editing `.pptx`
- `pillow` for preparing images, thumbnails, and simple composites before insertion
- `lxml` for low-level XML inspection only when necessary

Hard limits:
- No `pptxgenjs`, LibreOffice, `soffice`, or poppler in this runtime
- No reliable slide rendering to images from PowerPoint itself
- No shell commands or package installation

## Library Selection Rules

Use the installed presentation libraries before claiming the task cannot be done.

- Use `python-pptx` for creating, editing, updating text, adding images, building tables, and assembling standard decks
- Use `pillow` to preprocess images, make grids, resize assets, or create simple visual composites before insertion
- Use `lxml` only for inspection or last-resort low-level checks; do not begin with XML editing for normal deck tasks

## Preferred Workflow

When a `.pptx` file is involved, follow this order:

1. Decide whether the task is extraction, editing, or generation
2. If a template exists, inspect its slide layouts and reuse them
3. Use `python-pptx` for the actual deck changes
4. Use `pillow` only for image prep, not as a replacement for slide editing
5. Save to `/mnt/uploads/` and reopen the deck to confirm it loads

Default choices:
- Extract slide text: `python-pptx`
- Update existing deck text/images: `python-pptx`
- Build new slide deck: `python-pptx`
- Prepare visuals before insertion: `pillow`
- Slide-to-image rendering or PDF conversion: clearly state that this is outside Pyodide

## Quick Reference

| Task | Best tool |
|------|-----------|
| Create a slide deck | `python-pptx` |
| Edit text in an existing deck | `python-pptx` |
| Replace images in a slide | `python-pptx` + `pillow` |
| Extract slide text | `python-pptx` |
| Build simple chart/table slides | `python-pptx` |
| Create image grids before insertion | `pillow` |

## File Handling

Read `.pptx` and image inputs from `/mnt/uploads/`.

Write outputs back to `/mnt/uploads/`.

If the user gives a template deck:
1. Inspect existing slide layouts first
2. Reuse the closest layout instead of rebuilding everything
3. Preserve theme, sizing, and overall structure unless the user asked for a redesign

## Common Workflows

### Extract slide text

```python
from pptx import Presentation

prs = Presentation('/mnt/uploads/input.pptx')

for slide_index, slide in enumerate(prs.slides, start=1):
    print(f'--- Slide {slide_index} ---')
    for shape in slide.shapes:
        if hasattr(shape, 'text') and shape.text:
            print(shape.text)
```

### Create a new deck

```python
from pptx import Presentation
from pptx.util import Inches

output_path = '/mnt/uploads/presentation.pptx'
prs = Presentation()
slide = prs.slides.add_slide(prs.slide_layouts[1])
slide.shapes.title.text = 'Quarterly Update'
slide.placeholders[1].text = 'Revenue grew 12%\nCustomer retention improved\nNew markets launched'

slide2 = prs.slides.add_slide(prs.slide_layouts[5])
textbox = slide2.shapes.add_textbox(Inches(1), Inches(1.5), Inches(8), Inches(3))
textbox.text_frame.text = 'Key takeaways'

prs.save(output_path)
print(output_path)
```

### Update an existing template deck

```python
from pptx import Presentation

prs = Presentation('/mnt/uploads/template.pptx')

slide = prs.slides[0]
if slide.shapes.title:
    slide.shapes.title.text = 'Updated Title'

for shape in slide.shapes:
    if hasattr(shape, 'text') and 'PLACEHOLDER' in shape.text:
        shape.text = shape.text.replace('PLACEHOLDER', 'Actual content')

output_path = '/mnt/uploads/updated-template.pptx'
prs.save(output_path)
print(output_path)
```

### Insert prepared images

```python
from PIL import Image, ImageOps
from pptx import Presentation
from pptx.util import Inches

img = Image.open('/mnt/uploads/source.png')
img = ImageOps.contain(img, (1200, 800))
prepared = '/mnt/uploads/prepared-image.png'
img.save(prepared)

prs = Presentation()
slide = prs.slides.add_slide(prs.slide_layouts[6])
slide.shapes.add_picture(prepared, Inches(0.75), Inches(0.75), width=Inches(8.5))

output_path = '/mnt/uploads/image-slide.pptx'
prs.save(output_path)
print(output_path)
```

## Design Guidance

- Use real slide layouts, not blank slides by default
- Keep titles short and body text sparse
- Favor one visual idea per slide
- Use simple tables or charts only when they help comprehension
- Avoid dense paragraphs on slides
- Do not undersell capabilities: normal deck creation and editing are supported with `python-pptx`
- Do not promise visual validation that depends on native PowerPoint or LibreOffice rendering

## QA Guidance

Because slide rendering is limited in Pyodide, verify structurally:
1. Re-open the saved file with `Presentation(output_path)`
2. Count slides and inspect shape text
3. Check image dimensions before insertion
4. Keep text volume conservative to reduce overflow risk

## Limits and Fallbacks

- Do not promise pixel-perfect rendering without native office tooling
- Do not claim slide-to-image export from `.pptx` inside Pyodide
- For heavy template surgery, prefer minimal text/image replacement over deep layout rewriting

## Output Checklist

Before finishing:
1. Confirm the output deck exists in `/mnt/uploads/`
2. Re-open the generated file to ensure it loads
3. Print a quick text summary of slide titles and major text boxes
4. State clearly if visual verification was limited by the browser runtime
