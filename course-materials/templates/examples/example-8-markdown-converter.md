# PRD Example 8: Markdown to HTML Converter CLI

> **Difficulty:** Beginner | **Project Type:** CLI Tool | **Time:** 2-3 hours

---

## Overview

| | |
|---|---|
| **What** | A command-line tool that converts Markdown files to HTML |
| **Who** | Developers and writers who need quick Markdown conversion |
| **Why** | Enables fast, local Markdown to HTML conversion without online tools |

---

## Core Features (MVP)

1. **Convert File:** `mdconv input.md` → Outputs HTML to stdout
2. **Output to File:** `mdconv input.md -o output.html` → Saves HTML to file
3. **Watch Mode:** `mdconv input.md --watch` → Re-converts on file change
4. **Wrap in Template:** `mdconv input.md --full` → Outputs complete HTML document with `<head>` and `<body>`

---

## Non-Goals

**Will NOT build:**
- Support for all Markdown extensions (GFM, tables, footnotes)
- Syntax highlighting for code blocks
- Custom CSS styling options
- Batch conversion of multiple files
- Live preview server
- PDF or other format output
- Front matter parsing (YAML headers)

**Will NOT use:**
- External Markdown libraries (must parse manually)
- External APIs
- Web frameworks
- File watching libraries (use stdlib polling)

---

## Technical Constraints

| | |
|---|---|
| **Language** | Python 3.11+ |
| **Framework** | None (stdlib only) |
| **Dependencies** | None — stdlib only |
| **Markdown Support** | Headers (#), bold (**), italic (*), links, code blocks, lists |
| **Testing** | pytest, minimum 5 tests |
| **Code Style** | Black formatter, type hints |

---

## Success Criteria

- [ ] Converts `# Header` to `<h1>Header</h1>` (h1-h6)
- [ ] Converts `**bold**` to `<strong>bold</strong>`
- [ ] Converts `*italic*` to `<em>italic</em>`
- [ ] Converts `[text](url)` to `<a href="url">text</a>`
- [ ] Converts code blocks to `<pre><code>...</code></pre>`
- [ ] Converts `- item` lists to `<ul><li>item</li></ul>`
- [ ] `--full` wraps output in valid HTML5 document
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Basic Parsing
**Goal:** Parse headers, bold, italic, links

**Tasks:**
1. Create `mdconv.py` with file reading
2. Implement regex-based parsing for headers (h1-h6)
3. Parse bold (`**text**`) and italic (`*text*`)
4. Parse links (`[text](url)`)

**Verification:**
```bash
echo "# Hello **World**" | python mdconv.py
# Output: <h1>Hello <strong>World</strong></h1>

echo "[Click here](https://example.com)" | python mdconv.py
# Output: <p><a href="https://example.com">Click here</a></p>
```

**Deliverables:** `mdconv.py` with basic parsing

---

### Phase 2: Code Blocks & Lists
**Goal:** Handle code blocks and unordered lists

**Tasks:**
1. Parse fenced code blocks (```)
2. Parse inline code (`code`)
3. Parse unordered lists (`- item`)
4. Handle nested content (bold inside lists)

**Verification:**
```bash
echo "- Item **one**
- Item two" | python mdconv.py
# Output:
# <ul>
# <li>Item <strong>one</strong></li>
# <li>Item two</li>
# </ul>
```

**Deliverables:** Code block and list support

---

### Phase 3: Output Options & Testing
**Goal:** Add file output, full document mode, and tests

**Tasks:**
1. Add `-o` flag for file output
2. Add `--full` flag for complete HTML document
3. Add `--watch` mode with polling
4. Write pytest tests for each Markdown feature

**Verification:**
```bash
python mdconv.py input.md -o output.html
# Creates output.html file

python mdconv.py input.md --full
# Output:
# <!DOCTYPE html>
# <html>
# <head><title>Document</title></head>
# <body>
# <h1>Content here</h1>
# </body>
# </html>

pytest
# All tests pass
```

**Deliverables:** Complete CLI with tests

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Escape HTML entities in input | Before adding new Markdown syntax | Use external Markdown libraries |
| Validate input file exists | Before changing output format | Parse unsupported Markdown extensions |
| Handle empty files gracefully | Before adding CSS support | Modify input files |
| Output valid HTML5 | | Add syntax highlighting |
