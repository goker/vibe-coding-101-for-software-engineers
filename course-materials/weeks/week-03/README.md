# Week 3: From PRD to Deployed Prototype

> "If you're not embarrassed by the first version of your product, you've launched too late." — Reid Hoffman, LinkedIn Co-founder

---

## Quick Pulse Check (Start)

Before we begin:
1. On a scale of 1-10, how scared are you to show imperfect code to strangers?
2. What's ONE thing that could go wrong today that would make you want to quit?

---

## Prerequisites Checklist

**You MUST have these ready before class:**

| Requirement | How to Check | If Missing |
|-------------|--------------|------------|
| **PRD Written** | `docs/PRD.md` exists in your repo | See Week 1-2 materials |
| **AI Tool Installed** | `claude --version` or `gemini --version` | Install guide below |
| **CLAUDE.md Created** | File exists in project root | Example below |
| **Deployment Account** | Can log into Vercel/Railway/Replit | Sign up links below |
| **Git Repo Ready** | `git status` works, `.gitignore` has `.env` | `git init` |

### Quick Install (If Missing)

**Gemini CLI (FREE - Recommended for students):**

**macOS/Linux:**
```bash
npm install -g @google/gemini-cli
gemini --version
```

**Windows PowerShell:**
```powershell
npm install -g @google/gemini-cli
gemini --version
```

**Claude Code:**

**macOS/Linux:**
```bash
curl -fsSL https://claude.ai/install.sh | bash
claude --version
```

**Windows PowerShell:**
```powershell
winget install Anthropic.Claude
claude --version
```

### Sign Up for Deployment (FREE)

Pick ONE:
- **Vercel** (Best for Next.js): https://vercel.com
- **Render** (Best for simplicity): https://render.com
- **Replit** (Best for beginners): https://replit.com

---

## Learning Objectives

By the end of this week, you will:
- **Execute** a rapid prototyping workflow: PRD → Deployed prototype
- **Select** the right platform for your project type
- **Configure** environment variables and manage secrets securely
- **Debug** common deployment failures using logs and AI assistance
- **Write** one test before deploying (TDD Light)

---

## This Week You SHIP

By the end of this week, every student has a working prototype accessible via a **public URL**. This is the most energizing week of the course.

---

## CLAUDE.md Example

Create this file in your project root. This file tells AI tools about your project.

**Official Documentation:** https://docs.anthropic.com/en/docs/claude-code/overview

```markdown
# Project: [Your Project Name]

## Commands
- `npm run dev` — Start development server
- `npm run build` — Create production build
- `npm test` — Run all tests

## Testing
- Framework: Vitest (FREE)
- Location: `src/__tests__/`
- Coverage target: 80%

## Project Structure
src/
├── components/   # React components
├── hooks/        # Custom hooks
├── utils/        # Helper functions
└── __tests__/    # Test files

## Code Style
- Use TypeScript strict mode
- Prefer named exports
- Use async/await over .then()

## Boundaries (NEVER)
- Never hardcode API keys
- Never skip tests for "speed"
- Never deploy without build passing
```

---

## PRD Reminder: What Does a Good PRD Look Like?

Your PRD from Weeks 1-2 should include:

| Section | What It Contains |
|---------|-----------------|
| **Overview** | What, Who, Why (one sentence each) |
| **Features** | 3-5 core features with examples |
| **Non-Goals** | What you WON'T build (critical for AI) |
| **Technical Constraints** | Language, framework, dependencies |
| **Phases** | Step-by-step build plan |
| **Agent Rules** | Always/Ask First/Never table |

### Example PRDs (See `templates/examples/`)

**1. Todo CLI (Beginner)**
```markdown
## Overview
| | |
|---|---|
| **What** | A command-line todo list manager |
| **Who** | Developers who prefer terminal workflows |
| **Why** | Quick task capture without leaving terminal |

## Features
1. `todo add "Buy groceries"` → Adds task
2. `todo list` → Shows all tasks
3. `todo done 1` → Marks task complete
4. `todo delete 1` → Removes task

## Non-Goals
- Due dates or reminders
- Priority levels or tags
- Cloud sync
```

**2. Code Review Skill (Intermediate)**
```markdown
## Overview
| | |
|---|---|
| **What** | An AI skill that reviews code changes |
| **Who** | Solo developers wanting quick feedback |
| **Why** | Catch bugs before they reach production |

## Features
1. `/review` → Analyzes staged changes
2. Structured output: Bugs, Security, Style
3. Severity levels: Critical, Warning, Info

## Non-Goals
- Auto-fix capabilities
- GitHub/GitLab integration
- Custom rule configuration
```

**3. Weather Dashboard API (Advanced)**
```markdown
## Overview
| | |
|---|---|
| **What** | REST API aggregating weather data |
| **Who** | Developers building weather apps |
| **Why** | Clean, cached interface to weather data |

## Features
1. `GET /weather/{city}` → Current weather
2. 10-minute response caching
3. `GET /stats` → Usage analytics
4. Simple HTML dashboard

## Non-Goals
- User accounts or authentication
- Weather forecasts
- Multiple API providers
```

