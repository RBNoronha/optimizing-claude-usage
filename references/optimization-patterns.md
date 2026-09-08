# Optimization Patterns and Deep Dives

## Lever 1: Prompt Cache Preservation (90% Discount)

Prompt caching matches request tokens from the beginning forward. Any modification to the prefix invalidates the cache for all subsequent tokens.

- **Lock `/model` and `/effort` at the start:** Changing either parameter mid-conversation busts the cache, causing the entire conversation history to be re-prefilled at full price. (Note: turning Fast mode off is free, but activating it mid-session forces a re-prefill at Fast mode rates).
- **Time your `/compact` calls strategically:**
  - **Before stepping away:** Prompt cache expires after 1 hour (subscription) or 5 minutes (API key). Compaction requires reading the entire history once to write the summary. Running `/compact` while the cache is warm is up to 90% cheaper than compacting after returning from an idle break.
  - **Amortize compaction cost:** Compaction itself has an upfront token cost. Only compact at phase boundaries if there is meaningfully more work left in the session to recoup that cost across subsequent turns. Never compact right before closing a session.
- **Use `/rewind` over `/compact` for rollbacks:** If the last few turns went off track, `/rewind` trims turns from the tail, leaving the earlier cached prefix completely intact and costing 0 extra tokens.

## Lever 2: Context Window & Session Hygiene

Every turn resends the entire conversation history (or its compaction summary). Keep what enters the context minimal and precise.

- **Run `/clear` between tasks:** Do not drag context from a completed feature or bugfix into an unrelated task. Use `/rename` if you want to archive the session, then `/clear` to start with an empty context.
- **Audit loaded instructions with `/context`:** Check what is injected at session start (e.g., `CLAUDE.md`, system prompts, active MCP tools). Keep `CLAUDE.md` concise and shift workflow-specific logic into on-demand skills. Disable unneeded MCP servers via `/mcp`.
- **Prefer `@-mentions` over typing paths:** Referencing `@src/utils.ts` in your prompt attaches the file directly to your message payload. Naming the file without `@` forces Claude to spend expensive output tokens calling `Read`, plus input tokens reading it back.
- **Keep edits targeted:** Instruct Claude to produce targeted surgical diffs rather than rewriting full 500-line files, drastically lowering output token generation (which is priced ~5x higher than input).

## Lever 3: Advanced Tool Use & MCP Management

According to Anthropic's engineering benchmarks, 5 connected MCP servers can inject 55k+ tokens before any conversation begins, reaching 100k+ with tools like Jira. Moreover, unmanaged raw tool outputs quickly flood the context window.

- **Avoid the "MCP Tax":** Audit connected MCP servers at the start with `/mcp` or `/context`. If a session only requires git and local tests, disable external database, Slack, or ticketing MCPs. When running from the CLI, consider `--strict-mcp-config` or restricting tools via `--tools`.
- **Programmatic tool calling & output filtering:** Never dump multi-megabyte log files or full API JSON payloads into context. Filter at the source using bash pipelines (e.g. `grep`, `awk`, `head -n 50`) or node/python scripts so only synthesized findings enter the conversation.
- **Batch independent tool calls (Fable 5.1 / Opus 5 / Sonnet 5):** When reading multiple files or checking multiple symbols, batch them into a single turn rather than sequential turns. This cuts down intermediate round-trips and repeated prompt cache reads.
- **Enforce quiet flags on repeated commands:** Test runners and linters printing hundreds of passing lines bloat context. Use dot/compact reporters (e.g., `vitest run <file> --reporter=dot`, `pytest -q`). Store these exact command invocations inside `CLAUDE.md`.
- **Control subagent delegation:**
  - **Delegate when:** A task creates massive intermediate noise (searching raw server logs, broad grepping, evaluating large directories) where only the final synthesis matters. Subagents operate in clean, isolated context windows, shielding the parent session.
  - **Do inline when:** Executing single greps, single file inspections, or quick edits. The briefing overhead and independent setup of a subagent costs more than running it directly.
  - **Model alignment:** For repetitive background lookups, explicitly assign subagents to run on smaller models (e.g., Haiku).

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

## Lever 5: Prompting Precision & Output Control

Unnecessary verbosity in model output is expensive because output (decode) tokens cost roughly 5x more than input tokens.

- **Eliminate conversational preambles:** Instruct Claude directly: *"Provide concise, focused code and explanations. Skip conversational filler, pleasantries, and restating what was asked."*
- **Use XML structure for multi-source inputs:** When supplying multiple files, requirements, and examples, wrap them in clear semantic tags (`<context>`, `<specification>`, `<examples>`). This improves retrieval accuracy on long contexts and prevents confusion.
- **State negative constraints clearly:** Tell the model what **not** to touch (e.g., *"Do not modify dependencies, do not refactor surrounding functions, do not add extra documentation"*).
