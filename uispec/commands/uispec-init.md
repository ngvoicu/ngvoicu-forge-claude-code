---
description: "Initialize the uispec directory structure in the frontend project. Scans existing backend endpoints and generates initial specs. Usage: /uispec-init"
allowed-tools: Read, Write, Bash, Glob, Grep, LS
---

# UISpec Init

Initialize the `uispec/` directory structure in the frontend project and generate initial specs from existing backend endpoints.

## Workflow

### 1. Determine Frontend Project

Ask the user for the frontend project directory name. This becomes `{{PRJ-frontend}}`.

Verify that `../{{PRJ-frontend}}/` exists. If not, error: "Frontend project directory `../{{PRJ-frontend}}/` not found. Check the path and try again."

If `../{{PRJ-frontend}}/uispec/` already exists, warn: "UISpec directory already exists. Existing files will be preserved — only missing files will be created. Use `/uispec-sync` to update existing specs."

### 2. Create Directory Structure

**Skip rule:** Before creating each file below, check if it already exists. If it does, **skip it** and note "Skipped (already exists): `<filename>`". Never overwrite an existing file.

Create the following files in `../{{PRJ-frontend}}/uispec/`:

**`openapi.yaml`** — OpenAPI 3.1 skeleton:
```yaml
openapi: "3.1.0"
info:
  title: "{{project name}} API"
  version: "1.0.0"
  description: "API specification for {{project name}}"
paths: {}
components:
  schemas: {}
  securitySchemes: {}
```

**`UISPEC.md`** — Index template:
```markdown
# UISpec Index

API specifications and UI guidelines for the frontend.

## Endpoints

| Endpoint | Method | Path | Spec |
|----------|--------|------|------|
<!-- Generated entries will go here -->

## Files

- [Design System](./design-system.md)
- [Components](./components.md)
- [OpenAPI Spec](./openapi.yaml)
```

**`design-system.md`** — Empty template:
```markdown
# Design System

## Colors
<!-- Primary, secondary, neutral, semantic colors -->

## Typography
<!-- Font families, sizes, weights, line heights -->

## Spacing
<!-- Spacing scale -->

## Breakpoints
<!-- Responsive breakpoints -->

## Shadows
<!-- Box shadow definitions -->

## Z-Index
<!-- Z-index scale -->
```

**`components.md`** — Empty template:
```markdown
# UI Components

## Layout Components
<!-- Page layouts, grids, containers -->

## Form Components
<!-- Inputs, selects, checkboxes, form patterns -->

## Data Display
<!-- Tables, lists, cards, detail views -->

## Feedback / Status
<!-- Alerts, toasts, loading indicators, empty states, error displays -->

## Navigation
<!-- Nav bars, breadcrumbs, tabs, pagination -->
```

**`endpoints/`** — Empty directory (create with a `.gitkeep` if needed)

### 3. Scan Backend Endpoints

Search the current backend project for:
- Route definitions (Express routes, FastAPI paths, Rails routes, etc.)
- Controller files
- API handler files

Identify each endpoint's method, path, and basic request/response shape.

### 4. Generate Initial Specs

For each discovered endpoint:
1. Add the path to `openapi.yaml` — **merge** into existing paths (do not overwrite the file; add new paths only, skip paths that already exist)
2. Create `endpoints/<endpoint-name>.md` using the endpoint template from the `uispec-backend` skill — **only if the file doesn't already exist** (skip with note if it does)
3. Add the endpoint to the `UISPEC.md` index table — **only if not already listed** (skip if a row for this endpoint already exists)

### 5. Report

Show the user:
- Number of endpoints discovered
- Files created (list each new file)
- Files skipped (list each with "already exists" note)
- Reminder: "Add the CLAUDE.md snippet to your backend project to enforce sync. See `uispec-backend` skill for the snippet."
