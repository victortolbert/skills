---
name: ralph
description: "Convert PRDs to prd.json format for the Ralph autonomous agent system. Use when you have an existing PRD and need to convert it to Ralph's JSON format. Triggers on: convert this prd, turn this into ralph format, create prd.json from this, ralph json."
---

# Ralph PRD Converter

Converts existing PRDs to the prd.json format that Ralph uses for autonomous execution.

---

## The Job

Take a PRD (markdown file or text) and convert it to `plan/prd.json`.

---

## Output Format

```json
{
  "project": "[Project Name]",
  "branchName": "ralph/[feature-name-kebab-case]",
  "description": "[Feature description from PRD title/intro]",
  "testStrategy": {
    "runner": "vitest (+ playwright for behaviors JSDOM can't honor)",
    "projects": {
      "nuxt": {
        "path": "test/nuxt/",
        "use": "Nuxt-runtime components + composables"
      },
      "unit": {
        "path": "test/unit/",
        "use": "Pure logic + source-grep conformance"
      },
      "e2e": { "path": "test/e2e/", "use": "Playwright SSR + cross-browser" }
    },
    "commands": {
      "watch": "pnpm test",
      "once": "pnpm vitest run",
      "scoped": "pnpm test:nuxt | pnpm test:unit",
      "e2e": "pnpm test:e2e"
    },
    "failingTestsFirst": [
      {
        "story": "US-001",
        "file": "test/unit/foo.test.ts",
        "suite": "...",
        "asserts": "one-sentence summary of what the test asserts"
      }
    ],
    "exceptions": [
      {
        "story": "US-00X",
        "reason": "why this story has no failing test",
        "verification": "how we know it's done instead"
      }
    ],
    "rhythm": "write test → confirm red for the RIGHT reason → implement → confirm green → full suite green → typecheck"
  },
  "userStories": [
    {
      "id": "US-001",
      "title": "[Story title]",
      "description": "As a [user], I want [feature] so that [benefit]",
      "failingTest": {
        "location": "test/unit/foo.test.ts",
        "asserts": "one-sentence summary — the behavior that doesn't exist yet",
        "expectedRed": "the failure you expect to see before implementation"
      },
      "exception": null,
      "acceptanceCriteria": [
        "Failing test written and confirmed red for the right reason",
        "Criterion 1",
        "Criterion 2",
        "All tests green (no regressions)",
        "Typecheck passes"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    },
    {
      "id": "US-00X",
      "title": "Story that cannot or should not carry a failing test",
      "description": "As a [user], I want [...] so that [...]",
      "failingTest": null,
      "exception": {
        "reason": "Pure find-and-replace / docs-only / CSS-only / pure refactor with existing coverage. Wrapping in a .test.ts is test-theater.",
        "verification": "grep -rE '...' returns 0 results | snapshot matches | etc."
      },
      "acceptanceCriteria": [
        "Criterion verifiable via the exception.verification path",
        "All tests green",
        "Typecheck passes"
      ],
      "priority": 99,
      "passes": false,
      "notes": ""
    }
  ]
}
```

**Required-field rules.**

- Every story carries `failingTest` OR `exception` — never both, never neither. When `failingTest` is set, `exception` is `null`. When `exception` is set, `failingTest` is `null`.
- Top-level `testStrategy` is required. `failingTestsFirst` enumerates one row per non-exception story. `exceptions` mirrors the story-level exception entries for quick scan.
- `testStrategy.projects` lists every test project the PRD touches. If a PRD introduces a new test project (e.g. Playwright `test/e2e/`), add it here and mention that the directory is created by the story that introduces it.
- `passes: false` and `notes: ""` on every story at write-time. Ralph flips `passes: true` and fills `notes` as stories land.

**When a PRD is a stub** (deferred, reserved scope, no real stories yet): do NOT generate a `prd.json` for it. Stubs live as markdown only. See the "Stub PRD Pattern" section below.

---

## Story Size: The Number One Rule

**Each story must be completable in ONE Ralph iteration (one context window).**

Ralph spawns a fresh Claude instance per iteration with no memory of previous work. If a story is too big, the LLM runs out of context before finishing and produces broken code.

### Right-sized stories:

- Add a database column and migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list

### Too big (split these):

- "Build the entire dashboard" - Split into: schema, queries, UI components, filters
- "Add authentication" - Split into: schema, middleware, login UI, session handling
- "Refactor the API" - Split into one story per endpoint or pattern

**Rule of thumb:** If you cannot describe the change in 2-3 sentences, it is too big.

---

## Story Ordering: Dependencies First

Stories execute in priority order. Earlier stories must not depend on later ones.

**Correct order:**

