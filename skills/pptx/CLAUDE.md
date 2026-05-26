# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture

This is a skill for working with `.pptx` files. Two main workflows:

1. **Edit existing presentations** (template-based): unpack XML → edit → clean → pack
2. **Create from scratch**: generate via PptxGenJS (Node.js)

Key docs:
- `SKILL.md` - entry point, quick reference, QA process, design guidelines
- `editing.md` - XML-based editing workflow
- `pptxgenjs.md` - PptxGenJS API reference and pitfalls

## Scripts

| Script | Usage |
|--------|-------|
| `scripts/office/unpack.py` | `uv run scripts/office/unpack.py input.pptx unpacked/` |
| `scripts/add_slide.py` | `uv run scripts/add_slide.py unpacked/ slide2.xml` |
| `scripts/clean.py` | `uv run scripts/clean.py unpacked/` |
| `scripts/office/pack.py` | `uv run scripts/office/pack.py unpacked/ output.pptx --original input.pptx` |
| `scripts/thumbnail.py` | `uv run scripts/thumbnail.py input.pptx` |
| `scripts/office/soffice.py` | `uv run scripts/office/soffice.py --headless --convert-to pdf output.pptx` |

## Dependencies

Python script deps (`defusedxml`, `Pillow`, `lxml`) are declared inline via PEP 723 and installed automatically by `uv run`.

```bash
# Text extraction (used as CLI, not via uv run)
pip install "markitdown[pptx]"
# Create-from-scratch
npm install -g pptxgenjs react react-dom react-icons sharp
# System: LibreOffice (soffice) + Poppler (pdftoppm) for image conversion
```

## XML Editing Rules

- Use `defusedxml.minidom`, not `xml.etree.ElementTree` (corrupts namespaces)
- Use the Edit tool for content changes, not sed/Python scripts
- Smart quotes: use XML entities (`&#x201C;` `&#x201D;` `&#x2018;` `&#x2019;`)
- Never use unicode bullets (`•`); use `<a:buChar>` or `<a:buAutoNum>`
- Bold headers with `b="1"` on `<a:rPr>`
- Multi-item content: separate `<a:p>` elements, never concatenate into one string

## PptxGenJS Rules

- Never use `#` prefix on hex colors — corrupts file
- Never encode opacity in hex color strings (8-char hex) — use `opacity` property
- Never reuse option objects (e.g. shadow) across calls — PptxGenJS mutates in-place; use factory functions
- Use `bullet: true`, never unicode `•`
- Use `breakLine: true` between array items

## QA Process

1. Content check: `python -m markitdown output.pptx`
2. Check for placeholders: `python -m markitdown output.pptx | grep -iE "xxxx|lorem|ipsum"`
3. Visual QA: convert to images via `soffice` + `pdftoppm`, inspect with subagents
4. Fix issues and re-verify — never declare success without at least one fix-and-verify cycle
