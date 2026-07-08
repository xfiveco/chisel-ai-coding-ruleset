# New Session Prompt

**Launch your agent from inside the theme directory** (the one containing `CLAUDE.md`) so the `chisel-*` skills auto-discover. Skills live at `ai/skills/{name}/SKILL.md` and are only found when the agent's working directory is the theme root.

Then paste this at the start of every new Chisel session (new project or new conversation) before asking for any work:

> Read `CLAUDE.md` first and confirm you have it — it owns the loading flow, scaffolding rules, progress-file procedure, and the per-phase plan-review gate. Follow it.
>
> First check `ai-progress/INDEX.md` — if an effort is already in progress, open its roadmap and **resume the next phase** instead of scoping from scratch.
>
> Otherwise, work the task in phases, one at a time, **never chaining**:
>
> 1. **Scope** — restate the task, list what you'll touch and what's out of scope, ask any clarifying questions. **Pause.**
> 2. **Plan** — phased plan, each phase small enough to review/commit alone (Figma: one phase per section, top-to-bottom). **Pause for approval.**
> 3. **Execute** — one phase at a time, each as three separate steps with a hard stop between:
>    a. **Expand the phase plan** in the progress file (what you'll do, files/blocks/tokens, decisions). **Then STOP — do not write any code.** Wait for my review; I may change the plan.
>    b. On my explicit go-ahead, **build that phase only**, then summarize (what changed, what to verify). **STOP.**
>    c. Wait for "next" before touching the following phase.
> 4. **Wrap-up** — final summary + follow-ups after the last phase.
>
> If anything is ambiguous, ask before starting — don't guess.

TASK:
[YOUR TASK HERE] (Describe what you want.)

README: <https://github.com/xfiveco/chisel-ai-coding-ruleset>
