---
description: "Validate uispec directory for consistency and completeness. Checks openapi.yaml validity, endpoint coverage, orphaned specs, and index accuracy. Usage: /uispec-validate"
allowed-tools: Read, Bash, Glob, Grep, LS
---

# UISpec Validate

Validate the `uispec/` directory for consistency, completeness, and correctness.

## Workflow

### 1. Locate UISpec Directory

Read the backend project's CLAUDE.md or ask the user for the frontend project name to determine `{{PRJ-frontend}}`.

Verify `../{{PRJ-frontend}}/uispec/` exists. If not: "No uispec directory found. Run `/uispec-init` to set it up."

### 2. Run Validation Checks

Perform all checks and collect results. Do not stop at the first failure — run all checks and report everything.

**Check 1: OpenAPI validity**
- Read `openapi.yaml`
- Verify it is valid YAML (no syntax errors)
- Verify it follows OpenAPI 3.x structure (has `openapi`, `info`, `paths` keys)
- Verify all `$ref` references resolve to existing definitions
- Report: PASS/FAIL with specific errors

**Check 2: Endpoint coverage**
- Scan the backend project for all route definitions
- Check that each route has a corresponding file in `endpoints/`
- Report: list of endpoints missing spec files

**Check 3: Orphaned specs**
- List all files in `endpoints/`
- Check that each file corresponds to an actual backend endpoint
- Report: list of spec files with no matching backend route

**Check 4: Index accuracy**
- Read `UISPEC.md`
- Compare its endpoint listing against actual files in `endpoints/`
- Report: missing entries, stale entries, incorrect links

**Check 5: Spec completeness**
- For each endpoint file, check that required sections exist:
  - API section (Method, Path)
  - At least one of Request or Response
- Report: endpoint files with missing required sections

### 3. Report Results

Show a summary:

```
UISpec Validation Report
========================

OpenAPI validity:     PASS/FAIL
Endpoint coverage:    X/Y endpoints covered
Orphaned specs:       N found
Index accuracy:       PASS/FAIL
Spec completeness:    X/Y specs complete

Issues:
- [endpoint-name] Missing spec file
- [old-endpoint] Orphaned spec (no backend route)
- [UISPEC.md] Missing entry for [endpoint-name]
- [openapi.yaml] Invalid $ref: #/components/schemas/Missing

Suggested fixes:
- Run `/uispec-sync` to create missing specs and remove orphans
- Manually fix openapi.yaml $ref references
```