1. Schema/database changes (migrations)
2. Server actions / backend logic
3. UI components that use the backend
4. Dashboard/summary views that aggregate data

**Wrong order:**

1. UI component (depends on schema that does not exist yet)
2. Schema change

---

## Acceptance Criteria: Must Be Verifiable

Each criterion must be something Ralph can CHECK, not something vague.

### Good criteria (verifiable):

- "Add `status` column to tasks table with default 'pending'"
- "Filter dropdown has options: All, Active, Completed"
- "Clicking delete shows confirmation dialog"
- "Typecheck passes"
- "Tests pass"

### Bad criteria (vague):

- "Works correctly"
- "User can do X easily"
- "Good UX"
- "Handles edge cases"

### Always include as final criterion:

```
"Typecheck passes"
```

For stories with testable logic, also include:

```
"Tests pass"
```

### For stories that change UI, also include:

```
"Verify in browser using Playwright MCP tools (browser_navigate, browser_snapshot)"
```

Frontend stories are NOT complete until visually verified. Ralph will use Playwright MCP tools to navigate to the page, take snapshots, and confirm changes work.

---

## Conversion Rules

1. **Each user story becomes one JSON entry**
2. **IDs**: Sequential (US-001, US-002, etc.) - **IDs reset per PRD** (each new feature starts at US-001)
3. **Priority**: Based on dependency order, then document order
4. **All stories**: `passes: false` and empty `notes`
5. **branchName**: Derive from feature name, kebab-case, prefixed with `ralph/`
6. **Always add**: "Typecheck passes" to every story's acceptance criteria

---

## Splitting Large PRDs

If a PRD has big features, split them:

**Original:**

> "Add user notification system"

**Split into:**

1. US-001: Add notifications table to database
2. US-002: Create notification service for sending notifications
3. US-003: Add notification bell icon to header
4. US-004: Create notification dropdown panel
5. US-005: Add mark-as-read functionality
6. US-006: Add notification preferences page

Each is one focused change that can be completed and verified independently.

---

## Example

**Input PRD:**

```markdown
# Task Status Feature

Add ability to mark tasks with different statuses.

## Requirements

- Toggle between pending/in-progress/done on task list
- Filter list by status
- Show status badge on each task
- Persist status in database
```

**Output prd.json:**

```json
{
  "project": "TaskApp",
  "branchName": "ralph/task-status",
  "description": "Task Status Feature - Track task progress with status indicators",
  "userStories": [
    {
      "id": "US-001",
      "title": "Add status field to tasks table",
      "description": "As a developer, I need to store task status in the database.",
      "acceptanceCriteria": [
        "Add status column: 'pending' | 'in_progress' | 'done' (default 'pending')",
        "Generate and run migration successfully",
        "Typecheck passes"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    },
    {
      "id": "US-002",
      "title": "Display status badge on task cards",
      "description": "As a user, I want to see task status at a glance.",
      "acceptanceCriteria": [
        "Each task card shows colored status badge",
        "Badge colors: gray=pending, blue=in_progress, green=done",
        "Typecheck passes",
        "Verify in browser using Playwright MCP tools"
      ],
      "priority": 2,
      "passes": false,
      "notes": ""
    },
    {
      "id": "US-003",
      "title": "Add status toggle to task list rows",
      "description": "As a user, I want to change task status directly from the list.",
      "acceptanceCriteria": [
        "Each row has status dropdown or toggle",
        "Changing status saves immediately",
        "UI updates without page refresh",
        "Typecheck passes",
        "Verify in browser using Playwright MCP tools"
      ],
      "priority": 3,
      "passes": false,
      "notes": ""
    },
    {
      "id": "US-004",
      "title": "Filter tasks by status",
      "description": "As a user, I want to filter the list to see only certain statuses.",
      "acceptanceCriteria": [
        "Filter dropdown: All | Pending | In Progress | Done",
        "Filter persists in URL params",
        "Typecheck passes",
        "Verify in browser using Playwright MCP tools"
      ],
      "priority": 4,
      "passes": false,
      "notes": ""
    }
  ]
}
```

---

## Stub PRD Pattern

Some PRDs live as markdown-only for a while — the scope is reserved, but the work hasn't been planned or isn't ready to execute. Signals:

- Header includes `**Status:** Deferred — do NOT start until …` (or similar).
- No concrete user stories; just story seeds / audit scope / open questions.
- Footer explicitly says: _"Do not invoke `/ralph` against this PRD until graduated."_

**Rule:** when asked to convert a stub PRD, refuse politely and flag that the PRD is a stub. Do not write a `prd.json`. Point the user at the PRD's header/footer and ask what's needed to graduate it to live.

