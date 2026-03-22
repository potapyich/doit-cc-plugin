# Refactoring Plan v1

Architecture review of the current MVP. Captures what's working, what's fragile, and
priorities for the next iteration.

---

## What Works Well

**Right problem.** The core pain of LLM codegen is "jumped into code without context."
Pipeline enforces PRD → interview → plan → impl plan → tasks → execute. Reduces rewrites.

**Two-level abstraction (steps/ vs impl/).** Best architectural decision in the codebase.
Separating "what to do" (steps/) from "how to do it" (impl/) enables:
- Swapping executors (ralph ↔ default) without touching the pipeline
- Plugging in alternative interview modes (MCP-based, form-based)
- Changing "how" without breaking the step contract

**Approval gates.** Gates on plan and impl_plan are critical. Without them the LLM runs on
its own interpretation. Forced pause for user review is what separates useful pipeline from
a dangerous one.

**Zero-dependency.** No runtime beyond Claude Code. Pure markdown. Install = git clone +
one line in settings.json. Minimal friction.

**Shell injection for state.** `` !`cat .pipeline/state.json 2>/dev/null || echo '{}'` ``
is elegant. Native Claude Code mechanism for injecting live data into prompts. Reliable,
no custom runtime needed.

**Default executor (embedded).** Not depending on ralph is the right default. Progressive
disclosure: works out of the box, ralph is the advanced option.

**Verification as shell commands.** `verification: ["npm test ...", "curl ..."]` makes
execution verifiable. Exit code 0/non-0 is a simple and reliable contract.

**Stuck task handling.** 2 attempts → propose split → user approval. Doesn't silently fail,
doesn't loop, doesn't change scope without asking.

---

## What's Fragile

### 1. Linear pipeline — main limitation

Hard sequence: init → interview → plan → impl_plan → tasks → execute.
In practice:
- After plan, often need to revisit PRD (realized it was wrong)
- After executing 3 tasks, discover plan needs to change
- Interview may be unnecessary (user arrives with a ready PRD)

Only way to "go back" today: reset and start over. No `goto`, no `back`, no partial rollback.
Acceptable for MVP, but the first thing that will start causing friction.

### 2. State management is fragile

Each step file hardcodes the full expected state JSON:
```json
{
  "step": "plan",
  "completed": ["init", "interview_setup", "interview"],
  "pending": ["plan", "impl_plan", "tasks", "execute"]
}
```
Add or remove a step → must update JSON in all subsequent step files. No single source of
truth for step sequence. SKILL.md has a step table, but step files duplicate the sequence.

### 3. One pipeline = one feature

No support for multiple parallel pipelines (e.g., feature A in execute, feature B in
interview). State lives in one `.pipeline/state.json`. Will become a bottleneck on larger
projects with multiple concurrent features.

### 4. No state validation

Nothing checks that `state.json` is consistent. Manual edits or corruption can put the
pipeline in an invalid state. LLM will try to interpret garbage.

### 5. Context budget detection is approximate

"At ~40-50% estimated usage" — Claude has no precise API for measuring context usage. This
will work as a rough heuristic at best, not at all at worst. Especially relevant for the
default executor, which runs in the same session.

### 6. No feedback loop from execution to planning

If execution reveals that the architectural approach is wrong, pipeline doesn't have an
"escalate to plan" path — only stuck task split. Sometimes you need to change `plan.md`,
not subdivide a task.

### 7. Interview UX — no fast path

One question at a time is right for depth, but for large projects: 20-30 questions × wait
for answer. No fast path for experienced users who can dump architecture docs, existing
ADRs, or a full technical spec upfront and skip interview.

### 8. `implementation_plan.json` is a flat list

No explicit dependencies between tasks (only implicit ordering). If task 3.1 depends on
2.3, it's not expressed anywhere. Sequential execution works, but no support for
reordering or parallelization if needed.

### 9. Ralph integration is manual handoff

"Tell the user to run /ralph-loop" is an instruction, not an integration. Noticeable
friction: user must manually switch between two skills. Known limitation of Claude Code
(no skill-to-skill invocation).

### 10. Steps are not idempotent

If pipeline interrupted mid-step (network error, context overflow), re-running
`/armchair-architect` restarts the step from scratch. For init/interview this is tolerable.
For execute — partial code changes + restart may create conflicts.

### 11. Bilingual artifacts multiply surface area

`_prd_ru.md`, `_plan_ru.md`, `_implementation_plan_ru.md` — 3 extra files to sync. Will
drift in practice. Sync at phase boundaries helps, but Russian versions become stale between
phases. Open question: are they needed at all, or is one version in the user's language enough?

### 12. Verification commands are executable by design

JSON is LLM-generated, verification commands execute via Bash. No sandboxing or whitelisting.
Acceptable for user-approved workflow, but worth being aware of: `implementation_plan.json`
content is executable.

---

## Scores

| Dimension | Score | Notes |
|---|---|---|
| Idea | 8/10 | Right problem, right approach, good abstraction level |
| Technical | 7/10 | Strong for pure-markdown MVP; fragility is in state management and linear flow |

Main risk: pipeline linearity will start causing friction on real projects.

---

## Priority Roadmap

### P1 — High impact, relatively contained

1. **Non-linear transitions** — at minimum `back` to previous step without full reset
2. **Centralized step registry** — single source of truth for step sequence, not duplicated
   in every step file's state JSON
3. **Skip paths** — skip interview with a ready PRD, skip planning with a ready plan

### P2 — Medium impact

4. **Multi-pipeline support** — namespace state by feature (`.pipeline/<feature>/state.json`)
5. **State validation** — basic schema check on load, graceful error if invalid
6. **Step idempotency** — detect and handle mid-step restarts cleanly

### P3 — Nice to have

7. **Interview fast path** — accept document dump, infer answers, skip to confirmation
8. **Execution → planning escalation path** — structured way to say "this task reveals
   a plan-level problem"
9. **Task dependencies in JSON schema** — explicit `dependsOn` field

---

## What Not to Change

- Shell injection for state (`!` preprocessing) — keep as-is, it works
- steps/impl two-level abstraction — core architectural strength, don't collapse
- Approval gates on plan and impl_plan — non-negotiable
- Zero-dependency philosophy — no build steps, no external runtime
