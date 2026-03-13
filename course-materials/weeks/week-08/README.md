# Week 8: Skills, Agents & Automation — Building Reusable AI Workflows

**Vibe Coding 101** | Instructor: Goker Ezberci | Audience: Software Engineering Students

---

## Overview

This week, we shift from treating AI as a one-shot question-answer tool to building **reusable AI workflows** that automate your actual dev tasks. You'll learn to compose multiple AI tools into coordinated agents that handle the tedious, repetitive work engineers do every single day.

### The Big Idea

Every software engineer does these tasks repeatedly:
- Run tests before committing
- Check for linting errors
- Write commit messages
- Generate PR descriptions
- Deploy applications
- Check deployment status

Instead of doing these manually (or copy-pasting explanations to AI), we'll build a **Personal DevOps Agent**—a CLI tool that automates your entire workflow. This is practical: the patterns you learn this week will save you hours every month for the rest of your career.

---

## 1. From One-Shot to Workflow

### The Traditional Approach (One-Shot)
```
Student: "AI, write me a function to parse JSON"
AI: [returns code]
Student: paste, use it, move on
```

This works for simple tasks. But what about workflows?

### The Workflow Approach (Reusable Systems)
```
Developer: "Run my CI/CD workflow"
System:
  → Check git status
  → Run tests
  → Lint code
  → If all pass: generate commit message
  → Create PR with description
  [All automated, reproducible, composable]
```

**Key shift**: From "ask AI for help on one task" to "build systems where AI is a component that repeats reliably."

### When to Build Workflows

Build a workflow when:
1. You repeat the same set of steps regularly
2. The steps follow a consistent pattern
3. You want the same quality/format every time
4. The task involves multiple tools or external services

---

## 2. Claude Code Skills

Claude Code lets you create custom **Skills**—reusable commands that Claude auto-invokes when relevant.

### What's a Skill?

A skill is a SKILL.md file that:
- Defines a custom command (e.g., `/commit-message`)
- Describes what it does and when to use it
- Shows examples of input/output
- Can call tools, scripts, or external APIs

### Example: Commit Message Generator Skill

Create a file `~/.claude/skills/commit-message/SKILL.md`:

```markdown
# Commit Message Skill

## Command
/commit-message

## Description
Reads staged git changes and generates a conventional commit message.

## When to Use
- After staging files with `git add`
- Before running `git commit`

## Example
Input: (Unstaged changes to auth.js and tests/auth.test.js)
Output: feat(auth): add JWT token refresh mechanism

## How It Works
1. Runs `git diff --cached` to see staged changes
2. Passes diff to Claude with commit guidelines
3. Returns formatted message following Conventional Commits
```

### How Claude Code Invokes It

Once registered, you can use it in Claude Code:

```
You: "Help me commit my work"
Claude: [auto-invokes /commit-message]
Claude: [reads git diff]
Claude: Suggests: "feat(auth): add JWT refresh token logic"
```

**Key Benefit**: Consistent commit messages without context-switching to AI chat.

**Reference**: https://code.claude.com/docs/en/skills

---

## 3. MCP (Model Context Protocol)

MCP is an open standard for connecting AI systems to external tools, data sources, and templates.

### Three Core Primitives

#### 1. **Tools** (Functions/APIs)
```json
{
  "name": "run_tests",
  "description": "Execute test suite and return results",
  "inputSchema": {
    "type": "object",
    "properties": {
      "test_path": {"type": "string", "description": "Path to test file"}
    }
  }
}
```

#### 2. **Resources** (Data/State)
```json
{
  "name": "git_status",
  "mimeType": "text/plain",
  "description": "Current git repository status and staged files"
}
```

#### 3. **Prompts** (Templates)
```json
{
  "name": "pr_description_template",
  "description": "Template for generating PR descriptions",
  "arguments": [{"name": "commits", "description": "List of commit messages"}]
}
```

### Building a Simple MCP Server

Here's a minimal Node.js MCP server that exposes a "run-tests" tool:

```javascript
// test-runner-mcp.js
const stdio = require("stdio");

const server = new stdio.Server({
  tools: [
    {
      name: "run_tests",
      description: "Run test suite",
      inputSchema: {
        type: "object",
        properties: {
          testPath: {
            type: "string",
            description: "Test file or directory (default: all tests)"
          }
        },
        required: []
      },
      handler: async (input) => {
        const { testPath = "tests" } = input;
        const { execSync } = require("child_process");

        try {
          const output = execSync(`npm test -- ${testPath}`, {
            encoding: "utf-8"
          });
          return {
            status: "success",
            output: output,
            passed: output.includes("tests passed")
          };
        } catch (error) {
          return {
            status: "failure",
            error: error.message,
            output: error.stdout || ""
          };
        }
      }
    }
  ]
});

server.start();
```

