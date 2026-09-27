# Requirements: Todo List

- **Application name:** Todo List

This file is the product source of truth for `app-architect` and `app-builder`. Do not add features that are not listed here.

## Stack

- **Vite + TypeScript** web app
- Domain logic in a testable store module (e.g. `src/todoStore.ts`); thin UI on top
- Persistence: browser **localStorage**
- Tests: **Vitest** unit/integration only (jsdom/happy-dom is OK). No Playwright / E2E as the pass/fail source

## Product

- One list of todo **items** (not multiple named lists)
- Users can **add**, **view**, **edit title**, **change status**, and **delete** items
- Empty state when there are no items

## Item shape

```ts
{
  id: string
  title: string
  status: "open" | "in progress" | "done"
  createdAt: string // ISO-8601
}
```

- Title: required, trim whitespace, reject empty
- Default status on create: `"open"`

## Look and feel

The UI is part of the demo: it must look **professional and clean**, not a default unstyled page and not a flashy marketing site.

- **Theme:** one light theme only. Neutral page background, white content surface, dark text.
- **Accent:** one primary color for the main action (add). Suggested: `#2563eb` (blue). Hover/focus uses a slightly darker shade of the same hue.
- **Status colors** (text or small badge, not rainbow backgrounds): open = slate/gray, in progress = the primary blue, done = green (`#16a34a`).
- **Type:** system UI stack (`system-ui`, `-apple-system`, `Segoe UI`, `sans-serif`). Clear hierarchy: app title, item title, secondary meta (`createdAt` smaller and muted).
- **Layout:** centered column, max width ~40rem, comfortable padding and spacing between add form, list, and empty state.
- **Components:** add form, item row (title, status control, edit, delete), empty state with a short calm message (e.g. “No todos yet”). Buttons are simple solid/outline; delete is a quiet danger style (red text or outline, not a large red block).
- **Motion:** none required. **Do not** add animation libraries, loaders, confetti, parallax, or transitions longer than ~150ms. Instant state updates are preferred.
- **Do not:** dark mode, gradients, glassmorphism, drop shadows beyond a light card shadow, icons packs unless already in the stack, extra fonts from the network.

## Acceptance criteria (testable)

- Add item: new item has `id`, trimmed `title`, status `"open"`, and `createdAt` (ISO-8601)
- Reject empty or whitespace-only title on add and on edit
- List/view all items
- Edit title (valid update succeeds; invalid title is rejected)
- Status can change among `"open"`, `"in progress"`, and `"done"`
- Delete item removes it from the list
- Persist and reload from `localStorage` (mock `localStorage` in unit tests)
- Empty list is a valid state (empty UI / empty store)
- UI uses a single light theme with CSS custom properties (or equivalent) for background, surface, text, primary, and status colors; the add control uses the primary color
- No animation library dependency; no CSS `animation` / infinite keyframes for decorative motion

## Out of scope

Auth, backend, due dates, priority, filters, multiple lists, undo, sharing, dark mode, custom webfonts, animation libraries
