# New Session Prompt

**Launch your agent from inside the theme directory** (the one containing `CLAUDE.md`) so the `chisel-*` skills auto-discover. Skills live at `ai/skills/{name}/SKILL.md` and are only found when the agent's working directory is the theme root.

Then paste this at the start of every new Chisel session (new project or new conversation) before asking for any work:

> Read `CLAUDE.md` first (it's the always-loaded entry point — confirm you have it), then follow the **CLAUDE.md → reference → skill** flow for every task I give you. Before scaffolding anything (block, pattern, CPT, theme.json change, component), open the matching reference doc in `ai/rules/reference/` before the skill.
>
> **Work in phases, pause between them — wait for my explicit go-ahead before moving on.** Do not chain phases.
>
> 1. **Scope** — restate the task in your own words, list what you'll touch (files, blocks, tokens) and what's out of scope. Ask any clarifying questions. **Pause.**
> 2. **Plan** — propose a phased plan: ordered list of changes, each phase small enough to review and commit on its own. For Figma imports, one phase per section, top-to-bottom. **Always create a progress file at the theme root** using the [`chisel-plan` skill](ai/skills/chisel-plan/SKILL.md) — that skill owns the format and update procedure — unless I explicitly tell you to skip it. **Pause for approval.**
> 3. **Execute one phase at a time** — make the changes for _just that phase_, then stop and summarize: what was changed (file:line where useful), what to verify, what's deferred. Flip the phase status in the progress file. Wait for "next" / feedback / commit instruction. **Do not start the next phase.**
> 4. **Wrap-up** — after the last phase, give a one-paragraph summary + any follow-ups. Append a final session log entry to the progress file.
>
> If anything is ambiguous (file shape, design intent, reuse vs. new, scope, naming), ask before starting — don't pattern-match a guess. Treat memory entries as starting context, not current truth: read the actual file before relying on a documented value.
>
> Let me know if you see the chisel theme Claude.md and related rules and skills before the start

TASK:
[YOUR TASK HERE] (Describe what you want.)

README: <https://github.com/xfiveco/chisel-ai-coding-ruleset>
