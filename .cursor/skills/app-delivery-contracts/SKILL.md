---
name: app-delivery-contracts
description: >-
  Shared file contract for app-architect and app-builder: requirements
  path resolution, GITBRANCH.md schema, who owns DESIGN/TASKS/RUN/QA.
  Use whenever those subagents run or when maintaining the docs/ delivery loop.
---

# App delivery contracts

## Resolve requirements path (architect)

1. Path from this invocation (slash argument or parent prompt), if present
2. Else `requirements` already in `docs/GITBRANCH.md`
3. Else `docs/REQUIREMENTS.md`

Stop if that file is missing or has no **Application name**. Tell the parent to add it. Do not invent a product. Do not expand scope beyond that file. Do not rewrite it unless the parent explicitly asks.

Use a posix, repo-relative path. Persist it in `docs/GITBRANCH.md` so the builder never guesses the filename.

## `docs/GITBRANCH.md` (architect writes; builder reads)

```markdown
# Git branch

- **branch:** `cursor/<slug>` or `cursor/<slug>-<datetime>` if the base name was taken
- **application name:** <name from requirements>
- **requirements:** <resolved path, posix, repo-relative>
```

## Other files

| File | Owner | Purpose |
|------|--------|---------|
| The resolved requirements file | Product (input) | Application name, stack, acceptance criteria |
| `docs/GITBRANCH.md` | Architect | App git branch + resolved requirements path |
| `docs/DESIGN.md` | Architect | Modules, data, UI, stack usage, **How to run** |
| `docs/TASKS.md` | Architect owns the board; each builder worker may set **only its assigned task** to `implemented` | Task board (`files`, `depends_on`) |
| `docs/RUN.md` | Architect | Run start/end, QA log, board-complete summary |
| `docs/QA-REPORT.md` | Builder (serial only) | Last serial test run |
| `docs/QA-REPORT-<id>.md` | Builder worker for that task id | Parallel (and allowed for serial) per-task QA |

## Parallel builders (explicit only)

Default is **one builder worker = one task**. Parallel runs happen only when the user/parent **explicitly** names multiple task ids (e.g. T1–T4) and asks for parallel.

**Conflict rules** (must pass before any parallel start):

- Two tasks **conflict** if their `files` lists overlap (same path, or one path is a parent of the other), or if A `depends_on` B and B is not yet `done` or `implemented`.
- Missing `files` on either task → treat as **conflict** (unknown overlap).
- Conflicting tasks must **not** run in the same wave. Return the conflicting pairs and a **wave plan** (independent ids this wave; dependents later). Do not start a conflicting set.

**Isolation:**

- Spawn **one `app-builder` worker per non-conflicting id** in the same parent turn (parallel Task calls).
- Prefer **isolated git worktrees** (or Cursor isolated subagent environments) when more than one worker runs, so `docs/TASKS.md` and `src/` cannot clobber each other. Those copies are ephemeral; they are not extra product feature branches. After workers return, the parent merges onto the app branch from `docs/GITBRANCH.md` before spawning `app-architect`.
- If isolation is unavailable **and** `files` are disjoint: same worktree is allowed. Each worker may edit only its `files` list, only its row in `docs/TASKS.md`, and only `docs/QA-REPORT-<id>.md`. Never write another worker’s sources or `docs/QA-REPORT.md` in a parallel wave.

**Coordinator vs worker:** a single `/app-builder` invocation that received multiple ids + “parallel” is the **coordinator**: conflict-check, spawn isolated workers, do not implement. A worker prompt names **exactly one** task id and must not spawn further subagents.

## Shell commands (Windows and macOS)

Pick the command from the **shell that is actually running**. Do not assume macOS. Cursor on macOS is usually zsh; Cursor on Windows is usually PowerShell. Git Bash on Windows is bash, so it uses the Unix commands.

### UTC timestamps

Never guess a timestamp. Do not run `date -u` in PowerShell: there `date` is `Get-Date`, and `-u` fails.

**bash, zsh, or sh** (macOS, Linux, Git Bash):

```bash
date -u +"%Y-%m-%dT%H:%M:%SZ"
date -u +"%Y%m%dT%H%M%SZ"
```

**PowerShell** (`powershell` or `pwsh`). `InvariantCulture` keeps `-` and `:` stable on non-US locales:

