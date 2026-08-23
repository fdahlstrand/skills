---
name: structure-source-workspace
description: Design, scaffold, or reorganize source-code workspaces that keep shared source and CI, personal development tooling, and optional delivery composition in separately version-controlled areas. Use when establishing repository boundaries or keeping IDE, agent, and deployment choices out of a neutral source tree.
---

# Structure Source Workspace

Create a workspace with explicit ownership boundaries rather than treating one repository as the home for every concern. Preserve the central invariant:

> Personal productivity and delivery metadata may read, edit, build, or consume the source, but must not create configuration entries inside the source tree.

"Not versioned" is weaker than "not present." Ignoring `.idea/`, `.vscode/`, agent files, or similar entries does not satisfy a strict clean-source boundary.

## Establish the context

Inspect the existing directory and Git layout before proposing changes. Preserve unrelated files, repository histories, remotes, and user changes.

Before generating tool-specific files, obtain explicit answers to these questions when they are not already stated by the user:

- Which IDEs or editors must open the workspace?
- Which LLM, agent, or assisted-development tools must operate on it?
- Is this a new workspace or a reorganization of existing repositories?
- Which language, build system, and CI provider does the source use?
- Is delivery/composition in scope now, optional for later, or deliberately excluded?
- Does "clean source" prohibit only personal configuration, or also generated build output and caches?

Ask the user about the target IDE and LLM/agent tooling; do not infer those choices solely from files already present. Combine missing questions into a concise prompt. Do not block a purely conceptual comparison on details that would not affect it.

## Use the ownership model

Default to an unversioned container with sibling repositories:

```text
project/
|-- source/       Shared component repository
|-- workbench/    Personal productivity repository
`-- delivery/     Optional shared delivery/composition repository
```

The container is organizational only. Do not initialize it as another repository. Do not use nesting, submodules, or symlinks unless the user deliberately chooses their coupling and accepts the consequences.

Read [references/ownership-model.md](references/ownership-model.md) when classifying files, designing delivery composition, or explaining CI and SBOM boundaries.

Apply these dependency directions:

- `workbench` may refer to and operate on `source`.
- `delivery` may consume versioned artifacts produced from one or more source repositories.
- `source` must not depend on `workbench` or a particular delivery repository.
- Prefer delivery composition of immutable artifacts over rebuilding several raw source trees together.

Keep CI with the integration boundary it validates. Component CI belongs with component source. Delivery may have its own validation for composition, environment definitions, promotion, and system-level tests; it must not silently redefine the component build.

## Adapt to the selected tools

After the user identifies the tools, determine their current workspace/project discovery behavior. Consult current official documentation when behavior is uncertain or version-sensitive.

For IDEs and editors:

- Prefer a multi-root workspace, external content-root, wrapper-project, or equivalent whose metadata lives under `workbench`.
- Make `source` the working directory for shared build, test, lint, and debug commands when required.
- Keep portable references relative to the workbench where the tool supports them.
- Do not open `source` directly if that causes the tool to create project metadata there.

For LLM and agent tools:

- Store personal prompts, instructions, skills, tool connections, and agent state under `workbench`.
- Configure the tool to operate on `source` from outside it using supported workspace roots, launch options, or external configuration locations.
- Do not put an instruction file or symlink in `source` merely because a tool discovers configuration only by walking the source tree.

If a selected tool cannot operate without writing metadata inside `source`, explain the limitation and present alternatives. Ask the user to choose before making a compromise. Do not silently weaken the boundary.

Redirect personal caches and transient state to an ignored location such as `workbench/state/` when supported. Treat build-system output separately: it may be an intentional property of the shared build contract, but redirect it too when the user requires a literally pristine checkout.

## Plan and implement

Before moving or creating files, show the proposed tree and a short ownership map. Resolve ambiguous files by purpose rather than filename alone. For example, a formatter configuration belongs in `source` when it defines a shared convention, even if an IDE consumes it.

When authorized to implement:

1. Create only the repositories and directories in scope.
2. Preserve existing Git histories instead of copying tracked files into fresh repositories.
3. Put shared build, test, lint, packaging, contributor documentation, and component CI definitions in `source`.
4. Put selected IDE and LLM/agent workspace definitions, personal notes, and convenience wrappers in `workbench`.
5. Put artifact selection, composition, environment, promotion, deployment, and product-level supply-chain definitions in `delivery` when requested.
6. Make personal wrappers invoke the canonical commands exposed by `source`; do not duplicate compiler or linter policy in `workbench`.
7. Avoid adding a catalogue of personal tools to source `.gitignore` or `.git/info/exclude` as a substitute for the physical boundary.

Do not configure remotes, publish repositories, or move destructive targets unless those actions are explicitly within scope. Stop and ask before any migration whose repository history or ownership is unclear.

## Verify the result

Verify observable boundaries rather than only checking committed files:

- Confirm that `source`, `workbench`, and optional `delivery` resolve to the intended independent Git roots.
- Confirm that the outer container is not an additional repository created by this setup.
- Compare the source filesystem before and after opening each selected IDE or agent tool; check tracked, untracked, and ignored additions.
- Confirm that workspace tasks invoke the shared build contract from the correct source working directory.
- Confirm that personal configuration and state remain under `workbench`.
- If delivery is present, confirm that it pins identifiable artifacts and preserves links to their SBOMs or provenance rather than depending on mutable source state.

Report the final tree, repository ownership, opening commands for the selected tools, verification performed, and any tool limitation that could still write into `source`.
