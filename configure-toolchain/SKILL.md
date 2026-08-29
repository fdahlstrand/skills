---
name: configure-toolchain
description: Configure or migrate a project's development toolchain so project-specific executable tooling — compiler, runtime, package manager, build system, linters, test runners — runs in a project-owned, minimally privileged rootless Podman container instead of being installed on the developer host. Owns the containerized build contract, the least-privilege container profile, credential isolation from dependency-supplied code, and the split between ordinary, privileged, and secret-bearing execution modes. Use when setting up a development environment, adding a Node, TypeScript, C, C++, Rust, Go, Zig, Python or Java toolchain, configuring IDE integration against a containerized toolchain, or improving development-time supply-chain isolation — or when the user says "containerize the toolchain", "set up a dev container", "run the build in Podman", or "stop installing this on my host". Do NOT use this to decide which repository a file belongs to — that is the structure-source-workspace skill.
---

# Configure Toolchain

Configure the project so that the **project owns its executable development toolchain** and that toolchain runs inside a minimally privileged container.

The container is both:

1. a reproducible project environment; and
2. a security boundary around build-time and dependency-supplied code.

Do not treat containerization as an IDE-specific feature. The command-line project workflow is authoritative; IDEs and coding agents are clients of that workflow.

Requires a repository or project directory, and rootless Podman on the developer host. Do not install or reconfigure host software without explicit approval.

## Core policy

Apply these rules unless the project has a documented reason to deviate:

- Keep developer-facing workstation tools on the host: IDE/editor, Git client, terminal, Podman, credential manager, SSH client, browser, and similar personal tools.
- Put project-specific executable tooling in the project container: compiler/runtime, package manager, dependency resolver, build system, code generators, formatter, linter, test runner, and project-specific CLI tools.
- Prefer rootless Podman.
- Do not expose the host home directory to the toolchain container.
- Do not expose credential directories, credential-agent sockets, cloud credentials, package-publishing credentials, or the Podman/Docker socket.
- Bind-mount only the project working tree and explicitly required files or devices.
- Use named volumes for dependency caches and build caches where practical.
- Run the toolchain as a non-root user and preserve usable ownership of files written to the working tree.
- Drop Linux capabilities unless one is demonstrably required.
- Enable `no-new-privileges`.
- Prefer a read-only container root filesystem with writable `tmpfs` locations and explicit writable volumes.
- Give the toolchain network access only when the workflow needs it. Prefer offline build/test/lint operations when practical.
- Never bake secrets into an image, Containerfile, Compose file, build argument, repository file, or general container environment.
- Treat container definitions, dependency policy files, and toolchain wrappers as security-sensitive source code.
- Keep privileged debugging, hardware access, cloud access, or runtime secrets out of the default toolchain profile. Add narrowly scoped opt-in profiles when genuinely required.

## Workspace boundaries

This skill decides *where code executes and with what privilege*. It does not decide *which repository owns a file*. That is a separate axis, owned by the `structure-source-workspace` skill, which separates shared component `source` from personal `workbench` and optional `delivery`.

The two axes are orthogonal and must both be satisfied. The containerized toolchain **is** the shared build contract, so it is source-owned; everything tool-specific about consuming it is workbench; everything about shipping artifacts is delivery.

If the workspace separates ownership areas, place artifacts as follows:

| Artifact | Area |
| --- | --- |
| `Containerfile.dev`, `compose.dev.yaml`, pinned versions, dependency policy, lockfiles | `source` |
| The entry point (`./dev` or the existing task runner) — the build contract | `source` |
| Component CI invoking the same entry point | `source` |
| Toolchain caches — Cargo registry, `GOMODCACHE`/`GOCACHE`, npm/pnpm store, Maven/Gradle repository, ccache | named volumes, outside every tree. The default |
| A toolchain cache a host-side IDE or language server must read | the exception. Bind-mount a path the caller supplies, which the workbench sets to `workbench/state/`. Source still defaults to the named volume |
| The IDE's or agent's own indexes and session state | `workbench` — never enters the container |
| IDE remote-toolchain configuration, multi-root workspace, `.idea`, `.vscode`, `.devcontainer` | `workbench` |
| Personal wrappers that call the entry point | `workbench` |
| Host prerequisites — Podman, udev rules, GPU runtime | the host, documented in `source` |
| Production/runtime image, composition, deployment | `delivery`. Out of scope for this skill |

