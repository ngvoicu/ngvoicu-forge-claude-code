---
name: specsmith-management
description: "Awareness of the .specsmiths/ directory for multi-spec workflows. Triggers on: 'the plan', 'the spec', 'my plans', 'switch to', 'go back to', 'resume', 'continue where we left off', 'what was I working on', 'show my plans', 'where am I', 'actually let's change', 'we also need', 'I realized we should', or when a .specsmiths/ directory exists. Also triggers when the user describes a feature requiring changes to more than 2 files."
---

# Plan Management Awareness

## Trigger Phrases

Activate this skill when the user says anything matching these patterns:

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
- These imply an implicit `/specsmith edit` — update the spec, don't just implement ad-hoc

**Complexity detection:**
- When the user describes a feature that clearly requires changes to more than 2 files, or involves multiple components/layers (API + DB + UI, etc.), proactively suggest: "This is getting complex — want me to create a spec with `/specsmith new`?"

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

When resuming a spec (via explicit `/specsmith resume` or conversational triggers):

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

Say: "This is getting complex — want me to create a spec to track this properly? `/specsmith new <suggested-name>`"
