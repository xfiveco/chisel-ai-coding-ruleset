# New Session Prompt

**Launch your agent from inside the theme directory** (the one containing `CLAUDE.md`) so the `chisel-*` skills auto-discover. Skills live at `ai/skills/{name}/SKILL.md` and are only found when the agent's working directory is the theme root.

Then paste this at the start of every new Chisel session (new project or new conversation) before asking for any work:

> Read `CLAUDE.md` first and confirm you have it — it owns the loading flow, scaffolding rules, progress-file procedure, and the per-phase plan-review gate. Follow it.
>
> Work the task in phases, one at a time, **never chaining**:
>
> 1. **Scope** — restate the task, list what you'll touch and what's out of scope, ask any clarifying questions. **Pause.**
> 2. **Plan** — phased plan, each phase small enough to review/commit alone (Figma: one phase per section, top-to-bottom). **Pause for approval.**
> 3. **Execute** — per phase, expand its plan then build then summarize, with a pause before and after (per the gate in CLAUDE.md). One phase per go-ahead.
> 4. **Wrap-up** — final summary + follow-ups after the last phase.
>
> If anything is ambiguous, ask before starting — don't guess.

TASK:
[YOUR TASK HERE] (Describe what you want.)

README: <https://github.com/xfiveco/chisel-ai-coding-ruleset>
