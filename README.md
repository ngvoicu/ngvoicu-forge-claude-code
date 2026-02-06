# ngvoicu-forge-claude-code

Claude Code plugin marketplace with six plugins for spec-driven development, deep research, living documentation, surgical code editing, and UI specification workflows. All plugins are markdown-based — no build system, no dependencies.

## Quick Start

```bash
# Add the marketplace
/plugin marketplace add https://github.com/ngvoicu/ngvoicu-forge-claude-code.git

# Install what you need
/plugin install specsmith@ngvoicu-forge-claude-code
/plugin install grimoire@ngvoicu-forge-claude-code
/plugin install runebook@ngvoicu-forge-claude-code
/plugin install chisel@ngvoicu-forge-claude-code
/plugin install uispec-backend@ngvoicu-forge-claude-code
/plugin install uispec-frontend@ngvoicu-forge-claude-code
```

---

## Plugins

### specsmith

Research-driven implementation specs. Every `/specsmith-new` triggers a structured process:

1. **Discovery interview** — Rounds of focused questions about requirements, edge cases, failure modes, constraints, and boundaries. Doesn't stop until the problem is deeply understood. All Q&A is logged to `.specsmiths/<n>.questions.md`.
2. **Deep research** — Adaptive subagents (1-6) investigate based on what discovery revealed. If grimoire is installed, uses its specialized agents for richer research. Findings go to `.specsmiths/<n>.research.md`.
3. **Spec creation** — A full specification with requirements, architecture, data model, API design, edge case table, implementation steps (each with What/Where/How/Verify), risks, testing strategy, and rollback plan.

The result is a spec you could hand to any developer and they'd know exactly what to build. All state lives in `.specsmiths/` as plain markdown — commit it to git.

```
/specsmith-new <name>       # discovery -> research -> full spec
/specsmith-implement        # start active spec implementation
/specsmith-resume           # re-read everything, continue active spec
/specsmith-switch <name>    # save progress, switch active spec
/specsmith-edit <name>      # revise spec mid-flight
/specsmith-status           # dashboard of all specs with phase progress
/specsmith-list             # interactive list with picker
/specsmith-show [name]      # display spec without activating
/specsmith-archive <name>   # shelve completed/paused spec
/specsmith-unarchive <name> # restore archived spec
/specsmith-drop <name>      # delete spec + research + questions
```

### grimoire

Research suite with 6 specialized agents that can be invoked individually or in parallel:

| Stream | Agent | Purpose |
|--------|-------|---------|
| `docs` | grimoire-docs | Library documentation via Context7 MCP + web fallback |
| `practices` | grimoire-practices | Best practices, pitfalls, security considerations |
| `codebase` | grimoire-codebase | Architecture, patterns, files affected |
| `history` | grimoire-history | Git log, TODOs, ADRs, prior attempts |
| `schema` | grimoire-schema | DB schema, migrations, indexes |
| `contracts` | grimoire-contracts | External API docs, rate limits, auth |

Integrates with specsmith Phase 2 — when grimoire is installed, specsmith uses its agents for research instead of generic subagents.

```
/grimoire                          # show available streams
/grimoire docs <lib> <query>       # library documentation lookup
/grimoire practices <topic>        # best practices & pitfalls
/grimoire codebase <area>          # codebase analysis
/grimoire history <area>           # git history & prior work
/grimoire schema <area>            # DB schema analysis
/grimoire contracts <service>      # external API contracts
/grimoire research <context>       # full parallel research (all relevant agents)
```

### runebook

Living documentation that tracks how an application works. Maintains structured docs for endpoints, jobs, flows, services, and integrations — always in sync with code changes.

Runebook is **assertive**: it reads entries before code work and updates them after changes automatically, without asking. All state lives in `.runebook/` as plain markdown — commit it to git.

```
/runebook-init              # scan codebase, create .runebook/
/runebook-update [name]     # update specific entry or auto-detect from changes
/runebook-validate          # check staleness, coverage, broken refs
/runebook-show [name]       # display entry or master index
/runebook-status            # dashboard with coverage and staleness
```

### chisel

