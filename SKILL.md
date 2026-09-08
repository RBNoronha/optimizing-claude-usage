---
name: optimizing-claude-usage
description: Use when managing Claude usage/budget — context is trending past ~150k tokens, a session has run for hours, subagents are being spawned reflexively, or the same large content is being re-sent. Focuses on prompt cache preservation, context size, and model-task fit.
---

# Optimizing Claude Usage

## Overview

Reduce token consumption and redundant API calls without degrading output quality. The cost of a session is driven by **what you put in the context**, **how long it stays there**, and whether you preserve the **prompt cache**. 

**Core principle:** the cheapest token is the one you never resend. Most waste comes from carrying stale or irrelevant context forward, not from any single expensive call.

## When to use this skill

- Context is approaching ~150k tokens and there's a natural phase boundary (feature done, bug fixed, investigation concluded).
- A session has been running long enough that you can no longer summarize in one sentence what's still relevant from early in the conversation.
- You're about to spawn a subagent for something that's actually a single, trivial operation.
- The same large artifact (a spec, a log dump, a big file) has been pasted or read more than once in the conversation.

**Don't use this skill to:**
- Justify skipping context a task genuinely needs (full codebase review, large log correlation, image analysis).
- Force premature model downgrades on tasks with real ambiguity or multi-step reasoning.

## Lever 1: Context Size & Session Management

Every turn resends the full conversation history (or its compacted form) as input. Everything added to the context (files read, command outputs) gets sent back on every subsequent turn.

**What actually reduces this:**
- **Run `/clear` between tasks.** Don't drag old context into new, unrelated work. Start fresh. (You can `/rename` the session before clearing if you want to keep the history).
- **Audit your context with `/context`.** Run this in a fresh session to see what loads by default. Keep `CLAUDE.md` to specific instructions and move workflow-specific ones into skills. Disable unused MCP servers with `/mcp`.
- **@-mention files instead of typing their names.** When you say "fix the test in `@utils.test.ts`", Claude Code attaches the file to your first message. If you just type the name, Claude has to spend expensive output tokens to make a `Read` tool call, and then input tokens to read it.
- **Use `/rewind` instead of `/compact` for immediate mistakes.** If the last few turns went in the wrong direction, `/rewind` cuts them off without breaking the cache for the turns before them. `/compact` rewrites the whole conversation, which always costs something.

## Lever 2: Prompt Cache Preservation

Prompt caching makes re-reading context 90% cheaper. The cache matches your context from the beginning of the request. Changing anything at the front breaks the cache for everything after it.

- **Set your `/model` and `/effort` before you start.** Switching them mid-conversation busts the cache, forcing the entire history to be re-prefilled at full price. (Note: turning Fast mode off is free, but turning it on mid-session breaks the cache).
- **Compact at the right time:**
  - **Before you step away:** The prompt cache expires after 1 hour (on a subscription) or 5 minutes (API key). Summarizing a conversation with `/compact` requires the model to read the whole history. If you do this *before* a break while the cache is warm, the read is cheap. After a break, it happens at full price.
  - **At phase boundaries:** Compact when a feature ships or bug is fixed, but *only* if there's meaningfully more work left in the session to amortize the compaction cost against. Compacting right before ending a session wastes tokens.

## Lever 3: Subagents & Noisy Commands

**Command outputs are added to the conversation just like files.** A test runner that prints 400 passing lines adds those lines to every remaining turn.

- **Add quiet flags to commands.** Run tests with `npx vitest run <file> --reporter=dot`. Put these command templates in your `CLAUDE.md` so you always use them.
- **Use subagents to protect context, not avoid cost.** A subagent does not inherit the parent conversation's history. It starts clean. Use a subagent when:
  - The work would dump intermediate noise (broad searches, log dumps) into the main context that you don't need to keep.
  - The work is parallelizable.
  - For repeated noisy jobs, define a specific subagent with a smaller model like Haiku.
- **Do it inline instead when:** It's one grep, one file read, or one small edit. Briefing a subagent would cost more than just doing it.

## Lever 4: Model Selection

Match the model to the task's actual difficulty — but choose deliberately, don't default to the cheapest model and escalate on failure.

| Task profile | Reasonable default |
|---|---|
| Mechanical: formatting, extraction, simple lookups, boilerplate | Smallest capable model |
| Standard engineering: refactors, typical bug fixes, well-specified features | Mid-tier model |
| Ambiguous requirements, multi-step reasoning, architecture decisions | Strongest available model |

**Why not "always start cheap and escalate on failure":** Every failed attempt still costs tokens, plus your time diagnosing it. A wrong answer that ships silently is worse than an expensive correct one. Pick the tier that matches the task up front.

## Decision checklist

**Before compacting:**
- [ ] Is there enough remaining work in this session to amortize the compaction's own cost?
- [ ] Am I about to take a break long enough to let the prompt cache expire?

**Before spawning a subagent:**
- [ ] Would this work dump a lot of raw intermediate output into my main context?
- [ ] Can I brief it in less effort than doing the work inline?

**Before picking a model:**
- [ ] What's the actual difficulty of this task?
- [ ] If I'm tempted to start cheap, how expensive is it to detect and recover from a wrong answer?

## Common mistakes

| Mistake | Why it fails | Fix |
|---|---|---|
| Changing `/model` mid-session | Busts the prompt cache, causing cost spikes | Set model up front |
| Compacting at the end of a session | Compaction has an upfront cost; with no work left, it's wasted | Compact at phase boundaries or before breaks |
| Avoiding subagents to "save context" | Subagents isolate the parent from raw output — avoiding them keeps noise in your main context | Use subagents for broad/parallel work |
| Always starting with the cheapest model | A subtly wrong answer can cost more than tokens saved | Match model to task difficulty up front |
| Re-pasting the same document | Duplicates tokens | Reference what's already in the conversation or use `@-mentions` |
