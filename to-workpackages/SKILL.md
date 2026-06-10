---
name: to-workpackages
description: Generate detailed Work Package files from an existing Work Plan in the project's .scratch/<workplan-slug>/ folder. Each Work Package is a cohesive, single-sitting, non-trivial implementation task with an acceptance criterion and a task/subtask breakdown. Supports generating all packages at once (bulk) or a single named/next package. Hard-blocks any package affected by an unresolved Open Question rather than fabricating the answer. Use this whenever the user wants to flesh out, detail, or generate the work packages for a work plan — e.g. "generate the work packages", "detail WP-3", "write up the next work package", or "expand the work plan into packages". Do NOT use this to create the Work Plan itself — that is the to-workplan skill.
---

# to-workpackages

Take an existing **Work Plan** (in `.scratch/`) and produce detailed **Work Package** files — one file per package — containing the implementation plan that a developer (human or agent) executes in a single sitting.

## Inputs

- A Work Plan file, typically `.scratch/<workplan-slug>/workplan.md`. If the user doesn't name one and several exist, ask which. If exactly one exists, use it.
- Optionally, a target: a specific package (`detail WP-3`), "the next ready one", or "all" (bulk). Default to **all** packages that aren't blocked (see Open Questions handling).

## Core principle: synthesize from the plan + context + codebase, never fabricate

Build each package's detail from the Work Plan, the conversation/context, and the codebase. Read the relevant code before writing tasks that touch it — reference real modules, functions, and interfaces, not guessed ones.

Where a needed fact is unavailable from all three sources, do **not** invent it.

## Open Questions: hard-block affected packages

The Work Plan has an **Open Questions** section with `OQ-n` IDs, and packages may be annotated `blocked by OQ-n`.

- For each package you're asked to generate, determine whether an unresolved Open Question affects it — either it's explicitly annotated `blocked by OQ-n`, or detailing it correctly would require answering an open question.
- If so, **hard-block that package**: do not generate its file, and do not guess past the gap. Report which package is blocked by which OQ.
- In **bulk** mode, generate every package that is *not* blocked, and skip the blocked ones with a clear report. Don't make it all-or-nothing — deliver what can be delivered safely.

## Where files go

Write each package alongside its Work Plan, in the same plan folder:

```
.scratch/<workplan-slug>/
├── workplan.md
├── wp1-<slug>.md
├── wp2-<slug>.md
└── ...
```

Name each file `wp<id>-<slug>.md` (e.g. `wp3-parse-expressions.md`), where `<slug>` derives from the package name in the plan. By convention the Work Plan's list entry maps to this filename; **do not write back to the Work Plan** to register the file — the naming convention is the link, and the plan stays stable as a single-writer coordination surface.

## What a Work Package owns (and doesn't)

A Work Package file owns the **implementation detail** of one package:
- the user story or scope statement
- the acceptance criterion
- the task/subtask breakdown

It does **not** own its dependencies or its status — those live only in the Work Plan. Don't restate `deps:` or `[ ]/[~]/[x]` inside the package file; that would create two sources of truth that drift.

## Work Package structure

Use this template:

```markdown
# WP-<id>: <Package Name>

> Part of Work Plan: <workplan-slug> (see workplan.md in this folder). Transient planning artifact.

## Scope

<Prefer a user story: "As a <role>, I want <capability>, so that <benefit>." If this
package is architectural runway with no standalone user value, instead give a short
bounded-scope statement explaining what it builds and which later package(s) consume
it. Keep scope tight — this is one cohesive, single-sitting unit.>

## Acceptance Criterion

<The observable state that proves this package delivered its value. Concrete and
checkable — e.g. "`archie validate` exits 0 on testdata/sample.arch", "the evaluator
returns 42 for (+ 40 2)", "Spec §4.2 updated and reviewed". This is the package's
done-gate, distinct from completing every task.>

## Tasks

<Ordered, concrete steps to implement the package, sized so the whole package fits one
sitting. Use subtasks where a step has meaningful internal structure. Reference real
code (files, functions, types) discovered in the codebase. Each task should be
actionable without further design work — if a task would require answering an open
question, the package is blocked and should not have been generated.>

- [ ] Task 1: <...>
  - [ ] Subtask 1a: <...>
- [ ] Task 2: <...>

## Notes

<Optional: gotchas, pointers to relevant ADRs or to other packages. Lightweight and
transient. Durable decisions with rationale belong in ADRs, not here.>
```

(Task checkboxes here are an in-file aid for the developer executing the single sitting; the *package-level* status still lives in the Work Plan.)

## Sizing reminder

Each package must stay **cohesive** (top priority), **single-sitting**, and **non-trivial**. If, while detailing a package, you discover it genuinely cannot fit one sitting, don't silently produce a bloated package — flag it to the user and suggest the Work Plan split it (this is a planning issue that belongs back in `to-workplan`, since the plan owns the package list).

## After writing

List the files created, and **explicitly report any packages skipped due to blocking Open Questions**, naming the OQ for each. If blocked packages remain, remind the user those OQs need resolving (e.g. another grill-me/design pass, then updating the Work Plan) before those packages can be generated.
