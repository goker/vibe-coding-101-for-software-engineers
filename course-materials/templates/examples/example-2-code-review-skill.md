# PRD Example 2: Code Review Skill

> **Difficulty:** Intermediate | **Project Type:** Custom Skill | **Time:** 4-6 hours

---

# Code Review Skill

## Overview

| | |
|---|---|
| **What** | An AI skill that reviews code changes and provides structured feedback |
| **Who** | Solo developers who want quick feedback before committing |
| **Why** | Catch bugs, style issues, and security problems before they reach production |

## Features

1. **Review Staged Changes:** `/review` → Analyzes `git diff --staged` and provides feedback
2. **Structured Output:** Returns findings in categories: Bugs, Security, Style, Suggestions
3. **Severity Levels:** Each finding marked as Critical, Warning, or Info
4. **File-Specific Review:** `/review src/auth.py` → Reviews only specified file

## Non-Goals

**Will NOT build:**
- Auto-fix capabilities (suggestions only)
- Integration with GitHub/GitLab PR systems
- Custom rule configuration
- Historical review tracking
- Team collaboration features

**Will NOT use:**
- External linting APIs
- Database storage
- Web interfaces
- Third-party code analysis tools

## Technical Constraints

| | |
|---|---|
| **Skill Format** | `.claude/skills/code-review/SKILL.md` (or `.gemini/skills/`) |
| **References** | `references/review-checklist.md`, `references/security-patterns.md` |
| **Dependencies** | Git (assumes repo context) |
| **Testing** | Manual testing with sample diffs |

**Skill Structure:**
```
.claude/skills/code-review/
├── SKILL.md              # Main skill definition
└── references/
    ├── review-checklist.md   # What to check for
    └── security-patterns.md  # Common security issues
```

**Output Format:**
```markdown
## Code Review: [filename]

### Critical
- **Line 42:** SQL injection vulnerability in user input

### Warnings
- **Line 15:** Function exceeds 50 lines, consider splitting

### Suggestions
- **Line 8:** Consider using `const` instead of `let`
```

## Phases

### Phase 1: Basic Review Skill
**Tasks:** Create SKILL.md with triggers, basic review logic, and output format.
**Verify:**
```bash
# Stage a file with an obvious bug
git add test.py
claude  # or gemini
> /review
# Should output structured feedback
```

### Phase 2: Reference Documents
**Tasks:** Create `review-checklist.md` and `security-patterns.md` with common patterns.
**Verify:** Skill references these docs and catches patterns listed in them.

### Phase 3: File-Specific Review
**Tasks:** Add argument parsing for specific file paths.
**Verify:** `/review src/auth.py` only reviews that file, not all staged changes.

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Show file and line numbers | Before suggesting refactors | Auto-modify code |
| Categorize by severity | Before adding new categories | Access files outside repo |
| Explain why something is an issue | | Make up security vulnerabilities |
