---
name: specsmith-uispec
description: "UISpec integration for specsmith. Backend: initialize and sync API contracts to frontend's .uispec/. Frontend: detect gaps, create UI implementation specs, validate conformance. Triggers on: 'sync uispec', 'init uispec', 'detect uispec', 'validate ui', 'build ui for', 'implement ui', or when .uispec/ or .specsmiths/uispec.json exists."
user-invocable: false
---

# Specsmith UISpec Skill

Integrates UISpec into specsmith for full-stack workflows. Backend projects push API contracts to `.uispec/` in the frontend project. Frontend projects read `.uispec/` and create specsmith specs for UI implementation.

## State

**Backend project stores:**
- `.specsmiths/uispec.json` — config: `{"frontend_path": "../my-frontend", "last_synced": "<ISO>"}`

**Frontend project stores:**
- `.uispec/` — API contracts pushed by backend
- `.specsmiths/<endpoint>-ui.md` — UI implementation specs (created by `/specsmith-uispec-new`)
- `.specsmiths/<endpoint>-ui.research.md` — research findings

## Directory Structure

```
.uispec/
├── openapi.yaml         ← API contract (source of truth, backend-owned)
├── UISPEC.md            ← Index of all endpoints
├── design-system.md     ← Design tokens (frontend-owned)
├── components.md        ← Shared component patterns (backend seeds, frontend owns)
└── endpoints/
    └── <endpoint>.md    ← Per-endpoint spec
```

## Ownership Rules

| Section | Owner | Backend | Frontend |
|---------|-------|---------|----------|
| openapi.yaml | Backend | Read + Write | Read only |
| Method, Path, Auth, Request, Response, Errors | Backend | Read + Write | Read only |
| Suggested Implementation | Backend generates | Seeds it | Can override |
| Related Endpoints | Backend | Read + Write | Read only |
| UI Guidelines, Components, States, Accessibility | Frontend | Read only | Read + Write |
| design-system.md | Frontend | Read only | Read + Write |
| components.md (Shared Patterns section) | Backend | Read + Write | Read only |
| components.md (everything else) | Frontend | Read only | Read + Write |
| Changelog | Shared | Append only | Append only |

---

## Action: init

Initialize `.uispec/` in the frontend project. First-time setup for full-stack workflow.

### Step 1: Configure Frontend Path

Ask the user: "What is the frontend project directory name? (e.g., `my-frontend` — I'll create `.uispec/` at `../my-frontend/.uispec/`)"

Verify `../<frontend>/` exists. If not: "Directory `../<frontend>/` not found. Check the name and try again."

If `.uispec/` already exists in the frontend:
- Ask: "`.uispec/` already exists at `../<frontend>/.uispec/`. Re-initialize (overwrites templates, preserves endpoint files and manual edits)? Or abort?"
- If abort: stop
- If re-init: preserve existing endpoint files with manual edits, regenerate templates

Save config to `.specsmiths/uispec.json`:
```json
{"frontend_path": "../<frontend>", "last_synced": "<ISO date>"}
```

Create `.specsmiths/` directory if it doesn't exist.

### Step 2: Create Directory Structure

Create the following in `../<frontend>/.uispec/`:

**openapi.yaml** — OpenAPI 3.1 skeleton:
```yaml
openapi: "3.1.0"
info:
  title: "<project> API"
  version: "1.0.0"
paths: {}
```

**UISPEC.md** — Index template:
```markdown
# UI Specifications

API contracts and UI guidelines for frontend development.

## Endpoints

| Endpoint | Method | Path | Status |
|----------|--------|------|--------|
```

**design-system.md** — Empty template with sections:
- Colors, Typography, Spacing, Breakpoints, Shadows, Z-Index

**components.md** — Empty template with sections:
- Shared Patterns (backend-seeded), Layout Components, Form Components, Data Display, Feedback/Status, Navigation

