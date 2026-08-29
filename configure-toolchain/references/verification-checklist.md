# Verification checklist

Work through this before declaring a toolchain configuration complete. Verify observable
behaviour rather than the presence of a setting.

Use safe negative tests — check whether a path exists or whether a variable name is set. Never
print a secret value.

## The build works through the project

- The project builds through the project entry point.
- Tests run through it.
- Lint, format, and check commands work.
- Generated files have sensible host ownership.
- The toolchain is not accidentally using a host compiler, runtime, or package manager.
- Expected cache volumes persist and can be recreated from empty.

## The container is a boundary

- Host credential directories are absent inside the container.
- Sensitive environment variables are absent.
- `SSH_AUTH_SOCK` is absent.
- The Podman/Docker socket is absent.
- The container is not privileged.
- Unnecessary capabilities are absent.
- The container cannot read arbitrary host-home files.
- Each opt-in profile grants only what it documents, and the default profile still grants none of it.

## The source stays self-contained

The rule and its rationale are owned by the `structure-source-workspace` skill; this is the
executable form of its "Verify the result" section.

- Copy `source` alone to a scratch directory, with no sibling directories present, and run the
  unparameterized entry point. Build and test must pass.
- No committed file in `source` names `../` or the workbench, including CI definitions,
  `.gitignore`, and contributor documentation.
- Component CI invokes the entry point with default parameters.

## The ownership boundary holds

- Compare the source filesystem before and after opening each configured IDE or agent tool. Check
  tracked, untracked, **and ignored** additions — an ignored file is still a file in the tree.
- IDE and agent configuration resolves under the workbench.
- Personal caches and transient state are outside `source`.
- Any cache exposed for a host tool arrived through a caller-supplied parameter, not a committed
  path.

## IDE integration

- IDE integration, if configured, uses the same project environment rather than a parallel host
  installation.
- Any tool limitation that could still write into `source` is reported to the user, not silently
  accepted.
