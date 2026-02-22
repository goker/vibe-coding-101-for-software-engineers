# Week 3: From PRD to Deployed Prototype — Slides Outline

> "If you're not embarrassed by the first version of your product, you've launched too late." — Reid Hoffman, LinkedIn Co-founder

**Instructor:** Goker Ezberci

---

## Slide 1: Title Slide

**Week 3: From PRD to Deployed Prototype**

*Ship it or it doesn't count.*

> "If you're not embarrassed by the first version of your product, you've launched too late." — Reid Hoffman

---

## Slide 2: Pulse Check (Start)

**Before we begin:**

1. On a scale of 1-10, how scared are you to show imperfect code to strangers?
2. What's ONE thing that could go wrong today that would make you want to quit?

*Write down your answers. We'll revisit at the end.*

---

## Slide 3: Prerequisites Checklist

**You MUST have these ready before we continue:**

| Requirement | How to Check |
|-------------|--------------|
| **PRD Written** | File exists in your repo: `docs/PRD.md` |
| **AI Tool Installed** | `claude --version` or `gemini --version` works |
| **CLAUDE.md Created** | File exists in project root |
| **Deployment Account** | Can log into Vercel/Railway/Replit dashboard |
| **Git Repo Ready** | `git status` works, `.gitignore` has `.env` |

**If ANY box is unchecked, fix it NOW.**

---

## Slide 4: CLAUDE.md Example

```markdown
# Project: Grocery List App

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

## Slide 5: Learning Objectives

By the end of this class, you will:

1. **Execute** a rapid prototyping workflow: PRD → Deployed prototype
2. **Select** the right platform for your project type
3. **Configure** environment variables and manage secrets securely
4. **Debug** common deployment failures using logs and AI assistance
5. **Write** one test before deploying (TDD Light)

---

## Slide 6: The 80/20 Rule

**Get 80% working. Then deploy immediately.**

| 80% Done | 20% Remaining |
|----------|---------------|
| Core features work | Edge cases |
| UI is basic but usable | Polish |
| No error handling | Robustness |
| Database schema is rough | Optimization |

**Deployed 80% > Perfect 0%**

Localhost perfectionists ship nothing.
Real users are your best QA team.

---

## Slide 7: The 70% Wall

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

**WEEKS 1-3:** Get to 70% fast with AI
**WEEKS 4-12:** Push past the wall with discipline

---

## Slide 8: Visionary Quote

> "Make something people want. Then launch early and often."

— Paul Graham, Y Combinator

*Your users don't care about your code quality. They care about whether it solves their problem.*

---

## Slide 9: Tool Selection Matrix (All Have FREE Options)

| Project Type | FREE Tools | Paid Tools |
|--------------|-----------|------------|
| **Web App** | Bolt.new (free tier), v0 (free), Replit | Cursor ($20/mo), Lovable |
| **CLI Tool** | Gemini CLI (FREE!), Claude Code | Claude Max |
| **API/Backend** | Replit (free), Windsurf (free tier) | Cursor, Windsurf Pro |
| **Mobile App** | Expo (FREE) + any AI tool | Replit Mobile |

**Key insight:** Start with free tools. Upgrade when you hit limits.

**Gemini CLI is completely FREE with 1M token context!**

---

## Slide 10: Deployment Platforms (All Have FREE Tiers)

| Platform | Best For | Free Tier | Cost After |
|----------|----------|-----------|------------|
| **Vercel** | Next.js, React | 100GB bandwidth | $20/mo |
| **Railway** | Any language | $5 credit/mo | Pay-as-you-go |
| **Render** | Simple apps | 750 hours/mo | $7/mo |
| **Replit** | Learning | Free with ads | $25/mo |
| **Netlify** | Static sites | 100GB | $19/mo |

**Recommendation for students:** Start with **Vercel** (generous free tier) or **Replit** (zero config).

---

## Slide 11: Secrets Management

**The Golden Rule:** Never commit `.env` files. Never put API keys in code.

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
const API_KEY = process.env.OPENAI_API_KEY;  // Safe!
```

*If you commit a key, assume it's compromised. Rotate immediately.*

