# Week 1: Slides Outline

> "There's a new kind of coding I call 'vibe coding', where you fully give in to the vibes, embrace exponentials, and forget that the code even exists." — Andrej Karpathy, February 3, 2025

---

## Slide 1: Title

**Vibe Coding 101: Week 1**
*The History of Vibe Coding & PRD-First Development*

---

## Slide 2: Pulse Check (Start)

**Before we begin, rate your confidence (1-5):**

1. How comfortable are you using AI coding tools?
2. Have you ever written a PRD before?

---

## Slide 3: The Tweet That Changed Everything

> Show Karpathy's original tweet (February 3, 2025)

"There's a new kind of coding I call 'vibe coding', where you fully give in to the vibes, embrace exponentials, and forget that the code even exists."

— Andrej Karpathy

*Source: [x.com/karpathy/status/1886192184808149383](https://x.com/karpathy/status/1886192184808149383)*

---

## Slide 4: Who is Andrej Karpathy?

- Former OpenAI researcher
- Former Director of AI at Tesla (Autopilot)
- PhD from Stanford (computer vision)
- Created popular deep learning courses
- 2023: "The hottest new programming language is English"
- 2025: Coined "vibe coding"

---

## Slide 5: The Problem with Pure Vibe Coding

### Risks

- **Hallucinations:** AI can invent packages or APIs that don't exist
- **AI Slop:** Code that looks right but doesn't work
- **Security:** Slopsquatting attacks (malicious packages with hallucinated names)
- **Tech Debt:** "Wide but shallow" code—hairballs that are hard to maintain

---

## Slide 6: The Spectrum

```
PURE VIBE CODING              DISCIPLINED VIBE CODING
(Weekend Projects)            (This Course)

• "Accept All" always         • Review diffs critically
• Skip reading code           • Understand what's generated
• Copy-paste errors           • Verify with tests & PRD
• Works until it doesn't      • Production-ready mindset
```

---

## Slide 7: Our Solution: PRD-First Development

### Before You Write a Single Prompt...

Create a **Product Requirements Document (PRD)**

Why?
- Clarity for YOU
- Guidance for the AI
- Prevents scope creep
- Catches hallucinations early

---

## Slide 8: The PRD Structure

1. **Overview** - What/Who/Why
2. **Core Features** - MVP only
3. **Non-Goals** - CRITICAL for AI!
4. **Technical Constraints** - Language, deps, standards
5. **Success Criteria** - How do we know it works?
6. **Phases** - Sequential, verifiable chunks

---

## Slide 9: Why Non-Goals Matter

> "AI cannot infer from omission. If the specification does not explicitly state 'do not implement user authentication in this phase,' an agent might reasonably add it."

**Without Non-Goals:**
- AI adds features you didn't ask for
- Scope keeps expanding
- You spend time removing unwanted code

---

## Slide 10: The Six Essential Areas

From analysis of many agent configurations:

1. **Commands** - Full commands with flags
2. **Testing** - Framework, coverage, locations
3. **Project Structure** - Directory organization
4. **Code Style** - Real examples, not just rules
5. **Git Workflow** - Branches, commits, PRs
6. **Boundaries** - What agents should NEVER do

*Source: Addy Osmani*

---

## Slide 11: Tool Setup

**Claude Code (macOS/Linux):**
```bash
curl -fsSL https://claude.ai/install.sh | bash
claude
/init       # Generate CLAUDE.md
```

**Gemini CLI (macOS/Linux):**
```bash
npm install -g @google/gemini-cli
gemini
```

**Windows PowerShell:**
```powershell
# Claude Code
winget install Anthropic.Claude
claude

# Gemini CLI
npm install -g @google/gemini-cli
gemini
```

---

## Slide 12: The Workflow Cycle

```
1. EXPLORE → 2. PLAN → 3. CODE → 4. COMMIT
     ↑                              |
     └──────────────────────────────┘
              (Iterate)
```

---

## Slide 13: PRD to Code

1. Show the greeting-cli PRD
2. Start AI session
3. Share PRD with AI
4. Ask for plan (don't code yet!)
5. Review plan together
6. Implement Phase 1 only
7. Verify it works
8. Continue to Phase 2...

---

## Slide 14: Pulse Check (End)

**Reflect on today's session:**

1. What's your biggest takeaway about disciplined vibe coding?
2. What project will you build your first PRD for?

---

## Slide 15: What's Next

**Week 2: How AI Coding Actually Works**

- Tokenization, context windows, temperature
- The AI tool landscape
- Prompt engineering fundamentals

---

## Slide 16: Questions?

**Resources:**
- Course repo: [link]
- Claude Code docs: code.claude.com
- Gemini CLI docs: ai.google.dev/gemini-cli

---

**Instructor:** Goker Ezberci | gokerez@gmail.com
