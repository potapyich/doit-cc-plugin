# doit-cc-plugin

This is the source repo for the doit-cc-plugin Claude Code pipeline plugin.

## What This Repo Is

A collection of markdown prompts implementing a feature development pipeline.
No build step, no dependencies. Files are markdown prompt templates.

## Stack

- Format: Claude Code skills (`skills/<name>/SKILL.md`)
- Runtime state: `.pipeline/state.json` (in user's project, not here)
- Shell injection: `` !`<command>` `` for live data in prompts
- Executor: ralph (via Claude's Bash tool)

## Structure

```
skills/armchair-architect/
  SKILL.md               ← /armchair-architect entry point
  steps/                 ← WHAT to do at each phase (stable contract)
  impl/                  ← HOW to do it (swappable)
```

## Testing a Change

1. Copy (or symlink) `skills/armchair-architect/` to `~/.claude/skills/armchair-architect/`
2. Open any test project in Claude Code
3. Run `/armchair-architect status` — should show state or "no state"
4. Run `/armchair-architect` — should propose next step or init

Or configure as plugin in `~/.claude/settings.json`:
```json
{ "plugins": ["/path/to/doit-cc-plugin"] }
```

## Conventions

- Step files (`steps/`) describe WHAT the step does — stable interface, avoid breaking changes
- Impl files (`impl/`) describe HOW — freely replaceable
- All user-facing text in English
- Russian drafts (`_*.md`) are derived; sync at phase boundaries only
- Do not hardcode paths — use `${CLAUDE_SKILL_DIR}` when referencing step files

## Key Reference

See `DESIGN.md` for full architecture and design decisions.
