## UISpec (MANDATORY)

UI specs live in `../{{PRJ-frontend}}/uispec/`. **ALWAYS sync after API changes.**

### When to Sync

You MUST run the uispec sync workflow after ANY of these changes:
- Creating a new endpoint (route, controller, handler)
- Modifying an endpoint's request or response shape
- Changing authentication or authorization on an endpoint
- Removing or deprecating an endpoint
- Adding new error response formats
- Changing pagination, filtering, or sorting behavior

### How to Sync

After creating/modifying/removing any endpoint:
1. Update `../{{PRJ-frontend}}/uispec/openapi.yaml`
2. Create/update `../{{PRJ-frontend}}/uispec/endpoints/<endpoint>.md`
3. Update `../{{PRJ-frontend}}/uispec/components.md` if new UI patterns needed
4. Update `../{{PRJ-frontend}}/uispec/UISPEC.md` index if endpoints added/removed

No exceptions. Frontend depends on this.

### First-Time Setup

If `../{{PRJ-frontend}}/uispec/` doesn't exist yet, use the `uispec-backend` skill's first-time setup workflow or run `/uispec-init` to initialize the directory structure and generate specs from existing endpoints.

### Error Handling

- If `../{{PRJ-frontend}}/` doesn't exist, warn: "Frontend project directory not found. Verify the project name in this snippet."
- If spec files were manually edited by the frontend team, preserve their additions — merge changes, don't overwrite.
- After sync, validate that openapi.yaml is valid and all endpoint files are up to date.

### Conflict Handling

If a spec file has been manually modified (e.g. by the frontend team adding UI guidelines):
- Read the existing content before modifying
- Preserve any sections not owned by the backend (UI Guidelines, Components, Accessibility)
- Only update API-related sections (Method, Path, Auth, Request, Response, Error Responses)
- Add a Changelog entry noting the update
