# Week 5: Testing AI-Generated Code

> "Code that compiles is not code that works. Code that works once is not code that works always. AI gives you the first part for free — the rest is on you."

---

## Quick Pulse Check (Start)

Before we begin, rate yourself honestly (1-5):

| Question | 1 | 2 | 3 | 4 | 5 |
|----------|---|---|---|---|---|
| How many tests does your Week 3 project have? | Zero | 1-2 | 3-5 | 6-10 | 10+ |
| How confident are you that your deployed code has no bugs? | Not at all | Slightly | Moderately | Very | Completely |
| Could you explain what "test coverage" means? | No idea | Vaguely | Roughly | Well | Teach it |
| Have you ever written a test BEFORE writing the code? | Never | Once | A few times | Often | Always |

Keep these numbers — we'll revisit at the end.

---

## Learning Objectives

By the end of this week, you will be able to:

1. **Explain why AI-generated code requires more testing**, not less, and identify the three categories of AI-specific bugs
2. **Apply TDD (Test-Driven Development) with AI** — write the test first, let AI implement
3. **Build a test suite** with unit tests, edge case tests, and integration tests using free tools (pytest, Jest, Vitest)
4. **Design and run evals** — systematic measurements that test the AI itself, not just the code it produces
5. **Connect testing to your PRD** — turn Success Criteria into automated tests and eval checklists
6. **Avoid common AI testing pitfalls** — tests that pass by definition, over-mocking, and eval theater

---

## The False Confidence Problem

AI-generated code has a dangerous property: **it looks correct**. It compiles, it runs, and the variable names are sensible. This creates false confidence.

Here's what actually happens when you let AI write code without tests:

### Real Examples of AI Bugs

**1. The Off-By-One Error**

You asked AI to paginate results, 10 per page:

```python
# AI generated this — looks correct
def get_page(items, page_number):
    start = page_number * 10
    end = start + 10
    return items[start:end]
```

Page 1 starts at index 10 instead of 0. Users see nothing on the first page. The AI assumed zero-indexed page numbers but your UI sends 1-indexed.

**2. The Hallucinated API**

```javascript
// AI generated this — confident, clean code
import { formatCurrency } from 'intl-currency-format';

const price = formatCurrency(29.99, 'USD');
```

The package `intl-currency-format` doesn't exist. AI invented it based on patterns it's seen. `npm install` fails in production.

**3. The Happy-Path-Only Logic**

```python
# AI generated this — works for the demo
def divide_tasks(tasks, num_groups):
    group_size = len(tasks) // num_groups
    return [tasks[i:i+group_size] for i in range(0, len(tasks), group_size)]
```

What happens when `num_groups` is 0? `ZeroDivisionError`. What about when `tasks` is empty? What about when `num_groups` is larger than the number of tasks? AI tested none of these.

**4. The Silent Data Loss**

```python
# AI generated this — no errors, no warnings
def save_config(config, filepath):
    with open(filepath, 'w') as f:
        json.dump(config, f)
```

If `config` contains a `datetime` object, `json.dump` raises `TypeError`. But AI didn't add error handling — your users' configs silently fail to save, and the file might be truncated (partially written).

### The Pattern

Every one of these bugs shares three traits:

1. **The code looks professional** — clean formatting, good variable names
2. **It works for the obvious case** — the demo scenario passes
3. **It fails silently** — no error messages, just wrong behavior

This is why AI code needs **more** testing than human code, not less.

---

## Why AI Code Needs MORE Testing, Not Less

### The Paradox

| Without AI | With AI |
|------------|---------|
| Write code slowly | Generate code in seconds |
| Naturally think about edge cases while writing | Skip thinking — it "just works" |
| Small surface area — fewer functions | Large surface area — many generated functions |
| Bugs are in code you understand | Bugs are in code you skimmed |

### The Three Categories of AI Bugs

**Category 1: Confident-But-Wrong**

AI writes syntactically correct code that produces the wrong output. The code compiles, tests at first glance "look fine," but the logic is subtly flawed.

- Wrong sort order (ascending vs. descending)
- Incorrect boundary conditions
- Off-by-one errors in loops and pagination
- Wrong operator precedence

**Category 2: Plausible-But-Hallucinated**

AI references things that don't exist — packages, API endpoints, function signatures, configuration options.

- Non-existent npm/pip packages
- Deprecated API methods (AI trained on old data)
- Made-up configuration flags
- Libraries with wrong version APIs

**Category 3: Works-For-Demo-Only**

AI optimizes for the happy path because that's what training data emphasizes. Edge cases, error handling, and boundary conditions are afterthoughts.

