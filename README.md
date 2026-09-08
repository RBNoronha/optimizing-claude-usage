# Optimizing Claude Code Usage Skill

A specialized skill for **Claude Code** designed to help you manage your token budget, preserve the prompt cache, and keep your context window clean. 

Based on Anthropic's official best practices for [Maximizing the value of your Claude Code sessions](https://claude.com/blog/maximizing-the-value-of-your-claude-code-sessions), this skill guides Claude (and you) on how to operate efficiently within the CLI environment.

## 🚀 What it does

When this skill is active, it advises Claude on how to:
- Preserve the **Prompt Cache** (which makes context reading 90% cheaper) by avoiding actions that break it.
- Use **Subagents** correctly to isolate "noisy" outputs (like reading large logs) from polluting the main context window.
- Apply **Context Size Management** techniques, such as using `/clear` between tasks, `/context` to audit the prompt, and `@-mentioning` files instead of making expensive `Read` tool calls.
- Optimize **Model Selection** based on the actual difficulty of the task, rather than just defaulting to the cheapest model and escalating on failure.

## 📦 Installation

In Claude Code, skills are simply markdown files placed in a `.claude/skills` directory. You can install this globally (for all your projects) or locally (just for one project).

### Global Installation (Recommended)

To make this skill available across all your Claude Code sessions, add it to your user home directory:

**Mac/Linux:**
```bash
mkdir -p ~/.claude/skills/optimizing-claude-usage
curl -L -o ~/.claude/skills/optimizing-claude-usage/SKILL.md https://raw.githubusercontent.com/RBNoronha/optimizing-claude-usage-skill/main/SKILL.md
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path $HOME\.claude\skills\optimizing-claude-usage
Invoke-WebRequest -Uri https://raw.githubusercontent.com/RBNoronha/optimizing-claude-usage-skill/main/SKILL.md -OutFile $HOME\.claude\skills\optimizing-claude-usage\SKILL.md
```

### Local Installation (Per Project)

To use it only in a specific project, run this inside the project's root folder:

**Mac/Linux:**
```bash
mkdir -p .claude/skills/optimizing-claude-usage
curl -L -o .claude/skills/optimizing-claude-usage/SKILL.md https://raw.githubusercontent.com/RBNoronha/optimizing-claude-usage-skill/main/SKILL.md
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path .claude\skills\optimizing-claude-usage
Invoke-WebRequest -Uri https://raw.githubusercontent.com/RBNoronha/optimizing-claude-usage-skill/main/SKILL.md -OutFile .claude\skills\optimizing-claude-usage\SKILL.md
```

## 🛠️ How it works

Once installed, Claude Code will automatically detect the skill. It is configured to trigger automatically when:
- The context window is trending past ~150k tokens.
- A session has run for hours without a clear phase boundary.
- Subagents are being spawned reflexively for trivial tasks.
- The same large content (docs, logs, specs) is being re-sent across turns.

You can also explicitly invoke it by telling Claude Code:
> "Review your usage based on the optimizing-claude-usage skill"

## 💡 Top Tips for Claude Code Users
1. **Set `/model` and `/effort` up front.** Changing them mid-session breaks the prompt cache!
2. **Type `/compact` before you take a coffee break.** The cache expires after 1 hour. Compacting while the cache is warm is extremely cheap; doing it later costs full price.
3. **Use `/clear` between tasks.** Don't drag old context into a new bug fix.
4. **Use quiet flags for commands.** (e.g., `npm test --reporter=dot`). Long terminal outputs get stuck in your context forever!
