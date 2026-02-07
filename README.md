# ngvoicu-forge-claude-code

Claude Code plugin marketplace with four plugins for spec-driven development, deep research, living documentation, and surgical code editing. UI specification workflows are integrated into specsmith. All plugins are markdown-based — no build system, no dependencies.

## Quick Start

```bash
# Add the marketplace
/plugin marketplace add https://github.com/ngvoicu/ngvoicu-forge-claude-code.git

# Install what you need
/plugin install specsmith@ngvoicu-forge-claude-code    # specs + UISpec integration
/plugin install grimoire@ngvoicu-forge-claude-code     # research suite
/plugin install runebook@ngvoicu-forge-claude-code     # living documentation
/plugin install chisel@ngvoicu-forge-claude-code       # surgical code editor
```

---

## Plugins

### specsmith

Research-driven implementation specs. You describe what you want to build, specsmith researches your codebase and the ecosystem, then produces a full spec with phased implementation steps.

#### How it works

Every `/specsmith-new` triggers a three-phase process:

1. **Discovery** — Provide a name and brief in one shot. Specsmith checks if your brief has enough substance (specific behavior, a trigger or problem, not just a vague phrase). If the brief is comprehensive, it moves straight to research. If not, it asks focused follow-up questions — edge cases, constraints, flows. All Q&A is logged to `.specsmiths/<n>.questions.md`.
2. **Deep research** — Adaptive subagents (1-6) investigate based on what discovery revealed: codebase analysis, library docs, best practices, git history, DB schema, external APIs. If grimoire is installed, specsmith uses its specialized agents for richer research. Findings go to `.specsmiths/<n>.research.md`.
3. **Spec creation** — A full specification with requirements, architecture, data model, API design, edge case table, implementation steps (each with What/Where/How/Verify), risks, testing strategy, and rollback plan.

The result is a spec you could hand to any developer and they'd know exactly what to build. All state lives in `.specsmiths/` as plain markdown — commit it to git.

For simple changes touching 1-2 files with existing dependencies, you can request lightweight research: just tell specsmith "keep it light" or "skip the deep research" and it will limit to codebase analysis only.

#### Commands

```
/specsmith-new <name>: <brief>                  [BE+FE]  one-shot: discovery -> research -> full spec
/specsmith-new <name>                           [BE+FE]  name only: asks for brief next
/specsmith-implement                            [BE+FE]  start active spec implementation
/specsmith-resume                               [BE+FE]  re-read everything, continue active spec
/specsmith-switch <name>                        [BE+FE]  save progress, switch active spec
/specsmith-edit <name>                          [BE+FE]  revise spec mid-flight
/specsmith-status                               [BE+FE]  dashboard of all specs with phase progress
/specsmith-list                                 [BE+FE]  interactive list with picker
/specsmith-show [name]                          [BE+FE]  display spec without activating
/specsmith-archive <name>                       [BE+FE]  shelve completed/paused spec
/specsmith-unarchive <name>                     [BE+FE]  restore archived spec
/specsmith-drop <name>                          [BE+FE]  delete spec + research + questions
/specsmith-uispec-init                          [BE]     initialize .uispec/ in frontend project
/specsmith-uispec-sync                          [BE]     sync API changes to .uispec/
/specsmith-uispec-new [endpoint]                [FE]     create UI spec from .uispec/ endpoint
/specsmith-uispec-new <endpoint>: <context>     [FE]     create UI spec with implementation guidance
/specsmith-uispec-detect                        [FE]     find gaps between .uispec/ and UI
/specsmith-uispec-validate                      [FE]     validate UI against .uispec/ (PASS/FAIL)
```

### grimoire

Research suite with 6 specialized agents. Just describe what you need — grimoire analyzes your query and picks the right agents automatically. No need to specify which stream to use.

#### How it works

Grimoire has 6 agents: library docs (Context7 + web fallback), best practices, codebase analysis, git history, DB schema, and external API contracts. When you run `/grimoire <query>`, it reads your query, picks 1-6 relevant agents based on the context, runs them all in parallel, and returns a synthesized report.

For example, `/grimoire add user authentication` might spawn codebase analysis + best practices + library docs + git history — all at once.

#### Typical workflow

```bash
# Just describe what you need — grimoire auto-dispatches:
/grimoire add JWT refresh token rotation to the auth middleware
/grimoire how does Stripe Connect handle marketplace payouts
/grimoire why is the payment service timing out

# For a quick single-stream lookup, prefix with the stream name:
/grimoire docs zod how to validate nested objects
/grimoire practices CSRF protection for REST APIs
/grimoire codebase payment processing
/grimoire history websocket migration
/grimoire schema users table indexes
/grimoire contracts stripe webhooks
```

Grimoire is what specsmith Phase 2 uses automatically for research when installed.

#### Commands

```
/grimoire <query>                  # auto-dispatch to relevant agents (default)
/grimoire docs <lib> <query>       # library docs only
/grimoire practices <topic>        # best practices only
/grimoire codebase <area>          # codebase analysis only
/grimoire history <area>           # git history only
/grimoire schema <area>            # DB schema only
/grimoire contracts <service>      # external API docs only
/grimoire                          # show available streams
```

### runebook

Living documentation that tracks how your application works. Runebook maintains two kinds of documentation:

**Entries** — structured docs for individual components: endpoints, jobs, flows, services, integrations, pages, components, and hooks. Each entry covers what the component does, how it works, its dependencies, edge cases, and recent changes.

**System guides** — narrative markdown documents organized by feature domain (authentication, payments, onboarding) that explain how things work end-to-end. Unlike entries, guides read like a walkthrough written by a senior engineer onboarding a new teammate. They're auto-generated from component relationships and kept in sync as code changes, while preserving any human-written sections (tribal knowledge, gotchas, common tasks).

Runebook is **assertive**: once installed, it reads entries and guides before code work and updates them after changes — automatically, without asking.

#### Typical workflow

