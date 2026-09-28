# AI Academy Coding

A hands-on demo of using Cursor agents to build a web application from a written brief.

You stay in the chat and invoke two agents. **app-architect** reads the requirements and splits them into tasks. **app-builder** implements those tasks and writes a QA report. You send the architect back to read that report and mark the task done. The product they build is **Truck2Hand**, a marketplace for second-hand trucks.

The application source is not in this repository yet. This tree is the brief, the UI references, and the agent setup.

Thai version: [README_TH.md](README_TH.md).

## Before you start

Check out the exercise branch:

```text
git switch agents-ex02
```

- Open this repository in Cursor and use **Agent** chat.
- Invoke the agents yourself with `/app-architect` and `/app-builder`. The handoff between them is manual: neither agent starts the other.
- Node.js **24.20.0** is the runtime the brief pins (`.nvmrc` is created during the build). You need it when you run the finished app.
- The agents create a git branch named `cursor/<slug>`. They do not commit or push unless you ask.

## Repository structure

```text
.
├── .cursor/
│   ├── agents/
│   │   ├── app-architect.md      # design and task-board agent
│   │   └── app-builder.md        # implementation and QA agent
│   └── skills/
│       ├── app-delivery-contracts/SKILL.md
│       ├── app-architect-playbook/SKILL.md
│       └── app-builder-playbook/SKILL.md
├── docs/
│   ├── REQUIREMENTS.md           # product source of truth
│   └── specs/                    # acceptance criteria, API, database, UI images
├── backend/src/db/demo-photos/   # seed listing photos
└── README.md
```

### Agents

An agent is a role Cursor can run as a subagent. Each file under `.cursor/agents/` is one role.

| Agent | Invoke | What it does |
| --- | --- | --- |
| `app-architect` | `/app-architect` | Reads the requirements, writes the design and the task board, and later marks tasks done from QA reports. It does not write application code. |
| `app-builder` | `/app-builder` | Implements **one** task, runs tests, and writes a QA report. It marks that task `implemented` only when tests pass. It does not mark tasks `done`. |

Your chat is the coordinator. After the architect finishes, you start the builder. After the builder finishes, you start the architect again.

### Skills

A skill is a markdown playbook in `.cursor/skills/<name>/SKILL.md`. It is not an agent. The agent files tell each role to read its skills before doing any work, so the same rules apply every run.

| Skill | Who reads it | What it is for |
| --- | --- | --- |
| `app-delivery-contracts` | Both agents | Which requirements file to use, the git branch name, which doc each agent may edit, and the rules for running tasks at the same time. |
| `app-architect-playbook` | `app-architect` | How to write `docs/DESIGN.md`, `docs/TASKS.md`, and `docs/RUN.md`, how to read a QA report, and the two-round fix limit. |
| `app-builder-playbook` | `app-builder` | How to implement a single task, run tests once, and write `docs/QA-REPORT-<id>.md`. |

## How the demo runs

### 1. Design

Start with:

```text
/app-architect
```

The architect reads `docs/REQUIREMENTS.md` (or another path if you pass one, for example `/app-architect docs/REQUIREMENTS.md`). On the first run it:

- creates the app branch and writes `docs/GITBRANCH.md`
- writes `docs/DESIGN.md` and `docs/TASKS.md`
- records a start time in `docs/RUN.md`
- tells you the next single task, and which task ids can run together

Each task has an id (`T1`, `T2`, …), a `files` list, and `depends_on`. A dependency must be `implemented` or `done` before a later task starts.

### 2. Build

Default is one task:

```text
/app-builder Implement T1
```

To run every task the architect marked as safe together, copy those ids and say they should run at the same time:

```text
/app-builder Implement T2, T3, and T5 at the same time
```

`in parallel` means the same thing. Name the ids from the architect’s wave. Tasks that depend on unfinished work, or that edit the same files, stay in a later round. The builder will refuse that set and tell you which ids can go together.

For each task the builder implements only that task’s files, runs the tests once, and writes `docs/QA-REPORT-<id>.md`. A pass sets the task to `implemented`. A fail leaves it open.

### 3. Accept the work

Send the architect back in:

```text
/app-architect Review the new QA reports and update the task board
```

On a pass it sets the task to `done`. On a fail it adds a fix task (`T1-FIX1`, then at most `T1-FIX2`). You run `/app-builder` on that fix task, then `/app-architect` again. After two failed fix rounds the architect marks the original task `failed` and stops that line of work.

