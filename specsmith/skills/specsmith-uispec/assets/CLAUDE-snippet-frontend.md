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

### Integration with Other Plugins

- **specsmith**: UI specs are stored as `.specsmiths/<endpoint>-ui.md` — use `/specsmith-status` to see all specs
- **runebook**: Read endpoint entries for additional API context
- **grimoire**: `/grimoire docs` for component library APIs, `/grimoire practices` for a11y best practices
- **chisel**: `/chisel rename` when renaming components that reference endpoints

### Notes

- If `.uispec/` doesn't exist, ask the backend team to run `/specsmith-uispec-init`
- If an endpoint spec doesn't exist, flag it for the backend team
- If API response doesn't match spec, flag the discrepancy immediately
