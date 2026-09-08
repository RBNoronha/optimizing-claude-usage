# Optimizing Claude Code Usage & Prompting Skill ⚡

![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)
![Claude Code](https://img.shields.io/badge/Claude_Code-Plugin-orange.svg)

*( 🇧🇷 Para Português, veja [README.pt-br.md](README.pt-br.md) )*

A specialized plugin and skill for **Claude Code**, meticulously designed to manage your token budget, shield the **Prompt Cache**, calibrate models/effort, and keep the context window surgical and clean during your development sessions.

**Claude Code** is a powerful agentic coding tool, but continuous usage in complex projects can quickly lead to context bloat, unnecessary API costs (due to continuous token reprocessing), and execution confusion if the context window isn't well managed.

This skill acts as a built-in "prompt engineering and cost mentor" for your agent, teaching Claude to be more economical, direct, and efficient while working for you.

---

## 📚 Foundation & Philosophy

The development of this skill is not based on guesswork, but on **Anthropic's** official documentation and engineering guidelines. It embodies the best practices recommended in the following foundational guides:

1. **Session Efficiency & Costs**
   - [Maximizing the value of your Claude Code sessions](https://claude.com/blog/maximizing-the-value-of-your-claude-code-sessions): Core guide on how to avoid expensive reprocessing and when to use commands like `/compact` and `/rewind`.
2. **Advanced Tool Use**
   - [Advanced Tool Use: Search, Programmatic Calling & Examples](https://www.anthropic.com/engineering/advanced-tool-use): Guidelines on how the model should interact with the terminal and file system optimally (batching, source filtering).
3. **Prompt Engineering & Model Guidance**
   - [Prompting Best Practices & Model Guidance](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices): Rules for structuring prompts using XML tags and eliminating preambles.
   - Specific recommendations per reasoning model, addressing how to handle verbosity and contain loops:
     - [Prompting Claude Fable 5 & 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1)
     - [Prompting Claude Opus 4.8 & 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5)
     - [Prompting Claude Sonnet 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5)
4. **Skill Authoring Best Practices**
   - Built to comply fully with *The Complete Guide to Building Skills for Claude*, featuring Progressive Disclosure (nested reference files), explicit conversational examples, and structured troubleshooting blocks.

---

## 📦 Installation (Recommended via Claude Code)

You can install directly using Claude Code's native plugin/marketplace system:

### 1. Add the Marketplace
In the Claude Code chat (or terminal):
```bash
/plugin marketplace add RBNoronha/optimizing-claude-usage
```
*Or via conventional terminal:*
```bash
claude plugin marketplace add RBNoronha/optimizing-claude-usage
```

### 2. Install the Skill / Plugin
```bash
/plugin install optimizing-claude-usage@optimizing-claude-usage
```
*Or via conventional terminal:*
```bash
claude plugin install optimizing-claude-usage@optimizing-claude-usage
```

---

## 🚀 What the Skill Does (The 5 Optimization Levers)

With the skill installed and active, Claude Code strictly follows **5 strategic levers for economy and efficiency**:

### 1. Prompt Cache Shielding (Up to 90% Token Discount)
The Claude API offers huge discounts for unchanged contexts (Prompt Caching). The skill teaches Claude not to break this cache unnecessarily.
- **Lock Parameters Early:** Changing `/model`, `/effort`, or toggling *Fast mode* mid-conversation invalidates the prefix cache, causing the entire history to be reprocessed at full cost. The skill instructs locking these parameters at session start.
- **Strategic `/compact` Timing:** The prompt cache expires after 1 hour. Compiling and summarizing the history requires reading the entire conversation; the skill advises running `/compact` **before long breaks**, taking advantage of the still "warm" cache at a 90% discount.
- **Using `/rewind` over `/compact`:** For recent deviations or errors, it directs pruning bad turns with `/rewind`, keeping the previous history 100% cached at **zero additional token cost**.

### 2. Context Hygiene & Session Management
A bloated context is the biggest enemy of an LLM's accuracy.
- **Task Isolation with `/clear`:** Prevents dragging context from an already resolved task into a new feature or bugfix.
- **Instruction Auditing with `/context`:** Detects bloated rules in the project guidelines (`CLAUDE.md`) and advises migrating highly repetitive instructions to on-demand skills.
- **Direct Attachment via `@-mentions`:** Forces referencing files like `@src/service.ts`. Without the `@`, Claude spends expensive output tokens calling file system tools or making avoidable grep searches.
- **Surgical Edits (Targeted Diffs):** Avoids rewriting entire 500+ line files when only 5 lines need to change, always focusing on in-place modifications.

### 3. Advanced Tool Management & Eliminating the "MCP Tax"
MCP (Model Context Protocol) tools add functionality, but their schemas consume invisible tokens at the start of every prompt.
- **Combating MCP Bloat:** MCP servers (GitHub, Slack, Jira, etc.) can inject massive amounts of schema tokens before you even type anything. The skill teaches Claude to suggest disabling unnecessary MCPs via `/mcp` or strict flags (`--strict-mcp-config`).
- **Programmatic Calling & Source Filtering:** Prohibits injecting massive raw logs or database dumps directly into the chat. Directs using bash pipelines (`grep`, `awk`, `head`) or temporary scripts so only filtered information enters the context.
- **Tool-Call Batching:** Groups reads and inspections of multiple independent files in the same turn, reducing the number of requests sent to the API and speeding up the response.
- **Quiet Flags for Commands:** Enforces the use of compact flags in tests and linters (`--reporter=dot`, `-q`), preventing hundreds of success lines from polluting the history.

### 4. Model and Effort Calibration by Scenario
Classifies the task and prevents the *"start cheap and escalate on failure"* trap (because the cost of diagnosing corrupted code is much higher than using the correct model from the start):
- **Claude Fable 5.1 / Mythos 5.1:** Ideal for multi-day tasks, autonomous agent development, and end-to-end generation with explicit compaction rules.
- **Claude Opus 5 / Opus 4.8:** For complex architectural refactors and highly sensitive code reviews, where all specs must be "up-front" and over-verification loops must be contained.
- **Claude Sonnet 5:** Recommended for agile daily development, unit test creation, small bugfixes, and new features with strong verbosity control.
- **Claude Haiku:** The "tractor" model, excellent for mechanical data extraction, repetitive boilerplate creation, and fast investigations (triage) operating as a subagent.

### 5. Prompting Precision & Preamble Elimination
How Claude thinks and delivers results to the user:
- Cuts empty conversational introductions (*"Certainly, I will analyze and do what you asked..."*), unnecessary greetings, and repetitions of the original question. The focus is on generating only code and truly actionable explanations.
- Rigidly structures complex contexts, always using semantic XML tags (`<context>`, `<specification>`, `<examples>`), which maximizes assertiveness when the context window is full.

---

## 🛠️ How the Skill is Triggered in Claude Code

1. **Automatic Trigger (Background):**  
   Claude Code detects the installed skill and consults it automatically during moments of context stress, for example:
   - When the context starts to exceed the safety margin (~150k tokens).
   - When the session has been running for several consecutive hours without history compaction.
   - When it notices multiple non-essential MCP servers are active, generating high passive overhead.

2. **Manual Trigger (You in Control):**  
   You can explicitly direct Claude to adopt the optimization stance at any moment:
   > *"Claude, review our token consumption and session hygiene based on the optimizing-claude-usage skill."*
   > *"I'm going to start a major refactor now. Strictly follow the guidelines from optimizing-claude-usage so we don't blow the cache unnecessarily."*

---

## 📂 Alternative Manual Installation (Without Marketplace)

If you don't want to add the repository marketplace, you can simply download and save the skill directly to your local Claude Code configuration folder:

**Mac / Linux:**
```bash
mkdir -p ~/.claude/skills/optimizing-claude-usage
curl -L -o ~/.claude/skills/optimizing-claude-usage/SKILL.md https://raw.githubusercontent.com/RBNoronha/optimizing-claude-usage/main/SKILL.md
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path $HOME\.claude\skills\optimizing-claude-usage
Invoke-WebRequest -Uri https://raw.githubusercontent.com/RBNoronha/optimizing-claude-usage/main/SKILL.md -OutFile $HOME\.claude\skills\optimizing-claude-usage\SKILL.md
```
