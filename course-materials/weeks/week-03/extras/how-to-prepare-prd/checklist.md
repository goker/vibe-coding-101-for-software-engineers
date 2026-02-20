# PRD Quality Checklist

> Print this page and check each box before you start coding

---

## Overview Section
- [ ] **What:** One sentence describing the project
- [ ] **Who:** Specific target user (not just "users")
- [ ] **Why:** Clear problem being solved

## Core Features (MVP)
- [ ] 3-5 features maximum (no more!)
- [ ] Each feature is numbered
- [ ] Each feature includes command/usage example
- [ ] Features are specific (not "make it work")

## Non-Goals (CRITICAL!)
- [ ] Listed at least 5 features you WON'T build
- [ ] Listed technologies you WON'T use
- [ ] Answered: "What might AI add that I don't want?"
- [ ] No vague terms ("nothing complicated" ❌)

## Technical Constraints
- [ ] Language with specific version (e.g., "Python 3.11+")
- [ ] Framework specified (or "None / stdlib only")
- [ ] Dependencies listed (or "None")
- [ ] Data storage location specified
- [ ] Testing framework specified

## Success Criteria
- [ ] Each criterion is testable
- [ ] No vague terms ("works well" ❌)
- [ ] Includes specific commands or checks
- [ ] Maps to Core Features

## Implementation Phases
- [ ] 2-4 phases total
- [ ] Each phase has:
  - [ ] **Goal:** One sentence
  - [ ] **Tasks:** Numbered list
  - [ ] **Verification:** Copy-paste commands
  - [ ] **Deliverables:** Files created

## Agent Rules
- [ ] **Always** column has 3+ items
- [ ] **Ask First** column has 2+ items
- [ ] **Never** column has 3+ items
- [ ] Includes "Never add features not in PRD"

---

## Final Check

Before sharing with AI:

- [ ] Re-read entire PRD — does it make sense?
- [ ] Ask: "What's missing that AI might assume?"
- [ ] Verification commands are copy-paste ready
- [ ] No TODOs or placeholders remaining

---

## Quick Reference

**Good Non-Goals:**
```
- User authentication
- Database (use JSON)
- Cloud sync
- Due dates/reminders
- External APIs
```

**Good Agent Rules:**
```
| Always | Ask First | Never |
| Validate input | Before schema change | External packages |
| Handle errors | Before new commands | Add features |
| Show help text | Before refactoring | Skip tests |
```

---

*Vibe Coding 101 — Week 3 Extra Material*
