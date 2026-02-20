# PRD Example 7: Expense Tracker CLI

> **Difficulty:** Beginner | **Project Type:** CLI Tool | **Time:** 2-3 hours

---

## Overview

| | |
|---|---|
| **What** | A command-line expense tracker for logging and viewing spending |
| **Who** | Anyone who wants to track daily expenses without a complex app |
| **Why** | Provides quick expense logging and simple monthly summaries from the terminal |

---

## Core Features (MVP)

1. **Add Expense:** `expense add 25.50 "Lunch" food` → Logs expense with amount, description, category
2. **List Expenses:** `expense list` → Shows all expenses for current month
3. **Summary:** `expense summary` → Shows spending by category for current month
4. **Delete:** `expense delete 5` → Removes expense by ID

---

## Non-Goals

**Will NOT build:**
- Income tracking or budgeting
- Multiple currencies or conversion
- Recurring expenses
- Receipt photos or attachments
- Export to CSV/Excel
- Charts or visualizations
- Date range filtering
- Multi-user or shared accounts

**Will NOT use:**
- External APIs
- Database (use JSON file)
- Web or GUI interface
- Financial libraries

---

## Technical Constraints

| | |
|---|---|
| **Language** | Node.js 20+ |
| **Framework** | None (stdlib only) |
| **Dependencies** | None — stdlib only |
| **Data Storage** | JSON file at `~/.expenses/data.json` |
| **Testing** | Node.js built-in test runner |
| **Code Style** | ESLint, JSDoc comments |

---

## Success Criteria

- [ ] `expense add` creates entry with amount, description, category, timestamp
- [ ] `expense list` shows current month's expenses with running total
- [ ] `expense summary` groups by category with subtotals
- [ ] `expense delete` removes by ID with confirmation
- [ ] Amounts are stored as cents (integers) to avoid float issues
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Data Model & Add Command
**Goal:** Set up expense storage and creation

**Tasks:**
1. Create project with `expense.js` and `package.json`
2. Define expense schema: `{id, amount, description, category, date}`
3. Implement `add` command with argument parsing
4. Store amounts as cents (multiply by 100)

**Verification:**
```bash
node expense.js add 25.50 "Lunch" food
# Output: "Expense #1 added: $25.50 for Lunch (food)"

cat ~/.expenses/data.json
# Shows: {"expenses": [{"id": 1, "amount": 2550, "description": "Lunch", ...}]}
```

**Deliverables:** `expense.js` with add functionality

---

### Phase 2: List & Summary
**Goal:** Display expenses and summaries

**Tasks:**
1. Implement `list` command filtering by current month
2. Format amounts as currency ($XX.XX)
3. Implement `summary` command grouping by category
4. Show running totals

**Verification:**
```bash
node expense.js list
# Output:
# February 2025 Expenses:
# #1  02/12  $25.50  Lunch (food)
# #2  02/12  $4.50   Coffee (food)
# #3  02/11  $50.00  Gas (transport)
# Total: $80.00

node expense.js summary
# Output:
# February 2025 Summary:
# food:      $30.00 (2 items)
# transport: $50.00 (1 item)
# Total:     $80.00
```

**Deliverables:** List and summary commands

---

### Phase 3: Delete & Testing
**Goal:** Complete CRUD and add tests

**Tasks:**
1. Implement `delete` command with ID validation
2. Add confirmation prompt before delete
3. Write tests using Node.js test runner
4. Handle edge cases (empty data, invalid ID)

**Verification:**
```bash
node expense.js delete 1
# Output: "Delete Expense #1: $25.50 Lunch? (y/n): "
# User types: y
# Output: "Expense #1 deleted"

node --test
# All tests pass
```

**Deliverables:** Complete CLI with tests

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Store amounts as integers (cents) | Before changing JSON schema | Use floating point for money |
| Validate expense exists before delete | Before adding new categories | Store data outside `~/.expenses/` |
| Confirm before destructive actions | Before changing date filtering | Add budget or income tracking |
| Format currency consistently ($XX.XX) | | Delete without confirmation |
