---
name: uispec-frontend
description: "Reads UI specifications from uispec/ in the frontend project to guide component development and API integration. Triggers on: 'check uispec', 'what does the spec say', 'build page for', 'update design', 'new component', or automatically before any API-connected UI work."
---

# UISpec Frontend Skill

Reads `uispec/` in the current frontend project to guide component development, page building, and API integration. This is the consumer side of specs written by the `uispec-backend` skill.

## Structure

```
uispec/
├── openapi.yaml         ← API contract (READ ONLY — backend-owned)
├── UISPEC.md            ← Overview/index
├── design-system.md     ← Tokens, colors, typography (editable)
├── components.md        ← Reusable UI patterns (editable)
└── endpoints/
    └── <endpoint>.md    ← Per-endpoint UI specs (UI sections editable)
```

## Ownership Rules

**Can read (backend-owned, do not modify):**
- `openapi.yaml` — the API contract; changes come from the backend only

**Can read and update (frontend-owned sections):**
- `design-system.md` — design tokens, colors, typography, spacing
- `components.md` — reusable UI component patterns and usage guidelines
- UI Guidelines sections within `endpoints/<endpoint>.md` files (Layout, Components, States, Validation, Accessibility)

**Cannot modify:**
- API sections within endpoint files (Method, Path, Auth, Request, Response) — these are backend-owned
- `openapi.yaml` — backend-owned source of truth

## When to Use

Activate this skill when:
- Building a new page or component that connects to an API endpoint
- The user says "check uispec", "what does the spec say", "build page for [endpoint]"
- The user says "update design", "new component", or references design system tokens
- Before any API-connected UI work — read the endpoint spec first to understand the contract
- When implementing error states, loading states, or form validation — the spec defines expected behavior

## Workflow

### Before Building UI for an Endpoint

1. Read `uispec/endpoints/<endpoint>.md` for the full API contract and UI guidelines
2. Read `uispec/components.md` for reusable patterns that apply
3. Read `uispec/design-system.md` for tokens and styling rules
4. Implement according to the spec:
   - Use the correct request/response shapes from the endpoint file
   - Follow the UI Guidelines section for layout and component choices
   - Handle all documented states (loading, success, error, empty)
   - Apply validation rules specified in the spec
   - Use design system tokens, not hardcoded values

### Updating Frontend-Owned Specs

When the frontend team establishes new patterns:

**Updating `design-system.md`:**
- Add new design tokens as they're established
- Document color palette changes, typography scales, spacing systems
- Keep in sync with the actual CSS/theme implementation

**Updating `components.md`:**
- Add new reusable component patterns as they're built
- Document component props, variants, and usage guidelines
- Reference design system tokens

**Updating UI sections in endpoint files:**
- After building a page, update the UI Guidelines section with actual implementation details
- Document which components were used, specific accessibility attributes added
- Add notes about state management approach

## Rules

1. Always read the endpoint spec before building API-connected UI
2. Never modify `openapi.yaml` — if the API contract is wrong, flag it for the backend team
3. Never modify API sections (Method, Path, Auth, Request, Response) in endpoint files
4. When the spec doesn't cover something, make a reasonable choice and document it in the UI Guidelines section
5. If an endpoint file doesn't exist for an API you need, ask: "No uispec found for this endpoint. Should I request the backend team to create one?"

## CLAUDE.md Snippet

Add this to the frontend project's CLAUDE.md to enforce spec-first UI development. See `assets/CLAUDE-snippet.md` for the full snippet.
