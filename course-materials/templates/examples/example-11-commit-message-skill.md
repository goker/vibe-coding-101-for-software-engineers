# PRD Example 11: Git Commit Message Generator Skill

> **Difficulty:** Intermediate | **Project Type:** AI Skill | **Time:** 4-6 hours

---

## Overview

| | |
|---|---|
| **What** | A Claude Code / Gemini CLI skill that generates conventional commit messages from staged changes |
| **Who** | Developers who want consistent, well-formatted commit messages |
| **Why** | Saves time and ensures commit messages follow team conventions |

---

## Core Features (MVP)

1. **Generate Message:** Analyzes `git diff --staged` and generates a conventional commit message
2. **Multiple Options:** Provides 3 message options (concise, detailed, with scope)
3. **Apply Message:** User selects option, skill runs `git commit -m "message"`
4. **Customize Format:** Supports conventional commits format (feat:, fix:, docs:, etc.)

---

## Non-Goals

**Will NOT build:**
- Integration with issue trackers (Jira, GitHub Issues)
- Multi-commit squashing or rebasing
- Branch name parsing
- Automatic commit without user confirmation
- Commit message history or learning
- Support for non-conventional formats
- GUI or web interface

**Will NOT use:**
- External APIs beyond the AI tool's native capabilities
- Database or file storage
- Third-party git libraries

---

## Technical Constraints

| | |
|---|---|
| **Language** | TypeScript |
| **Framework** | Claude Code Skill format / Gemini CLI compatible |
| **Dependencies** | None — uses git CLI directly |
| **Output Format** | Conventional commits (type(scope): description) |
| **Testing** | Manual testing with sample diffs |
| **Compatibility** | Both Claude Code and Gemini CLI |

---

## Success Criteria

- [ ] Skill reads `git diff --staged` output correctly
- [ ] Generates 3 commit message options
- [ ] Messages follow conventional commits format
- [ ] User can select and execute commit
- [ ] Works in both Claude Code and Gemini CLI
- [ ] Handles empty staging area gracefully

---

## Implementation Phases

### Phase 1: Skill Structure & Diff Reading
**Goal:** Set up skill and read git diff

**Tasks:**
1. Create skill directory structure (`SKILL.md`, `prompts/`)
2. Define skill trigger phrases ("generate commit", "commit message")
3. Implement git diff reading via bash
4. Parse diff to understand changes

**Verification:**
```bash
# In Claude Code:
claude "generate commit message"
# Skill activates and reads staged changes

# Shows: "Analyzing 3 files changed..."
```

**Deliverables:** `SKILL.md` with basic structure

---

### Phase 2: Message Generation
**Goal:** Generate conventional commit options

**Tasks:**
1. Create prompt for analyzing diff and generating messages
2. Generate 3 options: concise, detailed, with scope
3. Format as conventional commits (feat:, fix:, etc.)
4. Present options to user for selection

**Verification:**
```bash
# After staging a new feature file:
claude "generate commit"

# Output:
# Based on your changes, here are 3 commit message options:
#
# 1. feat: add user authentication module
# 2. feat(auth): add user authentication with JWT support
# 3. feat(auth): implement user authentication
#
#    - Add login/logout endpoints
#    - Implement JWT token generation
#    - Add middleware for protected routes
#
# Which option would you like to use? (1/2/3)
```

**Deliverables:** Message generation logic

---

### Phase 3: Commit Execution & Edge Cases
**Goal:** Execute commit and handle errors

**Tasks:**
1. Execute `git commit -m "selected message"` on user selection
2. Handle empty staging area
3. Handle very large diffs (summarize)
4. Add confirmation before committing

**Verification:**
```bash
# User selects option 2:
claude "use option 2"
# Output: "Committed: feat(auth): add user authentication with JWT support"

# With empty staging area:
claude "generate commit"
# Output: "No staged changes found. Use 'git add' first."
```

**Deliverables:** Complete skill with error handling

---

## Skill File Structure

```
.claude/skills/commit-message/
├── SKILL.md           # Skill definition and triggers
└── prompts/
    └── analyze-diff.md  # Prompt for diff analysis
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Show diff summary before generating | Before committing | Commit without user confirmation |
| Provide multiple message options | Before amending previous commit | Push to remote |
| Follow conventional commits format | | Generate messages for unstaged changes |
| Confirm the selected message | | Include sensitive data in messages |
