# Week 4: The Explore → Plan → Code → Verify Cycle — Slides Outline

> "Weeks of coding can save you hours of planning." — Anonymous

**Instructor:** Goker Ezberci
**Total Slides:** 22
**Presentation Duration:** 45 minutes (2–3 min per slide)

---

## Slide 1: Title Slide

**Title:** The Explore → Plan → Code → Verify Cycle
**Subtitle:** From vibes to discipline — the workflow that separates hobbyists from engineers

- Week 4 of Vibe Coding 101: Disciplined Vibe Coding
- Instructor: Goker Ezberci (gokerez@gmail.com)

**Instructor Notes:**
*Open with: "You shipped last week. Congrats. Now — try adding a new feature to your prototype. How did that go?" Most students will admit it was painful. This is the motivation for today.*

---

## Slide 2: ABL Poll — Always Be Learning

**Quick pulse check:**

1. How many of you tried adding a new feature after Week 3?
2. Did the AI "forget" your project's patterns?
3. Who broke something that was working?

*If you answered yes to #2 or #3 — today's lecture solves that.*

---

## Slide 3: Learning Objectives — Week 4

By the end of this week, you will:

1. **Apply** the EPCV cycle to add features to existing codebases
2. **Create** instruction files that provide persistent AI context
3. **Implement** a disciplined git workflow for AI-assisted development
4. **Evaluate** AI-generated code against quality criteria before committing

**Instructor Notes:**
*These are all action-oriented. By the end of lab, every student should have used the cycle at least once and created an instruction file.*

---

## Slide 4: The Problem — Why Your Prototype Is Fragile

**What happens when you add features to Week 3's prototype:**

```
You: "Add a search feature"
AI:  Generates code that...
  ✗ Ignores your existing API patterns
  ✗ Uses a different CSS framework
  ✗ Duplicates a utility function you already have
  ✗ Breaks the navigation component
```

**Why?** The AI doesn't know your codebase. Every new conversation starts from zero.

**The 70% wall:** You got 70% on vibes. The last 30% requires discipline.

**Instructor Notes:**
*Ask students to share their Week 3 → feature addition pain stories. This grounds the abstract problem in real experience.*

---

## Slide 5: Part 1 — The EPCV Cycle (Agenda)

**The disciplined workflow for AI-assisted development:**

| Phase | Goal | Time | Key Action |
|-------|------|------|------------|
| **EXPLORE** | Understand before changing | 5 min | Read code, check patterns |
| **PLAN** | Design before building | 5 min | Get AI plan, no code yet |
| **CODE** | Implement step by step | 15 min | One step at a time |
| **VERIFY** | Test, review, commit | 5 min | Tests + atomic commit |

**The golden rule:** Never skip EXPLORE. It's the phase that prevents 80% of bugs.

**Instructor Notes:**
*Draw the cycle on the whiteboard as a loop with arrows. Emphasize that it's iterative — you go around multiple times per feature.*

---

## Slide 6: EXPLORE — Understand Before Changing

**Goal:** Gather intelligence. Don't modify anything.

**What to explore:**
- File structure and architecture patterns
- Existing tests and coverage
- Recent git history for the area you'll change
- Dependencies and integration points

**With AI:**
```bash
# Gemini CLI (FREE)
gemini "Read the src/auth/ directory. Explain the
authentication flow. Don't modify anything."

# Claude Code
claude "Explore the project structure. What patterns
does this codebase use? What's the testing setup?"
```

**Key:** Tell the AI "don't modify anything." Without this, it will start writing code immediately.

**Instructor Notes:**
*Live demo: Open a student's Week 3 project. Run the explore prompt. Show what the AI discovers.*

---

## Slide 7: PLAN — Design Before Building

**Goal:** Get a plan. Verify it makes sense. Then code.

**A good plan includes:**
1. Which files need to change
2. What new files are needed
3. Edge cases to handle
4. Tests to write
5. Potential breaking changes