- No null/undefined checks
- No empty input handling
- No error recovery
- No timeout handling for network calls
- No validation of user input

---

## The Testing Pyramid for AI-Assisted Development

The classic testing pyramid still applies, but with AI-specific additions at every layer:

```
            /\
           /  \         E2E Tests
          / E2E\        Does the deployed app work?
         /------\
        /        \      Integration Tests
       / Integr.  \     Do components work together?
      /------------\
     /              \   Unit Tests
    /    Unit Tests   \  Does each function work correctly?
   /--------------------\
  /                      \  AI-Specific Tests
 /  Hallucination + Edge  \ Do dependencies exist? Are edge cases covered?
/==========================\
```

### Layer 1: AI-Specific Tests (Foundation)

These tests are unique to AI-generated code:

**Hallucination Tests** — Verify that imported packages and APIs actually exist:

```python
# test_dependencies.py
import importlib

REQUIRED_PACKAGES = ['json', 'os', 'argparse']  # from your PRD constraints

def test_all_dependencies_exist():
    """Verify AI didn't hallucinate any packages."""
    for pkg in REQUIRED_PACKAGES:
        assert importlib.import_module(pkg), f"Package '{pkg}' not found"

def test_no_external_dependencies():
    """Verify AI didn't add packages outside the PRD."""
    import pkg_resources
    installed = {p.key for p in pkg_resources.working_set}
    # Should only have stdlib — no extras
    forbidden = installed - {'pip', 'setuptools', 'wheel'}
    # Adjust based on your PRD constraints
```

**Edge Case Tests** — Test the cases AI skips:

```python
def test_empty_input():
    assert get_tasks([]) == []

def test_none_input():
    with pytest.raises(TypeError):
        get_tasks(None)

def test_single_item():
    assert len(get_tasks(["one"])) == 1

def test_very_large_input():
    tasks = [f"task-{i}" for i in range(10000)]
    result = get_tasks(tasks)
    assert len(result) == 10000
```

**Boundary Tests** — Test limits, zeros, negatives:

```python
def test_page_zero():
    assert get_page(items, 0) == items[:10]  # or raises ValueError?

def test_page_negative():
    with pytest.raises(ValueError):
        get_page(items, -1)

def test_page_beyond_end():
    assert get_page(items, 9999) == []
```

### Layer 2: Unit Tests

Standard unit tests, but with AI-specific focus areas:

```python
# test_todo.py
import pytest
from todo import add_task, list_tasks, complete_task

class TestAddTask:
    def test_add_creates_task(self):
        result = add_task("Buy groceries")
        assert result['text'] == "Buy groceries"
        assert result['completed'] is False
        assert 'id' in result

    def test_add_assigns_unique_ids(self):
        t1 = add_task("Task 1")
        t2 = add_task("Task 2")
        assert t1['id'] != t2['id']

    def test_add_empty_text_raises(self):
        with pytest.raises(ValueError):
            add_task("")

    def test_add_whitespace_only_raises(self):
        with pytest.raises(ValueError):
            add_task("   ")
```

### Layer 3: Integration Tests

Test that components work together correctly:

```python
def test_add_then_list():
    """Integration: add a task, then verify it appears in the list."""
    add_task("Integration test")
    tasks = list_tasks()
    assert any(t['text'] == "Integration test" for t in tasks)

def test_add_complete_then_list():
    """Integration: full workflow — add, complete, verify status."""
    task = add_task("Complete me")
    complete_task(task['id'])
    tasks = list_tasks()
    target = next(t for t in tasks if t['id'] == task['id'])
    assert target['completed'] is True
```

### Layer 4: E2E Tests

Test the actual user-facing behavior:

```bash
# test_e2e.sh — run these after deployment
python todo.py add "E2E Test Task"      # Should print task ID
python todo.py list                      # Should show the task
python todo.py done 1                    # Should mark complete
python todo.py list                      # Should show [x]
```

---

## TDD with AI: The Red-Green-Refactor Loop

**TDD (Test-Driven Development)** is the most powerful workflow for AI-assisted coding. Here's why: when you write the test first, you become the **specification author** and AI becomes the **implementation partner**.

### The Workflow

```
1. RED:    Write a failing test (human writes this)
2. GREEN:  Ask AI to make it pass (AI writes the implementation)
3. REFACTOR: Ask AI to clean up (AI refactors, tests keep passing)
```

### Step-by-Step Example

**Step 1: RED — You write the test**

