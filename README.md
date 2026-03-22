# doit-cc-plugin

A feature development pipeline for Claude Code. Takes a project from raw idea through
structured PRD, planning, and automated execution.

## Pipeline Overview

```
/pipeline → PRD → Interview → plan.md → implementation_plan.md → implementation_plan.json → ralph
```

1. **Init** — describe your project, get a structured PRD
2. **Interview** — targeted questions to fill gaps, deepen technical detail
3. **Plan** — high-level work blocks, user-approved
4. **Implementation Plan** — concrete steps per block, user-approved
5. **Tasks** — execution-ready JSON with verification commands
6. **Execute** — ralph runs the tasks autonomously

## Install

### Global (all projects)

```bash
git clone https://github.com/potapyich/doit-cc-plugin ~/.claude/plugins/doit-cc-plugin
```

Add to `~/.claude/settings.json`:
```json
{
  "plugins": ["~/.claude/plugins/doit-cc-plugin"]
}
```

### Per-project

```bash
git clone https://github.com/potapyich/doit-cc-plugin .claude/plugins/doit-cc-plugin
```

Add to `.claude/settings.json` in your project:
```json
{
  "plugins": [".claude/plugins/doit-cc-plugin"]
}
```

### Update

```bash
cd ~/.claude/plugins/doit-cc-plugin && git pull
```

## Usage

```bash
/pipeline              # start or continue from current step
/pipeline status       # show current state
/pipeline reset        # clear state and start over
/pipeline use execute ralphex   # switch executor
```

## Requirements

- [Claude Code](https://claude.ai/code)
- [ralph](https://github.com/anthropics/ralph) for execution phase

## Files Created in Your Project

| File | When | What |
|---|---|---|
| `prd.md` | Step 1 | Product requirements (English) |
| `_prd_ru.md` | Step 1 (if input in Russian) | PRD in Russian |
| `plan.md` | Step 4 | Strategy plan |
| `implementation_plan.md` | Step 5 | Detailed steps |
| `implementation_plan.json` | Step 6 | Execution tasks for ralph |
| `progress.md` | Step 7 | Execution log (written by ralph) |
| `.pipeline/state.json` | Throughout | Pipeline state (gitignored) |

## Design

See [DESIGN.md](./DESIGN.md) for full architecture documentation.
