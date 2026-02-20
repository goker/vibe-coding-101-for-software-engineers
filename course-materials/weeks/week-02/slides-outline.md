# Week 2: How AI Coding Actually Works — Slides Outline

> "The hottest new programming language is English." — Andrej Karpathy

---

## Slide 1: Title

**Week 2: How AI Coding Actually Works**
*Tokenization, Context, Prompts, and Tools*

---

## Slide 2: Pulse Check (Start)

**Before we begin, rate your confidence (1-5):**

1. Do you understand how AI models process your code?
2. How would you explain "context window" to a friend?

---

## Slide 3: Learning Objectives

By the end of this week, you will:

1. Understand tokenization, context windows, and next-token prediction
2. Analyze AI coding tools across categories and tradeoffs
3. Apply prompt engineering and context engineering techniques
4. Evaluate AI-generated code for quality and maintainability

---

## Slide 4: Tokenization — How AI Reads Your Code

AI doesn't see words or characters—it sees **tokens** (subword chunks)

```
"function"              → [function]                    = 1 token
"XMLHttpRequest"        → [XML] [Http] [Request]        = 3 tokens (3x cost!)
const greeting = "Hello World";
  → [const] [_greeting] [_=] [_"] [Hello] [_World] [";] = 7 tokens
```

**Key Insight:** Common words = fewer tokens. Rare/compound words = more tokens.

---

## Slide 5: Context Windows — Why AI "Forgets"

| Model | Context | Notes |
|-------|---------|-------|
| Claude 3.5 Sonnet | 200K tokens | Industry standard |
| Gemini 2.5 Pro | 1M tokens | 5x larger |
| GPT-4 | 128K tokens | Strong reasoning |
| Local models | 8K–32K | Resource-constrained |

**200K tokens ≈ 150,000 words ≈ 300 pages of text**

**Engineering implication:** Start fresh sessions for new features. The AI isn't choosing to forget—the architecture can't see earlier messages once context is full.

---

## Slide 6: Temperature & Sampling

**Temperature** = how random the model is.

```
Temperature = 0.0    Deterministic (always same output)
            0.3    Consistent (minor variations)
            0.7    Balanced (some creativity)
            1.0+   Creative/Chaotic
```

**When to use:**
- **Production code:** 0.0–0.3 (consistency)
- **Brainstorming:** 0.7–1.0 (creativity)

---

## Slide 7: Next-Token Prediction

An LLM doesn't think. It **predicts the next token** repeatedly.

```
Input: "function add(a, b) {"
         ↓
Transformer: "What token should come next?"
         ↓
Output probabilities:
  " return" → 65%
  "\n  return" → 15%
  "const" → 3%
         ↓
Generate: "function add(a, b) { return"
         ↓
Repeat thousands more times...
```

---

## Slide 8: Why Hallucinations Happen

Hallucinations aren't bugs—they're features of next-token prediction.

The model has no "I'm not sure" mechanism. It's always confident.

**Defense:**
1. Know your domain
2. Always test the code
3. Validate APIs against docs
4. Human review required

---

## Slide 9: Where AI Excels

- Boilerplate (React components, API handlers, CRUD)
- Code transformation (refactoring, formatting)
- Generating from specifications (tests from code)
- Documentation (docstrings from signatures)
- Language translation (Python ↔ JavaScript)
- Explaining code

**Why?** High-pattern tasks with thousands of examples in training data.

---

## Slide 10: Where AI Struggles

- Novel algorithms (not in training data)
- Complex math (requires symbolic reasoning)
- Long chains of logic
- Security analysis (global perspective needed)
- Performance optimization (needs measurement)
- Guarantees & proofs

**Rule of thumb:** Use AI for "implement this pattern." Don't use AI for "solve this hard problem."

---

## Slide 11: AI Tool Categories

**IDE-Native:**
- Cursor, Windsurf, GitHub Copilot
- Best for: integrated editing, multi-file changes

**Terminal-Native:**
- Claude Code, Gemini CLI, aider
- Best for: backend, scripting, automation

**Rapid Prototyping:**
- Bolt.new, v0, Lovable, Replit
- Best for: MVPs, hackathons, learning

---

## Slide 12: Tool Selection Matrix

| Task | Best Tool | Why |
|------|-----------|-----|
| Boilerplate API | Cursor/Windsurf | IDE integration |
| Rapid MVP | Bolt.new | Fastest to live |
| Large refactor | Windsurf/Claude | Multi-file, terminal |
| Backend/scripting | Claude Code | No GUI needed |
| Cost-sensitive | Gemini CLI | Free + 1M context |

**The principle:** Tools change every 3 months. Disciplines last decades.

---

## Slide 13: Prompt Engineering Evolution

**Level 1 - Vague:**
```
"Make a website"
```

**Level 2 - Structured:**
```
Create a React todo app with:
- Scope: CRUD, localStorage, dark mode
- Non-Goals: Backend, auth, real-time
- Constraints: <5 KB gzipped
```

**Level 3 - Context Engineering:**
```
[Provide codebase structure]
[Provide style examples]
[Provide constraint files]
Then: "Implement logout endpoint"
```

---

## Slide 14: Pulse Check (End)

**Reflect on today's session:**

1. What surprised you most about how AI generates code?
2. Which tool category fits your workflow best?

---

## Slide 15: What's Next

**Week 3: From PRD to Deployed Prototype**

- Ship something live
- Platform selection
- Debugging deployment failures

---

**Instructor:** Goker Ezberci | gokerez@gmail.com
