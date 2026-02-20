# PRD: Greeting CLI Tool

> This is the sample PRD for Week 1's lab exercise.
> Students should use this as a reference when writing their own PRDs.

## 1. Overview

### What
A command-line tool that greets users by name with customizable greeting styles.

### Who
Developers learning PRD-first AI-assisted development.

### Why
Practice the disciplined vibe coding workflow: PRD → Plan → Code → Verify.

## 2. Core Features (MVP)

1. **Basic Greeting:** Accept a name as a command line argument and print "Hello, [name]!"
2. **Formal Mode:** `--formal` flag changes greeting to "Good day, [name]."
3. **Time Mode:** `--time` flag appends current time to the greeting
4. **Help Text:** `--help` displays usage information

## 3. Non-Goals

### Features We Will NOT Build
- [ ] GUI or web interface
- [ ] Database storage of greetings
- [ ] User authentication
- [ ] Multi-language support
- [ ] Greeting history

### Technologies We Will NOT Use
- [ ] External HTTP libraries
- [ ] Database packages
- [ ] Web frameworks

### Scope Boundaries
- [ ] This is a single-file Python script
- [ ] No configuration files
- [ ] No persistent state

## 4. Technical Constraints

### Language & Framework
- **Primary Language:** Python 3.11+
- **Framework:** None (stdlib only)
- **Package Manager:** pip (for dev dependencies only)

### Dependencies
- **Allowed:** Python standard library only (argparse, datetime)
- **Prohibited:** requests, flask, django, any web/db packages
- **Prefer:** argparse for CLI argument parsing

### Code Standards
- **Style Guide:** PEP 8
- **Type Checking:** Type hints recommended but not required
- **Testing Framework:** pytest or unittest
- **Minimum Coverage:** 3 test cases minimum

## 5. Success Criteria

### Functional Tests
- [ ] `python main.py Alice` outputs "Hello, Alice!"
- [ ] `python main.py Bob --formal` outputs "Good day, Bob."
- [ ] `python main.py Carol --time` outputs "Hello, Carol! The time is [HH:MM]."
- [ ] `python main.py --help` shows usage information

### Quality Gates
- [ ] All unit tests pass
- [ ] Code runs without errors
- [ ] Help text is clear and accurate

## 6. Implementation Phases

### Phase 1: Basic Greeting

**Goal:** Get the simplest version working.

**Tasks:**
1. Create `main.py` file
2. Use argparse to accept a name argument
3. Print "Hello, [name]!"

**Verification:**
```bash
python main.py Alice
# Expected output: Hello, Alice!
```

**Dependencies:** None

**Deliverables:**
- [ ] `main.py` with basic functionality

---

### Phase 2: Add Flags

**Goal:** Implement --formal and --time options.

**Tasks:**
1. Add `--formal` flag to argparse
2. Add `--time` flag to argparse
3. Implement conditional greeting logic
4. Import datetime for time functionality

**Verification:**
```bash
python main.py Alice --formal
# Expected: Good day, Alice.

python main.py Alice --time
# Expected: Hello, Alice! The time is 14:30.

python main.py Alice --formal --time
# Expected: Good day, Alice. The time is 14:30.
```

**Dependencies:** Phase 1 complete

**Deliverables:**
- [ ] Updated `main.py` with flag support

---

### Phase 3: Tests

**Goal:** Ensure code quality with automated tests.

**Tasks:**
1. Create `test_main.py`
2. Write test for basic greeting
3. Write test for formal mode
4. Write test for time mode

**Verification:**
```bash
python -m pytest test_main.py -v
# All tests should pass
```

**Dependencies:** Phase 2 complete

**Deliverables:**
- [ ] `test_main.py` with passing tests

## 7. Agent Boundaries

### Always Do
- [ ] Run the script after changes to verify it works
- [ ] Follow PEP 8 style guidelines
- [ ] Include docstrings for functions

### Ask First
- [ ] Before adding any external packages
- [ ] Before changing the output format

### Never Do
- [ ] Install packages outside stdlib
- [ ] Create additional files beyond main.py and test_main.py
- [ ] Add features not in this PRD

## 8. Reference Documentation

- [Python argparse documentation](https://docs.python.org/3/library/argparse.html)
- [Python datetime documentation](https://docs.python.org/3/library/datetime.html)
- [pytest documentation](https://docs.pytest.org/)