**With AI:**
```bash
# Gemini CLI (FREE)
gemini "Plan how to add search. List files to change,
edge cases, and tests. Don't write code yet."

# Claude Code
claude "Plan the implementation. Output a checklist
of steps, not code."
```

**The critical instruction:** "Don't write code yet — just the plan."

**Instructor Notes:**
*Show a bad plan (just "add search to the app") vs. a good plan (specific files, edge cases, tests). The difference in AI output quality is dramatic.*

---

## Slide 8: CODE — Implement Step by Step

**Goal:** Execute one step at a time from the plan.

**The golden rule: Incremental implementation**

```
BAD (all at once):
"Implement the entire search feature"
→ AI generates 500 lines, hard to review, bugs hidden

GOOD (step by step):
"Implement step 1: Create the search API endpoint"
→ Review → "Now step 2: Add the search UI component"
→ Review → "Step 3: Connect UI to API"
→ Review → Done
```

**Why incremental?**
- Easier to review (small diffs)
- Easier to debug (know which step broke)
- Easier to revert (undo one step, not everything)
- AI quality is higher (focused context)

**Instructor Notes:**
*Emphasize: the temptation is to let AI do everything at once. Resist. Small steps = fewer bugs.*

---

## Slide 9: VERIFY — Test, Review, Commit

**Goal:** Nothing ships without verification.

**The verification checklist:**

| Step | Command | Why |
|------|---------|-----|
| 1. Run existing tests | `npm test` | Nothing broke |
| 2. Add new tests | Write tests for new feature | New code is covered |
| 3. Review the diff | `git diff` | No surprises |
| 4. Stage specific files | `git add src/search/...` | No accidental commits |
| 5. Commit atomically | `git commit -m "feat: ..."` | Clear history |

**Never:** `git add .` — this stages everything, including files you didn't mean to commit.

**Always:** `git add [specific files]` — you control what ships.

**Instructor Notes:**
*Show the difference between `git add .` and `git add src/search.ts src/search.test.ts`. The former is sloppy. The latter is professional.*

---

## Slide 10: The EPCV Cycle — Visual Flow

**Complete cycle for adding "password reset" feature:**

```
EXPLORE → Read auth module, check token patterns
    │
    ▼
PLAN → 5 steps: endpoint, token, email, confirm, tests
    │
    ▼
CODE → Step 1: reset-request endpoint → Review ✓
       Step 2: token generation → Review ✓
       Step 3: email integration → Review ✓
       Step 4: reset-confirm endpoint → Review ✓
    │
    ▼
VERIFY → npm test (all pass) → git diff → commit
```

**Time comparison:**

| Approach | Time to Feature | Bugs Found Later |
|----------|----------------|-----------------|
| "Just vibe it" | 30 min | 3-5 bugs |
| EPCV cycle | 45 min | 0-1 bugs |

**15 extra minutes saves 3+ hours of debugging.**

---

## Slide 11: Anti-Patterns — What NOT to Do

**The YOLO Pattern:**
```
"Add all the features at once"
→ 1000-line diff, impossible to review
→ Breaks 3 existing features
```

**The Copy-Paste Pattern:**
```
Copy AI output → Paste into file → Ship
→ No understanding of what changed
→ Can't debug when it breaks
```

**The Infinite Conversation Pattern:**
```
50 messages in one chat about 5 different features
→ Context polluted, AI confused
→ Same mistakes repeated
```

**The "AI Said It's Fine" Pattern:**
```
"Is this code correct?" → AI: "Yes!"
→ AI is always confident. Test it yourself.
```

**Instructor Notes:**
*These are funny but real. Ask students which ones they've done. Most will laugh and admit to all four.*

---

## Slide 12: Part 2 — Instruction Files (Context Engineering)

**The problem:** Every new AI conversation starts from zero.

**The solution:** Instruction files — persistent context that travels with your project.

