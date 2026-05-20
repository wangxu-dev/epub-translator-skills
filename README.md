# EPUB Translator Toolkit

A practical toolkit for translating EPUB books with AI assistance. Handles text extraction, code preservation, translation injection, and EPUB rebuilding. Language-agnostic — works with any source/target language pair.

## Background

Built and battle-tested during two real translation projects. Version 0.0.2 includes fixes for container blocks, inline tag preservation, NCX navigation, and language metadata.

| Book | Segments | Volume |
|------|----------|--------|
| Programming (300pp) | 2,319 | ~305K chars |
| Programming (600pp) | 4,329 | ~2.2M chars |

**Total: 6,648 segments, ~2.5M characters processed.**

## How It Works

```
EPUB (ZIP of XHTML)
    │
    ▼
[export] Extract text by block elements
    │ Preserve <code> as ⌜CODE_N⌝ placeholders
    ▼
translations.json  ───▶  Translate segments
    │                       (AI / human / hybrid)
    ▼
[build] Inject translations
    │ Restore code placeholders
    ▼
Translated EPUB
```

### Phase 1: Export
- Parses EPUB (a ZIP of XHTML/HTML files)
- Groups text by block-level elements (`<p>`, `<h1>`, `<li>`, etc.)
- Replaces `<code>` blocks with `⌜CODE_N⌝` placeholders to prevent translation
- Produces a JSON file: `{"idx": 0, "file": "...", "text": "...", "translated": null}`

### Phase 2: Translate
The JSON acts as the single source of truth. Translate segments in any way:
- Use `batch_translate.py extract` to export plain-text batches
- Translate manually, via AI, or a mix
- Inject back with `batch_translate.py inject`

### Phase 3: Build
- Reads translated JSON
- Restores `⌜CODE_N⌝` to original `<code>` HTML
- Injects translations into the original EPUB structure
- Repacks as a new EPUB with identical formatting

## Installation

```bash
pip install beautifulsoup4
# or
uv add beautifulsoup4
```

## Usage

```bash
# Extract text from EPUB
python scripts/epub_translator.py export book.epub -o translations.json

# Check progress
python scripts/epub_translator.py info translations.json

# Verify translation before build
python scripts/epub_translator.py verify translations.json

# Build translated EPUB
python scripts/epub_translator.py build book.epub translations.json -o book_translated.epub

# Batch export for progressive translation
python scripts/batch_translate.py extract translations.json 0 100

# Inject batch translations
python scripts/batch_translate.py inject translations.json batch_0_100_translated.json
```

## Batch Workflow

```bash
# 1. Extract batch
python scripts/batch_translate.py extract translations.json 0 100

# 2. Translate batch_0_100.txt, create batch_0_100_translated.json
#    Format: {"0": "translated text", "1": "translated text", ...}

# 3. Inject back
python scripts/batch_translate.py inject translations.json batch_0_100_translated.json

# 4. Repeat
python scripts/batch_translate.py extract translations.json 100 200
```

## Design Decisions

### Code Preservation
Technical books contain code. `<code>` blocks are replaced with `⌜CODE_N⌝` placeholders during extraction and restored during injection. Code is never translated or corrupted.

### Block-Level Extraction
Text is extracted at the block-element level rather than individual text nodes. This prevents inline tags (`<strong>`, `<em>`, `<a>`) from fragmenting sentences into isolated segments.

### Progressive Translation
The toolkit supports translating in batches of 100-200 segments. This enables incremental progress, checkpointing, and mixing AI with human translation.

### Language Independence
No hardcoded language logic. The same workflow works for EN→ZH, EN→ES, DE→EN, or any other pair. The JSON format is fully language-agnostic.

## Lessons Learned

### What Worked
- Block-level grouping dramatically improved translation quality vs. node-level extraction
- Code placeholders are essential for technical books
- Progressive batches (100-200 segments) allowed steady progress with quality checks
- Single JSON as truth source simplified progress tracking and resumption

### Watch Out For
- A 300-page book produces ~300K translated chars; a 600-page book → ~1.5M chars
- Windows terminal (GBK) cannot print certain Unicode characters — always write output to files
- EPUBs often have duplicate text in TOC and body content — track progress by source file
- Translate in batches even with large AI context windows to maintain quality

## Project Structure

```
epub-translator-toolkit/
├── SKILL.md                  # Skill definition (workflow & guidance)
├── README.md                 # English documentation
├── README_ZH.md              # Chinese documentation
└── scripts/
    ├── epub_translator.py    # Main: export / info / verify / build
    └── batch_translate.py    # Helper: extract / inject batches

book-project/
├── book.epub                 # Original EPUB
├── book_translated.epub      # Translated EPUB
├── source/                   # Work files (JSON, batch intermediates)
└── tools/                    # Copy of scripts/
```

