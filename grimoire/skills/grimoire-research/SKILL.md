---
name: grimoire-research
description: "Research orchestration — picks agents, runs parallel research, synthesizes findings. Actions: research (full parallel), lookup (single stream), streams (list available). Triggers on: 'research this', 'look up docs for', 'find best practices', 'analyze the codebase', 'check git history', 'analyze the schema', 'check the API docs', or when invoked via /grimoire command."
user-invocable: false
---

# Grimoire Research

This skill orchestrates the grimoire agent suite — dispatching to specialized agents for focused research and synthesizing their findings.

## Action: streams

List available research streams. Display this table:

```
Grimoire Research Suite

  Just describe what you need — grimoire picks the right agents automatically.

  /grimoire add user authentication     Auto-dispatches to relevant agents
  /grimoire how does Stripe Connect work  Auto-dispatches to relevant agents

  For a quick single-stream lookup, prefix with the stream name:

  /grimoire docs react hooks            Library docs only
  /grimoire practices JWT rotation      Best practices only
  /grimoire codebase auth system        Codebase analysis only
  /grimoire history websocket migration Git history only
  /grimoire schema users table          DB schema only
  /grimoire contracts stripe API        External API docs only
```

---

## Action: lookup

Single-stream invocation. Dispatch to one specific agent.

**Input:** stream name + query

### Dispatch

1. Identify the target agent from the stream name:

   | Stream | Agent | subagent_type |
   |--------|-------|---------------|
   | `docs` | grimoire-docs | general-purpose |
   | `practices` | grimoire-practices | general-purpose |
   | `codebase` | grimoire-codebase | Explore |
   | `history` | grimoire-history | Explore |
   | `schema` | grimoire-schema | Explore |
   | `contracts` | grimoire-contracts | general-purpose |

2. Spawn the agent via the Task tool:
   - Read the agent's markdown file from the grimoire plugin directory: `${CLAUDE_PLUGIN_ROOT}/agents/grimoire-<stream>.md`
   - Use the agent's content as the system prompt, appending the user's query as research context
   - Use the appropriate `subagent_type` from the table above

   Example Task tool invocation for `grimoire-codebase`:
   ```
   Task tool:
     subagent_type: Explore
     description: "grimoire-codebase research"
     prompt: "<contents of ${CLAUDE_PLUGIN_ROOT}/agents/grimoire-codebase.md>

     ## Research Context
     <the user's query>"
   ```

3. Return the agent's findings directly to the user

---

## Action: research

Full parallel research. Analyze the context and spawn all relevant agents simultaneously.

**Input:** research context (what the user wants to build/change/investigate)

### Step 1: Analyze Context

Read the research context and determine which agents to spawn. Use these signals:

| Signal | Agent to Spawn |
|--------|----------------|
| Always | **grimoire-codebase** — every research needs codebase context |
| New/unfamiliar dependencies mentioned | **grimoire-docs** |
| Security-sensitive feature (auth, crypto, PII, payments) | **grimoire-practices** |
| Unfamiliar domain or novel architecture | **grimoire-practices** |
| Legacy system, bug fix, or "why does this work this way" | **grimoire-history** |
| Database tables, columns, migrations, queries | **grimoire-schema** |
| External API, third-party service, webhooks | **grimoire-contracts** |

Minimum: 1 agent (codebase). Maximum: all 6.

**Runebook context enrichment** (if runebook is installed):
- Check if `.runebook/` exists at the project root
- If yes:
  - From the research context, identify component names, file paths, or domain areas
    that might match runebook entries
  - Search `.runebook/` for matching entries by name, tags, or source_files
  - Read up to 5 most relevant entries (prioritize entries whose `source_files`
    overlap with the research area)
  - Check if any guide covers this domain (grep `guides/*.md` for matching domain/tags)
  - Append to the research context passed to each agent:
    ```
    ## Existing Documentation (from .runebook/)
    <For each entry: type, name, summary, dependencies, most recent changelog entry>
    <If a guide matched: the guide's Overview and How It Works sections>
    ```
  - This gives agents pre-existing knowledge about the components,
    reducing redundant scanning and improving research quality
- If `.runebook/` does NOT exist:
  - Skip silently — proceed without runebook enrichment

### Step 2: Spawn Agents in Parallel

For each selected agent, spawn via the Task tool simultaneously:

```
For each agent:
  1. Read the agent file: ${CLAUDE_PLUGIN_ROOT}/agents/grimoire-<name>.md
  2. Spawn with Task tool:
     - subagent_type: (per dispatch table above)
     - prompt: Include the agent's full content as system instructions,
               then append "## Research Context\n<the research context>"
     - description: "grimoire-<name> research"
```

**All agents run in parallel** — spawn them all in a single message with multiple Task tool calls.

### Step 3: Synthesize Findings

After all agents complete, combine their findings into a structured research report:

```markdown
# Grimoire Research: <context summary>

## Agents Invoked
- <agent>: <one-line summary of what was researched>

## Codebase Analysis
<findings from grimoire-codebase>

## Library Documentation
<findings from grimoire-docs, if invoked>

## Best Practices & Pitfalls
<findings from grimoire-practices, if invoked>

## Historical Context
<findings from grimoire-history, if invoked>

## Schema Analysis
<findings from grimoire-schema, if invoked>

## API Contracts
<findings from grimoire-contracts, if invoked>

## Key Decisions
For each decision point surfaced by the research:
- **Decision:** <what needs deciding>
- **Options:** <what was found>
- **Recommendation:** <which option and why>
- **Trade-offs:** <what you give up>

## Research Gaps
- <areas where research was incomplete or unavailable>
```

Present the synthesis to the user. If invoked by specsmith, write the synthesis to the spec's research file.

---

## Integration with Specsmith

When invoked during specsmith Phase 2:

1. Receive the discovery findings as research context
2. Run the full `research` action
3. Write the synthesis to `.specsmiths/<name>.research.md`
4. Return findings to specsmith for Phase 3 (spec creation)

The grimoire agents map directly to specsmith's research streams:

| Specsmith Stream | Grimoire Agent |
|------------------|----------------|
| Codebase Analysis | grimoire-codebase |
| Library Docs & Dependencies | grimoire-docs |
| Best Practices & Pitfalls | grimoire-practices |
| History & Existing Issues | grimoire-history |
| Database Schema Analysis | grimoire-schema |
| API Contract Review | grimoire-contracts |