Fast surgical code editor for renaming, moving, replacing, and making small targeted edits. Uses the haiku model for speed. Automatically updates all references and imports across any programming language.

Key capabilities:
- **Git-aware** — uses `git mv` for renames/moves to preserve file history
- **Framework-aware** — finds co-located tests, CSS modules, stories, barrel exports
- **Scope-aware** — respects word boundaries, warns on large scope, previews changes
- **Language-agnostic** — handles any language, framework, or file type

```
/chisel                                # show available operations
/chisel rename <old> to <new>          # rename + update all references
/chisel move <source> to <dest>        # move + update all import paths
/chisel replace <old> with <new>       # find-and-replace with scope analysis
/chisel edit <description>             # small targeted code change
/chisel <natural language>             # auto-detect operation
```

### uispec-backend

Backend plugin for maintaining UI specifications. After any API change, syncs `openapi.yaml`, endpoint spec files, and `components.md` in the frontend project's `uispec/` directory. Enforces strict ownership boundaries — backend owns API sections, frontend owns UI sections.

```
/uispec-init      # initialize uispec/ from existing backend endpoints
/uispec-sync      # sync after backend API changes
/uispec-validate  # check spec consistency and coverage
```

### uispec-frontend

Frontend plugin for consuming UI specifications. Reads specs from `uispec/` to guide component development. Knows which sections are backend-owned (API contract — read only) and which are frontend-owned (design system, components, UI guidelines — editable). Automatically activates before API-connected UI work.

```
/uispec-validate  # check spec consistency and coverage
```

---

## How Plugins Work Together

```
specsmith ──→ grimoire     Phase 2 uses grimoire agents for research
specsmith ──→ runebook     Read entries during implementation, update after
runebook  ──→ chisel       After rename/move, update runebook source_files
uispec    ──→ runebook     Keep endpoint docs consistent across both
grimoire  ──→ chisel       Research scope before large renames
chisel    ──→ uispec       Update spec files when renaming endpoints
```

---

## General CLAUDE.md Guidelines

Paste these into any project's `CLAUDE.md` as a baseline. They work standalone or alongside the plugin snippets below.

```markdown
## Agent Guidelines

### Read Before Acting
- NEVER speculate about code you haven't read — always open and read files before answering questions or making claims
- When the user references a file, read it first — give grounded, hallucination-free answers
- Before modifying code, read the surrounding context to understand patterns and conventions

### Plan Before Building
- Think through the problem before writing code
- For changes touching more than 2-3 files, check in with a plan before implementing
- Explain your approach at a high level before diving into details

### Keep It Simple
- Make every change as small and focused as possible
- Prefer editing existing files over creating new ones
- Don't over-engineer — no extra abstractions, no speculative features, no unnecessary refactoring
- If three lines of straightforward code work, don't create a helper function

### Explain As You Go
- Give a brief high-level explanation of each change you make
- After completing a task, summarize what was changed and why

### Verify Your Work
- After making changes, check that nothing is broken
- Run tests if they exist
- If you introduced a bug, fix it before moving on
```

---

## CLAUDE.md Snippets

Each plugin includes a ready-to-paste CLAUDE.md snippet for target projects. Add these to your project's `CLAUDE.md` (after the general guidelines above) to enforce plugin usage.

**Snippet locations:**
- Specsmith: managed via the specsmith-management awareness skill (auto-detects `.specsmiths/`)
- Grimoire: `grimoire/skills/grimoire-awareness/assets/CLAUDE-snippet.md`
- Runebook: `runebook/skills/runebook-awareness/assets/CLAUDE-snippet.md`
- Chisel: `chisel/skills/chisel-awareness/assets/CLAUDE-snippet.md`
- UISpec Backend: `uispec-backend/skills/uispec-backend/assets/CLAUDE-snippet.md`
- UISpec Frontend: `uispec-frontend/skills/uispec-frontend/assets/CLAUDE-snippet.md`

### For Backend Projects

Paste the following snippets into your backend project's `CLAUDE.md`:

<details>
<summary><strong>Runebook snippet</strong> (click to expand)</summary>

