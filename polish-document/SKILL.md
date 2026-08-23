---
name: polish-document
description: Rewrite or review one document against a named prose ruleset. Use when the user asks to polish, clean up, or unslop a document, wants STE rules applied to technical documentation, or mentions polish-document.
---

# Polish document

Apply one prose ruleset to one document.

## Workflow

1. Determine the ruleset. See below.
2. Tell the user which ruleset you picked and why, in one line.
3. Read the matching file in `rulesets/`. Read one file. Do not read the others.
4. Determine the mode. Review unless the user asks for a rewrite.
5. Act.
6. Report.

## Available rulesets

| Ruleset  | File                    | For                                              |
| -------- | ----------------------- | ------------------------------------------------ |
| `ste`    | `rulesets/ste.md`       | Documents the reader consults                    |
| `unslop` | `rulesets/unslop.md`    | Documents the reader receives                    |

Ask whether the reader consults the document or receives it.

Consulted, use `ste`: ADRs, architecture descriptions, glossaries, roadmaps, context documents, commit messages, CONTRIBUTING.md.
Received, use `unslop`: email, blog posts, pull request descriptions, release notes, README.md.

A ruleset named by the user overrides this inference.
If the document fits neither list and the test does not settle it, ask. Do not guess.

## Modes

Review is the default. Rewrite happens on request.

Review reports the changes it would make and edits nothing. Use it to get a feel for the result before committing to it.
Rewrite applies the mechanical rules to the file, then reports the judgement calls.

## Mechanical rules and judgement rules

Every ruleset file splits its rules in two.

Apply mechanical rules directly in rewrite mode. They have one correct output and they do not change what the document says.

Never apply judgement rules directly. Report them in both modes, with the location, the problem, and a proposed replacement. A wrong guess on these changes the technical content rather than the prose.

## Scope

These limits hold for every ruleset.

Edit prose only.
Leave code blocks, identifiers, log strings, error messages, config, test data, frontmatter, link targets, and text quoted from other sources unchanged.
Leave structural markers alone. `[!OPEN]`, `WIP`, `TODO(review):` and any other marker a tool or skill greps for are content, not prose.
If a skill or template defines the document's sections, keep them.
Polish one document per invocation.

## Report format

Open with the ruleset, the mode, and why.
Report mechanical changes as a count per rule, not one line per instance.
Report judgement calls one per line, with location and proposal.
Close with anything you left alone because a rule and the document's accuracy conflicted.

## Conflicts

If a rule and the document's accuracy conflict, keep the document accurate and report the conflict.
Direct instructions from the user override this skill.

## Adding a ruleset

Add a file to `rulesets/`, then add a row to the table above and a line to the selection test. Each ruleset file is self-contained and repeats any rule it shares with another ruleset. Do not factor shared rules into a common file, because a later ruleset may reject them.