```powershell
[DateTime]::UtcNow.ToString("yyyy-MM-dd'T'HH:mm:ss'Z'", [Globalization.CultureInfo]::InvariantCulture)
[DateTime]::UtcNow.ToString("yyyyMMdd'T'HHmmss'Z'")
```

**cmd.exe** (only if that is the shell): call PowerShell; do not use `date`.

```bat
powershell -NoProfile -Command "[DateTime]::UtcNow.ToString('yyyy-MM-dd''T''HH:mm:ss''Z''', [Globalization.CultureInfo]::InvariantCulture)"
powershell -NoProfile -Command "[DateTime]::UtcNow.ToString('yyyyMMdd''T''HHmmss''Z''')"
```

ISO form example: `2026-11-11T01:40:41Z`. Compact form example: `20261111T014041Z`.

### Git

Same git verbs on both systems. Quote every ref pattern with **double quotes** so zsh does not glob it and cmd does not treat quotes as literal characters. Single quotes are fine in zsh and PowerShell; they are not quotes in cmd.exe.

```bash
git status
git branch --show-current
git switch -c "<unique-branch>"
git branch --list --all -- "<name>"
git branch --list --all -- "*/<name>"
```

`<name>` is the full branch, e.g. `cursor/todo-list`. The second `--list` pattern matches a remote-tracking line such as `remotes/origin/cursor/todo-list`.

Do not run `git show-ref --verify` with an unquoted `refs/remotes/*/<name>`. `--verify` needs an exact ref, and in zsh an unmatched `*` exits with `no matches found` before git runs.

### Install, test, and start

Once Node is on `PATH`, these are the same in zsh, bash, PowerShell, and cmd:

```bash
npm install
npx vitest run
npm run dev
```

Write them as separate commands in **How to run**. Do not join them with `&&` (Windows PowerShell 5.1 rejects it). Do not add `open`, `start`, `xdg-open`, `export`, or `$env:`. The app URL is `http://localhost:5173` unless the dev script prints a different one.

## Branch name

From **Application name** in the resolved requirements file:

1. Trim, lowercase, replace spaces with `-`, keep only `a-z`, `0-9`, and `-`, collapse repeat hyphens, strip edge hyphens.
2. **Base name:** `cursor/<slug>` (example: `Todo List` → `cursor/todo-list`).
3. **Uniqueness (new branch only):** before creating a branch, check whether that name already exists **locally or on remotes**:
   - `git branch --list --all -- "<name>"` and `git branch --list --all -- "*/<name>"` (see **Shell commands**). An exact local ref check is `git show-ref --verify --quiet "refs/heads/<name>"`.
   - If the base name is **free**, use it.
   - If it **already exists**, do **not** switch to it. Compute a unique name `cursor/<slug>-<datetime>` where `<datetime>` is the compact UTC timestamp for the current shell (see **Shell commands**; example: `cursor/todo-list-20260922T152830Z`). Check that suffixed name the same way. If it still exists, append `-` plus a 2-digit counter (`-01`, `-02`, …) until free.
4. If `docs/GITBRANCH.md` already records a `branch` field, **reuse that name** (later runs). Do not invent another unique name.

## Git (architect, first kickoff)

On the **first** design run (`docs/TASKS.md` missing or empty):

1. `git status` and `git branch --show-current`.
2. If `docs/GITBRANCH.md` already has a `branch` value, use that name (do not re-unique).
3. Else compute a **unique** branch name (see **Branch name**) and write `docs/GITBRANCH.md` (including `requirements` and the chosen `branch`).
4. If already on that unique branch, keep it.
5. Else `git switch -c "<unique-branch>"` from current HEAD (do not reset, force-checkout, or rewrite history). **Never** `git switch` onto an existing leftover `cursor/<slug>` that you did not record in `docs/GITBRANCH.md`.

On later runs: read `docs/GITBRANCH.md` and switch to that stored branch. Never create extra feature branches. Never push. Never commit unless the parent asked.

**Builder:** stay on the `branch` field in `docs/GITBRANCH.md`. At start: `git branch --show-current`; if wrong, `git switch` to that branch. If `docs/GITBRANCH.md` is missing, stop and tell the parent to spawn `app-architect` first. Do not create a new branch, switch to `main`, commit, or push unless the parent asked.
