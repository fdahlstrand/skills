# Language-specific guidance

Read only the sections relevant to the project.

These are defaults, not fixed templates. Inspect the repository and current ecosystem behavior before choosing exact images, versions, cache paths, or security flags.

## Node.js / TypeScript

Containerize:

- Node.js;
- npm/pnpm/yarn as applicable;
- TypeScript compiler;
- ESLint/Prettier and project CLIs;
- test runners and code generators.

Prefer:

- a committed lockfile;
- current package-manager protections against install/lifecycle scripts;
- explicit approval of packages that need install-time execution where supported;
- restrictions on Git/remote URL dependencies where supported;
- a release-age/cooldown policy where supported;
- dependency trees and package caches in named volumes when IDE integration permits it.

Never expose npm publishing credentials to ordinary install/build/test operations.

For IDEs with remote Node runtime support, point the IDE at the container runtime/package manager rather than installing a second Node toolchain on the host.

## Rust

Containerize:

- `rustup` if used;
- pinned `rustc` toolchain;
- Cargo;
- rustfmt;
- Clippy;
- linker and native development libraries;
- project Cargo extensions.

Persist appropriate Cargo registry/git caches and build caches in named volumes.

Remember that `build.rs` and procedural macros execute code during build. Keep credentials out of the build environment.

Use `rust-toolchain.toml` when appropriate to make the intended Rust toolchain explicit in addition to the container image.

Debugging may require a separate profile with narrowly scoped `ptrace` capability.

## Go

Containerize:

- Go toolchain;
- `gopls` when the IDE can use a remote language server;
- Delve;
- static analysis and lint tools;
- code generators.

Persist `GOMODCACHE` and `GOCACHE` in named volumes when useful.

Keep `go.mod` and `go.sum` committed. Preserve Go checksum/module-security behavior unless the project has a documented private-module requirement.

Treat `go generate` and external generators as executable project tooling and run them inside the container.

Debugging may require a separate capability profile.

## C / C++

Containerize:

- GCC and/or Clang;
- libc and development headers;
- CMake/Meson/Autotools as applicable;
- Ninja/Make;
- Conan/vcpkg or other dependency tooling;
- clang-format, clang-tidy, static analyzers;
- GDB/LLDB;
- code generators;
- all system development packages required by the build.

Pin enough of the base distribution/toolchain to make ABI and library assumptions intentional.

Use build directories or named volumes that do not pollute the source tree when practical.

For IDE code models, use the same compiler, compile database, CMake presets, and sysroot as the containerized build rather than a host approximation.

Debugging may require a separate `SYS_PTRACE`/seccomp exception. Do not add that capability to the ordinary build container.

## Zig

Containerize:

- the pinned Zig compiler;
- project-specific build tools;
- optional native/system dependencies.

Persist Zig caches in named volumes where beneficial.

Treat the Zig compiler version as part of the project definition and update it deliberately.

## Python

Containerize:

- Python interpreter;
- pip/uv/Poetry/PDM or the project's package manager;
- build backends;
- formatters, linters, type checkers, test runners;
- native compilers and headers needed for extensions.

Use a committed lock or otherwise reproducible dependency description when supported by the project's package manager.

Do not mount personal pip configuration or cloud credentials into the normal environment.

Python source-distribution builds and build backends can execute code; treat dependency installation as untrusted execution.

## Java / JVM

Containerize:

- selected JDK;
- Maven/Gradle or wrapper-backed environment;
- project-specific JVM tools;
- native build dependencies where required.

Prefer Maven Wrapper or Gradle Wrapper when already used, while still running the wrapper inside the container. Cache artifact repositories in named volumes.

Do not expose repository publishing credentials to normal dependency resolution/builds.

## Debuggers and profilers

Create an explicit debug/profile mode if additional privileges are required. Grant only what the debugger or profiler needs, for example a specific ptrace capability or device.

Never use `privileged: true` as the first answer to a debugger problem.

## Embedded development

Keep compiler, linker, SDK, build system, and generators in the container.

Expose only the required serial/JTAG/USB device to an explicit hardware profile. Do not broadly mount `/dev`.

Host udev permissions or helper services may still be required; document these as host prerequisites.

## GPU development

Keep ordinary compilation/container tooling isolated. Create a GPU-specific execution profile exposing only the required GPU devices/runtime integration.

Do not add unrelated host credentials or a container-engine socket merely because GPU access is required.