```bash
# First time: scan the entire codebase and generate everything
/runebook-init

# Runebook creates:
#   .runebook/index.md              — master index
#   .runebook/endpoints/login.md    — entry for each endpoint
#   .runebook/services/auth.md      — entry for each service
#   .runebook/guides/authentication.md — guide for the auth domain
#   ... etc.

# After making changes, update a specific entry:
/runebook-update login

# Or let runebook auto-detect what changed:
/runebook-update

# Check if anything is stale, missing, or broken:
/runebook-validate

# Quick look at an entry or the full index:
/runebook-show login
/runebook-show

# Dashboard with coverage stats, staleness, and guide health:
/runebook-status
```

#### How guides work

Guides are generated from component clusters — when 3+ entries across 2+ types share tags, imports, or serve the same user flow, runebook identifies that as a feature domain and writes a narrative guide for it.

Guide sections are split into two categories. **Auto-derivable sections** (How It Works, Architecture at a Glance, Where Things Live) are updated from code whenever entries change. **Human sections** (Key Concepts, Common Tasks, Gotchas & Tribal Knowledge) are preserved always — runebook never overwrites what you write there.

#### Commands

```
/runebook-init              # scan codebase, create .runebook/ with entries + guides
/runebook-update [name]     # update specific entry or auto-detect from changes
/runebook-validate          # check staleness, coverage, broken refs, guide freshness
/runebook-show [name]       # display entry or master index
/runebook-status            # dashboard with coverage, staleness, and guide health
```

### chisel

Fast surgical code editor for renaming, moving, replacing, and making small targeted edits. Uses the haiku model for speed. Never manually rename or move files — chisel handles all reference updates across your entire codebase automatically.

#### Why use chisel instead of find-and-replace

Chisel is **git-aware** (uses `git mv` to preserve file history), **framework-aware** (finds co-located tests, CSS modules, stories, barrel exports), **scope-aware** (respects word boundaries, warns on large scope, previews changes), and **language-agnostic** (any language, framework, or file type).

A manual rename misses test files, barrel exports, and dynamic imports. Chisel catches them all.

#### Typical workflow

```bash
# Rename a component — updates all imports, tests, stories, CSS modules:
/chisel rename UserCard to ProfileCard

# Move a file to a new directory — updates all import paths:
/chisel move src/utils/auth.ts to src/services/auth.ts

# Replace a string across the codebase with scope analysis:
/chisel replace getUser with fetchUser

# Small targeted code change described in natural language:
/chisel edit add a loading prop to the Button component

# Or just describe what you want — chisel auto-detects the operation:
/chisel move the auth helpers into the services folder
```

After chisel operations, runebook entries referencing affected files are updated automatically.

#### Commands

```
/chisel                                # show available operations
/chisel rename <old> to <new>          # rename + update all references
/chisel move <source> to <dest>        # move + update all import paths
/chisel replace <old> with <new>       # find-and-replace with scope analysis
/chisel edit <description>             # small targeted code change
/chisel <natural language>             # auto-detect operation
```

---

## Specsmith Workflows

Specsmith adapts to your project type. Here are the three main flows — each is self-contained with the full lifecycle from creating a spec to archiving it.

### Backend-Only Flow

For API projects, microservices, CLIs — no frontend, no UISpec commands needed.

```bash
# 1. Create a spec
/specsmith-new auth-refactor: Refactor auth middleware to support JWT and session tokens

# 2. Review the spec
/specsmith-show auth-refactor

# 3. Start implementing
/specsmith-implement

# 4. Get interrupted? Resume later
/specsmith-resume

# 5. Requirements changed? Edit the spec
/specsmith-edit auth-refactor

# 6. Continue implementing
/specsmith-implement

# 7. Working on something else? Switch specs
/specsmith-switch payments-v2

# 8. Done? Archive
/specsmith-archive auth-refactor

# At any time: check status of all specs
/specsmith-status
```

Grimoire auto-runs during the research phase. Runebook auto-updates after code changes. No UISpec commands needed.

### Full-Stack Flow (BE + FE)

Backend creates the spec and implements it. API changes auto-sync to the frontend's `.uispec/` directory. Frontend picks up the changes and builds UI from the spec.

```bash
# --- Backend project ---

# 1. One-time setup: create .uispec/ in the frontend project
/specsmith-uispec-init                                    # [BE]

# 2. Create a spec for the backend feature
/specsmith-new create-user: REST endpoint for user registration with email verification

# 3. Implement — auto-syncs .uispec/ when API endpoints change
/specsmith-implement

# 4. If you made API changes outside a spec, sync manually
/specsmith-uispec-sync                                    # [BE]

# --- Switch to frontend project ---

# 5. See what's new/changed in .uispec/
/specsmith-uispec-detect                                  # [FE]

# 6. Build UI for an endpoint (full specsmith workflow)
/specsmith-uispec-new create-user                         # [FE]
# or with context about what changed:
/specsmith-uispec-new create-user: added email verification — show pending state

# 7. Resume if interrupted
/specsmith-resume

# 8. Validate UI matches the spec
/specsmith-uispec-validate                                # [FE]

# 9. Edit the spec if needed
/specsmith-edit create-user

# 10. Done? Archive
/specsmith-archive create-user
```

### Frontend-Only Flow

For frontend projects consuming an external API. The backend team (or you) provides `.uispec/` — you build UI from it.

```bash
# 1. Get .uispec/ from the backend team (or create manually)
# The backend team runs /specsmith-uispec-init from their project

# 2. See what endpoints are available
/specsmith-uispec-detect                                  # [FE]

# 3. Build UI for a specific endpoint
/specsmith-uispec-new login                               # [FE]

# 4. Resume if interrupted
/specsmith-resume

# 5. Validate
/specsmith-uispec-validate                                # [FE]
```

### Updating Existing Endpoints

Most of the time you're changing something that already exists — adding a field, tweaking validation, changing a response shape.

```bash
# 1. Make the backend change, then sync the updated contract
/specsmith-uispec-sync

# 2. In the frontend, see what's out of sync
/specsmith-uispec-detect

# 3. Implement with context about what changed
/specsmith-uispec-new create-user: Added email verification — show inline validation and pending state
# or without context if changes are self-explanatory:
/specsmith-uispec-new create-user

# 4. Validate
/specsmith-uispec-validate
```

The colon syntax provides context about **why** things changed and **how** you want the UI to handle it:

```
/specsmith-uispec-new login: Session tokens replaced JWT — remove token refresh logic, add cookie-based auth
/specsmith-uispec-new get-products: Added pagination — use infinite scroll, not page numbers
/specsmith-uispec-new create-order: New discount_code field — show collapsible promo section
```

