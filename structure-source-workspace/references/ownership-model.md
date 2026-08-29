# Ownership model

Use this reference when deciding where a file or responsibility belongs. Classify by authority and lifecycle, not by which tool happens to read the file.

## Source

`source` is the shared definition of a component and the integration boundary for changes to it.

It normally owns:

- implementation, tests, and component documentation;
- dependency manifests and lockfiles;
- compiler, build, test, lint, and normative formatting configuration;
- reproducible packaging needed to create an environment-neutral component artifact;
- the containerized development toolchain definition and the entry point that runs it;
- contributor guidance and architecture needed to change the component;
- component CI and its supply-chain checks;
- generation of an artifact-bound component SBOM and provenance.

A file does not become personal merely because an IDE consumes it. A formatter or linter configuration belongs in `source` when it defines a convention that contributors and CI must follow.

Source should expose a stable build contract such as `make verify`, `make test`, or `make package`. Personal and delivery automation should invoke that contract rather than reproducing its implementation.

The contract must be self-contained. A contributor may clone `source` alone, so it must build and test with no parameters and no sibling directories present. Any path outside the source tree is a caller-supplied parameter with a working default, never a committed literal — and not an untracked or ignored file inside `source` either, since "not versioned" is weaker than "not present." This is the operational answer to classification question 5, and it applies to every build contract, containerized or not.

## Workbench

`workbench` is a separately versioned personal productivity environment. It may be private even when source is public.

It normally owns:

- IDE or editor workspace and project metadata;
- personal LLM and agent instructions, prompts, skills, and tool connections;
- notes, review checklists, experiments, and scratch material;
- convenience scripts that delegate to the source build contract;
- host prerequisites, IDE remote-toolchain configuration, and the parameter values that point the source build contract at workbench paths;
- personal launch, navigation, and multi-root workspace definitions;
- ignored caches, indexes, and transient state when their location is configurable.

Workbench may depend on source. Source must not require workbench to build, test, review, or understand the component.

## Delivery

`delivery` is shared operational and product-composition policy. It is optional and may be maintained by a different group from the component.

It normally owns:

- selection of exact component artifacts;
- composition of multiple components into a product or distribution;
- runtime and production images, which are distinct from the development toolchain container;
- deployment topology and environment-specific configuration;
- release promotion, signing, rollout, and rollback policy;
- product-level integration and smoke tests;
- product SBOM composition, VEX context, and deployment inventory;
- provider-specific deployment automation.

One source component may feed multiple delivery repositories, and one delivery may consume artifacts from multiple source repositories.

Prefer references to immutable artifacts by digest. Bind each selected artifact to its component SBOM and provenance. Preserve component SBOMs as separately attributable documents and create a product-level SBOM that links them and adds composition relationships; do not flatten inventories blindly and lose provenance or completeness information.

## CI and delivery boundary

CI follows the integration boundary:

- Component CI validates source changes and produces a component artifact, SBOM, and provenance. It invokes the entry point with default parameters, which keeps source self-containment true continuously rather than at review time.
- Composition CI validates a selected set of component artifacts as a coherent product.
- Delivery automation promotes the already identified product composition into environments.

Delivery must not become an alternative location for language-specific compiler flags, dependency resolution, or component tests. If several source trees must be compiled together, make that joint integration boundary explicit rather than hiding it in deployment scripts.

## Classification questions

When ownership is unclear, ask:

1. Is this required to build, test, validate, or understand the component independently?
2. Is this an individual's choice of how to work on the component?
3. Does this select, compose, configure, promote, or operate component artifacts as a product?
4. Which repository must change when this decision changes?
5. Can the source still be built and verified without the other repositories?

The answers normally route the item to `source`, `workbench`, or `delivery` respectively. Split files that mix authorities instead of assigning the whole file by convenience.
