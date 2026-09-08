---
name: optimizing-claude-usage
description: Use when managing Claude usage/budget — context is trending past ~150k tokens, a session has run for hours, subagents are being spawned reflexively, or the same large content is being re-sent. Covers prompt cache preservation, context hygiene, effort calibration, tool-call batching, subagent controls, and model-specific nuances (Sonnet 5, Opus 5, Fable 5.1).
---

# Optimizing Claude Usage & Prompting Best Practices

## Overview

Reduce token consumption and redundant API/CLI calls without degrading output quality. The economics of a Claude Code session are driven by three interdependent variables:
1. **Context size:** How much text you force the model to re-read on each turn.
2. **Prompt cache preservation:** Keeping the prefix intact so history reads run at ~10% cost.
3. **Model & effort fit:** Matching both the model tier and thinking budget to actual task complexity.

**Core principle:** The cheapest token is the one you never resend. Most waste comes from carrying stale or irrelevant context forward and breaking prompt caching mid-flight.

---

## When to use this skill

- Context window is approaching ~150k tokens and a logical milestone is reached.
- A session has been running across multiple disparate tasks or features.
- Subagents are being spawned reflexively for trivial, one-step lookups.
- Large artifacts (logs, bundle outputs, full test suites, specs) are accumulating in history.
- Switching between different models (Sonnet 5, Opus 5, Fable 5.1) or tuning thinking effort.

**Don't use this skill to:**
- Skip essential context genuinely needed for multi-file refactors or architectural audits.
- Force premature model downgrades on ambiguous, high-stakes tasks where subtle errors require costly rewrites.

---

## Lever 1: Prompt Cache Preservation (90% Discount)

Prompt caching matches request tokens from the beginning forward. Any modification to the prefix invalidates the cache for all subsequent tokens.

- **Lock `/model` and `/effort` at the start:** Changing either parameter mid-conversation busts the cache, causing the entire conversation history to be re-prefilled at full price. (Note: turning Fast mode off is free, but activating it mid-session forces a re-prefill at Fast mode rates).
- **Time your `/compact` calls strategically:**
  - **Before stepping away:** Prompt cache expires after 1 hour (subscription) or 5 minutes (API key). Compaction requires reading the entire history once to write the summary. Running `/compact` while the cache is warm is up to 90% cheaper than compacting after returning from an idle break.
  - **Amortize compaction cost:** Compaction itself has an upfront token cost. Only compact at phase boundaries if there is meaningfully more work left in the session to recoup that cost across subsequent turns. Never compact right before closing a session.
- **Use `/rewind` over `/compact` for rollbacks:** If the last few turns went off track, `/rewind` trims turns from the tail, leaving the earlier cached prefix completely intact and costing 0 extra tokens.

---

## Lever 2: Context Window & Session Hygiene

Every turn resends the entire conversation history (or its compaction summary). Keep what enters the context minimal and precise.

- **Run `/clear` between tasks:** Do not drag context from a completed feature or bugfix into an unrelated task. Use `/rename` if you want to archive the session, then `/clear` to start with an empty context.
- **Audit loaded instructions with `/context`:** Check what is injected at session start (e.g., `CLAUDE.md`, system prompts, active MCP tools). Keep `CLAUDE.md` concise and shift workflow-specific logic into on-demand skills. Disable unneeded MCP servers via `/mcp`.
- **Prefer `@-mentions` over typing paths:** Referencing `@src/utils.ts` in your prompt attaches the file directly to your message payload. Naming the file without `@` forces Claude to spend expensive output tokens calling `Read`, plus input tokens reading it back.
- **Keep edits targeted:** Instruct Claude to produce targeted surgical diffs rather than rewriting full 500-line files, drastically lowering output token generation (which is priced ~5x higher than input).

---

## Lever 3: Tool Use, Batching & Noisy Commands

Terminal and tool outputs stay in context for every remaining turn. Unmanaged outputs quickly dominate token consumption.