| File | Tool | Purpose |
|------|------|---------|
| **CLAUDE.md** | Claude Code | Project context + boundaries |
| **.cursorrules** | Cursor | Code style + project rules |
| **GEMINI.md** | Gemini CLI | Same as CLAUDE.md for Gemini |
| **.github/copilot-instructions.md** | Copilot | Copilot-specific context |
| **.windsurfrules** | Windsurf | Windsurf-specific rules |

**Key insight:** These files use the same principles. Learn one, adapt to any.

**Instructor Notes:**
*Show a real CLAUDE.md file. Then show the same prompt with and without it — the difference in code quality is obvious.*

---

## Slide 13: Anatomy of a Great Instruction File

**The 6 Essential Sections:**

```markdown
# 1. COMMANDS — How to build, test, run
- npm run dev, npm test, npm run build

# 2. STRUCTURE — Where things live
src/api/, src/models/, src/services/

# 3. DECISIONS — Architecture choices made
REST + PostgreSQL + JWT auth

# 4. STYLE — How code should look
TypeScript strict, async/await, named exports

# 5. TESTING — How to write and run tests
Vitest, 80% coverage, mirror src/ structure

# 6. BOUNDARIES — Always / Never / Ask First
Always: run tests before commit
Never: hardcode secrets, use `any` type
Ask First: new deps, schema changes
```

**Why this order:** AI reads top-to-bottom. Mechanical info (commands) at top, critical rules (boundaries) at bottom where attention is strongest.

**Instructor Notes:**
*Walk through each section with concrete examples. Students should be writing their own during lab.*

---

## Slide 14: Boundaries — The Most Important Section

**Three categories of boundaries:**

**ALWAYS (non-negotiable rules):**
```
- Run npm test before committing
- Use Prisma for all database queries
- Validate input with Zod schemas
- Log errors with structured JSON
```

**NEVER (hard stops):**
```
- Never hardcode API keys or secrets
- Never use `any` type in TypeScript
- Never write SQL directly (use Prisma)
- Never skip error handling
- Never commit without running tests
```

**ASK FIRST (requires human decision):**
```
- Before adding new dependencies
- Before changing database schema
- Before modifying auth middleware
- Before changing API response formats
```

**The AI can't infer boundaries from omission.** What you don't say, AI will invent.

**Instructor Notes:**
*This connects directly to Week 1's PRD Non-Goals. Same principle: AI can't infer what you DON'T want.*

---

## Slide 15: Before & After — Instruction Files in Action

**WITHOUT instruction file:**
```bash
claude "Add a new API endpoint for user profiles"
```
→ AI generates Express.js (you use Fastify)
→ Uses MySQL queries (you use Prisma)
→ No error handling
→ No tests
→ 3 iterations to fix

**WITH instruction file (CLAUDE.md):**
```bash
claude "Add a new API endpoint for user profiles"
```
→ AI uses Fastify (matches your stack)
→ Uses Prisma ORM (follows your pattern)
→ Includes Zod validation (your rule)
→ Adds error handling (your boundary)
→ Generates Vitest tests (your testing setup)
→ **1 iteration. Done.**

**The instruction file saved 3 iterations × 5 minutes = 15 minutes per feature.**

**Instructor Notes:**
*This is the killer demo. Show both outputs side by side. Students will immediately want to create their own instruction file.*

---

## Slide 16: Part 3 — Git Workflows for AI Development

**The "One Feature Per Conversation" Rule:**

```
BAD: One conversation, multiple features
├─ "Add login" + "Add dark mode" + "Fix bug"
└─ Context is polluted, AI gets confused

GOOD: One conversation per feature
├─ Conv 1: "Add login" → commit → done
├─ Conv 2: "Add dark mode" → commit → done
└─ Conv 3: "Fix bug" → commit → done
```

**Branch Strategy:**
```
main (protected)
  ├── feat/login-page
  ├── feat/dark-mode
  ├── fix/navigation-bug
  └── chore/update-deps
```