Repeat steps 2 and 3 until the board has no `open` or `implemented` tasks. The architect then writes the end time, duration, and **How to run** into `docs/RUN.md`.

### 4. Run the application

When every task is `done` (or the board has stopped after the fix limit), follow **How to run** in `docs/RUN.md`. That section is also copied into `docs/DESIGN.md`. Use those commands to install dependencies, run tests, and start the app. Do this only after the board is complete.

### Task status

| Status | Meaning |
| --- | --- |
| `open` | Not built yet, or a new fix task. |
| `implemented` | Builder finished and tests passed. Waiting for the architect. |
| `done` | Architect accepted a passing QA report. |
| `failed` | QA still failed after two fix rounds. |

## The application

**Truck2Hand** is a small used-truck marketplace. The UI copy is Thai. There is no login. Anyone can open the back office from the header.

| Page | Route | What you can do |
| --- | --- | --- |
| Home | `/` | Browse a grid of listings. Filter by vehicle type, price, year, brand, and model. Sort by newest, price, or most viewed. |
| Listing detail | `/products/:id` | See the gallery, description, vehicle facts, and seller. Call the seller or use the LINE button. |
| Manage listings | `/manage` | Table of listings. Create, edit, or delete. Create and edit use one form. Delete asks for confirmation. |

Stack, pinned in `docs/specs/definition-of-done.md`:

- **Frontend:** Vue 3, vue-router, Vite, plain CSS. Dev server `http://localhost:5173`.
- **Backend:** Express 5, Drizzle ORM, SQLite. API `http://localhost:8080`, Swagger at `/api-docs`.

The full behaviour, validation rules, and API shape are in `docs/REQUIREMENTS.md` and `docs/specs/`. Agents must not add features that are not listed there.

### What the screens are meant to look like

These images are a visual guide for layout and tone. They are not a second specification. When a picture and `docs/REQUIREMENTS.md` disagree, follow the written requirements.

The built pages will not match these images exactly. Agents interpret the brief, so spacing, colours, labels, and which controls appear can differ. That difference is expected.

**Home** — header, filters, and a card grid:

![Home page](docs/specs/ui/homepage.png)

**Listing detail** — gallery on the left, price and seller on the right:

![Listing details](docs/specs/ui/listing_details.png)

**Manage listings** — table with edit and delete:

![Manage listings](docs/specs/ui/manage_listing.png)

**Create a listing** — modal form for a new truck:

![Create a new listing](docs/specs/ui/create_a_new_listing.png)

**Delete** — confirmation before the listing is removed:

![Confirm delete](docs/specs/ui/confirmation_to_delete_a_listing.png)

**Warning:** several things in the pictures are out of scope and must not be built. That includes the home search box, card bookmarks, a listing status column, and some of the headings and button labels. Examples:

| Picture | Shown in the image | What to build |
| --- | --- | --- |
| Home | Search box, bookmarks, result count, vehicle-type checkboxes | No search box and no bookmarks. One vehicle-type select, a brand-dependent model dropdown, and the four sort options. |
| Manage | Status column ("กำลังขาย"), heading "ระบบจัดการหลังบ้าน" | Story 3 columns and the heading "ระบบจัดการสินค้า". No status column. |
| Delete | Solid red "ลบรายการ" button | Confirm with "ยืนยันการลบสินค้านี้?". Delete stays a quiet red control. |
| Detail | "โพสต์เมื่อ", buttons "แชทผ่าน LINE" / "โทรติดต่อ" | Buttons are "ติดต่อผู้ขาย" and "แอดไลน์". No "โพสต์เมื่อ". |
| Create | No vehicle-type field, LINE marked required | Show ประเภทรถ and the verified-seller checkbox. LINE is optional. |

The full list is the table “Screenshot differences” in `docs/REQUIREMENTS.md`.

## Files the agents add

| File | Written by | Purpose |
| --- | --- | --- |
| `docs/GITBRANCH.md` | Architect | Branch name and the requirements path |
| `docs/DESIGN.md` | Architect | Modules, data, UI, and how to run |
| `docs/TASKS.md` | Architect owns the board; a builder may set only its own task to `implemented` | The task board |
| `docs/RUN.md` | Architect | Start and end time, QA log, summary, and how to run |
| `docs/QA-REPORT-<id>.md` | Builder | Test result for that task |

After the board is complete, follow **How to run** in `docs/RUN.md` to install, test, and open the app.
