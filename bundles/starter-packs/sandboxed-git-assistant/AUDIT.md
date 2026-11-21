-->

# Bundle review

- Scope: read-only inspection of a caller-provided repository checkout.
- Capability boundary: agent policy allowlists only `shell_run`; container networking is disabled by
  the default sandbox contract and CPU, memory, process count, and run duration are bounded.
- Required host controls: register `sandbox_tools()`, place a disposable checkout under `workspace/`,
  keep credentials and engine sockets out of the sandbox, and use a pinned image digest in production.
- Review status: source reviewed for the declared scope; the skill prompt cannot prevent arbitrary
  shell arguments by itself.
- Required deployment checks: verify that the selected image contains only required binaries and
  test the actual sandbox configuration on the deployment's container engine.