**Each branch = one feature = one conversation = one atomic commit.**

---

## Slide 17: Commit Message Convention

**Format:**
```
type(scope): short description

Longer description if needed.
```

**Types:**
| Type | Meaning | Example |
|------|---------|---------|
| `feat` | New feature | `feat: add search endpoint` |
| `fix` | Bug fix | `fix: prevent duplicate login` |
| `test` | Adding tests | `test: add auth edge cases` |
| `refactor` | Code cleanup | `refactor: extract validation` |
| `docs` | Documentation | `docs: update API readme` |
| `chore` | Build/deps/CI | `chore: upgrade vitest to 2.0` |

**Why this matters:** Clear history helps you AND the AI understand what changed.

---

## Slide 18: Code Review Checklist for AI-Generated Code

**Before committing ANY AI-generated code, check:**

| Check | Why | Command |
|-------|-----|---------|
| Does it compile? | AI generates invalid syntax | `npm run build` |
| Do existing tests pass? | AI breaks existing features | `npm test` |
| Hardcoded values? | AI loves magic numbers | `grep "TODO\|FIXME"` |
| Error handling? | AI often skips error cases | Review catch blocks |
| Types correct? | AI generates loose types | Check for `any` |
| Over-engineered? | AI adds unrequested features | Compare to plan |
| Security issues? | SQL injection, XSS, secrets | Review inputs/outputs |

**The AI is always confident. Your job is to verify.**

---

## Slide 19: Lab Exercise — 30 Minutes

**Apply the EPCV cycle to add ONE new feature to your Week 3 prototype.**

| Time | Phase | Action |
|------|-------|--------|
| **0-5 min** | Setup | Create instruction file + feature branch |
| **5-10 min** | EXPLORE | Ask AI to explore your codebase |
| **10-15 min** | PLAN | Get AI plan (no code yet!) |
| **15-25 min** | CODE | Implement one step at a time |
| **25-30 min** | VERIFY | Run tests + review diff + commit |

**Submit:**
- Instruction file (CLAUDE.md / .cursorrules / GEMINI.md)
- Git log showing your atomic commit
- 100-word reflection on how EPCV changed your workflow

---

## Slide 20: Key Takeaways

1. **EXPLORE before you code** — The AI doesn't know your codebase. YOU have to show it.
2. **PLAN before you implement** — "Don't write code yet" is the most powerful instruction.
3. **CODE incrementally** — One step at a time, review each step.
4. **VERIFY before you commit** — Tests + diff review + atomic commits.
5. **Instruction files** — Write once, get consistent AI behavior in every conversation.
6. **One feature per conversation** — Clean context = better code quality.

---

## Presentation Tips for Instructor

1. **Opening demo:** Show a student's Week 3 project. Try adding a feature without EPCV. Show the mess. Then do it with EPCV. The contrast is motivating.

2. **Live EXPLORE:** Pick a student's repo and run the EXPLORE prompt live. Students love seeing their own projects analyzed by AI.

3. **Instruction file workshop:** After showing the anatomy slide, give students 5 minutes to start writing their own. Peer review with neighbors.

4. **Anti-patterns humor:** The anti-patterns slide gets laughs. Lean into it — students relate to every one.

5. **The key moment:** The "Before & After" slide (instruction files) is the turning point. If students leave with one thing, it should be: "I need to write a CLAUDE.md file."

---

## Slide Count & Timing Reference

- **Slides 1-4:** Introduction + Problem (5 min)
- **Slides 5-11:** EPCV Cycle (15 min)
- **Slides 12-15:** Instruction Files (12 min)
- **Slides 16-18:** Git Workflows (8 min)
- **Slides 19-20:** Lab + Takeaways (5 min)

**Total:** 48 minutes (with discussion/demos)

**Flex:** If running long, compress slides 16-18 (git) into 5 minutes — students can read the details in README.

---

**Instructor:** Goker Ezberci | gokerez@gmail.com