In a single-repository project everything above lands in that repository. The privilege rules are unchanged.

## Goal state

Aim for this conceptual boundary:

```text
Developer host
├── IDE/editor
├── Git
├── rootless Podman
├── SSH and credential agents
├── cloud credentials
└── personal configuration
        │
        │ project source only
        ▼
Project toolchain container
├── compiler/runtime
├── package manager
├── dependency resolver
├── build/test/lint/format tools
├── project working tree
├── isolated caches
└── temporary files

Not visible by default:
  host $HOME
  ~/.ssh
  ~/.aws
  ~/.azure
  ~/.config/gh and similar credential stores
  SSH_AUTH_SOCK
  package publishing tokens
  cloud credentials
  Podman/Docker socket
```

## Workflow

### 1. Inspect before changing

Determine:

- languages used by the repository;
- current toolchain and versions;
- package/dependency managers;
- build, test, lint, format, code-generation, and debug commands;
- existing Containerfiles, Dockerfiles, Compose files, dev-container files, Makefiles, task runners, shell wrappers, and CI configuration;
- files that pin tool versions, such as `package.json`, lockfiles, `rust-toolchain.toml`, `go.mod`, `CMakePresets.json`, or equivalent;
- existing IDE-specific configuration;
- whether the project needs private dependencies, hardware devices, debuggers, GPUs, privileged networking, or cloud services;
- whether the workspace separates ownership areas, and if so which directory is `source`.

Preserve useful existing conventions. Do not replace a working project abstraction merely to impose preferred filenames.

Do not inspect, print, copy, or expose host secrets as part of discovery.

### 2. Establish the project-owned entry point

The repository should have a small, obvious command-line entry point for development operations. Prefer an existing task runner if one is already established. Otherwise create a lightweight wrapper such as `./dev`.

This entry point **is the source-owned build contract** — the stable interface that personal and delivery automation invoke rather than reproducing. Make common operations discoverable, for example:

```text
./dev shell
./dev install       # or deps
./dev build
./dev test
./dev check
./dev lint
./dev format
./dev run
```

Only add commands that make sense for the project.

The wrapper must execute project tooling through Podman rather than invoking a host compiler, runtime, or package manager.

Keep the wrapper thin. Business logic and complex build logic belong in the project's normal build system, not in the wrapper.

#### The contract must be self-contained

A contributor may clone `source` alone, with no workspace around it. Therefore:

- The entry point with no parameters must build and test a bare clone — named volumes only, the working tree as the sole bind mount, and no path outside the tree anywhere in the committed files.
- Extend through a small, documented parameter surface instead. An environment variable for an extra cache path, and one for extra Compose files, is usually enough:

  ```text
  podman compose -f compose.dev.yaml ${DEV_COMPOSE_EXTRA:+-f "$DEV_COMPOSE_EXTRA"} ...
  ```

  Source documents the parameter *names*. It never presumes a sibling layout and never names `workbench`.
- The override file itself lives in `workbench` — for example `workbench/compose.workbench.yaml` — and the personal wrapper passes it in. That is the permitted workbench-to-source direction; the reverse never appears.
- Do not place the override inside `source` as an untracked or git-ignored file. "Not versioned" is weaker than "not present."
- Component CI calls the unparameterized default, which keeps self-containment true continuously rather than at review time.

### 3. Define the toolchain container

If no suitable project container exists, create a project-owned container definition. Prefer neutral names such as:

```text
Containerfile.dev
compose.dev.yaml
```

unless the repository already has a better convention.

Pin deliberate toolchain versions. Avoid `latest` for compilers, runtimes, package managers, and other critical tools. If pinning an image digest, make the update process explicit so security patches are not accidentally frozen forever.

Install only the packages needed by the development toolchain.

Do not put project or user secrets in the image.

### 4. Apply minimum privilege

Configure and validate, where supported:

- rootless Podman;
- non-root process execution;
- host UID/GID preservation or another ownership-safe mapping;
- all capabilities dropped by default;
- `no-new-privileges`;
- read-only root filesystem;
- `tmpfs` for `/tmp` and other ephemeral writable paths;
- no privileged mode;
- no host PID namespace;
- no host network namespace unless specifically required;
- no Podman/Docker socket;
- no host credential or configuration directories;
- no SSH agent socket;
- no arbitrary host-device access.