### Why MCP Matters

- **Decoupling**: AI system doesn't need to know implementation details
- **Reusability**: Same tools work across different AI apps
- **Safety**: Explicit tool definitions prevent hallucinations
- **Composability**: Chain multiple MCP servers together

**Reference**: https://modelcontextprotocol.io/

---

## 4. Agent Design Patterns (Andrew Ng's Framework)

Andrew Ng identifies four core agent patterns. Let's see each with a DevOps Agent example.

### Pattern 1: Reflection

**Concept**: AI reviews its own output and iterates.

**Example: Commit Message Validator**
```
Stage 1 - Generate:
  Input: [git diff]
  Output: "Update authentication module"

Stage 2 - Reflect:
  Question: "Is this message descriptive enough?"
  Review: "Missing details about what changed"

Stage 3 - Improve:
  Output: "feat(auth): add JWT refresh token with expiry validation"
```

**Code Pattern**:
```python
def generate_commit_message(diff):
    # Step 1: Generate
    message = ai.generate(f"Write commit message for: {diff}")

    # Step 2: Reflect
    is_good = ai.evaluate(f"Is this good? {message}")

    # Step 3: Iterate if needed
    if not is_good:
        message = ai.improve(f"Improve this: {message}")

    return message
```

### Pattern 2: Tool Use

**Concept**: AI calls external APIs and services to gather info and take action.

**Example: Test-Driven Commit**
```
User: "Commit my changes"

Agent workflow:
  1. Tool: git_diff() → get staged changes
  2. Tool: run_tests() → verify tests pass
  3. Tool: lint_check() → verify code style
  4. Tool: generate_commit_message(diff) → create message
  5. Tool: git_commit(message) → commit
```

**Code Pattern**:
```python
def test_driven_commit():
    diff = tools.git_diff()

    # Verify quality first
    test_result = tools.run_tests()
    if not test_result.passed:
        return "❌ Tests failing. Fix before committing."

    lint_result = tools.lint_check()
    if lint_result.issues:
        return "❌ Linting errors found. Fix before committing."

    # Generate and commit
    message = tools.generate_commit_message(diff)
    tools.git_commit(message)
    return f"✅ Committed: {message}"
```

### Pattern 3: Planning

**Concept**: AI breaks complex tasks into sequential steps and tracks progress.

**Example: Full CI/CD Workflow**
```
User: "Deploy the app"

Agent Plan:
  Step 1: Verify all changes are committed
  Step 2: Run full test suite
  Step 3: Run security scan
  Step 4: Build Docker image
  Step 5: Push to registry
  Step 6: Update deployment config
  Step 7: Deploy to staging
  Step 8: Run smoke tests
  Status: Step 3/8 - Running security scan...
```

**Code Pattern**:
```python
def deploy_workflow():
    steps = [
        ("Check git status", check_git_clean),
        ("Run tests", run_tests),
        ("Security scan", run_security_scan),
        ("Build image", build_docker_image),
        ("Push image", push_to_registry),
        ("Deploy staging", deploy_staging),
        ("Smoke tests", run_smoke_tests),
    ]

    for i, (desc, func) in enumerate(steps, 1):
        print(f"Step {i}/{len(steps)}: {desc}...")
        result = func()
        if not result.success:
            print(f"Failed at step {i}")
            return result

    print("✅ Deployment complete!")
```

### Pattern 4: Multi-Agent

**Concept**: Multiple specialized AIs coordinate to solve a problem.

**Example: DevOps Team**
```
Workflow: Create PR

Agents:
  - Code Reviewer Agent: "Is the code quality good?"
  - Test Agent: "Do all tests pass?"
  - Security Agent: "Are there vulnerabilities?"
  - Documentation Agent: "Is it properly documented?"
  - Coordinator Agent: "All checks passed. Create PR."
```

**Code Pattern**:
```python
def create_pr_with_review(branch):
    reviewers = [
        CodeReviewerAgent(),
        TestAgent(),
        SecurityAgent(),
        DocAgent()
    ]

    results = {}
    for reviewer in reviewers:
        results[reviewer.name] = reviewer.evaluate(branch)

    all_passed = all(r.passed for r in results.values())

    if all_passed:
        CoordinatorAgent().create_pr(branch, results)
    else:
        print("Review failed:")
        for name, result in results.items():
            if not result.passed:
                print(f"  ❌ {name}: {result.message}")
```

**Reference**: https://www.deeplearning.ai/courses/agentic-ai/

---

## 5. Building the Personal DevOps Agent

Now let's build a complete example: a CLI tool that automates your daily workflow.

