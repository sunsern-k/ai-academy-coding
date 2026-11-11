# AI Academy Coding

Cursor demo that designs and builds a **Todo List** web app from [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md). Two custom agents share one task board. The same git, timestamp, install, test, and start commands work in macOS zsh and Windows PowerShell.

The product is a Vite + TypeScript app: one todo list, stored in `localStorage`, tested with Vitest. Users can add, view, edit, change status, and delete items. The source of truth for behavior is the requirements file. The agents do not add features that are not listed there.

Nothing in `src/` exists until the builder runs. This repository starts as the agents, their skills, and the requirements.

## Agents

| Agent | Role |
| --- | --- |
| `app-architect` | Reads the requirements, creates a git branch (`cursor/todo-list`, or a timestamped name if that branch exists), and writes `docs/DESIGN.md`, `docs/TASKS.md`, `docs/GITBRANCH.md`, and `docs/RUN.md`. After each QA report it marks the task `done` or adds a fix task. It does not write application code. |
| `app-builder` | Implements **one** open task, runs Vitest once, and writes `docs/QA-REPORT-<id>.md`. A passing task becomes `implemented`. The architect marks it `done`. |

Skills live under `.cursor/skills/`. Shell rules for Windows and macOS are in `.cursor/skills/app-delivery-contracts/SKILL.md` (**Shell commands**). A task may be fixed at most twice (`T1-FIX1`, then `T1-FIX2`).

## Prerequisites

- [Cursor](https://cursor.com)
- Git
- Node.js on `PATH` (needed when the builder installs and tests the app)

On macOS, Cursor’s shell is usually zsh. On Windows, it is usually PowerShell. Git Bash on Windows can use the Unix `date` commands; PowerShell cannot (`date` there is `Get-Date`).

## Run the demo

Open this folder in Cursor, then use a new Agent chat.

1. Design the app:

   ```text
   /app-architect
   ```

   Default requirements path is `docs/REQUIREMENTS.md`. To use another file: `/app-architect path/to/file.md`.

2. Build the next task (repeat until the board has no `open` work):

   ```text
   /app-builder
   ```

   Name a task when you want a specific one, for example `/app-builder T1`. After the builder writes a QA report, run `/app-architect` again so it can mark the task `done` or add a fix task.

3. When every original task is `done` or stopped after the fix limit, read **How to run** in `docs/RUN.md`.

Ask for parallel builders only by naming the task ids (for example T1–T4). The builder checks that their files do not overlap before starting them together.

The agents do not commit or push unless you ask.

## Run the Todo app

After the builder has added `package.json`, use the same three commands on macOS and Windows. Run them separately. PowerShell 5.1 does not accept `&&`.

```bash
npm install
npx vitest run
npm run dev
```

Open the URL the dev script prints. Vite’s default is [http://localhost:5173](http://localhost:5173).

## Layout

| Path | Purpose |
| --- | --- |
| `docs/REQUIREMENTS.md` | Product source: name, stack, acceptance criteria |
| `docs/DESIGN.md` | Written by the architect: modules, UI, how to run |
| `docs/TASKS.md` | Task board (`open`, `implemented`, `done`, `failed`) |
| `docs/GITBRANCH.md` | Branch name and the requirements path the builder must use |
| `docs/RUN.md` | Start time, QA log, and the final how-to-run summary |
| `docs/QA-REPORT-<id>.md` | One test report per task |
| `.cursor/agents/` | `app-architect` and `app-builder` |
| `.cursor/skills/` | Playbooks and the shared file and shell contract |