Do not weaken the default container merely to make debugging or integration easier. Add a separate, explicit profile for elevated capabilities such as `SYS_PTRACE`, USB/JTAG, GPU devices, or special networking.

### 5. Mount the minimum filesystem surface

The source working tree normally needs to be writable for development. Bind-mount it explicitly.

Do not mount the developer's home directory.

Prefer named volumes for:

- compiler/build caches;
- downloaded dependency caches;
- generated dependency trees when host access is unnecessary;
- other disposable, potentially large toolchain state.

Do not persist sensitive credentials in cache volumes.

If an IDE must inspect generated artifacts or dependencies, expose only the specific paths required and document why. Named volumes stay the default for every cache no host tool reads. An exposed path is a caller-supplied parameter that the workbench points at `workbench/state/` — never a literal outside-the-tree path committed in `source`.

### 6. Isolate credentials

Assume dependency installation, build scripts, procedural macros, code generators, plugins, compiler extensions, and tests may execute third-party code.

The default toolchain container must not receive:

- `NPM_TOKEN`, package-publishing tokens, or equivalent;
- GitHub/GitLab write tokens;
- AWS, Azure, GCP, or other cloud credentials;
- production secrets;
- `~/.ssh` or other private keys;
- `SSH_AUTH_SOCK`;
- desktop keyring or credential-agent sockets;
- Git credential stores;
- the Podman/Docker socket.

If private dependencies require authentication, prefer this order:

1. short-lived credentials;
2. read-only dependency-download scope;
3. secret/file mechanisms rather than general environment inheritance;
4. a fetch phase in which dependency-supplied executable hooks are disabled, when the ecosystem supports it;
5. removal of the credential before build/test execution.

If a package manager necessarily exposes a credential to code executed during dependency installation, document that residual risk. Never substitute a publishing-capable token when read-only access is sufficient.

### 7. Harden the ecosystem itself

Use the language/package-manager security mechanisms available to the project in addition to container isolation. Examples include:

- committed lockfiles or equivalent immutable dependency descriptions;
- install/build-script allowlists where supported;
- dependency provenance/signature verification where useful;
- delayed adoption of newly published packages where supported;
- checksum/sum databases where supported;
- explicit dependency update review;
- no arbitrary Git or URL dependencies unless intentionally approved.

Do not assume vulnerability scanners detect malicious-but-new packages. The container boundary must remain useful even when a dependency is actively malicious.

This skill contains the risk; it does not price it. The per-dependency evaluation — liability, security posture, licence, and the residual risks accepted by name — belongs in the Development view's External Dependencies section, owned by the `architecture-description` skill. Record residual risk there rather than leaving it in a commit message.

Read `references/language-guidance.md` when language-specific decisions are needed.

### 8. Separate ordinary, privileged, and secret-bearing execution

Prefer distinct modes rather than one all-powerful development container:

```text
toolchain/default
  build, dependency resolution, tests, lint, format
  no host credentials
  minimum privileges

 debug (optional)
  only additional debugger capability required
  still no unrelated credentials

 hardware (optional)
  only specifically required devices

 app-with-dev-secrets (optional)
  only narrowly scoped development credentials needed by the running app
  do not use for dependency installation
```

The default mode must remain the safest and most common path.

### 9. Make IDE integration secondary

After the CLI workflow works, configure the relevant IDE to use the same containerized environment if the IDE supports it.

Prefer remote/container compilers, runtimes, interpreters, package managers, language servers, test runners, and debuggers over parallel host installations.

Where the workspace separates ownership areas, IDE configuration is personal and belongs in `workbench`. Do not create `.idea/`, `.vscode/`, or `.devcontainer/` inside `source`. The split is by authority, not by filename: `Containerfile.dev` and `compose.dev.yaml` define the shared environment and stay in `source`; a `devcontainer.json` that references them is a tool-specific consumer and belongs in `workbench`. In a single-repository project all of it lands in that repository, and only the privilege rules below apply.

If a tool can only discover its configuration by walking the source tree, explain the limitation, present the alternatives, and ask the user to choose. Do not silently weaken the boundary. If IDE integration requires weakening the security model, prefer a minimal, documented exception over broad host exposure.

### 10. Document the rule for coding agents

