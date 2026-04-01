---
name: task-planner
description: >
  Systematically interview the user about a feature to resolve ambiguities,
  then generate a tasks.yaml and qa.md documenting all decisions.
  Triggers: /task-planner, plan tasks, create task breakdown, flesh out tasks,
  grill feature
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Agent
---

# Task Planner

Structured interview process that resolves design ambiguities one question at a time, then produces actionable task artifacts.

## Inputs

Parse `$ARGUMENTS` for:
- **`<file-path>`** (optional) — path to an issue doc, PRD, NOW.md entry, or rough notes
- **`<feature description>`** (optional) — free text describing the feature

If neither is provided, ask the user what feature to plan.

## Workflow

### Phase 1 — Intake

1. Read the input file/description thoroughly
2. Read `CLAUDE.md` for project context and conventions
3. Explore relevant existing code — components, services, schemas — so questions are informed, not generic
4. Identify **feature areas** (groups of related decisions) and **open questions** within each
5. Tell the user how many feature areas and rough question count you've identified

### Phase 2 — Grilling

Interview the user following these rules:

- **One question at a time** — wait for the answer before asking the next
- **Number sequentially**: Q1, Q2, Q3... across the entire session
- **Group by feature area** — announce the area when switching
- **Walk the design tree depth-first** — resolve foundational decisions before asking dependent ones
- **Read code before asking** — e.g., read an existing component before asking how to modify it
- **Propose options when useful** — "Option A: ..., Option B: ..." with trade-offs when the user is likely unsure or when choices have non-obvious consequences
- **Flag dependencies** — if an answer changes the scope of future questions, say so
- **Stay concrete** — reference actual file paths, component names, DB columns
- **Skip what's already decided** — if the input file answers a question, don't re-ask it

Question types to cover per feature area:
- **Scope**: What's in/out? Edge cases?
- **Data model**: New tables/columns? Relationships? Migrations?
- **UX flow**: Entry points, navigation, mobile vs desktop, empty states
- **Architecture**: Which layer owns this logic? Service vs domain vs UI?
- **Dependencies**: What existing code is affected? What must be built first?
- **Uncertainty**: Flag areas needing spikes or deeper analysis

### Phase 3 — Build Order

After all questions are resolved:
1. Summarize the feature areas and task count
2. Ask the user for their preferred **build order / priority** (or propose one with rationale)
3. Confirm the output directory path (default: `docs/issues/<feature-name>/`)

### Phase 4 — Generate Artifacts

Produce two files. See [REFERENCE.md](REFERENCE.md) for format specifications.

**`tasks.yaml`** — Task breakdown with:
- Phases grouping related tasks
- Task IDs (T1.1, T1.2, T2.1...), names, descriptions
- `depends_on` arrays forming an explicit dependency graph
- `estimated_complexity`: small / medium / large
- `output` listing affected file paths
- `status: pending` for all tasks
- Build order documented in the header comment

**`qa.md`** — Q&A decisions log with:
- Grouped by feature area with markdown headers
- Each Q&A as: `**Q1: question text?**` followed by answer with dash-prefixed bullets
- Build order as the final section

## References

- **Output formats**: [REFERENCE.md](REFERENCE.md)
