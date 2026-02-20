# Week 1: The History of Vibe Coding & PRD-First Development

> "There's a new kind of coding I call 'vibe coding', where you fully give in to the vibes, embrace exponentials, and forget that the code even exists." — Andrej Karpathy, February 3, 2025

---

## Quick Pulse Check (Start)

Before we begin, rate your confidence (1-5):
1. Have you used AI to help you write code before?
2. How comfortable are you defining project requirements before coding?

---

## Learning Objectives

By the end of this week, you will:
- Understand the origin and evolution of vibe coding
- Distinguish when vibe coding is appropriate vs. when discipline is required
- Write a Product Requirements Document (PRD) for AI-assisted development
- Set up your development environment with AI coding tools

---

## The Tweet That Started It All

On **February 3, 2025**, Andrej Karpathy—co-founder of OpenAI and former AI leader at Tesla—posted a tweet that defined a new era:

> "There's a new kind of coding I call 'vibe coding', where you fully give in to the vibes, embrace exponentials, and forget that the code even exists."

**Key quotes from Karpathy's thread:**
- "I 'Accept All' always, I don't read the diffs anymore"
- "The code grows beyond my usual comprehension"
- "It's not too bad for **throwaway weekend projects**"

**Critical insight:** Notice "throwaway weekend projects." This is NOT production software engineering. Our challenge:

> How do we harness the speed of vibe coding while maintaining professional standards?

---

## The Spectrum: Pure vs. Disciplined Vibe Coding

| Pure Vibe Coding | Disciplined Vibe Coding |
|-----------------|------------------------|
| "Accept All" always | Review diffs critically |
| Skip reading code | Understand what's generated |
| Copy-paste errors | Verify with tests & PRD |
| Works until it doesn't | Production-ready mindset |

This course teaches **disciplined vibe coding** — the professional approach.

---

## Timeline of AI Coding

| Date | Milestone |
|------|-----------|
| Feb 2025 | Karpathy coins "vibe coding" |
| Mar 2025 | Simon Willison: "Not all AI-assisted programming is vibe coding" |
| Late 2025 | Claude Code & Gemini CLI with structured workflows |
| 2026 | "Context engineering" replaces "prompt engineering" |

---

## Why PRD Before Prompts?

> "Planning in advance matters even more with an agent—you can iterate on the plan first, then hand it off to the agent." — Addy Osmani

**Without a PRD:**
- Scope creep (AI adds features you didn't ask for)
- Hallucinations (AI invents requirements)
- Inconsistent implementation
- No way to verify "done"

**AI cannot infer from omission.** If you don't say "no authentication," AI might add it.

---

## Essential PRD Structure

```markdown
# PRD: [Project Name]

## 1. Overview
- **What:** One sentence description
- **Who:** Target users
- **Why:** Problem being solved

## 2. Core Features (MVP)
- Feature 1
- Feature 2
- Feature 3

## 3. Non-Goals (Critical for AI!)
- What this will NOT do
- Technologies we won't use

## 4. Technical Constraints
- Language, dependencies, requirements

## 5. Success Criteria
- How we know it's done

## 6. Phases
- Phase 1: [Tasks + Verification]
- Phase 2: [Tasks + Verification]
```

---

## Setting Up Your AI Coding Tools

### Gemini CLI (FREE - 1,500 requests/day)

**macOS/Linux:**
```bash
npm install -g @google/gemini-cli
export GEMINI_API_KEY="your-key"  # from aistudio.google.com
gemini --version
```

**Windows PowerShell:**
```powershell
npm install -g @google/gemini-cli
$env:GEMINI_API_KEY = "your-key"
gemini --version
```

### Claude Code

**macOS/Linux:**
```bash
npm install -g @anthropic-ai/claude-code
export ANTHROPIC_API_KEY="your-key"
claude --version
```

**Windows PowerShell:**
```powershell
npm install -g @anthropic-ai/claude-code
$env:ANTHROPIC_API_KEY = "your-key"
claude --version
```

---

## The Explore → Plan → Code Cycle

The universal workflow for AI-assisted development:

```
1. EXPLORE → "Read and understand the project"
2. PLAN    → "Create a detailed plan, no code yet"
3. CODE    → "Implement and test"
4. VERIFY  → "Check against PRD, commit"
     ↑                              |
     └──────────────────────────────┘
              (Iterate)
```

---

## Your First PRD-Driven Session

### Step 1: Create a PRD

Create a file called `prd.md`:

```markdown
# PRD: Greeting CLI Tool

## Overview
- **What:** CLI tool that greets users by name
- **Who:** Developers learning vibe coding
- **Why:** Practice PRD-first workflow

## Core Features
1. Accept name as argument
2. Print greeting message
3. Support --formal flag

## Non-Goals
- No GUI, no web interface, no database

## Technical Constraints
- Python 3.11+, stdlib only

## Success Criteria
- All features work when tested manually

## Phases
### Phase 1: Basic greeting (name argument → output)
### Phase 2: Add --formal flag
```

### Step 2: Start an AI Session

**macOS/Linux:**
```bash
mkdir greeting-cli && cd greeting-cli
```

**Windows PowerShell:**
```powershell
mkdir greeting-cli; cd greeting-cli
```

### Step 3: Prompt the AI

**Gemini CLI:**
```bash
gemini "I have a PRD for a greeting CLI. [paste PRD]. First create a plan, then implement Phase 1 only."
```

**Claude Code:**
```bash
claude "I have a PRD for a greeting CLI. [paste PRD]. First create a plan, then implement Phase 1 only."
```

### Step 4: Verify Against PRD

Ask the AI: "Does this implementation satisfy Phase 1 of the PRD? What's missing?"

---

## Quick Pulse Check (End)

Reflect on today's session:
1. What's one thing you learned about vibe coding that surprised you?
2. What part of PRD writing feels most challenging?

---

## Resources

### Required Reading
- [Karpathy's original tweet](https://x.com/karpathy/status/1886192184808149383)
- [Simon Willison: Not all AI-assisted programming is vibe coding](https://simonwillison.net/2025/Mar/19/vibe-coding/)

### Course Templates
- [PRD Template](../../templates/prd-template.md)

### Tool Documentation
- [Gemini CLI Documentation](https://ai.google.dev/gemini-cli)
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)

---

**Instructor:** Goker Ezberci | gokerez@gmail.com
