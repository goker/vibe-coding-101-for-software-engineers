# Week 4: The Explore → Plan → Code → Verify Cycle

> "Weeks of coding can save you hours of planning." — Anonymous (every engineer who learned the hard way)

**Instructor:** Goker Ezberci (gokerez@gmail.com)
**Duration:** 45 minutes lecture + 30 minutes lab
**Prerequisites:** Week 3 (deployed prototype with public URL)
**Tools:** Your AI coding tool of choice + Git

---

## Learning Objectives

By the end of this week, students will be able to:

1. **Apply** the Explore → Plan → Code → Verify (EPCV) cycle to add features to existing codebases (Bloom's: Apply)
2. **Create** instruction files (CLAUDE.md, .cursorrules, GEMINI.md) that provide persistent project context to AI tools (Bloom's: Create)
3. **Implement** a disciplined git workflow for AI-assisted development with atomic commits (Bloom's: Apply)
4. **Evaluate** AI-generated code against quality criteria before committing (Bloom's: Evaluate)

---

## Opening Quote

> "The best AI-assisted developers don't type faster. They think first, then let the AI execute. The EPCV cycle is how you systematize that thinking."
>
> — Adapted from Anthropic's Claude Code Best Practices (2026)

---

## Why This Week Matters

In Week 3, you shipped a prototype. It works. Users can access it. You feel great.

But here's the truth: that prototype is fragile. Try adding a new feature. Watch what happens:

- The AI doesn't know your project structure
- It generates code that conflicts with existing logic
- You spend 3 hours debugging what should have taken 20 minutes
- You break something that was working

**This is the 70% wall.** You got 70% of the way there on vibes. The remaining 30% requires discipline.

The EPCV cycle is that discipline.

---

## Part 1: The EPCV Cycle (20 minutes)

### The Problem: Undisciplined AI Development

Most students (and many professionals) use AI like this:

```
1. Think of feature
2. Tell AI: "Add feature X"
3. Accept whatever AI generates
4. Wonder why things break
5. Repeat, more frustrated each time
```

This is "pure vibe coding." It works for throwaway projects. It fails for anything you want to maintain.

### The Solution: EPCV

```
EXPLORE → PLAN → CODE → COMMIT (repeat)
   │         │        │        │
   │         │        │        └─ Verify + commit atomically
   │         │        └─ AI generates, you guide
   │         └─ Design before building
   └─ Understand before designing
```

Each phase has specific actions and tools. Let's break them down.

### Phase 1: EXPLORE

**Goal:** Understand the current state before changing anything.

**Actions:**
- Read relevant source files
- Understand existing architecture and patterns
- Check test coverage for the area you'll modify
- Review git history for recent changes
- Identify dependencies and integration points

**With AI:**

**Gemini CLI (FREE):**
```bash
gemini "Read the src/auth/ directory. Explain the authentication flow:
how does a user log in, where are tokens stored, and which endpoints
are protected?"
```

**Claude Code:**
```bash
claude "Explore the project structure. What patterns does this codebase
use? What testing framework? What's the database schema?"
```

**Key principle:** Never let the AI modify code during EXPLORE. You're gathering intelligence, not making changes. In Claude Code, you can use the `--plan` flag to enforce this.

**What EXPLORE prevents:**
- AI generating code that duplicates existing functionality
- Breaking patterns the codebase already follows
- Missing integration points that cause bugs
- Reinventing utilities that already exist

### Phase 2: PLAN

**Goal:** Design the solution before writing code.

**Actions:**
- Define what files need to change
- Specify the API/interface changes
- List edge cases to handle
- Decide what tests to write
- Identify potential breaking changes

**With AI:**

**Gemini CLI (FREE):**
```bash
gemini "I need to add a password reset feature. Based on the auth
module you just explored, plan the implementation:
1. What files need to change?
2. What new files are needed?
3. What's the API endpoint design?
4. What edge cases should we handle?
5. What tests should we write?

Don't write code yet — just the plan."
```

**Claude Code:**
```bash
claude "Plan the implementation of password reset. Consider the
existing auth patterns. Output a checklist of steps, not code."
```

**The critical instruction:** "Don't write code yet — just the plan."

Without this, AI jumps straight to implementation. You lose the ability to catch design errors before they become code errors.

**What PLAN prevents:**
- Over-engineering (AI loves adding features you didn't ask for)
- Architectural mistakes that are expensive to fix later
- Missing edge cases that cause production bugs
- Scope creep (the plan becomes your contract)

### Phase 3: CODE

**Goal:** Implement the plan with AI assistance.

**Actions:**
- Implement one step at a time from the plan
- Review each generated file before moving to the next
- Run the app after each significant change
- Keep changes small and focused

**With AI:**

**Gemini CLI (FREE):**
```bash
gemini "Now implement step 1 from the plan: Create the password
reset request endpoint in src/auth/routes.ts. Follow the existing
pattern from the login endpoint."
```

**Claude Code:**
```bash
claude "Implement the password reset token generation. Follow the
plan from the previous message. Match the existing code style."
```

**The golden rule:** One step at a time. Don't let AI implement the entire plan in one shot. Review each step before proceeding.

**Why incremental coding matters:**
- Easier to review (small diffs vs. massive changes)
- Easier to debug (you know which step introduced the bug)
- Easier to revert (undo one step, not everything)
- AI quality is higher (focused context vs. sprawling implementation)

### Phase 4: VERIFY (Commit)

**Goal:** Validate the code works, then commit atomically.

**Actions:**
- Run existing tests (nothing broke)
- Add new tests for the feature
- Review the diff manually
- Write a meaningful commit message
- Commit as an atomic unit

**With AI:**

**Gemini CLI (FREE):**
```bash
gemini "Write tests for the password reset endpoint. Test:
1. Valid email sends reset token
2. Invalid email returns 404
3. Expired token is rejected
4. Token can only be used once"
```

**Claude Code:**
```bash
claude "Review the changes I've made. Are there any issues? Then
write tests for the password reset feature."
```

**The commit discipline:**

```bash
# 1. Run existing tests first
npm test

# 2. Review the diff
git diff

# 3. Stage specific files (NOT git add .)
git add src/auth/reset.ts src/auth/routes.ts

# 4. Write a clear commit message
git commit -m "feat: add password reset request endpoint

- POST /auth/reset-request accepts email
- Generates time-limited token (1 hour)
- Sends token via email service
- Tests: valid/invalid email, expired token"
```

**What VERIFY prevents:**
- Shipping broken features
- Committing unrelated changes together
- Losing track of what changed and why
- Regression bugs from untested code

---

## The EPCV Cycle in Practice

```
Feature request: "Add password reset"
                     │
                     ▼
    ┌─────────── EXPLORE ────────────┐
    │ Read auth module               │
    │ Check existing password logic  │
    │ Review email service setup     │
    │ Understand token patterns      │
    └────────────┬───────────────────┘
                 │
                 ▼
    ┌─────────── PLAN ───────────────┐
    │ 1. Add reset-request endpoint  │
    │ 2. Generate time-limited token │
    │ 3. Send email with reset link  │
    │ 4. Add reset-confirm endpoint  │
    │ 5. Update password in DB       │
    │ 6. Write 4 tests               │
    └────────────┬───────────────────┘
                 │
                 ▼
    ┌─────────── CODE ───────────────┐
    │ Step 1: reset-request endpoint │
    │   → Review → Next step         │
    │ Step 2: token generation       │
    │   → Review → Next step         │
    │ Step 3: email integration      │
    │   → Review → Next step         │
    │ ... (one at a time)            │
    └────────────┬───────────────────┘
                 │
                 ▼
    ┌─────────── VERIFY ─────────────┐
    │ Run all tests (nothing broke)  │
    │ Add 4 new tests                │
    │ Review full diff               │
    │ Commit: "feat: password reset" │
    └────────────────────────────────┘
```

**Time comparison:**

| Approach | Time to Feature | Bugs Found Later |
|----------|----------------|-----------------|
| "Just vibe it" | 30 min | 3-5 bugs in production |
| EPCV cycle | 45 min | 0-1 bugs (caught in VERIFY) |

The 15 extra minutes saves 3+ hours of debugging.

---

## Part 2: Context Engineering with Instruction Files (15 minutes)

### Why Instruction Files Matter

Every time you start a new AI conversation, the AI knows nothing about your project. You can either:

**Option A:** Re-explain everything every time (painful, inconsistent)

**Option B:** Write it once in an instruction file (automatic, consistent)

Instruction files are **context engineering at the project level.** They persist across sessions and ensure every AI interaction starts with your project's rules.

### The Instruction File Landscape (February 2026)

| File | Tool(s) | Location | Purpose |
|------|---------|----------|---------|
| **CLAUDE.md** | Claude Code | Project root | Project context, commands, boundaries |
| **.cursorrules** | Cursor | Project root | IDE-level instructions, code style |
| **GEMINI.md** | Gemini CLI | Project root | Same as CLAUDE.md for Gemini |
| **.github/copilot-instructions.md** | GitHub Copilot | `.github/` folder | Copilot-specific context |
| **.windsurfrules** | Windsurf | Project root | Windsurf-specific rules |
| **.aider.conf.yml** | aider | Project root | aider configuration and context |

**Key insight:** These files use the same principles. Learn one, adapt to any.

### Anatomy of a Great Instruction File

```markdown
# Project: TaskFlow (a project management CLI)

## Commands
- `npm run dev` — Start development server (port 3000)
- `npm run build` — Create production build
- `npm test` — Run Vitest test suite
- `npm run lint` — Run ESLint
- `npm run db:migrate` — Run database migrations

## Project Structure
src/
├── api/          # Express routes (REST endpoints)
├── models/       # Prisma models (database schema)
├── services/     # Business logic (keep routes thin)
├── middleware/    # Auth, error handling, validation
├── utils/        # Shared helpers
└── __tests__/    # Mirror of src/ structure

## Architecture Decisions
- **API style:** REST with JSON responses
- **Database:** PostgreSQL via Prisma ORM
- **Auth:** JWT tokens, stored in httpOnly cookies
- **Error handling:** All errors go through errorHandler middleware
- **Validation:** Zod schemas for request bodies

## Code Style
- TypeScript strict mode (no `any` types)
- Prefer named exports over default exports
- Use async/await, never raw Promises
- Error messages must be user-friendly (no stack traces in responses)
- All database queries go through service layer (never in routes)

## Testing
- Framework: Vitest
- Location: `src/__tests__/` (mirrors src/ structure)
- Coverage target: 80%
- Every new endpoint needs: happy path + error case + edge case tests
- Use test factories for mock data (see `src/__tests__/factories/`)

## Boundaries
### Always
- Run `npm test` before committing
- Use Prisma migrations for schema changes
- Log errors with structured JSON (never console.log in production)
- Validate all user input with Zod schemas

### Never
- Never hardcode API keys or secrets
- Never use `any` type in TypeScript
- Never write SQL directly (use Prisma)
- Never skip error handling
- Never commit without running tests

### Ask First
- Before adding new dependencies (check if existing util works)
- Before changing database schema
- Before modifying auth middleware
- Before changing API response formats
```

### The Golden Context Pattern

The most effective instruction files follow this structure:

```
1. COMMANDS     — How to build, test, run, deploy
2. STRUCTURE    — Where things live in the codebase
3. DECISIONS    — Architectural choices already made
4. STYLE        — How code should look and feel
5. TESTING      — How to write and run tests
6. BOUNDARIES   — Always/Never/Ask First rules
```

**Why this order matters:** AI reads top-to-bottom with recency bias. Put the most critical rules (boundaries) at the end where attention is strongest. Put mechanical info (commands, structure) at the top where it's referenced frequently.

### Instruction Files Across Tools

**CLAUDE.md** (for Claude Code):
```markdown
# Project: MyApp
## Commands
- npm run dev
- npm test
## Boundaries
- Never use `any` type
- Always run tests before committing
```

**Same content for .cursorrules** (Cursor):
```
You are working on MyApp, a Next.js project.
When generating code:
- Use TypeScript strict mode
- Never use `any` type
- Always run tests before suggesting changes
Commands: npm run dev, npm test
```

**Same content for GEMINI.md** (Gemini CLI):
```markdown
# Project: MyApp
## Commands
- npm run dev
- npm test
## Rules
- Never use `any` type
- Always run tests before committing
```

**The content is 90% the same.** Create one, adapt format slightly for each tool.

---

## Part 3: Git Workflows for AI-Assisted Development (10 minutes)

### The "One Feature Per Conversation" Rule

AI tools track context within a conversation. When you mix features in one conversation, the AI gets confused:

```
BAD: One conversation, multiple features
├─ "Add login page"
├─ "Also add dark mode"
├─ "And fix that bug from yesterday"
└─ Context is polluted, AI generates confused code

GOOD: One conversation per feature
├─ Conversation 1: "Add login page" → commit → done
├─ Conversation 2: "Add dark mode" → commit → done
├─ Conversation 3: "Fix navigation bug" → commit → done
└─ Each conversation has clean, focused context
```

### Branch Strategy

```
main (protected — never push directly)
  │
  ├── feat/login-page        (one feature)
  ├── feat/dark-mode          (one feature)
  ├── fix/navigation-bug      (one bug fix)
  └── chore/update-deps       (maintenance)
```

**The workflow:**

```bash
# 1. Create branch
git checkout -b feat/password-reset

# 2. Run EPCV cycle (Explore → Plan → Code → Verify)

# 3. Commit with conventional message
git commit -m "feat: add password reset flow

- POST /auth/reset-request endpoint
- POST /auth/reset-confirm endpoint
- Email integration with SendGrid
- Tests: 4 new tests, all passing"

# 4. Push and create PR
git push -u origin feat/password-reset
```

### Commit Message Convention

```
type(scope): short description

Longer description if needed.

Types:
  feat:     New feature
  fix:      Bug fix
  test:     Adding tests
  refactor: Code change that doesn't fix bug or add feature
  docs:     Documentation changes
  chore:    Build, deps, CI changes
```

**Why this matters with AI:**
- Clear commit messages help AI understand what changed
- When you ask AI to explore git history, structured messages are readable
- Future-you can understand what past-you was thinking

### Code Review Checklist for AI-Generated Code

Before committing any AI-generated code, check:

| Check | Why | How |
|-------|-----|-----|
| **Does it compile/run?** | AI sometimes generates invalid syntax | `npm run build` |
| **Do existing tests pass?** | AI might break existing features | `npm test` |
| **Are there hardcoded values?** | AI loves magic numbers and strings | `grep -r "TODO\|FIXME\|hardcode"` |
| **Is error handling present?** | AI often skips error cases | Review catch blocks |
| **Are types correct?** | AI generates loose types | Check for `any` |
| **Is it over-engineered?** | AI adds features you didn't ask for | Compare to plan |
| **Are there security issues?** | SQL injection, XSS, exposed secrets | Review inputs/outputs |

---

## Lab Exercise

**Duration:** 30 minutes

### Task

Apply the EPCV cycle to add ONE new feature to your Week 3 prototype.

### Steps

**Step 1: Setup (5 min)**
- Create an instruction file for your project (CLAUDE.md, .cursorrules, or GEMINI.md)
- Create a feature branch: `git checkout -b feat/your-feature`

**Step 2: EXPLORE (5 min)**
- Ask your AI tool to explore your codebase
- Understand the current structure and patterns
- Prompt: "Explore this project. What patterns does it use? What's the file structure?"

**Step 3: PLAN (5 min)**
- Ask AI to plan the new feature WITHOUT writing code
- Prompt: "Plan how to add [feature]. List files to change, edge cases, and tests. Don't write code yet."

**Step 4: CODE (10 min)**
- Implement one step at a time from the plan
- Review each step before proceeding
- Prompt: "Implement step 1 from the plan. Follow existing patterns."

**Step 5: VERIFY (5 min)**
- Run tests: `npm test`
- Review diff: `git diff`
- Write a clear commit message
- Commit: `git add [specific files] && git commit -m "feat: ..."`

### Submit

- The instruction file you created
- Your git log showing the atomic commit
- A 100-word reflection: How did the EPCV cycle change your workflow?

---

## Resources

### Essential Reading
- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Cursor Rules Documentation](https://docs.cursor.com/context/rules-for-ai)
- [Gemini CLI Configuration](https://ai.google.dev/gemini-cli)

### Git Workflows
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow)
- [How to Write Good Commit Messages](https://cbea.ms/git-commit/)

### Context Engineering
- [Anthropic: CLAUDE.md Best Practices](https://docs.anthropic.com/en/docs/claude-code/overview)
- [MIT Tech Review: From Vibe Coding to Context Engineering](https://www.technologyreview.com/2025/11/05/1127477/from-vibe-coding-to-context-engineering-2025-in-software-development/)

---

**Instructor:** Goker Ezberci | gokerez@gmail.com
