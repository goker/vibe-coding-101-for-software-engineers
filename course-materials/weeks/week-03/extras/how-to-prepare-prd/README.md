# How to Prepare a PRD for Vibe Coding

> Week 3 Extra Material — Deep dive into writing effective PRDs for AI-assisted development

---

## Overview

This extra lesson provides comprehensive guidance on writing Product Requirement Documents (PRDs) that work well with AI coding tools like Claude Code and Gemini CLI.

**The core insight:** AI cannot infer from omission. What you don't explicitly specify, AI will invent. A well-written PRD prevents scope creep, hallucinated features, and wasted iterations.

---

## Materials

| Resource | Description |
|----------|-------------|
| [Slides (PDF)](<slides/How to Prepare a PRD By Goker Ezberci.pdf>) | Presentation covering all PRD sections |
| [Checklist](checklist.md) | One-page printable quality checklist |
| [Examples](../../../../templates/examples/) | 24 complete PRD examples |

---

## The 7 Sections of a PRD

### 1. Overview
**Purpose:** Define What, Who, and Why in one sentence each

```markdown
| **What** | A command-line todo app for managing tasks |
| **Who** | Developers who prefer terminal workflows |
| **Why** | Quick task management without leaving the terminal |
```

**Common mistake:** Vague descriptions like "An app for users"

---

### 2. Core Features (MVP)
**Purpose:** 3-5 specific features with examples

```markdown
1. **Add Task:** `todo add "Buy groceries"` → Creates task with auto ID
2. **List Tasks:** `todo list` → Shows all tasks with [ ] or [x]
3. **Complete Task:** `todo done 3` → Marks task #3 as complete
```

**Common mistake:** Vague features like "Add things" or "Make it work well"

---

### 3. Non-Goals (CRITICAL!)
**Purpose:** Explicitly list what you WON'T build

```markdown
**Will NOT build:**
- User authentication or login
- Due dates or reminders
- Cloud sync or backup

**Will NOT use:**
- Database (SQLite, PostgreSQL, etc.)
- External APIs
- Rich terminal UI libraries
```

**Why this matters:** AI reads your PRD and wonders "Should I add authentication?" Without Non-Goals, it will guess (usually wrong).

**Rule:** If you wouldn't build it in v1, list it as a Non-Goal.

---

### 4. Technical Constraints
**Purpose:** Specify exact versions and requirements

```markdown
| **Language** | Python 3.11+ |
| **Framework** | None (stdlib only) |
| **Dependencies** | None |
| **Data Storage** | JSON at ~/.todo/tasks.json |
| **Testing** | pytest, minimum 3 tests |
```

**Common mistake:** Not specifying versions or allowing "any framework"

---

### 5. Success Criteria
**Purpose:** Testable checkboxes for "done"

```markdown
- [ ] `todo add "text"` creates task with unique ID
- [ ] `todo list` shows all tasks with status
- [ ] Data persists between sessions
- [ ] All tests pass
```

**Common mistake:** Vague criteria like "works well" or "is fast"

---

### 6. Implementation Phases
**Purpose:** Step-by-step build plan with verification

```markdown
### Phase 1: Core Data Model
**Goal:** Set up project and implement task storage

**Tasks:**
1. Create project structure
2. Define Task dataclass
3. Implement JSON read/write

**Verification:**
python todo.py add "Test"
cat ~/.todo/tasks.json  # Shows task

**Deliverables:** todo.py, JSON utilities
```

**Common mistake:** Missing verification commands

---

### 7. Agent Rules
**Purpose:** AI decision-making guardrails

```markdown
| Always | Ask First | Never |
|--------|-----------|-------|
| Validate task ID | Before schema change | Use external packages |
| Handle errors | Before new commands | Add features not in PRD |
| Show help text | Before refactoring | Skip tests |
```

**What each column means:**
- **Always:** Do this automatically
- **Ask First:** Get human approval before doing
- **Never:** Hard stop, don't do this

---

## Common Mistakes

### Mistake 1: Vague Features
❌ "Make it work well"
✅ `todo list` shows all tasks with `[ ]` or `[x]` prefix

### Mistake 2: Missing Non-Goals
❌ (No Non-Goals section)
✅ "Will NOT build: authentication, database, cloud sync..."

### Mistake 3: No Verification
❌ "Phase 1: Set up project"
✅ "Verification: `python todo.py add 'Test'` → Shows task ID"

---

## PRD Quality Checklist

Before starting to code, verify:

- [ ] Overview has What/Who/Why (all three)
- [ ] Features are numbered (3-5 max)
- [ ] Non-Goals list at least 5 things you WON'T build
- [ ] Technical constraints specify exact versions
- [ ] Success criteria are testable
- [ ] Each phase has verification commands
- [ ] Agent Rules table has all three columns

**If ANY is missing, fix it before coding.**

---

## Example PRDs

See `templates/examples/` for 24 complete examples:

**Beginner (2-3 hours):**
- Pomodoro Timer, Password Generator, Flashcard Quiz

**Intermediate (4-8 hours):**
- Recipe API, Bookmark Manager, Habit Tracker

**Advanced (1-4 weeks):**
- Real-time Chat, Finance Dashboard, SaaS Starter

---

## Key Takeaways

1. **AI cannot infer from omission** — Be explicit about everything
2. **Non-Goals are critical** — List what you WON'T build
3. **Use Agent Rules** — Define Always/Ask First/Never boundaries
4. **Include verification** — Every phase needs test commands
5. **Be specific** — Commands with expected output, not vague descriptions

---

*Part of Vibe Coding 101: From Idea to Shipped Product*