**Full examples:** See `templates/examples/` folder — **24 complete PRD examples** across beginner, intermediate, and advanced levels.

---

## Extra Material: How to Prepare a PRD

Want a deeper dive into writing effective PRDs? See the [How to Prepare a PRD](extras/how-to-prepare-prd/) extra lesson:

| Resource | Description |
|----------|-------------|
| [Slides (PDF)](<extras/how-to-prepare-prd/slides/How to Prepare a PRD By Goker Ezberci.pdf>) | Slide deck covering all PRD sections |
| [Checklist](extras/how-to-prepare-prd/checklist.md) | One-page printable quality checklist |
| [PPTX](<extras/how-to-prepare-prd/slides/How to Prepare a PRD By Goker Ezberci.pptx>) | Editable presentation version |
| [All Examples](../../templates/examples/) | 24 complete PRD examples |

**Key insight:** AI cannot infer from omission. What you don't say, AI will invent.

---

## The 80/20 Rule

Get 80% working in one sitting. This 80% is messy but functional:
- Core features work
- UI is basic but usable
- No edge case handling
- Database schema is provisional

**Then deploy this 80% version.** Get feedback. Fix the 20% that matters most.

**Deployed 80% > Perfect 0%**

Localhost perfectionists ship nothing. Real users are your best QA team.

---

## The "70% Wall"

```
Quality/Completeness
     ^
100% |                          .----- (Diminishing returns)
     |                    .----'
     |               .---'
 70% |          .---'  <-- THE WALL
     |     .---'         (Edge cases, scaling, maintenance)
     |.---'
  0% +-----------------------------------------> Time/Effort
          ^                    ^
     Quick wins          Hard problems
     (AI excels)         (Engineering discipline needed)
```

A surprising phenomenon: vibe-coded prototypes often work *remarkably well* initially. You might build something in 4 hours that a traditional team takes 2 weeks to plan.

But there's a wall at ~70% completeness:
- Edge cases surface
- Performance degrades
- Scaling becomes hard
- Maintenance becomes painful

**This is normal and expected.** The rest of this course teaches engineering discipline to push past this wall. For now: celebrate the 70% prototype.

---

## Choosing the Right Tool (All Have FREE Options)

| Project Type | FREE Tools | Paid Options |
|--------------|-----------|--------------|
| **Web App (React/Next.js)** | Bolt.new, v0, Replit | Cursor ($20/mo), Lovable |
| **CLI Tool** | Gemini CLI (FREE!), Claude Code | Claude Max |
| **API/Backend** | Replit (free), Windsurf (free tier) | Cursor, Windsurf Pro |
| **Mobile App** | Expo (FREE) + any AI tool | Replit Mobile |

**Key insight:** Start with free tools. Upgrade when you hit limits.

**Gemini CLI is completely FREE with 1M token context window!**

---

## Deployment Platforms (All Have FREE Tiers)

| Platform | Best For | Free Tier | Cost After |
|----------|----------|-----------|------------|
| **Vercel** | Next.js, React | 100GB bandwidth | $20/mo |
| **Railway** | Any language, containers | $5 credit/mo | Pay-as-you-go |
| **Render** | Simple apps, static sites | 750 hours/mo | $7/mo |
| **Replit** | Learning projects | Free with ads | $25/mo |
| **Netlify** | Static sites, JAMstack | 100GB | $19/mo |

**Recommendation for students:** Start with **Vercel** (generous free tier) or **Replit** (zero config).

---

## Environment Variables

**Golden rule:** Never commit `.env` files. Never put API keys in code.

**macOS/Linux:**
```bash
# Create .env file (git-ignored)
echo "OPENAI_API_KEY=your-key-here" >> .env
echo ".env" >> .gitignore
```

**Windows PowerShell:**
```powershell
# Create .env file (git-ignored)
Add-Content -Path .env -Value "OPENAI_API_KEY=your-key-here"
Add-Content -Path .gitignore -Value ".env"
```

**In your code:**
```javascript
// GOOD - reads from environment
const API_KEY = process.env.OPENAI_API_KEY;

// BAD - hardcoded secret (NEVER DO THIS)
const API_KEY = "sk-proj-abc123xyz";
```

Set real values in your platform's dashboard (Vercel → Settings → Environment Variables).

---

## The Deployment Checklist

Before you hit "Deploy":

| Step | macOS/Linux | Windows PowerShell |
|------|-------------|-------------------|
| 1. Does it work locally? | `npm run dev` | `npm run dev` |
| 2. Any secrets in code? | `grep -r "sk-" .` | `Select-String -Path .\* -Pattern "sk-"` |
| 3. Package.json exists? | `ls package.json` | `Test-Path package.json` |
| 4. Build passes? | `npm run build` | `npm run build` |
| 5. ONE test passes? | `npm test` | `npm test` |

**Don't skip the test. One test > Zero tests.**

---

## TDD Light: One Test Before You Deploy