### Architecture

```
┌─────────────────────────────────────────┐
│         DevOps Agent CLI                │
│  $ devops-agent --workflow commit       │
└──────────────┬──────────────────────────┘
               │
       ┌───────┴──────────┬──────────────┐
       │                  │              │
    ┌──▼──┐          ┌───▼──┐      ┌───▼───┐
    │ Git │          │Tests │      │ Lint  │
    │Tool │          │Tool  │      │Tool   │
    └─────┘          └──────┘      └───────┘
       │                  │              │
       └───────┬──────────┴──────────────┘
               │
        ┌──────▼────────┐
        │ Claude Agent  │
        │(via MCP/API)  │
        └──────┬────────┘
               │
        ┌──────▼──────────────┐
        │Generate Commit Msg  │
        │& Create PR          │
        └─────────────────────┘
```

### Step 1: Git Status Reader

```bash
#!/bin/bash
# tools/git-status.sh

echo "Staged changes:"
git diff --cached --stat

echo -e "\nChanged files:"
git diff --cached --name-only

echo -e "\nFull diff:"
git diff --cached
```

```python
# agent.py - integrate into agent
def get_git_status():
    """Tool 1: Read what changed"""
    import subprocess
    result = subprocess.run(
        ["bash", "tools/git-status.sh"],
        capture_output=True,
        text=True
    )
    return result.stdout
```

### Step 2: Test Runner

```bash
#!/bin/bash
# tools/run-tests.sh

echo "Running tests..."
npm test 2>&1

if [ $? -eq 0 ]; then
    echo "✅ All tests passed"
    exit 0
else
    echo "❌ Tests failed"
    exit 1
fi
```

```python
# agent.py - add test tool
def run_tests():
    """Tool 2: Verify tests pass"""
    result = subprocess.run(
        ["bash", "tools/run-tests.sh"],
        capture_output=True,
        text=True
    )
    return {
        "success": result.returncode == 0,
        "output": result.stdout
    }
```

### Step 3: Commit Message Generator

```python
# agent.py - call Claude to generate message
def generate_commit_message(git_diff):
    """Tool 3: AI generates commit message"""
    response = client.messages.create(
        model="claude-opus-4-6",
        max_tokens=100,
        system="""You are a commit message expert.
        Generate a single conventional commit message (type(scope): description).
        Be specific and concise. Do NOT output multiple options, just one message.""",
        messages=[
            {
                "role": "user",
                "content": f"Generate a commit message for:\n\n{git_diff}"
            }
        ]
    )
    return response.content[0].text.strip()
```

### Step 4: PR Description Writer

```python
# agent.py - comprehensive workflow
def create_pr_description(commits):
    """Tool 4: Summarize commits into PR description"""
    response = client.messages.create(
        model="claude-opus-4-6",
        max_tokens=500,
        system="""Write a professional PR description.
        Include: Summary, Changes Made, Testing Done, Breaking Changes (if any).
        Format as markdown.""",
        messages=[
            {
                "role": "user",
                "content": f"Create PR description for commits:\n{commits}"
            }
        ]
    )
    return response.content[0].text
```

### Complete Workflow Integration

```python
# devops_agent.py - full orchestration
import subprocess
import sys
from anthropic import Anthropic

client = Anthropic()

def devops_workflow():
    """Complete CI/CD workflow: test → lint → commit → pr"""

    # Step 1: Get status
    print("📋 Checking git status...")
    git_diff = get_git_status()
    if not git_diff:
        print("No staged changes. Stage files first.")
        return

    # Step 2: Run tests
    print("🧪 Running tests...")
    test_result = run_tests()
    if not test_result["success"]:
        print("❌ Tests failed. Fix issues before committing.")
        print(test_result["output"])
        return
    print("✅ Tests passed")

    # Step 3: Generate commit message
    print("💬 Generating commit message...")
    message = generate_commit_message(git_diff)
    print(f"Suggested: {message}")

    # Step 4: Commit
    confirm = input("Create commit? (y/n) ")
    if confirm.lower() == 'y':
        subprocess.run(["git", "commit", "-m", message])
        print("✅ Committed!")

    # Step 5: Generate PR description
    print("📄 Generating PR description...")
    commits = subprocess.run(
        ["git", "log", "--oneline", "-n", "5"],
        capture_output=True,
        text=True
    ).stdout

    pr_desc = create_pr_description(commits)
    print("\nPR Description:")
    print(pr_desc)

if __name__ == "__main__":
    devops_workflow()
```

---

## 6. Agent Orchestration Frameworks

When building complex agents, specialized frameworks help. Here's a comparison:

### LangGraph (State Graphs)

