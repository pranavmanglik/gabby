-->

# Bundle review

- Scope: explains results from one host-registered `sql_read_only_query` tool.
- Capability boundary: agent policy allowlists only that tool and limits calls to three per run.
- Required host control: the tool must use a read-only database principal, enforce query and row
  limits, reject multi-statement or mutating operations, and authorize tenant scope independently.
- Review status: source reviewed for the declared scope; prompts alone do not enforce SQL safety.
- Required deployment checks: test the actual tool handler against DDL, DML, excessive scans,
  cross-tenant queries, secrets, and malformed model-generated input before use.