```python
# test_todo.py — you write this FIRST
def test_add_task_returns_dict_with_id():
    result = add_task("Buy milk")
    assert isinstance(result, dict)
    assert 'id' in result
    assert 'text' in result
    assert result['text'] == "Buy milk"
    assert result['completed'] is False
```

Run it: `pytest test_todo.py` — it fails (RED). Good.

**Step 2: GREEN — Tell AI to make it pass**

Prompt to AI:
```
I have this failing test. Implement the add_task function
to make it pass. Use only Python stdlib. Store tasks in
a JSON file at ~/.todo/tasks.json.
```

AI implements. Run `pytest` again — it passes (GREEN).

**Step 3: REFACTOR — Ask AI to improve**

```
The test passes. Now refactor add_task for better error
handling. Keep all tests passing.
```

### Why TDD + AI Works

| Traditional Coding | TDD + AI |
|--------------------|----------|
| Think about implementation | Think about behavior |
| Write code, then test | Write test, then generate code |
| AI might go off-track | Tests keep AI on track |
| Review AI's design choices | AI follows YOUR design |
| Hours debugging | Minutes verifying |

> **The key insight: tests are a form of specification.** When you write tests first, you're telling AI exactly what "correct" means. This is more precise than any natural language prompt.

---

## Evals: Testing the AI Itself

Tests verify your **code**. Evals verify your **AI workflow**. Both are essential.

### What Are Evals?

An eval (evaluation) is a systematic measurement of AI output quality. In traditional ML, evals measure model accuracy. In vibe coding, evals measure whether your AI coding setup consistently produces good results.

Think of it this way:

| | Tests | Evals |
|---|---|---|
| **What they check** | Does this code work? | Does my AI workflow produce working code? |
| **When they run** | After code is written | After AI generates code |
| **What fails** | A specific function | Your prompt, PRD, or instruction file |
| **What you fix** | The code | Your AI configuration |

### Why Evals Matter

You changed one line in your CLAUDE.md. Did your AI outputs get better or worse? Without evals, you're guessing.

You tweaked your PRD's Non-Goals section. Does AI now respect the boundaries? Without evals, you're hoping.

You switched from Cursor to Gemini CLI. Does the new tool produce equivalent quality? Without evals, you're assuming.

### Three Levels of Evals

#### Level 1: Output Evals

*Does the generated code meet the spec?*

Run your AI on the same PRD multiple times and check the output:

```bash
# Simple eval script
#!/bin/bash
PASS=0
FAIL=0

for i in $(seq 1 5); do
    echo "=== Run $i ==="

    # Generate code from PRD (adjust for your tool)
    # claude-code "Implement the todo app from PRD.md" --output run_$i/

    # Check: Does it have the right file structure?
    [ -f "run_$i/todo.py" ] && ((PASS++)) || ((FAIL++))

    # Check: Do tests pass?
    (
      cd "run_$i" || exit 1
      pytest -q
    ) && ((PASS++)) || ((FAIL++))

    # Check: No external dependencies?
    ! grep -Eq '^(fastapi|requests|pandas|numpy)=' "run_$i/requirements.txt" \
      && ((PASS++)) || ((FAIL++))
done

echo "Pass rate: $PASS / $((PASS + FAIL))"
```

#### Level 2: Behavioral Evals

*Does AI follow your rules?*

Check whether the AI respects your Agent Rules, Non-Goals, and instruction files:

```python
# eval_behavioral.py
"""
Eval: Does AI respect Non-Goals from the PRD?
"""
import os
import ast

def check_no_external_imports(filepath):
    """Verify AI didn't import packages outside PRD constraints."""
    with open(filepath) as f:
        tree = ast.parse(f.read())

    stdlib = {'os', 'sys', 'json', 'argparse', 'pathlib', 'datetime'}
    violations = []

    for node in ast.walk(tree):
        if isinstance(node, ast.Import):
            for alias in node.names:
                if alias.name.split('.')[0] not in stdlib:
                    violations.append(alias.name)
        elif isinstance(node, ast.ImportFrom):
            if node.module and node.module.split('.')[0] not in stdlib:
                violations.append(node.module)

    return violations

def check_no_database_usage(directory):
    """Verify AI didn't add a database (Non-Goal)."""
    db_indicators = ['sqlite', 'postgresql', 'mysql', 'sqlalchemy',
                     'CREATE TABLE', 'SELECT *', '.db']
    violations = []
    for root, dirs, files in os.walk(directory):
        for f in files:
            if f.endswith('.py'):
                content = open(os.path.join(root, f)).read().lower()
                for indicator in db_indicators:
                    if indicator.lower() in content:
                        violations.append(f"{f}: contains '{indicator}'")
    return violations

# Run evals
import_violations = check_no_external_imports("todo.py")
db_violations = check_no_database_usage(".")

print(f"Import violations: {len(import_violations)}")
print(f"Database violations: {len(db_violations)}")

if import_violations:
    print(f"  Forbidden imports: {import_violations}")
if db_violations:
    print(f"  Database usage: {db_violations}")
```

