# PRD Example 9: Daily Journal CLI

> **Difficulty:** Beginner | **Project Type:** CLI Tool | **Time:** 2-3 hours

---

## Overview

| | |
|---|---|
| **What** | A command-line journal for writing and reviewing daily entries |
| **Who** | Anyone who wants to maintain a daily journaling habit |
| **Why** | Enables quick journal entries without leaving the terminal |

---

## Core Features (MVP)

1. **Write Entry:** `journal write` → Opens editor for today's entry
2. **Quick Add:** `journal add "Had a great day!"` → Appends text to today's entry
3. **Read Entry:** `journal read` → Shows today's entry (or `journal read 2025-02-10` for specific date)
4. **List Entries:** `journal list` → Shows all entry dates for current month

---

## Non-Goals

**Will NOT build:**
- Rich text formatting or Markdown
- Tags or categories
- Search across entries
- Mood tracking or analytics
- Image or media attachments
- Export to PDF or other formats
- Encryption or password protection
- Cloud sync or backup

**Will NOT use:**
- External APIs
- Database (use plain text files)
- GUI or web interface
- Text editor libraries (use system `$EDITOR`)

---

## Technical Constraints

| | |
|---|---|
| **Language** | Python 3.11+ |
| **Framework** | None (stdlib only) |
| **Dependencies** | None — stdlib only |
| **Data Storage** | Text files at `~/.journal/YYYY/MM/DD.txt` |
| **Editor** | Use `$EDITOR` env var (default: `nano`) |
| **Testing** | pytest, minimum 4 tests |
| **Code Style** | Black formatter, type hints |

---

## Success Criteria

- [ ] `journal write` opens system editor for today's file
- [ ] `journal add "text"` appends timestamped line to today's entry
- [ ] `journal read` displays today's entry
- [ ] `journal read YYYY-MM-DD` displays specific date's entry
- [ ] `journal list` shows all entry dates this month
- [ ] Files are stored as plain text in date-based folders
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Write & Add Commands
**Goal:** Create and append to journal entries

**Tasks:**
1. Create `journal.py` with directory structure logic
2. Implement `write` command that opens `$EDITOR`
3. Implement `add` command that appends with timestamp
4. Create date-based directory structure automatically

**Verification:**
```bash
python journal.py write
# Opens nano/vim with today's file

python journal.py add "Finished the project!"
# Output: "Added to February 12, 2025"

cat ~/.journal/2025/02/12.txt
# Shows:
# [14:30] Finished the project!
```

**Deliverables:** `journal.py` with write/add commands

---

### Phase 2: Read Command
**Goal:** Display journal entries

**Tasks:**
1. Implement `read` command for today's entry
2. Support optional date argument (`journal read 2025-02-10`)
3. Handle missing entries gracefully
4. Parse and validate date format

**Verification:**
```bash
python journal.py read
# Output:
# === February 12, 2025 ===
# [09:00] Morning thoughts...
# [14:30] Finished the project!

python journal.py read 2025-02-10
# Output:
# === February 10, 2025 ===
# [Entry content...]

python journal.py read 2025-01-01
# Output: "No entry for January 1, 2025"
```

**Deliverables:** Read command with date support

---

### Phase 3: List & Testing
**Goal:** List entries and add tests

**Tasks:**
1. Implement `list` command for current month
2. Show dates with entry counts (lines)
3. Write pytest tests for all commands
4. Handle edge cases (empty month, invalid dates)

**Verification:**
```bash
python journal.py list
# Output:
# February 2025 Entries:
# - 2025-02-12 (3 entries)
# - 2025-02-10 (5 entries)
# - 2025-02-08 (2 entries)

pytest
# All tests pass
```

**Deliverables:** Complete CLI with tests

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Create directories if they don't exist | Before changing file structure | Delete or overwrite entries |
| Use `$EDITOR` environment variable | Before adding new commands | Encrypt or password-protect |
| Validate date format (YYYY-MM-DD) | Before changing timestamp format | Store data outside `~/.journal/` |
| Handle missing files gracefully | | Add search or analytics features |