A stub graduates to live when it has: a user story list with failing-test blocks (or explicit exceptions), acceptance criteria, a test strategy, and the status is updated to `Draft v1`. At that point, `/ralph` can convert it.

---

## Prerequisite Chaining

PRDs can declare `**Blocks on:** plan/tasks/prd-foo.md` in their header to express a dependency on another PRD's completion. When converting:

- Read the referenced PRD's status and `plan/archive/` for a `-complete` directory.
- If the blocker isn't complete, flag to the user before writing the new `prd.json` — the Ralph run may fail or conflict if the prerequisite's work hasn't landed.
- Include the `Blocks on:` reference in the top-level `description` field so future operators understand the ordering.

---

## Archiving Previous Runs

Two archive variants:

**Pre-run archive** (`plan/archive/YYYY-MM-DD-feature-name/`) — snapshots the prd.json + progress.txt at the moment a new feature is staged, BEFORE the feature runs. Preserves the pristine starting state.

**Post-completion archive** (`plan/archive/YYYY-MM-DD-feature-name-complete/`) — snapshots the prd.json (all `passes: true`) + progress.txt (full execution log) at the moment the feature finishes, BEFORE transitioning to the next feature. Preserves the record of what was done and how.

When transitioning from a completed feature to a new one:

1. Read current `plan/prd.json`. If all stories have `passes: true`, the feature is complete.
2. If `branchName` differs from the new feature's branch name AND the feature is complete:
   - Create `plan/archive/YYYY-MM-DD-[completed-feature-name]-complete/` and copy the completed `prd.json` + `progress.txt` there. (The pre-run archive from this feature's start should already exist under `plan/archive/YYYY-MM-DD-[completed-feature-name]/`.)
   - Overwrite `plan/prd.json` with the new feature's content.
   - Reset `progress.txt` to a fresh header: `# Ralph Progress Log — [Feature Name] ([branchName])`.
3. If the current run is mid-execution (mixed `passes` values) and `branchName` differs from the new one: **pause and flag to the user**. Mid-run branch switches risk losing execution state.

**Folder structure:**

```
plan/
├── prd.json              # Current PRD being executed
├── archive/              # Historical runs (two variants per feature)
│   ├── 2026-04-14-loop-workflow-hero/            # pre-run snapshot (0/11 passing)
│   ├── 2026-04-14-loop-workflow-hero-complete/   # post-run snapshot (11/11 passing)
│   └── 2026-04-14-workbench-data-scaffolding/    # (and its -complete sibling when finished)
├── tasks/                # Original PRD markdown files
│   └── prd-*.md
└── ralph.sh              # Execution script

progress.txt              # Root-level progress log (appended by Ralph)
```

**Note:** Archiving is manual. Check before converting a new PRD to avoid overwriting completed work.

---

## Updating the Backlog

When staging a new `prd.json`, always update `AGENTS.md` § Backlog:

1. **Add the new feature** as a bullet if it isn't already listed.
2. **If replacing a partially-complete PRD** (some stories still `passes: false`), add a note to the backlog: `[feature] — X/Y stories complete, remaining: [list incomplete story titles]`. This prevents work from silently disappearing.
3. **If the previous PRD completed** (all `passes: true`), remove it from the backlog (it's in `plan/archive/`).

This ensures the backlog in AGENTS.md always reflects reality — completed work gets archived, incomplete work stays visible.

---

## Checklist Before Saving

Before writing `plan/prd.json`, verify:

- [ ] **PRD is not a stub** (if it is, refuse — point at the graduate-to-live requirements)
- [ ] **Prerequisite PRDs complete** (if the PRD header has `Blocks on:`, confirm the referenced PRD has a `-complete` archive or all-passing prd.json)
- [ ] **Previous run archived** (pre-run archive exists if this is a new feature; post-completion archive created if transitioning from a completed run)
- [ ] **AGENTS.md backlog updated** (new feature added, previous feature removed or marked incomplete)
- [ ] User story IDs start at US-001 (IDs reset per PRD)
- [ ] Each story is completable in one iteration (small enough)
- [ ] Stories are ordered by dependency (schema to backend to UI)
- [ ] **Every story carries `failingTest` OR `exception` — never both, never neither**
- [ ] **Top-level `testStrategy` object present** (runner, projects, commands, failingTestsFirst, exceptions, rhythm)
- [ ] **`testStrategy.failingTestsFirst` has one row per non-exception story; `testStrategy.exceptions` mirrors story-level exceptions**
- [ ] Every story has "Typecheck passes" as criterion
- [ ] UI stories have "Verify in browser using Playwright MCP tools" as criterion
- [ ] Acceptance criteria are verifiable (not vague)
- [ ] No story depends on a later story
- [ ] Root `progress.txt` reset with header only