#### Level 3: Regression Evals

*Did my changes make things better or worse?*

Track eval scores over time as you modify your PRD, instruction files, or prompts:

```python
# eval_regression.py
"""
Track eval scores over time.
Save results to evals/history.json.
"""
import json
import os
from datetime import datetime

def run_eval_suite(project_dir):
    """Run all evals and return a score dict."""
    scores = {
        "file_structure": check_file_structure(project_dir),
        "tests_pass": check_tests_pass(project_dir),
        "no_external_deps": check_no_external_deps(project_dir),
        "no_database": check_no_database(project_dir),
        "has_error_handling": check_error_handling(project_dir),
        "respects_non_goals": check_non_goals(project_dir),
    }
    scores["total"] = sum(scores.values()) / len(scores)
    return scores

def save_eval_result(scores, change_description):
    """Append eval result to history."""
    history_file = "evals/history.json"
    history = json.load(open(history_file)) if os.path.exists(history_file) else []
    history.append({
        "timestamp": datetime.now().isoformat(),
        "change": change_description,
        "scores": scores
    })
    json.dump(history, open(history_file, 'w'), indent=2)

# Example usage:
# scores = run_eval_suite("./todo-app")
# save_eval_result(scores, "Added error handling to Agent Rules")
# Compare: did scores improve?
```

### The Eval Checklist

For your Week 3 project, score each AI generation against this 5-point checklist:

| # | Check | Pass/Fail |
|---|-------|-----------|
| 1 | Correct file structure (matches PRD phases) | |
| 2 | All tests pass (your test suite) | |
| 3 | No forbidden dependencies (matches PRD constraints) | |
| 4 | No Non-Goal violations (no auth, no database, etc.) | |
| 5 | Agent Rules respected (no features added beyond PRD) | |

**Your pass rate = (checks passed) / (total checks across all runs)**

A good target: **80%+ pass rate across 3-5 runs** of the same PRD.

---

## Testing Tools (All Free)

### Python

| Tool | What It Does | Install |
|------|-------------|---------|
| **pytest** | Test runner + assertions | `pip install pytest` |
| **coverage.py** | Line-by-line coverage reports | `pip install coverage` |
| **pytest-cov** | Coverage integrated with pytest | `pip install pytest-cov` |

```bash
# Run tests
pytest

# Run tests with coverage
pytest --cov=. --cov-report=term-missing

# Generate HTML coverage report
pytest --cov=. --cov-report=html
open htmlcov/index.html
```

### JavaScript / TypeScript

| Tool | What It Does | Install |
|------|-------------|---------|
| **Vitest** | Fast test runner (Vite-native) | `npm install -D vitest` |
| **Jest** | Facebook's test framework | `npm install -D jest` |
| **c8** | V8 native coverage | `npm install -D c8` |

```bash
# Run tests (Vitest)
npx vitest run

# Run with coverage
npx vitest run --coverage

# Watch mode (re-runs on file change)
npx vitest
```

### Asking AI to Write Tests

You can ask AI to write tests — but you **must** review them:

```
Write pytest tests for todo.py. Include:
- Unit tests for each function
- Edge cases: empty input, None, very large input
- Integration test: add → list → complete → verify
- Boundary tests: page 0, negative page, page past end

Do NOT mock the file system — use a temp directory.
Do NOT write tests that pass by definition.
Every test must be able to FAIL.
```

> **Rule: If you can't explain why a test would fail, it's not a real test.**

---

## Common AI Testing Pitfalls

### Pitfall 1: Tests That Pass By Definition

AI writes a test that literally cannot fail:

```python
# BAD — this always passes
def test_add_task():
    result = add_task("test")
    assert result is not None  # Too weak — what IS the result?
```

```python
# GOOD — this tests actual behavior
def test_add_task():
    result = add_task("test")
    assert result['text'] == "test"
    assert result['completed'] is False
    assert isinstance(result['id'], int)
```

### Pitfall 2: Over-Mocking

AI loves to mock everything, which means your test doesn't test real behavior:

```python
# BAD — mocks the very thing you're testing
@mock.patch('todo.save_to_file')
@mock.patch('todo.load_from_file')
def test_add_task(mock_load, mock_save):
    mock_load.return_value = []
    result = add_task("test")
    mock_save.assert_called_once()  # Tests that code calls save, not that save works
```

