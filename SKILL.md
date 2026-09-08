---
name: optimizing-claude-usage
description: Audits Claude usage/budget and applies cost-saving practices. Use when context passes ~150k tokens, sessions run for hours, or subagents spawn reflexively. Covers prompt caching, context hygiene, MCP reduction, and batching.
---

# Optimizing Usage & Prompting Best Practices

## Important: Progressive Disclosure
For a deep dive into the 5 optimization levers (Prompt Caching, Context Hygiene, MCP Management, Model Calibration, Prompting Precision), read `references/optimization-patterns.md`.

## Core Instructions

## Session Optimization Workflow

Copy this checklist into your response and track your progress when auditing a session:

```
Optimization Progress:
- [ ] Step 1: Pre-Flight Context Audit
- [ ] Step 2: Session Hygiene Enforcement
- [ ] Step 3: Tool Execution & Output Control
- [ ] Step 4: Batching & Delegation
```

### Step 1: Pre-Flight Context Audit
When starting a task or when prompted to audit:
- Check active MCP servers via `/context` or `/mcp`. Advise disabling unused servers to reduce token overhead.
- Verify `/model` and `/effort` are appropriate for the task complexity and lock them in. Changing them mid-task breaks the prompt cache.

### Step 2: Session Hygiene Enforcement
- **Compaction Strategy:** Run `/compact` BEFORE stepping away for an hour, using the warm cache (90% discount). Do not compact if the session is ending.
- **Rollbacks:** Use `/rewind` instead of `/compact` if the last few turns were mistakes, preserving the cache.
- **Task Boundaries:** Enforce the use of `/clear` between unrelated tasks to prevent context drag.

### Step 3: Tool Execution & Output Control
- **Filter Raw Output:** Never dump raw multi-megabyte logs or API payloads. Use shell pipelines (e.g., `grep`, `awk`, `head`) to filter data at the source.
- **Quiet Flags:** Enforce quiet flags for linters/tests (e.g., `pytest -q`, `--reporter=dot`) to avoid polluting context with passing test logs.
- **Surgical Edits:** Apply targeted diffs rather than rewriting entire files. Output tokens are expensive.

### Step 4: Batching & Delegation
- **Batch Independent Reads:** Read multiple files or check multiple symbols in a single turn.
- **Subagents:** Only spawn subagents for heavy noise tasks (broad greps, large log parsing). Run single file reads or targeted greps inline.

## Examples

Example 1: Long session management
User says: "We've been working on this for a while, let's take a break."
Actions:
1. Advise the user to run `/compact` before the break.
2. Explain that the prompt cache expires in an hour, and compacting now uses the warm cache at a 90% discount.
Result: The user compacts the session affordably.

Example 2: Processing large logs
User says: "Check this 50MB server log for authentication errors."
Actions:
1. Do NOT use `Read` on the entire file.
2. Run an inline shell command: `grep -i "auth error" server.log | tail -n 50`.
Result: Only relevant lines enter the context window.

## Troubleshooting

### Error: "Context window limit reached"
Cause: Accumulation of uncompressed history, bloated MCP schemas, or raw tool outputs.
Solution: 
1. Use `/rewind` to undo recent bulky turns.
2. Advise the user to `/compact` the session.
3. Suggest disabling unused MCP servers.

### Error: "Prompt cache miss (0% cached)" mid-session
Cause: The user or agent changed the model (`/model`), effort (`/effort`), or toggled Fast mode.
Solution: Advise the user to set these parameters at the start of the session and lock them in. Use `/clear` before changing models if possible.
