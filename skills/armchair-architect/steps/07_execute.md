# Step 07 — Execute

## Goal

Run the implementation plan using the configured executor.

## Instructions

### 1. Resolve executor

If `impl.execute` in state is explicitly `"default"` — skip detection, use built-in executor.

Otherwise, auto-detect ralph:

```bash
[ -f ~/.claude/skills/ralph-loop/SKILL.md ] || [ -f .claude/skills/ralph-loop/SKILL.md ] && echo "found" || echo "not found"
```

**If ralph found:** use `ralph` executor. Load `${CLAUDE_SKILL_DIR}/impl/execute/ralph.md`.

**If ralph not found:** tell the user:

> Ralph Loop is not installed. It runs tasks autonomously in separate sessions.
>
> Options:
> **A)** Install ralph — add this to `~/.claude/settings.json`:
> ```json
> { "plugins": ["https://github.com/anthropics/claude-code-hooks-multi-agent-example"] }
> ```
> Then re-run `/armchair-architect` to use it.
>
> **B)** Run with built-in executor — I'll implement tasks here in this session (same logic, no install needed)

Wait for choice. On **A**: stop, let user install. On **B**: use `default` executor, load `${CLAUDE_SKILL_DIR}/impl/execute/default.md`.

### 2. Pre-flight check

Verify `implementation_plan.json` exists:

```bash
cat implementation_plan.json | jq 'length'
```

If missing, tell the user:
> `implementation_plan.json` not found. Run `/armchair-architect` from Step 06 to generate it first.

Show current task status:
```bash
cat implementation_plan.json | jq '[.[] | {id, description: .description[:60], passes}]'
```

Tell the user:
> Found <N> tasks, <M> remaining (passes: false).

### 3. Ask how to proceed

> How would you like to run?
>
> **A)** Run 3 tasks — supervised start to verify things work *(recommended)*
> **B)** Run all tasks — go to completion
> **C)** Pick a number
>
> Reply A, B, C (+ number for C).

Wait for user's reply, then follow the impl file instructions with the chosen task count.

### 4. Context handoff reminder

If you notice you're approaching context limits (~40-50%), warn the user:

> Approaching context limit. Before starting a new session:
> 1. `implementation_plan.json` has task status (`passes: true/false`)
> 2. In the new session, run `/armchair-architect` — it will resume from `execute` step
> 3. Execution will continue from the first `passes: false` task.

### 5. On task failure or stuck task

If a task fails after 2 fix attempts, propose a split:

> Task `<id>` appears stuck. Suggested split:
> - `<id>a`: <subtask description>
> - `<id>b`: <subtask description>
> - `<id>c`: <subtask description>
>
> Approve split? The original task will be replaced in `implementation_plan.json`.

Wait for user approval before modifying `implementation_plan.json`.

### 6. Completion

When all tasks have `passes: true`:

> All tasks complete.
>
> Summary:
> - Tasks completed: <N>
> - Verification commands run: <N>
>
> Run `git log --oneline` to review commits, or check `progress.md` for full execution log.

Update state:
```json
{
  "step": "done",
  "completed": ["init", "interview_setup", "interview", "plan", "impl_plan", "tasks", "execute"],
  "pending": []
}
```