**endpoints/** — Empty directory with `.gitkeep`

### Step 3: Scan Backend Endpoints

Scan the backend codebase for all route definitions. Look for framework-specific patterns:
- **Express**: `router.get/post/put/delete`, `app.get/post/put/delete`
- **FastAPI**: `@app.get/post/put/delete`, `@router.get/post/put/delete`
- **Rails**: `resources`, `get/post/put/delete` in routes.rb
- **Django**: `path()` in urls.py, `@api_view` decorators
- **NestJS**: `@Get/@Post/@Put/@Delete` decorators
- **Other**: adapt to whatever framework the project uses

For each discovered endpoint, extract: method, path, request body shape, response shape, auth requirements, error responses.

### Step 4: Generate Initial Specs

1. Generate `openapi.yaml` from discovered endpoints (merge if file already has content)
2. Create `endpoints/<endpoint-name>.md` for each route (skip if file exists)
   - Use the endpoint file template (see Endpoint Template section)
   - Populate API sections from code analysis
   - Generate Suggested Implementation (see Suggested Implementation section)
3. Run cross-endpoint component analysis (see Component Analysis section)
4. Update `UISPEC.md` index table

### Step 5: Report

```
UISpec Initialized
===================
Frontend:    ../<frontend>/.uispec/
Endpoints:   N discovered
Files:       N created, N skipped (already existed)

Next steps:
  - Add the backend CLAUDE snippet to your backend CLAUDE.md
  - Add the frontend CLAUDE snippet to your frontend CLAUDE.md
  - Run /specsmith-uispec-sync after any API changes
```

---

## Action: sync

Sync API changes to frontend's `.uispec/`. Requires prior `/specsmith-uispec-init`.

### Step 1: Load Config

Read `.specsmiths/uispec.json`. If not found: "No UISpec config found. Run `/specsmith-uispec-init` first."

Verify the frontend directory exists. If not: "Frontend directory `<path>` not found. Update `.specsmiths/uispec.json` or re-run `/specsmith-uispec-init`."

### Step 2: Detect Changes

Scan all backend endpoints (same framework detection as init). Compare against existing `.uispec/endpoints/`:

- **New endpoints** — routes in backend with no spec file
- **Modified endpoints** — routes whose method, path, request/response shapes, auth, or error responses differ from the spec
- **Removed endpoints** — spec files with no corresponding backend route

### Step 3: Apply Updates

**For new endpoints:**
1. Add path to `openapi.yaml`
2. Create `endpoints/<endpoint>.md` using template
3. Add row to `UISPEC.md` index

**For modified endpoints:**
1. Update path definition in `openapi.yaml`
2. Update ONLY backend-owned sections in `endpoints/<endpoint>.md`:
   - API section (Method, Path, Auth, Permissions)
   - Request section (parameters, body)
   - Response section (success, errors)
   - Suggested Implementation (regenerate)
   - Related Endpoints (update)
3. **PRESERVE** all frontend-owned sections (UI Guidelines, Components, States, Validation, Accessibility)
4. Append changelog entry: `- <date> API updated by specsmith-uispec-sync: <what changed>`

**For removed endpoints:**
1. Remove from `openapi.yaml`
2. Remove endpoint file
3. Remove row from `UISPEC.md` index
4. Check `components.md` for orphaned patterns referencing only this endpoint

### Step 4: Cross-Endpoint Component Analysis

Run the component analysis (see Component Analysis section) and update `components.md` Shared Patterns section.

### Step 5: Validation

Run all validation checks:
1. **YAML validity** — `openapi.yaml` is valid YAML and follows OpenAPI 3.x
2. **Endpoint coverage** — all backend routes have corresponding spec files
3. **No orphaned specs** — all spec files correspond to existing endpoints
4. **Reference integrity** — all `$ref` in `openapi.yaml` resolve correctly
5. **Index accuracy** — `UISPEC.md` index lists all current endpoint files

### Step 6: Update Config

Update `.specsmiths/uispec.json` with `"last_synced": "<ISO date>"`.

### Step 7: Report

```
UISpec Sync Report
===================
Added:     N new endpoints
Modified:  N endpoints updated
Removed:   N endpoints removed

Details:
  [new]     POST /api/users — created endpoint spec
  [updated] GET /api/users/:id — response shape changed (added email field)
  [removed] DELETE /api/legacy — endpoint removed

Components:
  [pattern] UserDisplay — now used by 3 endpoints
  [new]     PaginatedList — detected in list-users, list-orders

Validation: N passed, N issues
  [warn] endpoints/old-report — spec exists but route not found (orphaned?)

Last synced: <date>
```

---

## Action: new

Create a UI implementation spec from a `.uispec/` endpoint. This is the frontend equivalent of `/specsmith-new` — a full specsmith workflow driven by the API contract.

### Prerequisites

Verify `.uispec/` exists. If not: "No `.uispec/` directory found. Ask the backend team to run `/specsmith-uispec-init`."

### Input Parsing

Parse `{{ARGS}}`:
- **No args** → Discovery mode (show unimplemented endpoints, let user pick)
- **Endpoint name** → Start workflow for that endpoint
- **Endpoint: context** → Start workflow with implementation context (split on first colon)

### Discovery Mode (no args)

1. List all files in `.uispec/endpoints/*.md`
2. For each endpoint, check if `.specsmiths/<endpoint>-ui.md` exists:
   - If exists and `status: complete` → implemented
   - If exists and `status: implementing` → in progress
   - If not exists → unimplemented (new)
3. Check modification dates: if `.uispec/endpoints/<endpoint>.md` was modified after `.specsmiths/<endpoint>-ui.md`, mark as "changed"
4. Show list:

```
UISpec Endpoints
=================
  [new]     create-user    POST /api/users
  [new]     list-orders    GET /api/orders
  [changed] login          POST /api/auth/login — spec updated since last implementation
  [active]  get-user       GET /api/users/:id — Phase 2/3 (step 2.1)
  [done]    delete-user    DELETE /api/users/:id

Run /specsmith-uispec-new <endpoint> to start implementing.
```

5. Use AskUserQuestion to let user pick an endpoint

### Full Workflow (with endpoint)

#### Phase 1: Discovery

1. Read `.uispec/endpoints/<endpoint>.md` — full spec including Suggested Implementation and Related Endpoints
2. Read `.uispec/design-system.md` — available design tokens
3. Read `.uispec/components.md` — shared patterns and reusable components
4. Scan existing frontend code for any prior implementation of this endpoint's API path
5. Determine: new implementation vs. update to existing code
6. If context was provided (colon syntax), incorporate it into understanding

If the endpoint spec is comprehensive (has Suggested Implementation, clear states, validation rules), proceed directly to Phase 2 without asking questions.

If there are genuine gaps (ambiguous UX requirements, unclear component choices), ask 2-3 focused questions using AskUserQuestion.

Log discovery to `.specsmiths/<endpoint>-ui.questions.md` (brief + any Q&A).

#### Phase 2: Research

**Check if grimoire is installed** (look for grimoire skills/agents).

**If grimoire available**, use `grimoire:grimoire-research` skill:
- `grimoire-docs`: Component library docs based on project framework (React, Vue, Angular, Svelte, etc.)
- `grimoire-practices`: UI/UX best practices, a11y patterns, form validation patterns
- `grimoire-codebase`: Existing patterns, conventions, shared utilities in the frontend project

**If grimoire NOT available**, spawn inline subagents:

**Always spawn:**
- **Codebase Analysis** (subagent_type: explore, thoroughness: very thorough)
  - Detect framework, styling approach, state management
  - Find existing component patterns and naming conventions
  - Identify utilities, hooks, shared components to reuse
  - Map file structure conventions

**Spawn if relevant:**
- **Component Library Docs** (subagent_type: general-purpose) — if using a component library (MUI, Chakra, Radix, etc.)
- **Best Practices** (subagent_type: general-purpose) — if accessibility requirements, complex form validation, or unfamiliar patterns

Save research to `.specsmiths/<endpoint>-ui.research.md`.

#### Phase 3: Planning

Generate implementation spec at `.specsmiths/<endpoint>-ui.md`:

```markdown
---
status: planning
type: ui-implementation
endpoint: <endpoint-name>
created: <ISO date>
updated: <ISO date>
current_phase: 0
total_phases: <N>
uispec_file: .uispec/endpoints/<endpoint>.md
research: <endpoint>-ui.research.md
discovery: <endpoint>-ui.questions.md
---

# UI Spec: <endpoint>

## Overview
<What this UI does, which endpoint it connects to, the approach chosen>

## Requirements
### From API Contract
- <Extracted from .uispec endpoint spec>

### UI Requirements
- States: <loading, error, empty, success — from spec>
- Validation: <rules from spec>
- Accessibility: <requirements from spec>
- Design tokens: <relevant tokens from design-system.md>

### Edge Cases
| # | Scenario | Expected Behavior | Phase |
|---|----------|-------------------|-------|
| E1 | API returns 401 | Redirect to login | 1 |
| E2 | API returns 429 | Show rate limit message with retry timer | 1 |
| E3 | Empty response | Show empty state illustration | 1 |
| E4 | Form validation fails | Show inline errors per field | 1 |
| ... | ... | ... | ... |

## Implementation

### Phase 1: Core UI
- [ ] 1.1. <Step>
  **What:** <Description>
  **Where:** <File paths>
  **How:** <Approach — components, hooks, patterns from research>
  **Edge cases:** <E1, E3>
  **Verify:** <How to check>

...

**Phase 1 acceptance:** <What must be true>

### Phase 2: Polish & Validation
...
```

Quality bar: same as specsmith specs — every step implementable without follow-up questions.

Show the plan to the user, ask for approval before implementing.

After approval, set `status: implementing`, `current_phase: 1`, and update `.specsmiths/active.json`.

#### Phase 4: Implementation

Execute steps with specsmith's Step Completion Protocol:

1. Read step details and edge case references
2. Implement following **How** guidance and patterns from research
3. Use design tokens from `.uispec/design-system.md` — never hardcode colors, spacing, fonts
4. Follow component patterns from `.uispec/components.md`
5. Auto-detect framework conventions:
   - Framework: React (JSX/TSX), Vue (.vue), Angular, Svelte
   - Styling: CSS modules, Tailwind, styled-components, SCSS, CSS-in-JS
   - State management: React Query, SWR, Redux, Vuex, Pinia
   - File structure and naming conventions
   - **Follow whatever conventions the project uses. Do not impose different patterns.**
6. Handle referenced edge cases
7. Run **Verify** check
8. Mark step done: `- [ ]` → `- [x]`, update frontmatter, announce

**After all steps complete:** Update the endpoint's UI Guidelines section in `.uispec/endpoints/<endpoint>.md`:
- Which components were used
- Accessibility attributes added
- State management approach
- Any deviations from spec (and why)
- Append changelog: `- <date> UI implemented by specsmith-uispec-new`

**Never modify API sections** (Method, Path, Auth, Request, Response) — backend-owned.

#### Phase 5: Validation

Run 6 validation checks scoped to this endpoint (see Validation Checks section).

If all pass: mark `.specsmiths/<endpoint>-ui.md` as `status: complete`.
If any fail: fix issues, re-run checks, then mark complete.

Report:
```
UISpec Implementation Complete
================================
Endpoint:    <endpoint>
Spec:        .specsmiths/<endpoint>-ui.md
Status:      ✓ Complete

Files created/modified:
  - src/pages/...
  - src/components/...

Validation:
  API contract:    PASS
  Shapes:          PASS
  States:          PASS
  Design tokens:   PASS
  Accessibility:   PASS
  Form validation: PASS (or N/A)
```

---

## Action: detect

Quick check for gaps between `.uispec/` and UI code.

### Prerequisites

Verify `.uispec/` exists. If not: "No `.uispec/` directory found. Ask the backend team to run `/specsmith-uispec-init`."

### Check 1: Unimplemented specs

1. List all files in `.uispec/endpoints/*.md`
2. For each endpoint spec, extract the API path (Method + Path)
3. Search the codebase for components/pages that call this API path (fetch, axios, useSWR, useQuery, useMutation, $http, HttpClient, api., request())
4. Report specs where no UI implementation references the endpoint's API path

### Check 2: Undocumented API calls

1. Scan the codebase for API call patterns: fetch(), axios, useSWR, useQuery, useMutation, $http, HttpClient, api., request(
2. Extract API paths/URLs
3. Compare against `.uispec/endpoints/`
4. Report API calls with no corresponding endpoint spec

### Check 3: Shape mismatches

1. For each endpoint spec with a corresponding UI implementation:
2. Read spec's Request and Response sections for expected fields
3. Search implementation for TypeScript interfaces, type definitions, or data access patterns
4. Compare fields
5. Report differences: missing fields, extra fields, type mismatches

### Check 4: Design system violations

1. Read `.uispec/design-system.md` for available tokens
2. Scan component files for hardcoded values:
   - Colors: hex (#xxx, #xxxxxx), rgb(), rgba(), hsl(), hsla()
   - Font sizes: px/rem/em for font-size
   - Spacing: hardcoded px/rem margins and paddings
3. Report violations with file, line, hardcoded value, and suggested token

### Check 5: Missing state handling

1. For each endpoint spec, read the States section
2. Find the corresponding UI implementation
3. Check if the component handles all documented states (loading, error, empty, success)
4. Report missing states

### Output Format

```
UISpec Detection Report
========================
Unimplemented specs:      N endpoints
Undocumented API calls:   N calls
Shape mismatches:         N issues
Design system violations: N found
Missing state handling:   N gaps

Details:
  [spec]  endpoints/<name> — no UI implementation found
  [api]   GET /api/path — no endpoint spec exists
  [shape] endpoints/<name> — response missing `fieldName`
  [token] src/components/File.tsx:45 — #3B82F6 → use primary-500
  [state] endpoints/<name> — empty state not handled

Suggested fixes:
  - Run /specsmith-uispec-new <endpoint> for unimplemented specs
  - Request backend to sync specs for undocumented API calls
  - Update TypeScript interfaces to match spec shapes
  - Replace hardcoded values with design system tokens
  - Add missing state handling to components
```

---

## Action: validate

Validate that existing UI code follows `.uispec/` specs with PASS/FAIL checks across all implemented endpoints.

### Prerequisites

Verify `.uispec/` exists. If not: "No `.uispec/` directory found. Ask the backend team to run `/specsmith-uispec-init`."

List all endpoint specs in `.uispec/endpoints/`. For each spec that has a corresponding UI implementation, run all 6 checks.

### Validation Checks

**Check 1: API contract conformance**
- Verify correct HTTP method per spec
- Verify correct API path per spec
- Verify authentication headers/tokens per spec's Auth section
- PASS: method, path, auth all match. FAIL: any mismatch.

**Check 2: Request/response shapes**
- Compare TypeScript interfaces or data structures against spec's Request and Response sections
- All required fields present, field names match exactly
- PASS: all fields present and correctly named. FAIL: missing or misnamed fields.

**Check 3: State handling completeness**
- Verify component renders UI for each documented state (loading, error, empty, success)
- PASS: all documented states handled. FAIL: any state missing.

**Check 4: Design system compliance**
- Scan component files for hardcoded colors, font sizes, spacing
- Cross-reference against `.uispec/design-system.md` tokens
- PASS: all values use design tokens. FAIL: hardcoded values found.

**Check 5: Accessibility compliance**
- Read spec's Accessibility section for required ARIA attributes, labels, keyboard navigation
- Verify component includes specified accessibility features
- PASS: all requirements met. FAIL: any missing. N/A: no accessibility section.

**Check 6: Form validation compliance**
- Read spec's Validation section for required fields, format checks, error messages
- Verify component implements specified validation rules
- PASS: all rules implemented. FAIL: any rule missing. N/A: no form/validation requirements.

### Output Format

```
UISpec UI Validation Report
============================
                              PASS    FAIL    N/A
API contract conformance        N       N       N
Request/response shapes         N       N       N
State handling                  N       N       N
Design system compliance        N       N       N
Accessibility                   N       N       N
Form validation                 N       N       N

FAILURES:
  [API]    endpoints/<name> — uses GET but spec says POST
  [shape]  endpoints/<name> — response missing `fieldName`
  [state]  endpoints/<name> — empty state documented but not handled
  [token]  src/components/File.tsx:23 — #3B82F6 → colors.primary.500
  [a11y]   endpoints/<name> — missing aria-describedby on password field
  [form]   endpoints/<name> — missing email format validation

Suggested fixes:
  - Fix API method/path to match spec
  - Update interfaces to include missing fields
  - Add missing state handling components
  - Replace hardcoded values with design tokens
  - Add required ARIA attributes
  - Implement missing validation rules
```

---

## Endpoint File Template

Used by init and sync when creating new endpoint specs:

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

## Suggested Implementation
<!-- Backend-generated. Frontend can override in UI Guidelines. -->
- **Endpoint type**: [create-form | edit-form | detail-view | list-view | delete-action | action-trigger]
- **Component hierarchy**: [Parent → Children tree]
- **Key interactions**: [What the user does step by step]
- **State flow**: [idle → loading → success/error]
- **Cross-endpoint reuse**: [Shared patterns from components.md]
- **Watch out for**: [Rate limits, file uploads, nested objects, date formatting, case transforms]

## Related Endpoints
- [list-<resource>] — same resource, list view
- [update-<resource>] — same resource, edit form
- [delete-<resource>] — same resource, delete action

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

Remove sections that don't apply. Keep focused on what the frontend needs.

---

## Suggested Implementation Logic

When creating or updating endpoint files, populate Suggested Implementation based on endpoint patterns:

| Endpoint Pattern | Type | Suggested Components | Key Interaction |
|-----------------|------|---------------------|-----------------|
| POST with body (no ID) | create-form | FormBuilder, TextField, SelectField | User fills form, submits, sees success/error |
| PUT/PATCH with ID | edit-form | FormBuilder (pre-filled), TextField | User edits fields, submits changes |
| GET with ID (single resource) | detail-view | DetailCard, Badge, ActionMenu | User views details, takes actions |
| GET without ID (list/collection) | list-view | DataTable or CardGrid, Pagination, SearchBar | User browses, filters, paginates |
| DELETE with ID | delete-action | ConfirmDialog, DangerButton | User clicks delete, confirms in modal |
| POST (action, no resource creation) | action-trigger | Button, StatusIndicator | User triggers action, sees result |

**Component hierarchy** — not just flat component names but a tree:
```
CreateUserPage
├── PageHeader (title, breadcrumb)
├── UserForm
│   ├── TextField (name)
│   ├── TextField (email)
│   ├── SelectField (role)
│   └── SubmitButton
└── FormFeedback (success/error messages)
```

**State flow** — describe transitions:
```
idle → (user submits) → loading → success (redirect to detail) | error (show inline)
```

**Cross-endpoint reuse** — reference components.md shared patterns:
```
Shares UserDisplay pattern with list-users and get-order — reuse UserBadge component
```

**Watch out for** — scan endpoint API details for:
- Rate limits (429 response → mention limit)
- File uploads (multipart body → mention FormData)
- Nested response objects (→ mention flattening needs)
- Pagination (→ mention cursor vs offset, total count)
- Long-running operations (→ mention polling or WebSocket)
- Auth/permission requirements (→ mention role-based UI hiding)
- Date formatting (→ "API returns ISO strings — format for display")
- Case transforms (→ "API uses snake_case — transform to camelCase")

---

## Cross-Endpoint Component Analysis

Run during init and sync. Analyzes all endpoints together to identify shared patterns:

### Step 1: Group by type
- Categorize all endpoints as list-view, detail-view, create-form, edit-form, delete-action, action-trigger

### Step 2: Identify shared response shapes
- Find fields that appear in multiple endpoint responses (e.g., `user` object in list-users, get-order, get-comment)
- Find common sub-objects (embedded resources)

### Step 3: Detect patterns
- Pagination patterns across list endpoints (cursor vs offset, total_count)
- Filter/sort patterns
- Error response formats
- Auth patterns

### Step 4: Update components.md

Add/update the **Shared Patterns** section (backend-owned):

```markdown
## Shared Patterns

### User Display
- **Used by**: list-users, get-order, get-comment
- **Fields**: id, name, avatar, role
- **Suggested component**: UserBadge / UserCard

### Pagination
- **Used by**: list-users, list-orders, list-products
- **Pattern**: cursor-based with total_count
- **Suggested component**: PaginatedList

### Error Feedback
- **Format**: `{ "error": "<code>", "details": [...] }`
- **Used by**: all mutation endpoints
- **Suggested component**: ErrorAlert with per-field mapping
```

Preserve all other sections in components.md (frontend-owned).

---

## Rules

1. **Backend init/sync commands:**
   - Always update `openapi.yaml` first — it is the API source of truth
   - One endpoint = one file in `endpoints/`
   - Read existing files before modifying — preserve frontend-owned sections and manual additions
   - Always add changelog entries when updating
   - Always populate Suggested Implementation based on endpoint analysis
   - Always run cross-endpoint component analysis

2. **Frontend new/detect/validate commands:**
   - Always read the endpoint spec before building API-connected UI
   - Never modify `openapi.yaml` — backend-owned
   - Never modify API sections in endpoint files — backend-owned
   - Use design tokens from `design-system.md`, never hardcode values
   - Follow specsmith's Step Completion Protocol during implementation
   - Flag API discrepancies — if actual API doesn't match spec, flag for backend team

3. **Both:**
   - Changelog is append-only — never remove or modify existing entries
   - Respect ownership boundaries strictly