---

## UISpec Deep Dive

UISpec is built into specsmith. Backend projects push API contracts to `.uispec/` in the frontend project. Frontend projects read `.uispec/` and create full specsmith specs for UI implementation.

### What Each Side Owns

| | Backend-owned | Frontend-owned |
|---|---|---|
| **API contract** | `openapi.yaml`, Method/Path/Auth/Request/Response sections | Read-only |
| **Suggested implementation** | Backend generates (component suggestions, interactions, watchouts) | Can override in UI Guidelines |
| **UI guidelines** | Read-only | `design-system.md`, `components.md`, UI Guidelines sections |
| **Implementation spec** | `.specsmiths/` — full spec with phased steps | Can read, doesn't modify |
| **Documentation** | `.runebook/` — entries + guides (shared, both sides update) | `.runebook/` — entries + guides (shared, both sides update) |
| **Changelog** | Both append (append-only, never modify existing entries) | Both append |

### Cross-Endpoint Component Analysis

During init and sync, specsmith analyzes all endpoints together:
- Groups endpoints by type (list-view, detail-view, create-form, etc.)
- Identifies shared response shapes (e.g., `user` object in multiple endpoints)
- Detects common patterns (pagination, filters, error formats)
- Updates `components.md` with a Shared Patterns section

### Suggested Implementation

Each endpoint spec includes a backend-generated Suggested Implementation section:
- Component hierarchy (parent → children tree)
- State flow (idle → loading → success/error)
- Cross-endpoint reuse ("shares User pattern with list-users")
- Interaction sequences (click → modal → fill → submit → feedback)
- Watch-outs (rate limits, file uploads, date formatting, case transforms)

Frontend can override any of it in UI Guidelines.

### Auto-Sync During Implementation

When `/specsmith-implement` completes a step that creates or modifies an API endpoint and a frontend project is configured (`.specsmiths/uispec.json` exists), specsmith automatically syncs the changes to `.uispec/`:

- `openapi.yaml` gets the new/modified endpoints
- `endpoints/<name>.md` files are created or updated with request/response shapes, auth rules, error codes, Suggested Implementation, and Related Endpoints
- Cross-endpoint component analysis updates `components.md` with shared patterns
- Frontend-owned sections (UI guidelines, design system) are preserved — never overwritten

No manual sync needed during spec implementation.

---

## How Plugins Work Together

```
specsmith ──→ grimoire     Phase 2 uses grimoire agents for research
specsmith ──→ runebook     Read entries during implementation, update after
specsmith ──→ .uispec/     Auto-syncs API contracts when implementing endpoint steps
specsmith-uispec-new ──→ grimoire   Research phase for component libs, a11y, codebase patterns
runebook  ──→ chisel       After rename/move, update runebook source_files
grimoire  ──→ chisel       Research scope before large renames
chisel    ──→ .uispec/     Update spec files when renaming endpoints
```

### When to Use What

| You're about to... | Use this |
|---|---|
| Build a new feature | `/specsmith-new <name>: <brief>` |
| Research before coding | `/grimoire <query>` (auto-dispatches to relevant agents) |
| Understand existing code before changing it | `/grimoire <query>` and `/runebook-show <name>` |
| Rename or move files | `/chisel rename` or `/chisel move` |
| Set up UISpec for a frontend project | `/specsmith-uispec-init` (run from backend) |
| Sync API changes to frontend | `/specsmith-uispec-sync` (auto during spec implementation) |
| See what UI needs implementing | `/specsmith-uispec-detect` |
| Build UI for an endpoint | `/specsmith-uispec-new <endpoint>` |
| Update UI after a backend change | `/specsmith-uispec-new <endpoint>: <what changed>` |
| Check if UI matches specs | `/specsmith-uispec-validate`, `/runebook-validate` |
| Onboard to unfamiliar code | `/runebook-show` for entries, `/runebook-show <guide>` for domain guides |

---

## All Commands Reference

| Plugin | Command | Scope | Description |
|--------|---------|-------|-------------|
| specsmith | `/specsmith-new <name>: <brief>` | BE+FE | One-shot: discovery + research + full spec |
| specsmith | `/specsmith-new <name>` | BE+FE | Name only: asks for brief, then discovery + research + spec |
| specsmith | `/specsmith-implement` | BE+FE | Start active spec implementation |
| specsmith | `/specsmith-resume` | BE+FE | Re-read everything, continue active spec |
| specsmith | `/specsmith-switch <name>` | BE+FE | Save progress, switch to different spec |
| specsmith | `/specsmith-edit <name>` | BE+FE | Revise spec mid-flight |
| specsmith | `/specsmith-status` | BE+FE | Dashboard of all specs with phase progress |
| specsmith | `/specsmith-list` | BE+FE | Interactive spec list with picker |
| specsmith | `/specsmith-show [name]` | BE+FE | Display spec without activating |
| specsmith | `/specsmith-archive <name>` | BE+FE | Shelve completed/paused spec |
| specsmith | `/specsmith-unarchive <name>` | BE+FE | Restore archived spec |
| specsmith | `/specsmith-drop <name>` | BE+FE | Delete spec + research + questions |
| specsmith | `/specsmith-uispec-init` | BE | Initialize .uispec/ in frontend project |
| specsmith | `/specsmith-uispec-sync` | BE | Sync API changes to .uispec/ |
| specsmith | `/specsmith-uispec-new [endpoint]` | FE | Create UI spec from .uispec/ endpoint |
| specsmith | `/specsmith-uispec-new <ep>: <ctx>` | FE | Create UI spec with implementation guidance |
| specsmith | `/specsmith-uispec-detect` | FE | Find gaps between .uispec/ and UI |
| specsmith | `/specsmith-uispec-validate` | FE | Validate UI against .uispec/ (PASS/FAIL) |
| grimoire | `/grimoire <query>` | BE+FE | Auto-dispatch research (picks relevant agents) |
| grimoire | `/grimoire docs <lib> <query>` | BE+FE | Library docs only |
| grimoire | `/grimoire practices <topic>` | BE+FE | Best practices only |
| grimoire | `/grimoire codebase <area>` | BE+FE | Codebase analysis only |
| grimoire | `/grimoire history <area>` | BE+FE | Git history only |
| grimoire | `/grimoire schema <area>` | BE+FE | DB schema only |
| grimoire | `/grimoire contracts <service>` | BE+FE | External API docs only |
| runebook | `/runebook-init` | BE+FE | Scan codebase, create .runebook/ with entries + guides |
| runebook | `/runebook-update [name]` | BE+FE | Update entry or auto-detect from changes |
| runebook | `/runebook-validate` | BE+FE | Check staleness, coverage, broken refs, guide freshness |
| runebook | `/runebook-show [name]` | BE+FE | Display entry or master index |
| runebook | `/runebook-status` | BE+FE | Dashboard with coverage, staleness, and guide health |
| chisel | `/chisel` | BE+FE | Show available operations |
| chisel | `/chisel rename <old> to <new>` | BE+FE | Rename + update all references |
| chisel | `/chisel move <src> to <dest>` | BE+FE | Move + update all import paths |
| chisel | `/chisel replace <old> with <new>` | BE+FE | Find-and-replace with scope analysis |
| chisel | `/chisel edit <description>` | BE+FE | Small targeted code change |
| chisel | `/chisel <natural language>` | BE+FE | Auto-detect operation |

