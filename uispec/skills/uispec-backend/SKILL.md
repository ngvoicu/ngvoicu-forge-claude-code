---
name: uispec-backend
description: "Maintains UI specifications in ../{{PRJ-frontend}}/uispec/ for backend projects. MUST USE after any API change: new endpoint, modified request/response, removed endpoint. Syncs openapi.yaml, endpoint specs, and component patterns. Trigger words: 'sync uispec', 'update uispec', 'api changed', or automatically after any endpoint work."
---

# UISpec Backend Skill

Keeps `../{{PRJ-frontend}}/uispec/` in sync with backend API changes. `{{PRJ-frontend}}` is the frontend project directory name (e.g. `btb-frontend`).

## Structure

```
../{{PRJ-frontend}}/
└── uispec/
    ├── openapi.yaml         ← API contract (source of truth)
    ├── UISPEC.md            ← Overview/index
    ├── design-system.md     ← Tokens, colors, typography
    ├── components.md        ← Reusable UI patterns
    └── endpoints/
        └── <endpoint>.md    ← Per-endpoint UI specs
```

## First-Time Setup

If `../{{PRJ-frontend}}/uispec/` doesn't exist:

1. Ask the user to confirm the frontend project name (the `{{PRJ-frontend}}` value)
2. Create the directory structure:
   - `openapi.yaml` — basic OpenAPI 3.1 skeleton with project info
   - `UISPEC.md` — index template listing all endpoints and linking to specs
   - `design-system.md` — empty template with sections: Colors, Typography, Spacing, Breakpoints, Shadows, Z-Index
   - `components.md` — empty template with sections: Layout Components, Form Components, Data Display, Feedback/Status, Navigation
   - `endpoints/` — empty directory
3. Scan existing backend routes (controllers, route files, etc.)
4. Generate initial `openapi.yaml` from current endpoints — include paths, methods, basic request/response shapes
5. Create an endpoint file for each existing route using the endpoint template below
6. Inform the user: "UISpec initialized at `../{{PRJ-frontend}}/uispec/`. Add the CLAUDE.md snippet to your backend project to enforce sync."

Use `/uispec-init` as the command shortcut for this workflow.

## When to Sync

After ANY API change:
- New endpoint → create `endpoints/<n>.md`, update `openapi.yaml`
- Modified endpoint (request/response shape, auth, status codes) → update both files
- Removed endpoint → remove endpoint file, update `openapi.yaml`
- New response patterns → update `components.md`
- Auth changes → update affected endpoint files
- New error formats → update affected endpoint files and `components.md` error patterns

## Workflow

1. Detect API change in backend code
2. Update `../{{PRJ-frontend}}/uispec/openapi.yaml`
3. Create/update `../{{PRJ-frontend}}/uispec/endpoints/<endpoint>.md`
4. Update `components.md` if new UI patterns needed
5. Update `UISPEC.md` index if new endpoints added/removed
6. Run validation (see Validation section below)

## Endpoint File Template

```markdown
# <Endpoint Name>

## API
- **Method:** GET/POST/PUT/DELETE
- **Path:** /api/v1/...
- **Auth:** Required/Optional/Bearer
- **Permissions:** [list required roles/permissions]

## Request

### Path Parameters
- `id` (string, required): Description

### Query Parameters
- `page` (integer, optional, default: 1): Page number
- `limit` (integer, optional, default: 20, max: 100): Items per page

### Body
\```json
{ "field": "type and constraints" }
\```

### Example
\```json
{ "field": "example value" }
\```

## Response

### Success (200)
\```json
{ "data": "schema" }
\```

### Error Responses
- **400**: Validation error — `{ "error": "validation_error", "details": [...] }`
- **401**: Unauthorized — `{ "error": "unauthorized" }`
- **404**: Not found — `{ "error": "not_found" }`
- **429**: Rate limited — `{ "error": "rate_limit", "retry_after": 60 }`

## UI Guidelines
- **Layout**: Component/page structure
- **Components**: Which reusable components to use (reference components.md)
- **States**:
  - Loading: How to show loading
  - Success: What happens after success
  - Error: How to display errors
  - Empty: Empty state display
- **Validation**: Frontend validation rules (before API call)
- **Accessibility**: Required a11y attributes

## Changelog
- YYYY-MM-DD: Initial creation
```

Remove sections that don't apply (e.g. no Path Parameters if the endpoint has none). Keep the template focused on what the frontend actually needs.

## Validation

After each sync, verify:

1. **YAML validity**: Confirm `openapi.yaml` is valid YAML and follows OpenAPI 3.x structure
2. **Endpoint coverage**: Check that all endpoints in backend code have corresponding files in `endpoints/`
3. **No orphaned specs**: Check that no endpoint files exist for endpoints that were removed from the backend
4. **Reference integrity**: Verify all `$ref` references in `openapi.yaml` resolve correctly
5. **Index accuracy**: Check that `UISPEC.md` index lists all current endpoint files

If validation fails, report specific issues and suggest fixes. Do not silently ignore validation errors.

Use `/uispec-validate` as the command shortcut for running validation standalone.

## Rules

1. Always update `openapi.yaml` first
2. One endpoint = one file in `endpoints/`
3. Read existing files before modifying — preserve manual additions
4. Keep specs focused on what frontend needs — don't duplicate backend implementation details
5. Include a Changelog entry in endpoint files when updating

## CLAUDE.md Snippet

Add this to the backend project's CLAUDE.md (replace `{{PRJ-frontend}}` with your frontend project directory name). See the expanded version in `assets/CLAUDE-snippet.md`.