```markdown
## Runebook (MANDATORY)

Living documentation lives in `.runebook/`. **ALWAYS consult before changes, ALWAYS update after changes — automatically, without asking.**

### Before ANY Code Changes

You MUST read relevant runebook entries before modifying code:
1. Identify which components will be affected by your changes
2. Read their entries from `.runebook/<type>/<name>.md`
3. Understand current behavior, dependencies, edge cases, and recent changes

### After ANY Code Changes

You MUST update the runebook after changes — do it automatically, do not ask:
- New endpoint → Create `.runebook/endpoints/<name>.md`
- New job → Create `.runebook/jobs/<name>.md`
- New service → Create `.runebook/services/<name>.md`
- New integration → Create `.runebook/integrations/<name>.md`
- Modified component → Update the existing entry
- Removed component → Flag entry as removed

Commands: /runebook-init, /runebook-update, /runebook-validate, /runebook-show, /runebook-status
```

See `runebook/skills/runebook-awareness/assets/CLAUDE-snippet.md` for the full version with naming conventions, integration notes, and error handling.

</details>

<details>
<summary><strong>Grimoire snippet</strong> (click to expand)</summary>

```markdown
## Grimoire (MANDATORY)

Research suite with 6 specialized agents. **ALWAYS research before guessing.**

- NEVER guess library APIs — use `/grimoire docs <library> <question>`
- ALWAYS research best practices before security-sensitive features — `/grimoire practices <topic>`
- ALWAYS analyze codebase before major refactors — `/grimoire codebase <area>`
- For complex features — `/grimoire research <context>` (spawns all relevant agents)

Commands: /grimoire, /grimoire docs, /grimoire practices, /grimoire codebase, /grimoire history, /grimoire schema, /grimoire contracts, /grimoire research
```

See `grimoire/skills/grimoire-awareness/assets/CLAUDE-snippet.md` for the full version.

</details>

<details>
<summary><strong>Chisel snippet</strong> (click to expand)</summary>

```markdown
## Chisel (MANDATORY)

Fast surgical code editor. **NEVER manually rename or move files — always use chisel.**

- `/chisel rename <old> to <new>` — rename + update all references
- `/chisel move <source> to <dest>` — move + update all import paths
- `/chisel replace <old> with <new>` — find-and-replace with scope analysis
- `/chisel edit <description>` — small targeted code change

Git-aware (preserves history), framework-aware (finds tests, styles, stories), any language.
```

See `chisel/skills/chisel-awareness/assets/CLAUDE-snippet.md` for the full version.

</details>

<details>
<summary><strong>UISpec Backend snippet</strong> (click to expand)</summary>

```markdown
## UISpec — Backend (MANDATORY)

UI specs live in `../{{PRJ-frontend}}/uispec/`. **ALWAYS sync after API changes.**

After creating/modifying/removing any endpoint:
1. Update `../{{PRJ-frontend}}/uispec/openapi.yaml`
2. Update `../{{PRJ-frontend}}/uispec/endpoints/<endpoint>.md`
3. Update `../{{PRJ-frontend}}/uispec/components.md` if new UI patterns needed

No exceptions. Frontend depends on this.

Commands: /uispec-init, /uispec-sync, /uispec-validate
```

Replace `{{PRJ-frontend}}` with your frontend project directory name. See `uispec-backend/skills/uispec-backend/assets/CLAUDE-snippet.md` for the full version with ownership rules and conflict handling.

</details>

### For Frontend Projects

Paste the following snippets into your frontend project's `CLAUDE.md`:

<details>
<summary><strong>Runebook snippet</strong> (same as backend — click to expand)</summary>

Same snippet as above. See `runebook/skills/runebook-awareness/assets/CLAUDE-snippet.md`.

</details>

<details>
<summary><strong>Grimoire snippet</strong> (same as backend — click to expand)</summary>

Same snippet as above. See `grimoire/skills/grimoire-awareness/assets/CLAUDE-snippet.md`.

</details>

<details>
<summary><strong>Chisel snippet</strong> (same as backend — click to expand)</summary>

Same snippet as above. See `chisel/skills/chisel-awareness/assets/CLAUDE-snippet.md`.

</details>

<details>
<summary><strong>UISpec Frontend snippet</strong> (click to expand)</summary>

