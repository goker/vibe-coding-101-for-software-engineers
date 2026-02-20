# PRD Example 5: Password Generator CLI

> **Difficulty:** Beginner | **Project Type:** CLI Tool | **Time:** 2-3 hours

---

## Overview

| | |
|---|---|
| **What** | A command-line tool that generates secure, customizable passwords |
| **Who** | Anyone who needs to create strong passwords quickly |
| **Why** | Helps users create unique, secure passwords without relying on external services |

---

## Core Features (MVP)

1. **Generate Password:** `passgen` → Generates a 16-character password with mixed characters
2. **Custom Length:** `passgen --length 24` → Generates password of specified length
3. **Character Options:** `passgen --no-symbols` → Exclude special characters
4. **Multiple Passwords:** `passgen --count 5` → Generate multiple passwords at once

---

## Non-Goals

**Will NOT build:**
- Password storage or vault functionality
- Password strength checker for existing passwords
- GUI or web interface
- Clipboard integration (user copies manually)
- Password history or logging
- Pronounceable or memorable password generation
- Pattern-based passwords (e.g., "word-number-word")

**Will NOT use:**
- External APIs or services
- Cryptographic libraries beyond stdlib
- Database or file storage
- Configuration files

---

## Technical Constraints

| | |
|---|---|
| **Language** | Python 3.11+ |
| **Framework** | None (stdlib only) |
| **Dependencies** | `secrets` module (stdlib) for cryptographic randomness |
| **Output** | Plain text to stdout |
| **Testing** | pytest, minimum 4 tests |
| **Code Style** | Black formatter, type hints |

---

## Success Criteria

- [ ] Default generation produces 16-char password with uppercase, lowercase, digits, symbols
- [ ] `--length` flag correctly changes password length (8-128 range)
- [ ] `--no-symbols` and `--no-numbers` flags work correctly
- [ ] `--count` generates multiple unique passwords
- [ ] Uses `secrets` module (not `random`) for security
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Basic Generator
**Goal:** Generate secure random passwords

**Tasks:**
1. Create `passgen.py` with argument parsing
2. Implement password generation using `secrets.choice()`
3. Default character set: uppercase + lowercase + digits + symbols
4. Default length: 16 characters

**Verification:**
```bash
python passgen.py
# Output: "K9#mP2$xL5nQ8@wR" (random)

python passgen.py
# Output: "different-each-time"
```

**Deliverables:** `passgen.py` with basic generation

---

### Phase 2: Customization Options
**Goal:** Add length and character set options

**Tasks:**
1. Add `--length` argument with validation (8-128)
2. Add `--no-symbols` flag to exclude `!@#$%^&*()_+-=`
3. Add `--no-numbers` flag to exclude digits
4. Validate that at least one character type remains

**Verification:**
```bash
python passgen.py --length 24
# Output: 24-character password

python passgen.py --no-symbols
# Output: Password without special characters

python passgen.py --no-symbols --no-numbers
# Output: Password with only letters
```

**Deliverables:** Updated `passgen.py` with all flags

---

### Phase 3: Multiple Passwords & Testing
**Goal:** Generate multiple passwords and add tests

**Tasks:**
1. Add `--count` argument to generate multiple passwords
2. Write pytest tests for each feature
3. Test edge cases (min/max length, invalid inputs)
4. Add helpful error messages

**Verification:**
```bash
python passgen.py --count 5
# Output:
# K9#mP2$xL5nQ8@wR
# H3@jN7*pM4kL9#qT
# ... (5 unique passwords)

python passgen.py --length 5
# Error: "Length must be between 8 and 128"

pytest
# All tests pass
```

**Deliverables:** Complete CLI with tests

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Use `secrets` module, not `random` | Before changing default length | Store passwords to file |
| Validate all user input | Before adding new character sets | Use weak randomness |
| Show clear error messages | Before changing output format | Add clipboard functionality |
| Include at least one char from each enabled set | | Log or print passwords to files |
