---
name: app-builder-playbook
description: >-
  Builder playbook: one task per worker, tests from resolved requirements,
  run tests once, QA report, mark implemented only on PASS. When the parent
  explicitly names multiple task ids for parallel, conflict-check and spawn
  isolated workers. Use when running as app-builder.
---

# App builder playbook

Load `.cursor/skills/app-delivery-contracts/SKILL.md` first. The product source is the file named by `requirements` in `docs/GITBRANCH.md` — not a hard-coded path.

## Product source

Read `docs/GITBRANCH.md` and use the `requirements` field. Then read that file (application name, stack, acceptance criteria), then `docs/DESIGN.md` and `docs/TASKS.md`.

- Implement only what the current task and that requirements file allow.
- Do not add features that are not in that file.
- Do not change the requirements file to make tests pass.
- Do not weaken or delete tests to force a pass.

## Input

1. Confirm you are on the app git branch (`branch` in `docs/GITBRANCH.md`), or on the isolated worktree for this worker.
2. Read `docs/GITBRANCH.md`, the resolved requirements file, `docs/TASKS.md`, `docs/DESIGN.md`.
3. **Serial (default):** implement **only** the one task id the parent named. If the parent did not name one and did not ask for parallel, take the first `open` task (including `*-FIX1` / `*-FIX2`).
4. **Parallel (explicit only):** if the parent named **multiple** ids and asked for parallel, you are the **coordinator**. Follow **Parallel waves** below. Do not implement in the coordinator process.
5. A **worker** prompt names exactly one id. Implement only that task. Do not spawn subagents.
6. For a fix task, also read `docs/QA-REPORT.md` and `docs/QA-REPORT-<id>.md` and fix the cited failures. Do not refactor unrelated code.

## Parallel waves

Load conflict and isolation rules from the contracts skill.

1. Build the requested id list. Skip ids that are not `open`.
2. If any pair conflicts, **do not start**. Return conflicting pairs, a wave plan, and tell the parent to run the safe subset (or serial).
3. If all requested ids are non-conflicting: spawn one isolated `app-builder` worker per id in parallel. Each worker prompt: task id, worker role, `files` it may touch, write `docs/QA-REPORT-<id>.md`.
4. When workers return, report each PASS/FAIL and QA path to the parent. Parent merges isolated worktrees if used, then spawns `app-architect` once with all new QA files.
5. Never implement T1–Tn yourself in one process to “fake” parallelism.

## Implementation rules

- Follow the stack and design in the docs (e.g. store API, storage key, file layout).
- Keep domain logic unit-testable; keep UI thin.
- Add or update tests that map to **acceptance criteria** in the resolved requirements file for this task, and keep existing tests running.
- Use the project's test runner from requirements/design (for this repo's demo: Vitest). Do not start a long watch process; run once with `npx vitest run`. That command is the same in zsh, bash, PowerShell, and cmd (contracts skill, **Shell commands**).
- If the runner is not set up and the task or board expects it, add a minimal setup so tests can execute.
- You may iterate implement → test **within this one task**. If tests still fail, **stop**: write a FAIL report. Do not start the next feature task. Do not mark the task `implemented`. The architect owns further retries (`FIX1` / `FIX2`, max 2).
- Edit only this task’s `files` list (and its QA report + its `docs/TASKS.md` status). If you need a file not listed, stop and tell the parent (conflict with another task).
- If you add or change start/test scripts, update **How to run** in `docs/DESIGN.md` using the **Shell commands** format (separate lines, no `&&`, same on macOS and Windows).

## After you finish (always write a QA report)

Include:

- Timestamp
- Git branch tested (from `docs/GITBRANCH.md`)
- Requirements path used (from `docs/GITBRANCH.md`)
- Task id under test
- Command run
- Overall: `PASS` or `FAIL`
- Table of cases: requirement, test name, pass/fail
- On failure: test names, expected vs actual, file:line, excerpt of error
- Missing coverage list (a required behavior with no test is a **FAIL**)
- Suggested fix direction (for the architect), not a full rewrite

Write **`docs/QA-REPORT-<id>.md`** (always). Also write **`docs/QA-REPORT.md`** only in **serial** mode (single worker, not a parallel wave).

Then:

1. **PASS:** in `docs/TASKS.md`, set **that one task** to `implemented` (do not mark `done`; the architect does that). Do not rewrite other tasks’ rows.
2. **FAIL:** leave the task `open` (or already `open` on a fix task). Do not mark `implemented` or `done`.

Return to the parent:

- Git branch / worktree you worked on
- Requirements path used
- Task id (workers: one id; coordinator: the wave list)
- Files changed
- Overall `PASS` or `FAIL` and QA report path(s)
- How to run the app (if you added or changed scripts)
- Explicit next step: parent should spawn `app-architect` with the new report(s)

Do not start another feature task in the same **worker** run.
