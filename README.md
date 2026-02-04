# ngvoicu-forge

Claude Code plugins for spec-driven development and UI specification workflows.

## Install

```bash
# Add the marketplace
/plugin marketplace add gabriel/ngvoicu-forge

# Install what you need
/plugin install specsmith@ngvoicu-forge
/plugin install uispec@ngvoicu-forge
```

## Plugins

### specsmith

Research-driven implementation specs. Every `/specsmith new` triggers a structured process:

1. **Discovery interview** — Claude asks rounds of focused questions about requirements, edge cases, failure modes, constraints, and boundaries. Doesn't stop until the problem is deeply understood. Uses a concrete "done" checklist to know when discovery is complete. All Q&A is logged to `.specsmiths/<n>.questions.md`.
2. **Deep research** — Adaptive subagents (1-5) investigate based on what discovery revealed: codebase architecture, web best practices, library docs, git history, database schema, or API contracts. Only relevant agents are spawned. Findings go to `.specsmiths/<n>.research.md`.
3. **Spec creation** — A full specification with requirements, architecture, data model, API design, edge case table, implementation steps (each with What/Where/How/Verify), risks, testing strategy, and rollback plan.

The result is a spec you could hand to any developer and they'd know exactly what to build.

```
/specsmith new auth           # interview -> research -> full spec
/specsmith new caching        # another spec (independent)
/specsmith switch auth        # activate auth, start working
/specsmith switch caching     # save progress, switch to caching
/specsmith switch auth        # come back, resume where you left off
/specsmith resume             # continue active spec (re-reads everything)
/specsmith status             # dashboard of all specs
/specsmith list               # quick list
/specsmith archive auth       # shelve a spec without deleting
/specsmith unarchive auth     # bring it back
/specsmith drop caching       # remove spec + research + questions
```

The specsmith-management skill also auto-detects `.specsmiths/` directories and proactively suggests creating specs when work gets complex. It handles mid-implementation edits, context restoration on resume, and validates spec integrity (name collisions, corrupted state, merge conflicts).

All state lives in `.specsmiths/` as plain markdown — commit it to git.

### uispec

A two-skill system for keeping frontend UI specs in sync with backend API changes.

**Backend skill (`uispec-backend`):** After any API change, syncs `openapi.yaml`, endpoint spec files, and `components.md` in the frontend project's `uispec/` directory. Includes first-time setup workflow, comprehensive endpoint templates (with auth, permissions, request/response examples, error responses, UI guidelines, accessibility, changelog), and post-sync validation.

**Frontend skill (`uispec-frontend`):** Reads specs from `uispec/` to guide component development. Knows which sections are backend-owned (API contract — read only) and which are frontend-owned (design system, components, UI guidelines — editable). Automatically activates before API-connected UI work.

```
/uispec-init             # initialize uispec/ from existing endpoints
/uispec-sync             # manual sync after backend changes
/uispec-validate         # check spec consistency and coverage
```

Add the CLAUDE.md snippet to your backend project to enforce sync:

```
## UISpec (MANDATORY)

UI specs live in `../{{PRJ-frontend}}/uispec/`. ALWAYS sync after API changes.

After creating/modifying/removing any endpoint:
1. Update `../{{PRJ-frontend}}/uispec/openapi.yaml`
2. Update `../{{PRJ-frontend}}/uispec/endpoints/<endpoint>.md`
3. Update `../{{PRJ-frontend}}/uispec/components.md` if new UI patterns needed

No exceptions. Frontend depends on this.
```

Replace `{{PRJ-frontend}}` with your frontend project directory name (e.g. `btb-frontend`). See `uispec/skills/uispec-backend/assets/CLAUDE-snippet.md` for the expanded version with error handling and conflict resolution guidance.

Add the CLAUDE.md snippet to your frontend project to enforce spec-first development:

```
## UISpec (MANDATORY)

UI specs live in `uispec/`. ALWAYS read specs before building API-connected UI.

Before creating or modifying any page/component that calls an API endpoint:
1. Read `uispec/endpoints/<endpoint>.md` for the API contract and UI guidelines
2. Read `uispec/components.md` for reusable patterns
3. Read `uispec/design-system.md` for tokens and styling rules

Do not modify `openapi.yaml` or API sections in endpoint files — those are backend-owned.
```

See `uispec/skills/uispec-frontend/assets/CLAUDE-snippet.md` for the expanded version with ownership rules and post-implementation guidance.
