# Week 2: How AI Coding Actually Works

> "The hottest new programming language is English." — Andrej Karpathy

---

## Quick Pulse Check (Start)

Before we begin, rate your confidence (1-5):
1. How well do you understand what happens when an AI generates code?
2. Have you experimented with different AI coding tools?

---

## Learning Objectives

By the end of this week, you will:
- Understand how LLMs generate code (tokenization, context windows, next-token prediction)
- Recognize why AI hallucinations happen and how to prevent them
- Apply prompt engineering techniques to maximize AI effectiveness
- Compare different AI coding tools and select the right one for your task

---

## LLM Fundamentals for Engineers

### Tokenization: The Foundation

A token is not a word—it's a subword unit. Models process and charge by tokens.

```
"function" → 1 token
"XMLHttpRequest" → 4 tokens (XML / Http / Request)
"const myVariable = 42;" → 6 tokens
```

**Why this matters:**
- **Cost scaling**: Verbose prompts = higher cost, fewer tokens left for generation
- **Context efficiency**: Be precise to maximize useful output
- **Model differences**: Different models tokenize differently

### Context Windows: AI's Memory

| Model | Context Window |
|-------|---------------|
| Claude 3.5 Sonnet | 200K tokens |
| Gemini 2.5 Pro | 1M tokens |
| GPT-4 | 128K tokens |

**Practical insight**: By message 50 in a conversation, the AI begins "forgetting" earlier exchanges. Start fresh sessions for unrelated tasks.

### Temperature: Controlling Randomness

```
Temperature 0.0  → Deterministic (same output every time)
Temperature 0.7  → Moderate randomness (default)
Temperature 1.5  → High creativity (unpredictable)
```

**When to use:**
- **Code generation**: Temperature 0.0–0.3 (reproducibility)
- **Brainstorming**: Temperature 0.7–1.0 (creativity)

### Why Hallucinations Happen

The model has no "doubt" mechanism—it's always confident. It predicts the next token based on patterns, not understanding:

```javascript
// AI might generate this with high confidence:
function queryDB(id) {
  return DATABASE.query(`SELECT * FROM users WHERE id = ${id}`)
    .then(users => users[0].encryptionKey) // ← Hallucination!
}
```

**Always validate AI-generated code**, especially for:
- API calls (made-up endpoints)
- Database queries (non-existent columns)
- Library function names (close but wrong)

---

## The AI Coding Tool Landscape

### Tool Categories

| Category | Tools | Best For |
|----------|-------|----------|
| **IDE-Native** | Cursor, Windsurf, GitHub Copilot | Integrated development, tab completion |
| **Terminal-Native** | Gemini CLI, Claude Code, aider | Server-side work, scripting, automation |
| **Rapid Prototyping** | Bolt.new, v0, Lovable, Replit | Full-stack prototypes in minutes |

### Free Options for Students

| Tool | Free Tier |
|------|-----------|
| **Gemini CLI** | 1,500 requests/day (FREE) |
| **GitHub Copilot** | Free with GitHub Student Pack |
| **Cursor Pro** | 1 year free for students |
| **Bolt.new** | 300K daily tokens free |

**The principle**: Tools change every 3 months. Master the discipline, not the tool.

---

## Prompt Engineering for Code

### System Prompts vs. User Prompts

**System Prompt** (persistent context):
```
You are a Senior Software Engineer specializing in TypeScript.
You prioritize: Type safety, error handling, test coverage >85%.
Non-Goals: Don't optimize prematurely.
```

**User Prompt** (specific request):
```
Implement a function that fetches user data and caches the result in Redis for 10 minutes.
```

### The Specificity Spectrum

**Poor:**
```
Build a todo app
```

**Excellent:**
```
Create a React 19 todo app with:

SCOPE:
- Task CRUD operations
- Persistence: localStorage only
- UI: Tailwind CSS, responsive to 320px

NON-GOALS:
- Server synchronization
- User authentication

CONSTRAINTS:
- React Hooks only
- Zero external dependencies except React & Tailwind
```

### Few-Shot Examples

Show the AI examples of code patterns you want:

```javascript
// Style I want:
const [formData, setFormData] = useState({ email: '', password: '' });

const validate = () => {
  const newErrors = {};
  if (!formData.email.includes('@')) newErrors.email = 'Invalid email';
  return Object.keys(newErrors).length === 0;
};

// Now generate a similar hook for...
```

### Context Engineering

**Prompt engineering** = "How do I phrase my request?"
**Context engineering** = "How do I organize ALL information so the AI can't misunderstand?"

Provide:
1. Project structure
2. Existing code patterns
3. Test examples
4. Then ask for the specific change

---

## AI Tool Examples

### Exploring a Codebase

**Gemini CLI:**
```bash
gemini "Read all files in src/ and explain the architecture"
```

**Claude Code:**
```bash
claude "What's the structure of this codebase? List main components."
```

### Generating Code with Constraints

**Gemini CLI:**
```bash
gemini "Generate a login form in React with email validation. Use Tailwind CSS. No external libraries."
```

**Claude Code:**
```bash
claude "Create an Express endpoint for user registration with bcrypt password hashing and input validation."
```

---

## Where AI Excels vs. Struggles

| Strengths (Pattern-matching) | Weaknesses (Reasoning) |
|------------------------------|------------------------|
| Boilerplate code | Novel algorithms |
| Refactoring existing code | Complex math proofs |
| Generating test cases | Long chains of logic |
| Documentation from code | Security vulnerability discovery |
| Fixing syntax errors | Performance optimization |

**Rule of thumb**: AI is a search-by-pattern tool, not a reasoning engine. Give it patterns to follow.

---

## Quick Pulse Check (End)

Reflect on today's session:
1. What's one thing about how AI generates code that surprised you?
2. Which AI coding tool do you want to try first?

---

## Resources

### Fundamental Papers
- **"Attention Is All You Need"** (Vaswani et al., 2017) — The transformer architecture
- **"Language Models are Few-Shot Learners"** (Brown et al., 2020) — Why prompting works

### Prompt Engineering
- [Anthropic's Prompt Engineering Guide](https://docs.anthropic.com/en/docs/build-a-chatbook/prompt-engineering)
- [OpenAI's Prompt Engineering Best Practices](https://platform.openai.com/docs/guides/prompt-engineering)

### Tool Documentation
- [Gemini CLI Documentation](https://ai.google.dev/gemini-cli)
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Cursor Documentation](https://docs.cursor.sh/)

---

**Instructor:** Goker Ezberci | gokerez@gmail.com