```markdown
## UISpec — Frontend (MANDATORY)

UI specs live in `uispec/`. **ALWAYS read specs before building API-connected UI.**

Before creating or modifying any page/component that calls an API endpoint:
1. Read `uispec/endpoints/<endpoint>.md` for the API contract and UI guidelines
2. Read `uispec/components.md` for reusable patterns
3. Read `uispec/design-system.md` for tokens and styling rules

Do not modify `openapi.yaml` or API sections — those are backend-owned.

Command: /uispec-validate
```

See `uispec-frontend/skills/uispec-frontend/assets/CLAUDE-snippet.md` for the full version with ownership rules and post-implementation guidance.

</details>

---

## All Commands Reference

| Plugin | Command | Description |
|--------|---------|-------------|
| specsmith | `/specsmith-new <name>` | Discovery interview + research + full spec |
| specsmith | `/specsmith-implement` | Start active spec implementation |
| specsmith | `/specsmith-resume` | Re-read everything, continue active spec |
| specsmith | `/specsmith-switch <name>` | Save progress, switch to different spec |
| specsmith | `/specsmith-edit <name>` | Revise spec mid-flight |
| specsmith | `/specsmith-status` | Dashboard of all specs with phase progress |
| specsmith | `/specsmith-list` | Interactive spec list with picker |
| specsmith | `/specsmith-show [name]` | Display spec without activating |
| specsmith | `/specsmith-archive <name>` | Shelve completed/paused spec |
| specsmith | `/specsmith-unarchive <name>` | Restore archived spec |
| specsmith | `/specsmith-drop <name>` | Delete spec + research + questions |
| grimoire | `/grimoire` | Show available research streams |
| grimoire | `/grimoire docs <lib> <query>` | Library documentation lookup |
| grimoire | `/grimoire practices <topic>` | Best practices and pitfalls |
| grimoire | `/grimoire codebase <area>` | Codebase architecture analysis |
| grimoire | `/grimoire history <area>` | Git history and prior work |
| grimoire | `/grimoire schema <area>` | Database schema analysis |
| grimoire | `/grimoire contracts <service>` | External API contract review |
| grimoire | `/grimoire research <context>` | Full parallel research |
| runebook | `/runebook-init` | Scan codebase, create .runebook/ |
| runebook | `/runebook-update [name]` | Update entry or auto-detect from changes |
| runebook | `/runebook-validate` | Check staleness, coverage, broken refs |
| runebook | `/runebook-show [name]` | Display entry or master index |
| runebook | `/runebook-status` | Dashboard with coverage and staleness |
| chisel | `/chisel` | Show available operations |
| chisel | `/chisel rename <old> to <new>` | Rename + update all references |
| chisel | `/chisel move <src> to <dest>` | Move + update all import paths |
| chisel | `/chisel replace <old> with <new>` | Find-and-replace with scope analysis |
| chisel | `/chisel edit <description>` | Small targeted code change |
| uispec | `/uispec-init` | Initialize uispec/ from backend endpoints |
| uispec | `/uispec-sync` | Sync after backend API changes |
| uispec | `/uispec-validate` | Check spec consistency and coverage |

---

## Architecture

### Thin-Command-Over-Thick-Skill Pattern

Each plugin follows a consistent architecture:

- **Commands** (`commands/<name>.md`) — thin wrappers (~15 lines) that declare allowed tools and invoke a skill action
- **Skills** (`skills/<name>/SKILL.md`) — thick implementation containing all logic, state management, and workflows
- **Agents** (`agents/<name>.md`) — specialized subagents spawned via the Task tool for focused work
- **Awareness skills** (`skills/<name>-awareness/SKILL.md`) — proactive triggers that detect when a plugin should activate

### State Management

All state is plain markdown files designed to be committed to git:

- `.specsmiths/<name>.md` — spec with YAML frontmatter (status, phase, steps)
- `.specsmiths/<name>.research.md` — research findings from subagents
- `.specsmiths/<name>.questions.md` — discovery Q&A log
- `.runebook/index.md` — master index with links, tags, dates
- `.runebook/<type>/<name>.md` — entry with YAML frontmatter
- `uispec/` — UI specifications (created in frontend projects)
