Merge into the component's existing `AGENTS.md` or `CONTRIBUTING.md`. Do not create a new
agent-instruction file for this; if neither exists, put it in the contributor documentation the
project already has.

---

## Development toolchain

The project owns its development toolchain. Run project compilers, runtimes, package managers,
dependency installation, builds, tests, linters, formatters, and code generators through the
project-provided container workflow (`./dev` or the documented equivalent), not through
host-installed toolchains.

The entry point works from a clean clone with no arguments. Paths outside this repository are
never hard-coded; they arrive as documented parameters supplied by the caller.

Treat the toolchain container as a security boundary. Do not mount or forward the host home
directory, SSH agent, credential stores, cloud credentials, publishing tokens, or the
Podman/Docker socket unless the task explicitly requires a narrowly scoped exception. Do not
install or reconfigure host development toolchains to work around a project-container issue.

Container and toolchain definitions and their security settings are security-sensitive. Preserve
least privilege, use the default non-secret, non-privileged profile for normal work, and call out
any change that expands host access, credentials, devices, capabilities, or privileges.
