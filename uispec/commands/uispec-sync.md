---
description: "Manually sync uispec directory with current backend API state. Detects changes and updates spec files. Usage: /uispec-sync"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, LS
---

# UISpec Sync

Manually trigger a sync of the `uispec/` directory with the current backend API state.

## Workflow

### 1. Locate UISpec Directory

Read the backend project's CLAUDE.md or ask the user for the frontend project name to determine `{{PRJ-frontend}}`.

Verify `../{{PRJ-frontend}}/uispec/` exists. If not, suggest: "No uispec directory found. Run `/uispec-init` to set it up."

### 2. Detect Backend Changes

Scan the current backend project for all endpoints:
- Route definitions, controller files, API handlers
- Compare discovered endpoints against existing `uispec/endpoints/*.md` files and `openapi.yaml`

Categorize changes:
- **New endpoints**: Routes in backend with no corresponding spec file
- **Modified endpoints**: Routes whose request/response shapes differ from the spec
- **Removed endpoints**: Spec files with no corresponding backend route

### 3. Apply Updates

**For new endpoints:**
1. Add path to `openapi.yaml`
2. Create `endpoints/<endpoint-name>.md` using the endpoint template
3. Add entry to `UISPEC.md` index

**For modified endpoints:**
1. Update `openapi.yaml` with new path/method/schema details
2. Update API sections in the endpoint file (Method, Path, Auth, Request, Response, Error Responses)
3. Preserve frontend-owned sections (UI Guidelines, Components, States, Validation, Accessibility)
4. Add a Changelog entry with the date and what changed

**For removed endpoints:**
1. Remove path from `openapi.yaml`
2. Remove the endpoint file from `endpoints/`
3. Remove entry from `UISPEC.md` index
4. Check `components.md` for patterns only used by removed endpoints

### 4. Update Components

If new response patterns or UI patterns are introduced:
- Update `components.md` with new component suggestions
- Reference which endpoints use the new patterns

### 5. Validate

Run the validation checks (same as `/uispec-validate`):
1. Verify `openapi.yaml` is valid YAML
2. Check endpoint coverage (all backend routes have spec files)
3. Check for orphaned specs
4. Verify `UISPEC.md` index is accurate

### 6. Report

Show the user a summary:
- Endpoints added, modified, removed
- Files created, updated, deleted
- Any validation issues found
- Any manual review needed (e.g. UI Guidelines sections to update)