The toolchain rule has two halves.

The **contract** — build through the project entry point, do not install host toolchains, the container is a security boundary — is contributor guidance needed to change the component. Merge `snippets/source-toolchain-contract.md` into the project's existing `AGENTS.md` or `CONTRIBUTING.md`.

The **personal agent instruction** that points a particular tool at that contract is personal configuration. Use `snippets/workbench-agent-instructions.md`.

Where the workspace separates ownership areas, the contract belongs in `source` and the personal instruction in `workbench`; do not create a new agent-instruction file in `source` merely because a tool looks for one there. In a single-repository project both halves merge into that repository's existing instructions.

Merge into existing project instructions rather than replacing unrelated guidance.

### 11. Validate the setup

Read `references/verification-checklist.md` when validating a configured workspace, and work through it before declaring the configuration complete.

Use safe negative tests such as checking for path existence or variable names. Never print secret values.

### 12. Report what changed

Summarize:

- which files were added or changed, and in which ownership area;
- the selected toolchain/runtime versions;
- the normal `./dev` or equivalent commands, and any parameters a workbench may supply;
- caches/volumes created;
- security boundaries applied;
- any deliberate exceptions or residual risks;
- any host prerequisites that remain for the developer to install or configure.

## Decision rules

Several choices here are hard to reverse — whether to containerize at all, Podman versus another runtime, the version-pinning strategy, accepting residual credential exposure during dependency installation, and each privileged profile added. When one is settled by a real trade-off, record it as an ADR via the `adr` skill rather than leaving the reasoning in this session.

### Existing host toolchain

Do not require its removal. Configure the project so normal project commands do not depend on it.

### Existing Docker-based environment

Do not mechanically rewrite it for Podman. First determine whether it already works safely with rootless Podman. Prefer compatible OCI/Compose definitions over Podman-only constructs when they do not weaken the model.

### Existing dev container

Reuse useful container definitions where practical, but keep a CLI workflow independent of a particular IDE or agent.

### Cross-platform repositories

The project may support Podman on Linux while contributors use another OCI runtime elsewhere. Keep project semantics portable when possible, but do not weaken the user's requested security boundary merely for theoretical portability.

### Hardware- or kernel-coupled work

For kernel modules, eBPF, embedded/JTAG, GPU tooling, device access, system integration, performance counters, or similar work, containerize what remains useful and expose only the required integration points. Document that the container is a reduced boundary in those modes.

### CI

Reuse the same build commands and project abstractions where practical. Do not assume the local Podman wrapper must itself be used in CI; CI may already provide an isolated environment. The build semantics should remain consistent.

The development toolchain container is not the runtime image. Component CI belongs with `source` and reuses the same entry point. Production images, composition, promotion, and product SBOMs belong to `delivery`.

## Anti-patterns

Do not:

- install project compilers/runtimes/package managers on the host as the default solution;
- mount `$HOME` for convenience;
- forward every host environment variable into the container;
- mount `SSH_AUTH_SOCK` by default;
- mount the Podman/Docker socket into the toolchain container;
- run the default development container privileged;
- add `SYS_ADMIN` or broad capabilities to solve ordinary permission problems;
- store secrets in `.env` files committed to the repository;
- put publishing credentials in the ordinary dependency-install environment;
- use an IDE-specific container definition as the only executable project workflow;
- hard-code a path outside the source tree in a committed source file;
- silently change or delete an existing working development workflow without explaining why;
- claim the container makes untrusted code safe. It reduces accessible assets and privileges; it does not establish perfect isolation.

## Relationship to other skills

- **`structure-source-workspace`** owns the ownership axis — which repository a file belongs to, and the self-containment rule this skill's entry point must satisfy. It decides *where a file lives*; this skill decides *where code runs*. Consult it whenever the workspace has separate ownership areas.
- **`architecture-description`** owns the per-dependency evaluation record in the Development view's External Dependencies section. That record prices the supply-chain risk; the container configured here bounds it.
- **`adr`** records the hard-to-reverse choices listed under Decision rules.

## Success criteria

The configuration is successful when a new developer or coding agent can clone the repository, satisfy the small set of documented host prerequisites, and use one project-owned interface to build and test the code without installing the project's language toolchain on the host, while the ordinary toolchain environment cannot see the developer's unrelated credentials or host configuration.
