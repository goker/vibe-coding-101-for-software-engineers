# PRD Example 15: CLI File Organizer

> **Difficulty:** Intermediate | **Project Type:** CLI Tool | **Time:** 4-6 hours

---

## Overview

| | |
|---|---|
| **What** | A command-line tool that organizes files into folders by type or date |
| **Who** | Anyone with cluttered Downloads or Desktop folders |
| **Why** | Automates the tedious task of sorting files into organized folders |

---

## Core Features (MVP)

1. **Organize by Type:** `organize ./Downloads --by-type` → Moves files into Images/, Documents/, Videos/, etc.
2. **Organize by Date:** `organize ./Downloads --by-date` → Moves files into 2025/02/, 2025/01/, etc.
3. **Dry Run:** `organize ./Downloads --dry-run` → Shows what would happen without moving
4. **Undo:** `organize --undo` → Reverts last organization using log file

---

## Non-Goals

**Will NOT build:**
- Duplicate file detection
- File renaming or deduplication
- Recursive subfolder organization
- Custom rules or configuration file
- GUI or interactive mode
- File content analysis (only extension-based)
- Cloud storage integration
- Scheduled/automatic organization

**Will NOT use:**
- External APIs
- Database
- File watching libraries
- Configuration files

---

## Technical Constraints

| | |
|---|---|
| **Language** | Go 1.21+ |
| **Framework** | None (stdlib only) |
| **Dependencies** | None — stdlib only |
| **Log File** | `~/.organize/history.json` for undo |
| **Testing** | Go built-in testing |
| **Code Style** | gofmt, golint |

---

## Success Criteria

- [ ] `--by-type` creates folders: Images, Documents, Videos, Audio, Archives, Other
- [ ] `--by-date` creates folders: YYYY/MM format
- [ ] `--dry-run` shows plan without moving files
- [ ] `--undo` reverts last operation
- [ ] Handles filename conflicts (adds _1, _2, etc.)
- [ ] Skips hidden files and directories
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Project Setup & File Scanning
**Goal:** Set up project and scan directory

**Tasks:**
1. Create Go project with `main.go`
2. Implement directory scanning (non-recursive)
3. Define file type mappings (extension → category)
4. Parse command line arguments

**Verification:**
```bash
go run main.go ./Downloads --dry-run --by-type
# Output:
# Would move vacation.jpg → Images/vacation.jpg
# Would move report.pdf → Documents/report.pdf
# Would move song.mp3 → Audio/song.mp3
#
# 3 files would be organized (dry run)
```

**Deliverables:** `main.go` with scanning and dry-run

---

### Phase 2: Organize by Type
**Goal:** Move files into category folders

**Tasks:**
1. Create category folders if not exist
2. Move files to appropriate folders
3. Handle filename conflicts
4. Log operations to history file

**Verification:**
```bash
go run main.go ./Downloads --by-type
# Output:
# Moved vacation.jpg → Images/vacation.jpg
# Moved report.pdf → Documents/report.pdf
#
# 2 files organized

ls ./Downloads/Images
# vacation.jpg
```

**Deliverables:** Working `--by-type` command

---

### Phase 3: Organize by Date
**Goal:** Move files into date-based folders

**Tasks:**
1. Read file modification time
2. Create YYYY/MM folder structure
3. Move files to date folders
4. Log operations

**Verification:**
```bash
go run main.go ./Downloads --by-date
# Output:
# Moved vacation.jpg → 2025/02/vacation.jpg
# Moved old_doc.pdf → 2024/12/old_doc.pdf

ls ./Downloads/2025/02
# vacation.jpg
```

**Deliverables:** Working `--by-date` command

---

### Phase 4: Undo & Testing
**Goal:** Implement undo and add tests

**Tasks:**
1. Read history file for last operation
2. Reverse all moves
3. Clear history entry after undo
4. Write Go tests for all features

**Verification:**
```bash
go run main.go --undo
# Output:
# Reverted: Images/vacation.jpg → vacation.jpg
# Reverted: Documents/report.pdf → report.pdf
#
# 2 files restored

go test ./...
# All tests pass
```

**Deliverables:** Complete CLI with tests

---

## File Type Mappings

```go
var categories = map[string][]string{
    "Images":    {".jpg", ".jpeg", ".png", ".gif", ".svg", ".webp"},
    "Documents": {".pdf", ".doc", ".docx", ".txt", ".xlsx", ".pptx"},
    "Videos":    {".mp4", ".mov", ".avi", ".mkv", ".webm"},
    "Audio":     {".mp3", ".wav", ".flac", ".aac", ".ogg"},
    "Archives":  {".zip", ".tar", ".gz", ".rar", ".7z"},
    "Code":      {".py", ".js", ".ts", ".go", ".rs", ".java"},
}
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Skip hidden files (starting with .) | Before adding new categories | Delete any files |
| Handle filename conflicts safely | Before recursive organization | Modify file contents |
| Log all operations for undo | Before changing folder structure | Organize system directories |
| Show summary after operation | | Require config file |