- **Enforce quiet flags on repeated commands:** Test runners and linters printing hundreds of passing lines bloat context. Use dot/compact reporters (e.g., `vitest run <file> --reporter=dot`, `pytest -q`). Store these exact command invocations inside `CLAUDE.md`.
- **Batch independent tool calls (Fable 5.1 / Opus 5 / Sonnet 5):** When reading multiple files or checking multiple symbols, batch them into a single turn rather than sequential turns. This cuts down intermediate round-trips and repeated prompt cache reads.
- **Control subagent delegation:**
  - **Delegate when:** A task creates massive intermediate noise (searching raw server logs, broad grepping, evaluating large directories) where only the final synthesis matters. Subagents operate in clean, isolated context windows, shielding the parent session.
  - **Do inline when:** Executing single greps, single file inspections, or quick edits. The briefing overhead and independent setup of a subagent costs more than running it directly.
  - **Model alignment:** For repetitive background lookups, explicitly assign subagents to run on smaller models (e.g., Haiku).

---

## Lever 4: Model & Effort Calibration

Anthropic's latest models calibrate response verbosity, thinking depth, and instruction-following differently. Match both model tier and effort level to the task.

| Model Tier | Ideal Workload | Key Prompting & Optimization Nuance |
|---|---|---|
| **Claude Fable 5.1 / Mythos 5.1** | Complex multi-day tasks, autonomous agents, end-to-end features | Supports user-facing progress updates; batch tool calls in agent loops; specify explicit compaction rules to retain critical state. |
| **Claude Opus 5 / Opus 4.8** | Deep architectural refactors, ambiguous requirements, high-stakes review | Strongest coding capability; excels when given full specs up-front; avoid micromanaging intermediate steps; restrict excessive over-verification loops. |
| **Claude Sonnet 5** | Daily development, standard features, targeted bugfixes, test authoring | Adaptive thinking by default; response length scales to task complexity; specify concise formatting requirements if verbosity creeps up. |
| **Claude Haiku (e.g. 4.5)** | Fast lookups, boilerplate, subagent triage, simple file transformations | Lowest cost per token; use for mechanical scripts and subagent log processing. |

### The Anti-Pattern: "Start Cheap and Escalate on Failure"
Do not default to the cheapest model with the intent of escalating only if it fails. A subtle, plausible-looking bug generated by an under-powered model costs more in review time, debugging, and retries than using the right model up front. Pick the appropriate tier immediately based on ambiguity and difficulty.

---

## Lever 5: Prompting Precision & Output Control

Unnecessary verbosity in model output is expensive because output (decode) tokens cost roughly 5x more than input tokens.

- **Eliminate conversational preambles:** Instruct Claude directly: *"Provide concise, focused code and explanations. Skip conversational filler, pleasantries, and restating what was asked."*
- **Use XML structure for multi-source inputs:** When supplying multiple files, requirements, and examples, wrap them in clear semantic tags (`<context>`, `<specification>`, `<examples>`). This improves retrieval accuracy on long contexts and prevents confusion.
- **State negative constraints clearly:** Tell the model what **not** to touch (e.g., *"Do not modify dependencies, do not refactor surrounding functions, do not add extra documentation"*).

---

## Decision Checklist

**Before compacting:**
- [ ] Is there a natural milestone/phase boundary reached?
- [ ] Is there enough work remaining in this session to amortize the compaction overhead?
- [ ] Am I about to take a break long enough for the prompt cache to expire (~1 hour)?

**Before spawning a subagent:**
- [ ] Will this task generate screens of intermediate output that are useless once completed?
- [ ] Is the briefing prompt shorter and simpler than doing the work inline?

**Before kicking off a task:**
- [ ] Are `/model` and `/effort` locked in for the duration of this session?
- [ ] Are test commands configured with quiet flags?
- [ ] Are target files referenced via `@-mentions`?

---

## Common Mistakes & Solutions

| Mistake | Root Cause / Impact | Correct Strategy |
|---|---|---|
| Changing `/model` or `/effort` mid-task | Invalidates the prompt cache, causing expensive re-prefill | Set once at session start; `/clear` before changing |
| Compacting right before ending session | Pays summarization cost with 0 subsequent turns to benefit | Leave uncompacted if closing the session |
| Neglecting `/rewind` after bad turns | Clutters history with failed reasoning | Use `/rewind` to prune bad turns without cache penalty |
| Over-using subagents for single reads | Briefing and tool setup exceeds inline execution cost | Run single reads and edits inline |
| Running verbose test suites repeatedly | Hundreds of passing test lines accumulate in prompt history | Use `--reporter=dot` or `-q` in `CLAUDE.md` |
| Vague file naming ("fix the auth file") | Forces Claude to use output tokens calling `Read` and searching | Use `@src/auth/service.ts` directly in prompt |
