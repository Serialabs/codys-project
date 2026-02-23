# CLAUDE.md

## Core Principles

- **Simplicity First** — Make every change as simple as possible. Minimal code impact.
- **No Laziness** — Find root causes. No temporary fixes. Senior dev standards.
- **Minimal Impact** — Only touch what's necessary. Avoid introducing bugs.
- **DRY** — Flag repetition aggressively.
- **Engineered enough** — Not fragile/hacky, not over-abstracted. Explicit over clever.
- **Edge cases > speed** — Thoughtfulness wins. Well-tested code is non-negotiable.

---

## Workflow Orchestration

### Plan Mode (Default for non-trivial tasks)
Enter plan mode for any task with 3+ steps or architectural decisions.
- Write detailed specs upfront to reduce ambiguity
- If something goes sideways: STOP, re-plan, don't keep pushing
- Use plan mode for verification steps, not just building

**Plan mode review areas:** Architecture → Code Quality → Tests → Performance

### Subagent Strategy
- Use subagents to keep the main context window clean
- Offload research, exploration, and parallel analysis to subagents
- One focused tack per subagent
cod
### Autonomous Bug Fixing
- Given a bug report: just fix it. No hand-holding needed.
- Point at logs, errors, failing tests — resolve them directly
- Fix failing CI tests without being told how

---

## Task Management

1. **Plan First** — Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan** — Check in before starting implementation
3. **Track Progress** — Mark items complete as you go
4. **Explain Changes** — High-level summary at each step
5. **Document Results** — Add review section to `tasks/todo.md`
6. **Capture Lessons** — Update `tasks/lessons.md` after any correction

### Verification Before Done
- Never mark a task complete without proving it works
- Ask: *"Would a staff engineer approve this?"*
- Run tests, check logs, demonstrate correctness

---

## Self-Improvement Loop

After **any** correction from the user:
1. Update `tasks/lessons.md` with the pattern
2. Write a rule that prevents the same mistake
3. Ruthlessly iterate until mistake rate drops
4. Review lessons at the start of each session