Best for: Multi-step workflows with state management.

```python
from langgraph.graph import StateGraph

workflow = StateGraph()

# Define nodes (steps)
workflow.add_node("check_tests", run_tests)
workflow.add_node("generate_message", generate_message)
workflow.add_node("create_commit", create_commit)

# Define edges (transitions)
workflow.add_edge("check_tests", "generate_message")
workflow.add_edge("generate_message", "create_commit")

# Run workflow
result = workflow.invoke({"diff": git_diff})
```

**Strengths**: Visual state graphs, conditional routing, history tracking.

**Reference**: https://langgraph.com/

### CrewAI (Role-Based Teams)

Best for: Multi-agent coordination with specialized roles.

```python
from crewai import Agent, Task, Crew

# Define agents with roles
reviewer = Agent(
    role="Code Reviewer",
    goal="Review code quality",
    tools=[review_tool]
)

tester = Agent(
    role="Test Engineer",
    goal="Run tests and report",
    tools=[test_tool]
)

# Define tasks
review_task = Task(
    description="Review the code changes",
    agent=reviewer
)

# Create crew
crew = Crew(agents=[reviewer, tester], tasks=[review_task])
crew.kickoff()
```

**Strengths**: Human-like agent personalities, role clarity, tool delegation.

**Reference**: https://github.com/crewAIInc/crewAI

### When to Use Each

| Framework | Use When | Example |
|-----------|----------|---------|
| LangGraph | Sequential steps with branching logic | Test → lint → commit workflow |
| CrewAI | Multiple specialized agents coordinating | Code review + testing + deployment team |
| Custom (Direct API) | Simple workflows, learning | Personal DevOps Agent for this course |

**Reference**: https://www.deeplearning.ai/courses/agentic-ai/

---

## 7. Lab Exercise

Build your own automation workflow in three parts.

### Part A: Create a Custom Claude Code Skill

Create a file `~/.claude/skills/[your-skill]/SKILL.md`:

```markdown
# [Your Skill Name]

## Command
/[your-command]

## Description
[What your skill does and why it's useful]

## When to Use
[Specific scenarios or triggers]

## Example
Input: [Example input]
Output: [Example output]

## Implementation Notes
[How it works internally]
```

**Examples to choose from**:
- `/lint-report`: Analyzes linting errors and suggests fixes
- `/test-summary`: Runs tests and summarizes failures
- `/code-review`: Reviews code for best practices
- `/api-docs`: Generates API documentation from code

### Part B: Build One MCP Tool

Choose one:
1. **run-tests** — Execute test suite, return results
2. **lint-check** — Run linter, report issues
3. **deploy-status** — Check deployment status
4. **git-diff** — Retrieve staged changes

Create a Node.js/Python script that:
- Accepts input parameters
- Executes the tool logic
- Returns structured output
- Handles errors gracefully

### Part C: Chain Tools Into a Workflow

Create a CLI script that:
1. Calls 2+ tools in sequence
2. Passes output from one tool to the next
3. Uses AI (via Claude API) to make decisions
4. Produces a final, automated result

**Example workflow**:
```
User: "npm run deploy"
Workflow:
  1. Git tool: Check if repo is clean
  2. Test tool: Run test suite
  3. Lint tool: Check code style
  4. Build tool: Create production build
  5. Deploy tool: Push to server
  6. Verify tool: Check deployment health

Output: ✅ Deployed successfully to production
```

### Submission Checklist

- [ ] Part A: SKILL.md file created and documented
- [ ] Part B: Working MCP tool with proper schema
- [ ] Part C: Complete workflow that chains 2+ tools
- [ ] All tools tested and working
- [ ] Clear error handling
- [ ] Comments explaining each step

---

## Key Takeaways

1. **Workflows > One-shots**: Reusable systems beat one-time AI queries
2. **Skills + MCP**: Combine Claude Code Skills with MCP tools for powerful automation
3. **Patterns matter**: Reflection, Tool Use, Planning, Multi-Agent are your building blocks
4. **Practical automation**: Your DevOps Agent saves hours every month
5. **Composition**: Chain simple tools into complex workflows

---

## Resources

- **Anthropic MCP**: https://modelcontextprotocol.io/
- **Claude Code Skills**: https://code.claude.com/docs/en/skills
- **Andrew Ng - Agentic AI**: https://www.deeplearning.ai/courses/agentic-ai/
- **LangGraph**: https://langgraph.com/
- **CrewAI**: https://github.com/crewAIInc/crewAI
- **Code Execution with MCP**: https://www.anthropic.com/engineering/code-execution-with-mcp

---

**Week 8 Complete.** Next week: Advanced patterns, production deployment, and real-world agent systems.
