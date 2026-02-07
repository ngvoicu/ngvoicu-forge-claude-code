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

### Integration with Other Plugins

- **specsmith**: Auto-syncs during `/specsmith-implement` when API endpoints change
- **runebook**: Update endpoint entries after syncing
- **grimoire**: Use `/grimoire contracts` before designing endpoints
- **chisel**: Use `/chisel rename` to update both source and spec files