```python
# GOOD — uses real file system with temp directory
def test_add_task(tmp_path):
    config_dir = tmp_path / ".todo"
    result = add_task("test", config_dir=config_dir)
    # Verify the file was actually written
    saved = json.loads((config_dir / "tasks.json").read_text())
    assert len(saved) == 1
    assert saved[0]['text'] == "test"
```

### Pitfall 3: Testing Implementation, Not Behavior

```python
# BAD — tests HOW it works (implementation detail)
def test_add_task_uses_uuid():
    result = add_task("test")
    import uuid
    uuid.UUID(result['id'])  # Fails if we switch to auto-increment IDs

# GOOD — tests WHAT it does (behavior)
def test_add_task_has_unique_id():
    t1 = add_task("test 1")
    t2 = add_task("test 2")
    assert t1['id'] != t2['id']  # Works regardless of ID strategy
```

### Pitfall 4: Eval Theater

AI generates impressive-looking eval metrics that don't catch real issues:

```python
# BAD — "100% eval pass rate" that means nothing
def eval_output(code):
    return {
        "has_functions": "def " in code,      # True for any Python file
        "has_imports": "import " in code,      # True for any real code
        "not_empty": len(code) > 0,            # Obviously true
        "has_comments": "#" in code,            # Trivially true
    }
```

```python
# GOOD — evals that catch real AI failures
def eval_output(code, prd):
    return {
        "tests_pass": run_tests(code),                    # Actually runs tests
        "no_forbidden_imports": check_imports(code, prd),  # Checks against PRD
        "handles_empty_input": test_empty_input(code),     # Tests real edge case
        "respects_non_goals": check_non_goals(code, prd),  # Checks PRD compliance
    }
```

---

## Lab Exercise

### Part A: Add a Test Suite to Your Week 3 Project

**Requirements:**

1. **Minimum 10 tests** covering:
   - At least 4 unit tests (one per core function)
   - At least 3 edge case tests (empty input, None, boundary values)
   - At least 2 integration tests (multi-step workflows)
   - At least 1 hallucination test (verify dependencies exist)

2. **80% code coverage target** — run `pytest --cov` and check the report

3. **All tests must be able to fail** — for each test, describe one change to the source code that would make it fail

### Part B: Build a Mini Eval

**Requirements:**

1. Run your AI tool on your PRD **3 separate times** (fresh conversation each time)
2. Score each output against the 5-point eval checklist:
   - Correct file structure?
   - All tests pass?
   - No forbidden dependencies?
   - No Non-Goal violations?
   - Agent Rules respected?
3. Calculate your **pass rate** (checks passed / total checks)
4. Write a one-paragraph reflection: What did the AI get wrong? What would you change in your PRD to improve the pass rate?

### Submission

Push to your project repository:
- `tests/` directory with your test suite
- `evals/` directory with your eval script and results
- Updated README with a "Testing" section showing how to run tests
- Brief write-up of your eval results and pass rate

---

## Quick Pulse Check (End)

Rate yourself again (1-5):

| Question | 1 | 2 | 3 | 4 | 5 |
|----------|---|---|---|---|---|
| Can you explain the three categories of AI bugs? | | | | | |
| Could you write a test BEFORE writing the code? | | | | | |
| Do you know the difference between tests and evals? | | | | | |
| Can you set up a test suite with 80% coverage? | | | | | |

Compare with your start-of-week scores. What changed?

---

## Resources

### Testing Frameworks
- [pytest Documentation](https://docs.pytest.org/)
- [Vitest Documentation](https://vitest.dev/)
- [Jest Documentation](https://jestjs.io/)

### Coverage Tools
- [coverage.py](https://coverage.readthedocs.io/)
- [c8 (V8 Coverage)](https://github.com/bcoe/c8)

### Further Reading
- [Martin Fowler: Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html)
- [Kent Beck: Test-Driven Development](https://www.oreilly.com/library/view/test-driven-development/0321146530/)
- [Anthropic: Evaluating AI Systems](https://docs.anthropic.com/en/docs/build-with-claude/develop-tests)

### AI-Specific Testing
- [OWASP AI Security Guide](https://owasp.org/www-project-ai-security-and-privacy-guide/)
- [Google: Testing AI-Generated Code](https://testing.googleblog.com/)

---

*Instructor: Goker Ezberci | gokerez@gmail.com*
*Vibe Coding 101: From Idea to Shipped Product — Week 5*
