## UISpec (MANDATORY)

UI specs live in `uispec/`. **ALWAYS read specs before building API-connected UI.**

### Before Building UI

Before creating or modifying any page/component that calls an API endpoint:
1. Read `uispec/endpoints/<endpoint>.md` for the API contract and UI guidelines
2. Read `uispec/components.md` for reusable patterns
3. Read `uispec/design-system.md` for tokens and styling rules
4. Follow the spec's States section (loading, success, error, empty)
5. Apply the spec's Validation rules before API calls

### Ownership Rules

**Read only (backend-owned — do not modify):**
- `uispec/openapi.yaml` — API contract, changes come from backend only
- API sections in endpoint files (Method, Path, Auth, Request, Response)

**Editable (frontend-owned):**
- `uispec/design-system.md` — update as design tokens evolve
- `uispec/components.md` — add patterns as new components are built
- UI Guidelines sections in endpoint files (Layout, Components, States, Validation, Accessibility)

### After Building UI

After implementing a page/component, update the endpoint's UI Guidelines section with:
- Which components were actually used
- Accessibility attributes added
- State management approach
- Any deviations from the original spec (and why)

### Missing Specs

If no endpoint spec exists for an API you need: stop and flag it. Ask the backend team to create the spec before proceeding.
