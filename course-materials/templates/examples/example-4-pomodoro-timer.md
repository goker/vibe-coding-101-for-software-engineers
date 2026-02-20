# PRD Example 4: Pomodoro Timer CLI

> **Difficulty:** Beginner | **Project Type:** CLI Tool | **Time:** 2-3 hours

---

## Overview

| | |
|---|---|
| **What** | A command-line Pomodoro timer that tracks work sessions and breaks |
| **Who** | Students and developers who want to manage focus time |
| **Why** | Helps maintain productivity with timed work/break cycles without leaving the terminal |

---

## Core Features (MVP)

1. **Start Timer:** `pomo start` → Begins a 25-minute work session with countdown display
2. **Take Break:** `pomo break` → Starts a 5-minute break timer (or 15-min after 4 sessions)
3. **Show Status:** `pomo status` → Shows current session, time remaining, and total sessions today
4. **View Stats:** `pomo stats` → Shows daily/weekly summary of completed sessions

---

## Non-Goals

**Will NOT build:**
- GUI or desktop notifications
- Sound alerts or audio cues
- Customizable timer durations (fixed 25/5/15 pattern)
- Task names or descriptions for sessions
- Cloud sync or multi-device support
- Historical data beyond 7 days
- Integration with calendar or task apps

**Will NOT use:**
- External APIs
- Database (SQLite, PostgreSQL, etc.)
- Rich terminal UI libraries (blessed, rich, etc.)
- Background processes or daemons

---

## Technical Constraints

| | |
|---|---|
| **Language** | Python 3.11+ |
| **Framework** | None (stdlib only) |
| **Dependencies** | None — stdlib only |
| **Data Storage** | JSON file at `~/.pomo/sessions.json` |
| **Testing** | pytest, minimum 3 tests |
| **Code Style** | Black formatter, type hints |

---

## Success Criteria

- [ ] `pomo start` begins 25-minute countdown with live display
- [ ] `pomo break` correctly alternates between 5-min and 15-min breaks
- [ ] `pomo status` shows accurate time remaining
- [ ] `pomo stats` displays sessions from JSON file
- [ ] All tests pass with `pytest`
- [ ] No external dependencies beyond stdlib

---

## Implementation Phases

### Phase 1: Core Timer Logic
**Goal:** Implement the basic timer functionality

**Tasks:**
1. Create project structure with `pomo.py` and `tests/`
2. Implement countdown timer with terminal display
3. Add `start` command for 25-minute sessions
4. Save completed sessions to JSON file

**Verification:**
```bash
python pomo.py start
# Shows countdown: "25:00 remaining..."
# After completion: "Session complete! Take a break."

cat ~/.pomo/sessions.json
# Shows: {"sessions": [{"date": "2025-02-12", "completed": 1}]}
```

**Deliverables:** `pomo.py`, `~/.pomo/sessions.json`

---

### Phase 2: Break Timer & Status
**Goal:** Add break functionality and status display

**Tasks:**
1. Implement `break` command with 5/15-minute logic
2. Track session count to determine break length
3. Add `status` command showing current state
4. Handle Ctrl+C gracefully (cancel without crashing)

**Verification:**
```bash
python pomo.py break
# After 4 work sessions: "Long break: 15:00 remaining..."
# Otherwise: "Short break: 5:00 remaining..."

python pomo.py status
# Shows: "Session 3 of 4 | Next: Long break"
```

**Deliverables:** Updated `pomo.py` with break logic

---

### Phase 3: Stats & Testing
**Goal:** Add statistics display and tests

**Tasks:**
1. Implement `stats` command with daily/weekly view
2. Add pytest tests for timer logic
3. Test JSON file read/write
4. Add error handling for missing data file

**Verification:**
```bash
python pomo.py stats
# Shows:
# Today: 4 sessions (1h 40m focused)
# This week: 18 sessions (7h 30m focused)

pytest
# All tests pass
```

**Deliverables:** Complete CLI with tests

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Handle missing JSON file gracefully | Before changing JSON schema | Use external packages |
| Validate timer is not already running | Before adding new commands | Store data outside `~/.pomo/` |
| Show helpful error messages | Before changing countdown display format | Add features not in this PRD |
| Use type hints for all functions | | Implement background/daemon mode |