---

## Slide 12: The Deployment Checklist

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

## Slide 13: The Sprint — Step by Step

**This is NOT "write a PRD in 1 minute." You already have one.**

| Time | Task | Details |
|------|------|---------|
| **0-2 min** | Verify prerequisites | All boxes checked from Slide 3 |
| **2-5 min** | Open your PRD | Share with AI tool |
| **5-15 min** | Generate initial version | AI scaffolds based on PRD |
| **15-25 min** | Iterate and fix | "Fix this bug", "Add dark mode" |
| **25-30 min** | Write ONE test | TDD Light (next slide) |
| **30-40 min** | Deploy | Push to Vercel/Railway/Render |
| **40-45 min** | Share URL | Send to 1 person, get feedback |

---

## Slide 14: Why TDD? (Even One Test Matters)

**The Problem:**
- AI generates code that *looks* right but doesn't work
- 20% of AI-generated code has bugs you can't see
- Deployments can succeed but features can be broken

**One Test Catches:**
- "Core feature doesn't work" — before you embarrass yourself
- "API integration is broken" — before users see errors
- "Logic is wrong" — before you waste time debugging production

**Real Story:**
> "I shipped a calculator app. It looked perfect. Then someone tried 2+2 and got 22. One test would have caught it."

**One test takes 5 minutes. Debugging production takes 5 hours.**

---

## Slide 15: FREE Testing Tools

| Tool | Language | Cost | Best For |
|------|----------|------|----------|
| **Vitest** | JavaScript/TS | FREE | React, Vue, modern JS |
| **Jest** | JavaScript/TS | FREE | Node, React |
| **pytest** | Python | FREE | All Python projects |
| **Go test** | Go | FREE (built-in) | Go projects |

**Install (FREE):**

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

---

## Slide 16: TDD Light — One Test Before You Deploy

**Example test (takes 5 minutes with AI):**

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

**Run it:**

**macOS/Linux & Windows:**
```bash
npm test
```

*If this one test fails, your app is broken. Don't deploy broken apps.*

---

## Slide 17: TDD with AI — Red-Green-Refactor

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

## Slide 18: Common Deployment Failures

| Error | Cause | Fix |
|-------|-------|-----|
| **Build failed** | Missing dep or syntax error | Check logs, fix locally, redeploy |
| **Module not found** | Wrong import path | Verify path, `npm install` |
| **Env var undefined** | Not set in dashboard | Add to platform settings, redeploy |
| **App crashes on startup** | Missing API key | Check runtime logs, add key |
| **504 Gateway Timeout** | Function too slow | Check cold start, optimize |

---

## Slide 19: Debugging with AI

**When deploys fail, copy the error and ask:**

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

*AI is excellent at parsing error logs. Use it.*

---

## Slide 20: Pulse Check (End)

**Reflect on today:**

1. What's the URL you'll deploy to today? (Write it down)
2. Who will you share it with to get feedback?
3. Look at your Slide 2 answers — do you still feel the same way?

*The fear of shipping imperfect code is normal. Ship anyway.*

---

## Slide 21: Resources (All FREE)

**Deployment Platforms (FREE tiers):**
- Vercel: https://vercel.com
- Railway: https://railway.app
- Render: https://render.com
- Replit: https://replit.com
- Netlify: https://netlify.com

**AI Tools (FREE options):**
- Gemini CLI (FREE, 1M context!): https://ai.google.dev/gemini-api/docs/quickstart
- Bolt.new (free tier): https://bolt.new
- v0 (free): https://v0.dev
- Claude Code: https://claude.ai/code

**Testing (FREE):**
- Vitest: https://vitest.dev
- Jest: https://jestjs.io
- pytest: https://pytest.org

---

## Slide 22: What's Next

**Week 4: The Explore → Plan → Code → Verify Cycle**

- Context engineering for AI tools
- Instruction files (CLAUDE.md, .cursorrules)
- Git workflows for AI-assisted development
- Pushing past the 70% wall

*You've shipped. Now we build discipline.*

---

**Instructor:** Goker Ezberci | gokerez@gmail.com
