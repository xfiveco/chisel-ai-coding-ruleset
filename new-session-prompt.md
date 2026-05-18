# New Session Prompt

Paste this at the start of every new Chisel session (new project or new conversation) before asking for any work:

> Read `CLAUDE.md` and `rules/RULES.md` first, then follow the **RULES → reference → skill** flow for every task I give you. Before scaffolding anything (block, pattern, CPT, theme.json change, component), open the matching reference doc in `rules/reference/` before the skill.
>
> **Work in phases, pause between them — wait for my explicit go-ahead before moving on.** Do not chain phases.
>
> 1. **Scope** — restate the task in your own words, list what you'll touch (files, blocks, tokens) and what's out of scope. Ask any clarifying questions. **Pause.**
> 2. **Plan** — propose a phased plan: ordered list of changes, each phase small enough to review and commit on its own. For Figma imports, one phase per section, top-to-bottom. **Pause for approval.**
> 3. **Execute one phase at a time** — make the changes for *just that phase*, then stop and summarize: what was changed (file:line where useful), what to verify, what's deferred. Wait for "next" / feedback / commit instruction. **Do not start the next phase.**
> 4. **Wrap-up** — after the last phase, give a one-paragraph summary + any follow-ups.
>
> If anything is ambiguous (file shape, design intent, reuse vs. new, scope, naming), ask before starting — don't pattern-match a guess. Treat memory entries as starting context, not current truth: read the actual file before relying on a documented value.

TASK:
[YOUR TASK HERE] (Describe what you want.)
