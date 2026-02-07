---
name: specsmith-management
description: "Awareness of the .specsmiths/ directory for multi-spec workflows and .uispec/ for UI specification workflows. Triggers on: 'the plan', 'the spec', 'my plans', 'switch to', 'go back to', 'resume', 'continue where we left off', 'what was I working on', 'show my plans', 'where am I', 'actually let's change', 'we also need', 'I realized we should', 'implement the spec', 'let's build this', 'start implementation', 'execute the plan', 'sync uispec', 'build ui for', 'implement ui', 'check uispec', or when a .specsmiths/ or .uispec/ directory exists. Also triggers when the user describes a feature requiring changes to more than 2 files."
user-invocable: false
---

# Plan Management Awareness

**Do NOT use EnterPlanMode when specsmith is active.** Specsmith IS the planning process — it handles discovery, research, and spec creation. Using plan mode on top of specsmith is redundant and disruptive.

## Trigger Phrases

Activate this skill when the user says anything matching these patterns:

**Implementation triggers:**
- "implement the spec", "implement this", "let's implement"
- "let's build this", "start building", "begin implementation"
- "start implementation", "execute the plan", "execute the spec"
- "let's code this", "start coding", "begin coding"
- "make it happen", "build it", "do it"
- When any of these are detected, invoke `/specsmith-implement`

**Switch/Resume:**
- "go back to [spec name]", "continue where we left off", "switch to [spec name]"
- "resume", "pick up where I left off", "continue the other one"
- "let's work on [spec name] instead"

**Status/Discovery:**
- "what was I working on", "show my plans", "where am I"
- "what specs do I have", "what's the status", "how far along"

**Mid-implementation edits:**
- "actually let's change X", "we also need Y", "I realized we should..."
- "wait, what about...", "can we add...", "I forgot about..."
- These imply an implicit `/specsmith-edit` — update the spec, don't just implement ad-hoc

**Complexity detection:**
- When the user describes a feature that clearly requires changes to more than 2 files, or involves multiple components/layers (API + DB + UI, etc.), proactively suggest: "This is getting complex — want me to create a spec with `/specsmith-new`?"

## Auto-Detection Behavior

Before ANY implementation work, check for `.specsmiths/active.json` in the project root:

1. **If `.specsmiths/active.json` exists and has an active spec:**
   - Read the active spec file to understand current context
   - Verify that the user's current request aligns with the active spec
   - If work aligns: proceed, referencing the spec's current phase and step
   - If work doesn't align: ask the user:
     - "You have an active spec for `<name>`. This seems like different work. Want to: (a) switch to an existing spec, (b) create a new spec for this, or (c) deactivate the current spec and work without one?"

2. **If `.specsmiths/` directory exists but no active spec:**
   - Mention available specs: "You have specs for X, Y, Z but none is active. Want to activate one?"

3. **If no `.specsmiths/` directory exists:**
   - No action needed — the user hasn't started using specs yet

## Context Restoration

When resuming a spec (via explicit `/specsmith-resume` or conversational triggers):

1. Always re-read ALL companion files:
   - `.specsmiths/<n>.md` — the spec itself
   - `.specsmiths/<n>.research.md` — research findings, patterns, decisions
   - `.specsmiths/<n>.questions.md` — discovery Q&A, user requirements, constraints
2. Identify the current phase and first unchecked step
3. Summarize where things stand before proceeding
4. Restore the TodoWrite task list with remaining steps from the current phase

## Proactive Suggestions

When the conversation suggests complexity but no spec exists:
- Multiple components being discussed
- User keeps adding "oh and also..." requirements
- Implementation is spanning multiple files without a clear plan
- User seems uncertain about approach or order of operations

Say: "This is getting complex — want me to create a spec to track this properly? `/specsmith-new <suggested-name>`"

## UISpec Awareness

**Backend project** (`.specsmiths/uispec.json` exists):
- When the user mentions uispec, API contracts, or syncing:
  - Suggest: "Run `/specsmith-uispec-sync` to sync your latest API changes."
- When the user is about to create a new endpoint outside of a spec:
  - Suggest: "Run `/specsmith-uispec-sync` after creating this endpoint to update the frontend's .uispec/."
- During spec implementation, auto-sync happens automatically — no manual reminder needed.

**Frontend project** (`.uispec/` directory exists):
- When the user wants to build UI for an API endpoint:
  - Suggest: "Run `/specsmith-uispec-new <endpoint>` to create a full implementation spec from the API contract."
- When the user asks what needs implementing:
  - Suggest: "Run `/specsmith-uispec-detect` to find gaps between specs and UI."
- When the user asks about API compliance:
  - Suggest: "Run `/specsmith-uispec-validate` to check UI conformance."
- When `.uispec/` exists but the user starts building without reading specs:
  - Remind: "There's a `.uispec/` spec for this endpoint — read it first to ensure the UI matches the API contract."

**No UISpec configured** (no `.specsmiths/uispec.json` and no `.uispec/`):
- No UISpec-related suggestions. Don't nag about UISpec in backend-only projects.