### Why TDD Matters (Even One Test)

**The Problem:**
- AI generates code that *looks* right but doesn't work
- 20% of AI-generated code has bugs you can't see
- Deployments can succeed but features can be broken

**Real Story:**
> "I shipped a calculator app. It looked perfect. Then someone tried 2+2 and got 22. One test would have caught it."

**One test takes 5 minutes. Debugging production takes 5 hours.**

### FREE Testing Tools

| Tool | Language | Cost | Best For |
|------|----------|------|----------|
| **Vitest** | JavaScript/TS | FREE | React, Vue, modern JS |
| **Jest** | JavaScript/TS | FREE | Node, React |
| **pytest** | Python | FREE | All Python projects |
| **Go test** | Go | FREE (built-in) | Go projects |

**Install:**

**macOS/Linux:**
```bash
npm install -D vitest   # JavaScript
pip install pytest      # Python
```

**Windows PowerShell:**
```powershell
npm install -D vitest   # JavaScript
pip install pytest      # Python
```

### Example Test (Takes 5 Minutes with AI)

```javascript
// grocery-list.test.js
import { test, expect } from 'vitest';
import { GroceryList } from './grocery-list';

test('user can add an item', () => {
  const list = new GroceryList();
  list.add('Milk');
  expect(list.items).toContain('Milk');
});
```

### TDD with AI: Red-Green-Refactor

**Step 1: RED (Write failing test)**

**Gemini CLI (FREE):**
```bash
gemini "Write a Vitest test for adding items to a grocery list. It should fail initially."
```

**Claude Code:**
```bash
claude "Write a failing test for adding grocery items using Vitest."
```

**Step 2: GREEN (Make it pass)**
```bash
gemini "Make this test pass with the minimum code needed."
```

**Step 3: REFACTOR (Clean up)**
```bash
gemini "Refactor this code to be cleaner. Keep the test passing."
```

**This cycle takes 5-10 minutes with AI. Do it ONCE before deploy.**

---

## Debugging Deployment Failures

### Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| **Build failed** | Missing dep or syntax error | Check logs, fix locally, redeploy |
| **Module not found** | Wrong import path | Verify path, `npm install` |
| **Env var undefined** | Not set in dashboard | Add to platform settings, redeploy |
| **App crashes on startup** | Missing API key | Check runtime logs, add key |
| **504 Gateway Timeout** | Function too slow | Check cold start, optimize |

### Debugging with AI

When deploys fail, copy the error and ask:

**Gemini CLI (FREE):**
```bash
gemini "My Vercel build failed with this error:
[paste error log]

What's wrong and how do I fix it?"
```

**Claude Code:**
```bash
claude "Debug this deployment error: [paste error]"
```

AI is excellent at parsing error logs. Use it.

---

## The Sprint: Step by Step

**This is NOT "write a PRD in 1 minute." You already have one.**

| Time | Task | Details |
|------|------|---------|
| **0-2 min** | Verify prerequisites | All boxes checked from checklist |
| **2-5 min** | Open your PRD | Share with AI tool |
| **5-15 min** | Generate initial version | AI scaffolds based on PRD |
| **15-25 min** | Iterate and fix | "Fix this bug", "Add dark mode" |
| **25-30 min** | Write ONE test | TDD Light |
| **30-40 min** | Deploy | Push to Vercel/Railway/Render |
| **40-45 min** | Share URL | Send to 1 person, get feedback |

---

## Cross-Platform Deployment Commands

### Vercel

**macOS/Linux & Windows:**
```bash
npm i -g vercel
vercel
```

### Railway

**macOS/Linux & Windows:**
```bash
npm i -g @railway/cli
railway login
railway up
```

### Render

No CLI needed — connect GitHub repo and deploy from dashboard.

---

## Quick Pulse Check (End)

Reflect on today:
1. What's the URL you'll deploy to today? (Write it down)
2. Who will you share it with to get feedback?
3. Look at your start answers — do you still feel the same way?

*The fear of shipping imperfect code is normal. Ship anyway.*

---

## Resources (All FREE)

### Deployment Platforms
- **Vercel:** https://vercel.com
- **Railway:** https://railway.app
- **Render:** https://render.com
- **Replit:** https://replit.com
- **Netlify:** https://netlify.com

### AI Tools (FREE Options)
- **Gemini CLI (FREE, 1M context!):** https://ai.google.dev/gemini-api/docs/quickstart
- **Bolt.new (free tier):** https://bolt.new
- **v0 (free):** https://v0.dev
- **Claude Code:** https://claude.ai/code

### Testing (FREE)
- **Vitest:** https://vitest.dev
- **Jest:** https://jestjs.io
- **pytest:** https://pytest.org

### Learning Resources
- **Vercel Deployment Guide:** https://vercel.com/docs/deployments/overview
- **12-Factor App (Config):** https://12factor.net/config
- **Testing with Vitest:** https://vitest.dev/guide/

---

**Instructor:** Goker Ezberci | gokerez@gmail.com
