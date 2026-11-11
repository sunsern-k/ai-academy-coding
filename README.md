# AI Academy for Developers

Welcome to **AI Academy for Developers**. This repository is the course workspace for learning how to use AI effectively as a software developer—from writing better prompts, to running Cursor agents with skills, to orchestrating agents with a single command.

Material lives on **separate git branches**. Start here on `master` for the overview, then switch to the branch for the session you are in.

Thai version: [README_TH.md](README_TH.md).

## What you will learn

1. **Prompts** — Structure useful developer prompts (role, context, task, constraints) and reuse examples from the session.
2. **Agents** — Use Cursor custom agents and skills to design and build a simple app, invoking each agent yourself.
3. **Agents (complex app)** — Apply the same agent pattern to a fuller web marketplace.
4. **Autonomous agents** — Let an agent conductor orchestrate the same build with one command.

## Branches in this repository

| Branch | Session focus |
| --- | --- |
| [`prompts`](#prompts) | Useful developer prompts and RCTC examples from class |
| [`agents`](#agents) | Todo List demo with `app-architect` and `app-builder` (manual handoff) |
| [`agents-ex02`](#agents-ex02) | Truck2Hand marketplace with the same agents (manual handoff) |
| [`autonomous-agents-ex02`](#autonomous-agents-ex02) | Same Truck2Hand app, driven by `app-conductor` in one loop |

## How to switch branches

From the repository root in a terminal (or Cursor’s terminal):

```bash
# See available branches
git branch -a

# Switch to a session branch
git switch prompts
git switch agents
git switch agents-ex02
git switch autonomous-agents-ex02

# Return to this overview
git switch master
```

After switching, open that branch’s `README.md` (and `README_TH.md` if you prefer Thai) for full steps. If Cursor still shows files from another branch, reload the window or reopen the folder so the agent setup matches the branch.

**Tip:** Finish or stash local work before switching if you have uncommitted changes, so you do not mix session materials.

---

## prompts

**Useful prompts for developers**, including examples explained in the AI Academy session.

What you will find:

- `50_dev_prompts_en.md` / `50_dev_prompts_th.md` — Fifty software-development prompts (English and Thai), each shaped as **role → context → task → constraints** (RCTC).
- `examples/` — Side-by-side “without / with” examples for each RCTC piece:
  - Role
  - Context
  - Task
  - Constraint

Topics in the prompt set cover planning and requirements, architecture, coding, testing, debugging, documentation, and related developer work.

```bash
git switch prompts
```

Open the markdown or `.txt` files and try the prompts in Cursor Agent chat during the session.

---

## agents

A **simple Todo List** demo that applies Cursor **AI agents**, **subagents**, and **skills**.

You stay in the chat and call agents **one by one**. The product is a Vite + TypeScript todo app (localStorage, Vitest). Application source under `src/` is created only when the builder runs; the branch starts with agents, skills, and requirements.

| Agent | Invoke | Role |
| --- | --- | --- |
| `app-architect` | `/app-architect` | Reads requirements, creates the app branch, writes design and task board docs. Does not write app code. |
| `app-builder` | `/app-builder` | Implements **one** open task, runs tests once, writes a QA report. |

Skills under `.cursor/skills/` define playbooks and shared file/shell contracts. Typical flow: `/app-architect` → `/app-builder` (repeat) → architect again to mark tasks done or add fix tasks (at most two fix rounds).

```bash
git switch agents
```

Full walkthrough: open that branch’s `README.md` / `README_TH.md` after switching.

---

## agents-ex02

A **more complex** demo: **Truck2Hand**, a web marketplace to post and sell second-hand trucks.

Same agent concepts as the `agents` branch (`app-architect`, `app-builder`, skills), still with **manual** handoff in chat—you invoke each agent yourself. The tree holds the brief, UI references, and agent setup; app source is produced during the exercise.

Stack (pinned in the brief): Vue 3 + Vite frontend, Express 5 + Drizzle + SQLite backend. Node.js **24.20.0** is required to run the finished app. Backend install may need native build tools (`better-sqlite3`).

```bash
git switch agents-ex02
```

Then follow that branch’s README: design with `/app-architect`, build with `/app-builder`, review QA with the architect again, and run the app from **How to run** in `docs/RUN.md` when the board is complete.

---

## autonomous-agents-ex02

Builds the **same Truck2Hand** marketplace as `agents-ex02`, but introduces **`app-conductor`** to orchestrate the architect and builder.

You start with **one command**; the conductor runs the design → build → QA loop until the board is finished or a stop rule fires. You no longer need to call architect and builder on every task yourself (manual handoff remains optional).

| Agent | Invoke | Role |
| --- | --- | --- |
| `app-conductor` | `/app-conductor` | Spawns architect and builder in turn; writes `docs/LOOP.md`. Does not design or write app code. |
| `app-architect` / `app-builder` | Spawned by the conductor (or still callable manually) | Same roles as on `agents-ex02` |

Optional flags include pausing after design (`/app-conductor pause after design`) and allowing parallel waves (`/app-conductor allow parallel`).

**Speed tip:** serial is the default. To go faster, kick off with parallel mode so the conductor runs as many architect-marked safe tasks together as it can:

```text
/app-conductor allow parallel
```

An open fix task still runs alone. If a parallel wave leaves unmerged worktrees, the conductor stops (it does not merge or commit)—see that branch’s README for stop reasons and resume.

```bash
git switch autonomous-agents-ex02
```

Start the loop with `/app-conductor` (or the parallel form above), then use that branch’s README for stop reasons, resume (`continue`), and how to run the finished app.

---

## Prerequisites (all agent sessions)

- [Cursor](https://cursor.com) with **Agent** chat
- Git
- Node.js on `PATH` (version pinned on Truck2Hand branches: **24.20.0**)

Agents do not commit or push unless you ask. Prefer the README on the branch you switched to for exact commands, OS notes, and acceptance criteria.

## Suggested path through the course

1. `prompts` — Practice RCTC and session prompt patterns.
2. `agents` — Learn architect / builder / skills on a small Todo app.
3. `agents-ex02` — Scale the same pattern to Truck2Hand with manual handoffs.
4. `autonomous-agents-ex02` — Automate that workflow with `app-conductor`.

If you are unsure which branch to open, ask the instructor for the current session name, then `git switch` to that branch and read its README before invoking any agent.
