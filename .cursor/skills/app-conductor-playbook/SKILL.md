---
name: app-conductor-playbook
description: >-
  Conductor playbook: serial handoff between app-architect and app-builder,
  docs/LOOP.md, and stop rules (fix limit, no progress, handoff cap,
  pause after design). Use when running as app-conductor.
disable-model-invocation: true
---

# App conductor playbook

Load `.cursor/skills/app-delivery-contracts/SKILL.md` first. You schedule. The architect owns design, the board, and `docs/RUN.md`. The builder owns implementation and QA reports. You own `docs/LOOP.md` only.

Spawn with the Task tool, `subagent_type` `app-architect` or `app-builder`, and wait until that agent returns (`run_in_background` false). Put the full assignment in the prompt. Subagents do not see this chat. After each return, update `docs/LOOP.md` before the next spawn.

## Kickoff flags

| Flag | When | Effect |
| --- | --- | --- |
| Requirements path | A path in the kickoff | Pass it to the first architect spawn. Later spawns reuse `docs/GITBRANCH.md`. |
| Pause after design | `pause after design`, `review the plan first`, or `stop after the task board` | After the design spawn, stop. Do not spawn the builder. |
| Allow parallel | `allow parallel` on the kickoff | A build step may pass the architect’s parallel-safe ids to one builder coordinator. |
| Continue | `continue` or `resume` | Read `docs/LOOP.md` and resume. Do not redesign. |

Default: serial, no pause, requirements path `docs/REQUIREMENTS.md`.

## `docs/LOOP.md`

Write this file at the start and after every handoff. Get timestamps with `date -u +"%Y-%m-%dT%H:%M:%SZ"`. Do not guess.

```markdown
# Loop

- **phase:** `design` | `build` | `qa` | `paused` | `stopped`
- **mode:** `serial` | `parallel`
- **pause_after_design:** true | false
- **requirements:** <posix repo-relative path>
- **original_tasks:** T1, T2
- **max_builder_attempts_per_task:** 3
- **last_task:** T1
- **last_result:** PASS | FAIL | _(pending)_
- **last_action:** spawn app-architect | spawn app-builder | stop
- **stop_reason:** _(pending)_ | board complete | fix limit | no progress | handoff cap | scope expanded | missing QA report | parallel merge failed | paused after design
- **cycles:** 0

## Board snapshot

<id> <status>

## Handoff log

| # | time | agent | task | result | next |
|---|------|-------|------|--------|------|
```

`original_tasks` is fixed after the design spawn: ids like `T1` with no `-FIX` suffix. Do not add later feature ids to this list.

`cycles` increments by one when you start a builder spawn (one task or one parallel wave). The design spawn and a final “write the run summary” architect spawn do not increment it.

## Loop

A decision pass either stops, runs architect QA once, or runs one builder spawn and then architect QA. Do not spawn an extra builder or architect beyond that pass.

1. If `docs/LOOP.md` exists and the user did not ask to continue, and **phase** is `stopped` or `paused`, report that state and wait. Do not start a second run.
2. If the user asked to continue:
   - `paused after design` → go to the decision pass.
   - `stopped` → continue only when the user also says the investigation is done (for example `continue after investigation`). Then clear **stop_reason** and go to the decision pass.
   - `design`, `build`, or `qa` → go to the decision pass. Do not spawn a second design.
3. If `docs/TASKS.md` is missing or empty, set **phase** `design` and spawn `app-architect` once. Prompt: load its skills, resolve the requirements path, first design run, return the playbook return shape (branch, board, next action, parallel-safe ids). Then freeze `original_tasks`. If that list is empty, stop `no progress`. If **pause_after_design** is true, stop `paused after design` and return the board. Do not spawn the builder. If `docs/TASKS.md` already has tasks and `docs/LOOP.md` does not, do not redesign. Create `docs/LOOP.md` from the current board, freeze `original_tasks` from ids with no `-FIX` suffix, then start the decision pass.

### Decision pass

Read `docs/TASKS.md`. A fix line is exhausted when `<id>-FIX2` exists and its status is `failed`. The original id is often `failed` as soon as the first QA fails, while `<id>-FIX1` is still `open`. That open fix is still work. Do not stop on the first `failed` row.

Take the **first** match:

1. Any exhausted fix line → stop `fix limit`. List remaining `open` ids. Do not build other tasks.
2. An id that is not in `original_tasks` and is not `<original>-FIX1` or `<original>-FIX2` → stop `scope expanded`.
3. No `open` and no `implemented` → if `docs/RUN.md` has no **end**, spawn `app-architect` once and ask it to write **end**, **duration**, **summary**, and **How to run**. Then stop `board complete`.
4. Any `implemented` → run **Architect QA**, then start another decision pass.
5. Eligible `open` work → run **Build one step**, then **Architect QA**, then start another decision pass. An open `*-FIX1` or `*-FIX2` is eligible even when its original id is `failed`. Other `depends_on` must be `done` or `implemented`. Run an open fix before a later feature task.
6. No eligible `open` task → stop `no progress`.

### Build one step

Count handoff-log rows whose agent is `app-builder` and whose task is that original id or its `-FIX` ids. If that count is already 3, stop `handoff cap`.

Serial: one id. Parallel mode, and only when no fix is open: the architect’s latest non-conflicting ids, one coordinator prompt with the word parallel. If that list is missing, use the one serial id instead.

Set **phase** `build`, increment **cycles**, spawn `app-builder`. Serial prompt: worker role, do not spawn subagents, write `docs/QA-REPORT-<id>.md` and serial `docs/QA-REPORT.md`. Parallel prompt: coordinator role, same worktree, do not implement. If it reports unmerged worktrees, stop `parallel merge failed` and include the paths. Do not merge.

If a QA report file is missing for an id it ran, stop `missing QA report`. A `FAIL` leaves the task `open`. That is not `no progress` yet. **Architect QA** runs next.

### Architect QA

Set **phase** `qa`. This does not increment **cycles**. Spawn `app-architect`. Prompt: read the new QA reports, update the board, append QA attempts to `docs/RUN.md`, return the playbook return shape.

Compare the board snapshot from before this spawn with the board after it. Stop `no progress` when no status changed and no new fix id appeared. A new `*-FIX3` id is also `no progress`; do not run it.

The architect’s “next action” line is a hint. `docs/TASKS.md` wins. When a fix line is exhausted (`-FIX2` is `failed`), stop the whole run even if the architect names another open task. An open `-FIX1` or `-FIX2` still runs.

## Spawn prompts

Every child prompt must include:

- Load `.cursor/skills/app-delivery-contracts/SKILL.md` and that role’s playbook before work.
- Stay on the `branch` in `docs/GITBRANCH.md` after it exists. Do not commit or push.
- Return the return shape from that role’s playbook.
- Do not spawn `app-conductor`.

After the design spawn, `git branch --show-current` must match `branch` in `docs/GITBRANCH.md`. If it does not, stop `no progress` and report both names. Do not switch branches yourself.

## When you stop

Return to the user:

1. **stop_reason** and **phase**
2. App branch from `docs/GITBRANCH.md` (or the current branch if that file does not exist yet)
3. Board: id + status
4. **cycles**, and the last handoff-log rows
5. `board complete`: start, end, duration, and **How to run** from `docs/RUN.md`. The user runs the app and checks the screens. Do not start the dev server.
6. Any other stop: the task id, the QA report path, task notes, and remaining `open` ids. Point at those files for investigation. Do not edit code.

Do not commit.