---

## CLAUDE.md Guidelines and Snippets

### General Guidelines

Paste these into any project's `CLAUDE.md` as a baseline. They work standalone or alongside the plugin snippets below.

```markdown
## Agent Guidelines

### Read Before Acting
- NEVER speculate about code you haven't read — always open and read files before answering questions or making claims
- When the user references a file, read it first — give grounded, hallucination-free answers
- Before modifying code, read the surrounding context to understand patterns and conventions

### Plan Before Building
- Think through the problem before writing code
- For non-trivial features or changes touching more than 2-3 files, use `/specsmith-new` to create a proper spec before implementing
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

### Plugin Snippets

Copy the block matching your project type into your project's `CLAUDE.md` (after the general guidelines above).

### For Standalone Projects

For backend-only APIs, microservices, CLIs, or frontend-only SPAs that don't use UISpec.

````markdown
## Grimoire (MANDATORY)

Research suite with 6 specialized agents. **ALWAYS research before guessing — never hallucinate library APIs or best practices.**

### When to Use Grimoire

| Situation | Command |
|-----------|---------|
| Using unfamiliar library API | `/grimoire docs <library> <question>` |
| Security-sensitive implementation | `/grimoire practices <topic>` |
| Before major changes or refactors | `/grimoire codebase <area>` |
| Understanding why code exists | `/grimoire history <area>` |
| Database/schema changes | `/grimoire schema <area>` |
| External API integration | `/grimoire contracts <service>` |
| Complex feature (full research) | `/grimoire <context>` (auto-dispatches all relevant agents) |

### Rules

- **NEVER guess library APIs** — always use `/grimoire docs` to look up exact signatures, parameters, and return types
- **ALWAYS research best practices** before implementing security-sensitive features (auth, crypto, payments, PII)
- **ALWAYS analyze the codebase** before major refactors to understand existing patterns and conventions
- **ALWAYS check API contracts** before integrating with external services to understand rate limits, auth, and error handling
- **Prefer `/grimoire <query>`** for complex features — it auto-dispatches all relevant agents in parallel

### Getting Started

Run `/grimoire` with no arguments to see all available research streams and examples.

### Commands

```
/grimoire                              # Show available streams
/grimoire docs <library> <query>       # Library documentation lookup
/grimoire practices <topic>            # Best practices & pitfalls
/grimoire codebase <area>              # Codebase analysis
/grimoire history <area>               # Git history & prior work
/grimoire schema <area>                # Database schema analysis
/grimoire contracts <service>          # External API contracts
/grimoire <context>                     # Auto-dispatch to relevant agents (full parallel research)
```

---

## Runebook (MANDATORY)

Living documentation lives in `.runebook/`. **ALWAYS consult before changes, ALWAYS update after changes — automatically, without asking.**

### When to Use Runebook

| Situation | Action |
|-----------|--------|
| Before modifying any code | Read relevant `.runebook/<type>/<name>.md` entries |
| After creating a new endpoint | Create `.runebook/endpoints/<name>.md` |
| After creating a new scheduled job | Create `.runebook/jobs/<name>.md` |
| After creating a new service/module | Create `.runebook/services/<name>.md` |
| After creating a new integration | Create `.runebook/integrations/<name>.md` |
| After creating a new page/route | Create `.runebook/pages/<name>.md` |
| After creating a new reusable component | Create `.runebook/components/<name>.md` |
| After creating a new custom hook | Create `.runebook/hooks/<name>.md` |
| After modifying any existing component | Update the existing entry |
| After removing a component | Flag entry as removed (add `status: removed` to frontmatter) |
| New business flow identified | Create `.runebook/flows/<name>.md` (ask about steps) |

### Before ANY Code Changes

You MUST read relevant runebook entries before modifying code:
1. Identify which components will be affected by your changes
2. Read their entries from `.runebook/<type>/<name>.md`
3. Understand current behavior, dependencies, edge cases, and recent changes
4. Use this context to inform your implementation

**How to find the right entries:** Match changed files to runebook entries via `source_files` in frontmatter, or search by tag/name in `.runebook/index.md`.

### After ANY Code Changes

You MUST update the runebook after changes — do it automatically, do not ask:

For every update:
- Update the entry's behavior, dependencies, and request/response sections as needed
- Preserve human-written notes and context — **never overwrite them**
- Update frontmatter: `updated` date, `source_files`, `tags`
- Append to the changelog with today's date and what changed — **changelog is append-only**
- Update cross-references bidirectionally (if A depends on B, both entries reference each other)
- Update `.runebook/index.md` if metadata changed (new entries, changed methods/paths/schedules)

### Rules

- **ALWAYS read entries before code work** — never modify code without checking runebook first
- **ALWAYS update entries after changes** — automatically, without asking permission
- **NEVER overwrite human context** — notes, explanations, and annotations are sacred
- **NEVER modify existing changelog entries** — changelog is append-only
- **ALWAYS update cross-references bidirectionally** — if entry A references B, entry B must reference A

### Naming Conventions

| Type | File Name Pattern | Example |
|------|-------------------|---------|
| Endpoints | `<method>-<path-slug>.md` | `post-users.md`, `get-orders-by-id.md` |
| Jobs | `<job-name>.md` | `cleanup-expired-sessions.md` |
| Services | `<service-name>.md` | `auth-service.md`, `payment-processor.md` |
| Integrations | `<provider-name>.md` | `stripe.md`, `sendgrid.md` |
| Flows | `<flow-name>.md` | `user-onboarding.md`, `order-checkout.md` |
| Pages | `<route-path-slug>.md` | `users-by-id.md`, `dashboard.md` |
| Components | `<component-name>.md` | `button.md`, `data-table.md` |
| Hooks | `<hook-name>.md` | `use-auth.md`, `use-pagination.md` |

### First-Time Setup

If `.runebook/` doesn't exist yet, run `/runebook-init` to scan the codebase and create the runebook.

### Commands

```
/runebook-init              # Scan codebase, create .runebook/
/runebook-update [name]     # Update specific entry or auto-detect from changes
/runebook-validate          # Check staleness, coverage, broken refs
/runebook-show [name]       # Display entry or master index
/runebook-status            # Dashboard with coverage and staleness
```

---

## Chisel (MANDATORY)

Fast surgical code editor. **NEVER manually rename or move files — always use chisel to ensure all references are updated.**

### When to Use Chisel

| Situation | Command |
|-----------|---------|
| Rename a file, variable, function, or class | `/chisel rename <old> to <new>` |
| Move a file or directory | `/chisel move <source> to <destination>` |
| Find-and-replace text across files | `/chisel replace <old> with <new>` |
| Small CSS/HTML/config tweaks | `/chisel edit <description>` |
| Any rename, move, or targeted edit | `/chisel <natural language description>` |

### Rules

- **NEVER rename or move files manually** (with `mv`, manual delete+create, or IDE refactoring) — always use `/chisel` to ensure all imports, references, and config files are updated
- **NEVER do find-and-replace with sed/awk** — use `/chisel replace` which handles word boundaries, scope analysis, and verification
- **ALWAYS verify** that chisel found and updated all references — check the summary it produces
- **Use `/chisel rename`** for any entity rename: files, directories, variables, functions, classes, components, modules

### Commands

```
/chisel                                # Show available operations
/chisel rename <old> to <new>          # Rename + update all references
/chisel move <source> to <dest>        # Move + update all import paths
/chisel replace <old> with <new>       # Find-and-replace with scope analysis
/chisel edit <description>             # Small targeted code change
/chisel <natural language>             # Auto-detect operation
```

---

## Cross-Plugin Workflows

- **`/specsmith-new`** auto-uses grimoire agents for research when grimoire is installed — no manual invocation needed
- **`/specsmith-implement`** reads runebook entries before each step and updates them after each step
- **`/chisel` operations** (rename, move, replace) update runebook `source_files` references automatically
- **`/grimoire` research** reads `.runebook/` entries as input context for richer findings
- After any rename or move, always use `/chisel` to keep runebook references in sync
````

### For Full-Stack Backend Projects

````markdown
## Grimoire (MANDATORY)

Research suite with 6 specialized agents. **ALWAYS research before guessing — never hallucinate library APIs or best practices.**

### When to Use Grimoire

| Situation | Command |
|-----------|---------|
| Using unfamiliar library API | `/grimoire docs <library> <question>` |
| Security-sensitive implementation | `/grimoire practices <topic>` |
| Before major changes or refactors | `/grimoire codebase <area>` |
| Understanding why code exists | `/grimoire history <area>` |
| Database/schema changes | `/grimoire schema <area>` |
| External API integration | `/grimoire contracts <service>` |
| Complex feature (full research) | `/grimoire <context>` (auto-dispatches all relevant agents) |

### Rules

- **NEVER guess library APIs** — always use `/grimoire docs` to look up exact signatures, parameters, and return types
- **ALWAYS research best practices** before implementing security-sensitive features (auth, crypto, payments, PII)
- **ALWAYS analyze the codebase** before major refactors to understand existing patterns and conventions
- **ALWAYS check API contracts** before integrating with external services to understand rate limits, auth, and error handling
- **Prefer `/grimoire <query>`** for complex features — it auto-dispatches all relevant agents in parallel

### Getting Started

Run `/grimoire` with no arguments to see all available research streams and examples.

### Commands

```
/grimoire                              # Show available streams
/grimoire docs <library> <query>       # Library documentation lookup
/grimoire practices <topic>            # Best practices & pitfalls
/grimoire codebase <area>              # Codebase analysis
/grimoire history <area>               # Git history & prior work
/grimoire schema <area>                # Database schema analysis
/grimoire contracts <service>          # External API contracts
/grimoire <context>                     # Auto-dispatch to relevant agents (full parallel research)
```

---

## Runebook (MANDATORY)

Living documentation lives in `.runebook/`. **ALWAYS consult before changes, ALWAYS update after changes — automatically, without asking.**

### When to Use Runebook

| Situation | Action |
|-----------|--------|
| Before modifying any code | Read relevant `.runebook/<type>/<name>.md` entries |
| After creating a new endpoint | Create `.runebook/endpoints/<name>.md` |
| After creating a new scheduled job | Create `.runebook/jobs/<name>.md` |
| After creating a new service/module | Create `.runebook/services/<name>.md` |
| After creating a new integration | Create `.runebook/integrations/<name>.md` |
| After creating a new page/route | Create `.runebook/pages/<name>.md` |
| After creating a new reusable component | Create `.runebook/components/<name>.md` |
| After creating a new custom hook | Create `.runebook/hooks/<name>.md` |
| After modifying any existing component | Update the existing entry |
| After removing a component | Flag entry as removed (add `status: removed` to frontmatter) |
| New business flow identified | Create `.runebook/flows/<name>.md` (ask about steps) |

### Before ANY Code Changes

You MUST read relevant runebook entries before modifying code:
1. Identify which components will be affected by your changes
2. Read their entries from `.runebook/<type>/<name>.md`
3. Understand current behavior, dependencies, edge cases, and recent changes
4. Use this context to inform your implementation

**How to find the right entries:** Match changed files to runebook entries via `source_files` in frontmatter, or search by tag/name in `.runebook/index.md`.

### After ANY Code Changes

You MUST update the runebook after changes — do it automatically, do not ask:

For every update:
- Update the entry's behavior, dependencies, and request/response sections as needed
- Preserve human-written notes and context — **never overwrite them**
- Update frontmatter: `updated` date, `source_files`, `tags`
- Append to the changelog with today's date and what changed — **changelog is append-only**
- Update cross-references bidirectionally (if A depends on B, both entries reference each other)
- Update `.runebook/index.md` if metadata changed (new entries, changed methods/paths/schedules)

### Rules

- **ALWAYS read entries before code work** — never modify code without checking runebook first
- **ALWAYS update entries after changes** — automatically, without asking permission
- **NEVER overwrite human context** — notes, explanations, and annotations are sacred
- **NEVER modify existing changelog entries** — changelog is append-only
- **ALWAYS update cross-references bidirectionally** — if entry A references B, entry B must reference A

### Naming Conventions

| Type | File Name Pattern | Example |
|------|-------------------|---------|
| Endpoints | `<method>-<path-slug>.md` | `post-users.md`, `get-orders-by-id.md` |
| Jobs | `<job-name>.md` | `cleanup-expired-sessions.md` |
| Services | `<service-name>.md` | `auth-service.md`, `payment-processor.md` |
| Integrations | `<provider-name>.md` | `stripe.md`, `sendgrid.md` |
| Flows | `<flow-name>.md` | `user-onboarding.md`, `order-checkout.md` |
| Pages | `<route-path-slug>.md` | `users-by-id.md`, `dashboard.md` |
| Components | `<component-name>.md` | `button.md`, `data-table.md` |
| Hooks | `<hook-name>.md` | `use-auth.md`, `use-pagination.md` |

### First-Time Setup

If `.runebook/` doesn't exist yet, run `/runebook-init` to scan the codebase and create the runebook.

### Commands

```
/runebook-init              # Scan codebase, create .runebook/
/runebook-update [name]     # Update specific entry or auto-detect from changes
/runebook-validate          # Check staleness, coverage, broken refs
/runebook-show [name]       # Display entry or master index
/runebook-status            # Dashboard with coverage and staleness
```

---

## Chisel (MANDATORY)

Fast surgical code editor. **NEVER manually rename or move files — always use chisel to ensure all references are updated.**

### When to Use Chisel

| Situation | Command |
|-----------|---------|
| Rename a file, variable, function, or class | `/chisel rename <old> to <new>` |
| Move a file or directory | `/chisel move <source> to <destination>` |
| Find-and-replace text across files | `/chisel replace <old> with <new>` |
| Small CSS/HTML/config tweaks | `/chisel edit <description>` |
| Any rename, move, or targeted edit | `/chisel <natural language description>` |

### Rules

- **NEVER rename or move files manually** (with `mv`, manual delete+create, or IDE refactoring) — always use `/chisel` to ensure all imports, references, and config files are updated
- **NEVER do find-and-replace with sed/awk** — use `/chisel replace` which handles word boundaries, scope analysis, and verification
- **ALWAYS verify** that chisel found and updated all references — check the summary it produces
- **Use `/chisel rename`** for any entity rename: files, directories, variables, functions, classes, components, modules

### Commands

```
/chisel                                # Show available operations
/chisel rename <old> to <new>          # Rename + update all references
/chisel move <source> to <dest>        # Move + update all import paths
/chisel replace <old> with <new>       # Find-and-replace with scope analysis
/chisel edit <description>             # Small targeted code change
/chisel <natural language>             # Auto-detect operation
```

---

## UISpec — Backend (MANDATORY)

UI specs live in `../<frontend-project>/.uispec/`. **ALWAYS sync after API changes.**

### When to Sync

| Change | Action |
|--------|--------|
| New endpoint created | Run `/specsmith-uispec-sync` |
| Request/response shape modified | Run `/specsmith-uispec-sync` |
| Auth/permissions changed | Run `/specsmith-uispec-sync` |
| Endpoint removed or deprecated | Run `/specsmith-uispec-sync` |
| New error response formats | Run `/specsmith-uispec-sync` |
| Pagination/filtering changed | Run `/specsmith-uispec-sync` |

### Auto-Sync

When implementing a specsmith spec (`/specsmith-implement`), API endpoint changes are **automatically synced** to `.uispec/`. No manual sync needed during spec implementation.

Manual `/specsmith-uispec-sync` is for changes made outside of specsmith specs.

### Ownership Rules

| Section | Owner | Backend Can |
|---------|-------|-------------|
| openapi.yaml | Backend | Read + Write |
| Method, Path, Auth, Request, Response, Errors | Backend | Read + Write |
| Suggested Implementation | Backend | Generate (frontend can override) |
| Related Endpoints | Backend | Read + Write |
| UI Guidelines, States, Accessibility | Frontend | Read only |
| design-system.md | Frontend | Read only |
| components.md (Shared Patterns) | Backend | Read + Write |
| components.md (everything else) | Frontend | Read only |
| Changelog | Shared | Append only |

### Rules

- **ALWAYS update openapi.yaml first** — it is the API source of truth
- **NEVER modify frontend-owned sections** — UI Guidelines, design-system.md
- **ALWAYS preserve manual additions** — read existing files before modifying
- **ALWAYS add changelog entries** — every sync appends date + what changed

### First-Time Setup

Run `/specsmith-uispec-init` to create `.uispec/` in the frontend project and generate specs from existing endpoints.

### Commands

```
/specsmith-uispec-init      [BE]  First-time setup — create .uispec/ in frontend
/specsmith-uispec-sync      [BE]  Sync after backend API changes
```

Frontend commands (run these in the frontend project, not here):
```
/specsmith-uispec-detect    [FE]  Find gaps between specs and UI
/specsmith-uispec-new       [FE]  Create UI spec from .uispec/ endpoint
/specsmith-uispec-validate  [FE]  Validate UI against .uispec/ (PASS/FAIL)
```

---

## Cross-Plugin Workflows

- **`/specsmith-new`** auto-uses grimoire agents for research when grimoire is installed — no manual invocation needed
- **`/specsmith-implement`** reads runebook entries before each step, updates them after each step, and auto-syncs `.uispec/` when API endpoints change
- **`/chisel` operations** (rename, move, replace) update runebook `source_files` references and corresponding `.uispec/` endpoint spec files automatically
- **`/grimoire` research** reads `.runebook/` entries as input context for richer findings
- After any rename or move, always use `/chisel` to keep runebook and UISpec references in sync
- Use `/grimoire contracts` before designing new `.uispec/` endpoint specs
````

### For Full-Stack Frontend Projects

````markdown
## Grimoire (MANDATORY)

Research suite with 6 specialized agents. **ALWAYS research before guessing — never hallucinate library APIs or best practices.**

### When to Use Grimoire

| Situation | Command |
|-----------|---------|
| Using unfamiliar library API | `/grimoire docs <library> <question>` |
| Security-sensitive implementation | `/grimoire practices <topic>` |
| Before major changes or refactors | `/grimoire codebase <area>` |
| Understanding why code exists | `/grimoire history <area>` |
| Database/schema changes | `/grimoire schema <area>` |
| External API integration | `/grimoire contracts <service>` |
| Complex feature (full research) | `/grimoire <context>` (auto-dispatches all relevant agents) |

### Rules

- **NEVER guess library APIs** — always use `/grimoire docs` to look up exact signatures, parameters, and return types
- **ALWAYS research best practices** before implementing security-sensitive features (auth, crypto, payments, PII)
- **ALWAYS analyze the codebase** before major refactors to understand existing patterns and conventions
- **ALWAYS check API contracts** before integrating with external services to understand rate limits, auth, and error handling
- **Prefer `/grimoire <query>`** for complex features — it auto-dispatches all relevant agents in parallel

### Getting Started

Run `/grimoire` with no arguments to see all available research streams and examples.

### Commands

```
/grimoire                              # Show available streams
/grimoire docs <library> <query>       # Library documentation lookup
/grimoire practices <topic>            # Best practices & pitfalls
/grimoire codebase <area>              # Codebase analysis
/grimoire history <area>               # Git history & prior work
/grimoire schema <area>                # Database schema analysis
/grimoire contracts <service>          # External API contracts
/grimoire <context>                     # Auto-dispatch to relevant agents (full parallel research)
```

---

## Runebook (MANDATORY)

Living documentation lives in `.runebook/`. **ALWAYS consult before changes, ALWAYS update after changes — automatically, without asking.**

### When to Use Runebook

| Situation | Action |
|-----------|--------|
| Before modifying any code | Read relevant `.runebook/<type>/<name>.md` entries |
| After creating a new endpoint | Create `.runebook/endpoints/<name>.md` |
| After creating a new scheduled job | Create `.runebook/jobs/<name>.md` |
| After creating a new service/module | Create `.runebook/services/<name>.md` |
| After creating a new integration | Create `.runebook/integrations/<name>.md` |
| After creating a new page/route | Create `.runebook/pages/<name>.md` |
| After creating a new reusable component | Create `.runebook/components/<name>.md` |
| After creating a new custom hook | Create `.runebook/hooks/<name>.md` |
| After modifying any existing component | Update the existing entry |
| After removing a component | Flag entry as removed (add `status: removed` to frontmatter) |
| New business flow identified | Create `.runebook/flows/<name>.md` (ask about steps) |

### Before ANY Code Changes

You MUST read relevant runebook entries before modifying code:
1. Identify which components will be affected by your changes
2. Read their entries from `.runebook/<type>/<name>.md`
3. Understand current behavior, dependencies, edge cases, and recent changes
4. Use this context to inform your implementation

**How to find the right entries:** Match changed files to runebook entries via `source_files` in frontmatter, or search by tag/name in `.runebook/index.md`.

### After ANY Code Changes

You MUST update the runebook after changes — do it automatically, do not ask:

For every update:
- Update the entry's behavior, dependencies, and request/response sections as needed
- Preserve human-written notes and context — **never overwrite them**
- Update frontmatter: `updated` date, `source_files`, `tags`
- Append to the changelog with today's date and what changed — **changelog is append-only**
- Update cross-references bidirectionally (if A depends on B, both entries reference each other)
- Update `.runebook/index.md` if metadata changed (new entries, changed methods/paths/schedules)

### Rules

- **ALWAYS read entries before code work** — never modify code without checking runebook first
- **ALWAYS update entries after changes** — automatically, without asking permission
- **NEVER overwrite human context** — notes, explanations, and annotations are sacred
- **NEVER modify existing changelog entries** — changelog is append-only
- **ALWAYS update cross-references bidirectionally** — if entry A references B, entry B must reference A

### Naming Conventions

| Type | File Name Pattern | Example |
|------|-------------------|---------|
| Endpoints | `<method>-<path-slug>.md` | `post-users.md`, `get-orders-by-id.md` |
| Jobs | `<job-name>.md` | `cleanup-expired-sessions.md` |
| Services | `<service-name>.md` | `auth-service.md`, `payment-processor.md` |
| Integrations | `<provider-name>.md` | `stripe.md`, `sendgrid.md` |
| Flows | `<flow-name>.md` | `user-onboarding.md`, `order-checkout.md` |
| Pages | `<route-path-slug>.md` | `users-by-id.md`, `dashboard.md` |
| Components | `<component-name>.md` | `button.md`, `data-table.md` |
| Hooks | `<hook-name>.md` | `use-auth.md`, `use-pagination.md` |

### First-Time Setup

If `.runebook/` doesn't exist yet, run `/runebook-init` to scan the codebase and create the runebook.

### Commands

```
/runebook-init              # Scan codebase, create .runebook/
/runebook-update [name]     # Update specific entry or auto-detect from changes
/runebook-validate          # Check staleness, coverage, broken refs
/runebook-show [name]       # Display entry or master index
/runebook-status            # Dashboard with coverage and staleness
```

---

## Chisel (MANDATORY)

Fast surgical code editor. **NEVER manually rename or move files — always use chisel to ensure all references are updated.**

### When to Use Chisel

| Situation | Command |
|-----------|---------|
| Rename a file, variable, function, or class | `/chisel rename <old> to <new>` |
| Move a file or directory | `/chisel move <source> to <destination>` |
| Find-and-replace text across files | `/chisel replace <old> with <new>` |
| Small CSS/HTML/config tweaks | `/chisel edit <description>` |
| Any rename, move, or targeted edit | `/chisel <natural language description>` |

### Rules

- **NEVER rename or move files manually** (with `mv`, manual delete+create, or IDE refactoring) — always use `/chisel` to ensure all imports, references, and config files are updated
- **NEVER do find-and-replace with sed/awk** — use `/chisel replace` which handles word boundaries, scope analysis, and verification
- **ALWAYS verify** that chisel found and updated all references — check the summary it produces
- **Use `/chisel rename`** for any entity rename: files, directories, variables, functions, classes, components, modules

### Commands

```
/chisel                                # Show available operations
/chisel rename <old> to <new>          # Rename + update all references
/chisel move <source> to <dest>        # Move + update all import paths
/chisel replace <old> with <new>       # Find-and-replace with scope analysis
/chisel edit <description>             # Small targeted code change
/chisel <natural language>             # Auto-detect operation
```

---

## UISpec — Frontend (MANDATORY)

UI specs live in `.uispec/`. **ALWAYS read specs before building API-connected UI.**

### When to Use

| Situation | Action |
|-----------|--------|
| Check what needs implementing | `/specsmith-uispec-detect` |
| Build UI for an endpoint | `/specsmith-uispec-new <endpoint>` |
| Build with context about changes | `/specsmith-uispec-new <endpoint>: <context>` |
| Validate UI matches specs | `/specsmith-uispec-validate` |
| Resume UI implementation | `/specsmith-resume` |

### Workflow

```
/specsmith-uispec-detect                    # See what's new/changed
/specsmith-uispec-new login                 # Full workflow: discover → research → plan → implement → validate
/specsmith-uispec-new login: added 2FA      # With context about what changed
/specsmith-resume                           # Continue from where you left off
```

### Ownership Rules

| Section | Owner | Frontend Can |
|---------|-------|--------------|
| openapi.yaml | Backend | Read only |
| Method, Path, Auth, Request, Response | Backend | Read only |
| UI Guidelines, States, Accessibility | Frontend | Read + Write |
| design-system.md | Frontend | Read + Write |
| components.md (except Shared Patterns) | Frontend | Read + Write |

### Rules

- **NEVER modify `openapi.yaml`** — backend-owned
- **NEVER modify API sections** in endpoint files — backend-owned
- **ALWAYS read the spec first** before building API-connected components
- **ALWAYS use design tokens** from `design-system.md` — never hardcode values
- **ALWAYS update UI Guidelines** after building — document components used, a11y, state management
- **ALWAYS flag API discrepancies** — if actual API doesn't match spec, flag for backend team

### Commands

```
/specsmith-uispec-detect              [FE]  Find gaps between specs and UI
/specsmith-uispec-new                 [FE]  Show unimplemented endpoints, pick one
/specsmith-uispec-new <ep>            [FE]  Full workflow for specific endpoint
/specsmith-uispec-new <ep>: <context> [FE]  Full workflow with implementation guidance
/specsmith-uispec-validate            [FE]  PASS/FAIL conformance checks
```

Backend commands (run these in the backend project, not here):
```
/specsmith-uispec-init                [BE]  First-time setup — create .uispec/ in frontend
/specsmith-uispec-sync                [BE]  Sync after backend API changes
```

### Notes

- If `.uispec/` doesn't exist, ask the backend team to run `/specsmith-uispec-init`
- If an endpoint spec doesn't exist, flag it for the backend team
- If API response doesn't match spec, flag the discrepancy immediately

---

## Cross-Plugin Workflows

- **`/specsmith-new`** auto-uses grimoire agents for research when grimoire is installed — no manual invocation needed
- **`/specsmith-implement`** reads runebook entries before each step, updates them after each step, and auto-syncs `.uispec/` when API endpoints change
- **`/chisel` operations** (rename, move, replace) update runebook `source_files` references and corresponding `.uispec/` endpoint spec files automatically
- **`/grimoire` research** reads `.runebook/` entries as input context for richer findings
- After any rename or move, always use `/chisel` to keep runebook and UISpec references in sync
- Use `/grimoire docs` for component library APIs and `/grimoire practices` for a11y best practices when building UI from specs
````

---

## Architecture

### Thin-Command-Over-Thick-Skill Pattern

Each plugin follows a consistent architecture:

- **Commands** (`commands/<name>.md`) — thin wrappers (~15 lines) that declare allowed tools and invoke a skill action
- **Skills** (`skills/<name>/SKILL.md`) — thick implementation containing all logic, state management, and workflows
- **Agents** (`agents/<name>.md`) — specialized subagents spawned via the Task tool for focused work (grimoire, runebook, chisel)
- **Awareness skills** (`skills/<name>-awareness/SKILL.md`) — proactive triggers that detect when a plugin should activate

### State Management

All state is plain markdown files designed to be committed to git:

- `.specsmiths/<name>.md` — spec with YAML frontmatter (`status`, `current_phase`, `total_phases`)
- `.specsmiths/<name>.research.md` — research findings from subagents
- `.specsmiths/<name>.questions.md` — discovery Q&A log
- `.specsmiths/active.json` — tracks active spec (`{"active": "<name>", "last_switched": "<ISO date>"}`)
- `.specsmiths/uispec.json` — UISpec config (`{"frontend_path": "../frontend", "last_synced": "<ISO>"}`)
- `.specsmiths/<endpoint>-ui.md` — UI implementation spec (created by `/specsmith-uispec-new`)
- `.specsmiths/<endpoint>-ui.research.md` — UI research findings
- `.uispec/openapi.yaml` — API contract (backend-owned)
- `.uispec/endpoints/<endpoint>.md` — per-endpoint spec with API sections + Suggested Implementation + UI Guidelines
- `.uispec/design-system.md` — design tokens (frontend-owned)
- `.uispec/components.md` — shared component patterns (Shared Patterns section backend-owned, rest frontend-owned)
- `.runebook/index.md` — master index with links, tags, dates
- `.runebook/<type>/<name>.md` — entry with YAML frontmatter (endpoints, jobs, flows, services, integrations, pages, components, hooks)
- `.runebook/guides/<name>.md` — narrative system guides organized by feature domain
