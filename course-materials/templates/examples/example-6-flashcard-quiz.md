# PRD Example 6: Flashcard Quiz CLI

> **Difficulty:** Beginner | **Project Type:** CLI Tool | **Time:** 2-3 hours

---

## Overview

| | |
|---|---|
| **What** | A command-line flashcard app for studying with spaced repetition |
| **Who** | Students preparing for exams or learning new topics |
| **Why** | Enables quick study sessions directly in the terminal without distractions |

---

## Core Features (MVP)

1. **Add Card:** `flash add "What is Python?" "A programming language"` → Creates a flashcard
2. **Study Mode:** `flash study` → Shows cards one by one, user marks correct/incorrect
3. **List Cards:** `flash list` → Shows all cards with their success rates
4. **Delete Card:** `flash delete 3` → Removes card by ID

---

## Non-Goals

**Will NOT build:**
- Deck organization or categories
- Spaced repetition algorithm (simple random order only)
- Import/export from Anki or other formats
- Rich text or markdown in cards
- Images or media attachments
- Timed quizzes
- Multi-user or sharing features
- Statistics beyond simple success rate

**Will NOT use:**
- External APIs
- Database (use JSON file)
- Web interface
- Third-party flashcard libraries

---

## Technical Constraints

| | |
|---|---|
| **Language** | Python 3.11+ |
| **Framework** | None (stdlib only) |
| **Dependencies** | None — stdlib only |
| **Data Storage** | JSON file at `~/.flashcards/cards.json` |
| **Testing** | pytest, minimum 4 tests |
| **Code Style** | Black formatter, type hints |

---

## Success Criteria

- [ ] `flash add` creates cards with unique IDs
- [ ] `flash study` presents cards randomly, tracks correct/incorrect
- [ ] `flash list` shows all cards with success percentage
- [ ] `flash delete` removes cards by ID
- [ ] Data persists between sessions
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Data Model & Add Command
**Goal:** Set up card storage and creation

**Tasks:**
1. Create project structure with `flash.py`
2. Define card schema: `{id, question, answer, correct, attempts}`
3. Implement `add` command
4. Create JSON file in `~/.flashcards/` if not exists

**Verification:**
```bash
python flash.py add "Capital of France?" "Paris"
# Output: "Card #1 added!"

cat ~/.flashcards/cards.json
# Shows: {"cards": [{"id": 1, "question": "Capital of France?", ...}]}
```

**Deliverables:** `flash.py` with add functionality

---

### Phase 2: Study Mode
**Goal:** Interactive study session

**Tasks:**
1. Implement `study` command that shows cards one at a time
2. User presses Enter to reveal answer
3. User types `y` (correct) or `n` (incorrect)
4. Update success/attempt counters in JSON

**Verification:**
```bash
python flash.py study
# Shows: "Question: Capital of France?"
# User presses Enter
# Shows: "Answer: Paris"
# Shows: "Correct? (y/n): "
# User types: y
# Shows: "Correct! Next card..."
# After all cards: "Session complete! 4/5 correct (80%)"
```

**Deliverables:** Working study mode

---

### Phase 3: List, Delete & Testing
**Goal:** Complete CRUD and add tests

**Tasks:**
1. Implement `list` command showing all cards with stats
2. Implement `delete` command by card ID
3. Write pytest tests for all commands
4. Handle edge cases (empty deck, invalid ID)

**Verification:**
```bash
python flash.py list
# Shows:
# #1 "Capital of France?" → Paris (80% correct, 5 attempts)
# #2 "What is 2+2?" → 4 (100% correct, 3 attempts)

python flash.py delete 1
# Output: "Card #1 deleted"

pytest
# All tests pass
```

**Deliverables:** Complete CLI with tests

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Validate card ID exists before delete | Before changing JSON schema | Use external packages |
| Handle empty deck gracefully | Before adding new commands | Store data outside `~/.flashcards/` |
| Show helpful prompts during study | Before changing study flow | Add deck/category features |
| Preserve card stats on update | | Delete all cards without confirmation |
