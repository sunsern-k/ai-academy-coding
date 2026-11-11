---
name: app-architect-playbook
description: >-
  Architect playbook: DESIGN/TASKS/RUN, QA gating, max two fix rounds.
  Use when running as app-architect or when the parent asks to design
  or update the task board.
---

# App architect playbook

Load `.cursor/skills/app-delivery-contracts/SKILL.md` first. Requirements path, `docs/GITBRANCH.md`, git branch commands, and **Host OS and shell** (timestamps, portable setup/run docs) live there. Below, “the resolved requirements file” means that path (also stored as `requirements` in `docs/GITBRANCH.md`).

## Task board format (`docs/TASKS.md`)

Each task has:

- `id` (T1, T2, …; fix tasks `T1-FIX1`, `T1-FIX2`)
- `title`
- `status`: `open` | `implemented` | `done` | `failed`
- `files` — paths this task may create/edit (required so parallel waves can detect overlap)
- `depends_on` — task ids that must be `done` or `implemented` first (empty if none)
- `notes` (optional)

Meaning:

- `open` — not built yet (or a new fix task)
- `implemented` — builder finished **and tests passed**; waiting for you to mark `done`
- `done` — you mark this after a passing `docs/QA-REPORT.md`
- `failed` — tests failed; you set the task to `failed` and add a fix task when allowed

## Fix loop (locked)

- Maximum **2** fix rounds per original task (`…-FIX1`, then `…-FIX2`).
- If QA fails after `…-FIX2` was attempted, mark the original task `failed`, record residual failures in notes, and **do not** create `…-FIX3`. Tell the parent to stop that task and report to the user.

## Run clock and QA stats (`docs/RUN.md`)

You own the wall-clock for this delivery. Subagents do not share memory, so **persist it in `docs/RUN.md`**.

Get timestamps with a shell command, not by guessing. Use the cross-platform Node ISO-8601 UTC command from **Host OS and shell** in the contracts skill (do not use `date -u`).

**First prompt (start):** on the first design run, if `docs/RUN.md` is missing or has no **start**, write the file and set **start** to now. Never change **start** on later runs.

**Each QA report:** append one attempt row per report file (`docs/QA-REPORT.md` and/or `docs/QA-REPORT-<id>.md`):

- attempt number (1, 2, …)
- UTC timestamp (report timestamp if present, else now)
- task id
- overall `PASS` or `FAIL`
- test cases passed / failed / missing coverage (count rows in the report table)

**Board complete (end):** when no task is `open` or `implemented` (every original task is `done`, or the board stopped after max fixes with remaining `failed` and no further fix tasks):

1. Set **end** to now (do not change **start**).
2. Set **duration** as human-readable elapsed time (hours/minutes/seconds) from start to end.
3. Fill the **summary** (see template), including **How to run**.
4. Return the summary to the parent (required when stopping as complete).

Template for `docs/RUN.md`:

```markdown
# Run

- **start:** `YYYY-MM-DDTHH:MM:SSZ`
- **end:** _(pending)_
- **duration:** _(pending)_

## QA attempts

| # | time | task | result | passed | failed | missing |
|---|------|------|--------|--------|--------|---------|

## Summary

_(fill only when the board is complete)_

## How to run

_(copy from docs/DESIGN.md; commands to install, test, and start the app)_
```

Summary must include:

- Start time and end time
- How long all tasks took (**duration**), human-readable
- How many QA attempts ran, how many **FAIL**, how many **PASS**
- Test cases: how many **failed before** (sum of `failed` + `missing` on FAIL attempts, and failed counts on earlier runs) vs **how many now passed** (passed count on the latest PASS report, or on the final report if the board stopped failed)
- One line per original task: `done` on first QA vs needed `FIX1`/`FIX2`
- **How to run the app** so a human can try it (from `docs/DESIGN.md`). Follow **Host OS and shell** in the contracts skill: Node **24.20.0** (`.nvmrc` + Windows equivalent), `npm install` in `backend/` and `frontend/`, test via the resolved requirements (Truck2Hand: `npm test` in `backend/` with `node:test` — not Vitest), start both apps, note ports (e.g. `8080` / `5173`), and note native build tools for `better-sqlite3` on macOS vs Windows. If DESIGN is missing the section, infer from `package.json` scripts and write it into both DESIGN and RUN.

## When invoked: first design

If `docs/TASKS.md` is missing or empty:

1. Resolve the requirements path (contracts skill). Confirm the file and **Application name**.
2. Create/check out the git branch and write `docs/GITBRANCH.md` (include `requirements`).
3. Record **start** in `docs/RUN.md` (see **Run clock and QA stats**).
4. Write `docs/DESIGN.md` from the resolved requirements file (modules, data, persistence, UI, file layout). Include a **How to run** section (install, test once, start the app) that works for macOS and Windows per **Host OS and shell**. Point at the branch file; do not invent a second branch name.
5. Split work into **small tasks**. Prefer: domain/store + validation → persistence → UI, using `depends_on` for real order. Set disjoint `files` per task so an explicit parallel request can run a safe wave. Include a test-runner setup task only if the repo has no runner yet.
6. Return to the parent: branch name, resolved requirements path, start time, design summary, task ids, `depends_on` / `files`, the default **next single task**, and which ids are **parallel-safe** together (no overlap, dependencies satisfied).

## When invoked: QA report

Read `docs/TASKS.md` and every new QA file (`docs/QA-REPORT.md` and `docs/QA-REPORT-<id>.md`). Append each attempt to `docs/RUN.md` **before** changing the board. Process multiple reports in one invocation when a parallel wave finished.

- **Pass:** mark that report’s task `done`. Point the parent at remaining `open` work (single next task and/or the next parallel-safe wave), or declare the board complete and write **end** + **summary** + **How to run**.
- **Fail:** set the attempted task to `failed`. If fix count for that original task is `< 2`, add the next fix task with: failing test names, expected vs actual, files, and a precise repair instruction. If already 2 fixes, stop as specified above (still write **end** + **summary** + **How to run** if no more `open`/`implemented` work remains).

Do not rewrite the whole plan after a failure. Do not expand scope.

## Return shape (to parent)

Always end with:

1. Current git branch (must be the app branch from `docs/GITBRANCH.md`)
2. Resolved requirements path (`requirements` in `docs/GITBRANCH.md`)
3. What you changed in `docs/`
4. Current board (id + status)
5. Run clock: **start** (and **end** + **duration** + QA summary if the board is complete)
6. Next parent action: `spawn app-builder` (one task id) | `spawn app-builder parallel` (list of non-conflicting ids) | `stop` (reason)
7. **When the board is complete:** the full task summary, **how long all tasks took**, and **how to run the app** to try it (commands + expected URL if any)
