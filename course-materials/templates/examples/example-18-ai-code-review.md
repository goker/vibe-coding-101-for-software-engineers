# PRD Example 18: AI Code Review Bot Skill

> **Difficulty:** Advanced | **Project Type:** AI Skill | **Time:** 1-2 weeks

---

## Overview

| | |
|---|---|
| **What** | A Claude Code / Gemini CLI skill that reviews code changes and provides actionable feedback |
| **Who** | Development teams who want consistent, automated code review assistance |
| **Why** | Provides instant feedback on PRs while human reviewers focus on architecture and design |

---

## Core Features (MVP)

1. **Review PR:** Analyze git diff and provide structured feedback on code quality
2. **Check Categories:** Security, performance, readability, best practices, bugs
3. **Severity Levels:** Critical, Warning, Suggestion, Praise
4. **Inline Comments:** Format feedback as file:line references
5. **Summary Report:** Overall assessment with action items

---

## Non-Goals

**Will NOT build:**
- GitHub/GitLab integration (CLI only)
- Auto-fixing issues
- Custom rule configuration
- Historical tracking of issues
- Team-specific coding standards
- Language-specific linting (use existing linters)
- Test coverage analysis
- Documentation review

**Will NOT use:**
- External APIs beyond AI tool capabilities
- Database or file storage
- GitHub/GitLab APIs
- Static analysis tools (defer to existing)

---

## Technical Constraints

| | |
|---|---|
| **Language** | TypeScript |
| **Framework** | Claude Code Skill / Gemini CLI compatible |
| **Dependencies** | None — uses git CLI |
| **Input** | `git diff` or `git diff HEAD~N` output |
| **Output** | Markdown formatted review |
| **Testing** | Manual testing with sample diffs |

---

## Success Criteria

- [ ] Skill triggers on "review code", "review PR", "check changes"
- [ ] Analyzes staged changes or specified commit range
- [ ] Categorizes issues (security, performance, readability, bugs)
- [ ] Assigns severity (critical, warning, suggestion)
- [ ] Provides file:line references for each issue
- [ ] Generates actionable summary
- [ ] Works in both Claude Code and Gemini CLI

---

## Implementation Phases

### Phase 1: Skill Structure & Diff Parsing
**Goal:** Set up skill and parse git diff

**Tasks:**
1. Create skill directory structure
2. Define trigger phrases
3. Implement diff parsing (file, line numbers, changes)
4. Handle large diffs (truncate with summary)

**Verification:**
```bash
claude "review my code changes"
# Skill activates
# Shows: "Analyzing changes in 5 files..."
```

**Deliverables:** `SKILL.md` and diff parsing logic

---

### Phase 2: Review Analysis Prompts
**Goal:** Create prompts for code analysis

**Tasks:**
1. Create analysis prompt for security issues
2. Create analysis prompt for performance issues
3. Create analysis prompt for readability/best practices
4. Create analysis prompt for potential bugs
5. Combine into structured output

**Verification:**
```bash
# After code review:
# Output:
# ## Code Review Summary
#
# ### Critical Issues (1)
# - **Security**: SQL injection vulnerability
#   `src/db.ts:45` - User input directly in query
#   → Use parameterized queries
#
# ### Warnings (2)
# - **Performance**: N+1 query pattern
#   `src/users.ts:23` - Query in loop
#   → Batch fetch with WHERE IN
```

**Deliverables:** Analysis prompts

---

### Phase 3: Output Formatting
**Goal:** Format review as actionable report

**Tasks:**
1. Group issues by severity
2. Format with file:line references
3. Add suggested fixes for each issue
4. Generate summary statistics
5. Add praise for good patterns

**Verification:**
```markdown
## Code Review: feature/user-auth

### Summary
- 1 Critical issue (must fix)
- 2 Warnings (should fix)
- 3 Suggestions (nice to have)
- 2 Good practices noticed ✓

### Critical
🔴 **Security: Hardcoded secret**
`src/auth.ts:12`
```ts
const SECRET = "abc123";  // ← Hardcoded!
```
**Fix:** Use environment variable

### Good Practices ✓
✅ Proper error handling in `src/api.ts`
✅ Type safety with generics in `src/utils.ts`
```

**Deliverables:** Complete skill with formatted output

---

### Phase 4: Edge Cases & Polish
**Goal:** Handle edge cases and improve UX

**Tasks:**
1. Handle empty diff (no changes)
2. Handle binary files (skip with note)
3. Limit review to relevant file types
4. Add "focus on X" mode (e.g., "review security only")

**Verification:**
```bash
claude "review security only"
# Shows only security-related feedback

claude "review code" (with no changes)
# Shows: "No changes to review. Stage changes with git add."
```

**Deliverables:** Production-ready skill

---

## Output Format

```markdown
## Code Review: [branch-name]

**Files Changed:** 5 | **Lines:** +120 / -45

### 🔴 Critical (1)
[issue with file:line and fix]

### 🟡 Warnings (2)
[issues with file:line and fix]

### 💡 Suggestions (3)
[improvements with file:line]

### ✅ Good Practices
[positive feedback]

---
**Recommended Actions:**
1. Fix critical security issue before merge
2. Consider performance optimization in user query
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Show file:line for every issue | Before reviewing files outside diff | Auto-commit or push changes |
| Prioritize security issues first | Before suggesting major refactors | Access external services |
| Include suggested fix for each issue | Before changing review scope | Modify any files |
| Be specific and actionable | | Make assumptions about business logic |
