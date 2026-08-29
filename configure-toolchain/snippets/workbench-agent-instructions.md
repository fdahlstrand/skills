Place in the workbench's own agent instructions. This is personal configuration — it points a
particular tool at the component's contract and does not restate it.

---

## Working on the component

Build, test, lint, and format the component only through its own entry point (`./dev` or the
documented equivalent), run from the component's directory. That entry point is the component's
contract; this workbench does not reimplement compiler, linter, or container policy.

Invoke it with default parameters unless a documented parameter is genuinely needed. Personal
overrides — extra Compose files, cache paths, environment — live here in the workbench and are
passed in. Never add them to the component, not even as an ignored file.

Do not create IDE, editor, or agent configuration inside the component. It belongs here. If a tool
appears to require configuration inside the component to work at all, stop and say so rather than
adding it.

Do not install the component's language toolchain on the host to work around a container problem.
