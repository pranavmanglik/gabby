-->

# ADR 0126: Optional MCP, telemetry, and orchestrator integrations

- Status: accepted
- Date: 2026-11-12

## Context

Hosts need standard ways to connect Gabby's governed runtime to tool servers, monitoring systems,
and workflow orchestrators. These integrations must not move tenant identity, durable state, or
workflow ownership into the runtime, and must not imply that model instructions are enforcement.

## Decision

- Keep MCP, OpenTelemetry, LF TRACE, signed audit, and optional orchestrator clients behind
  independent optional dependencies or adapters.
- MCP clients use Streamable HTTP and expose only explicitly configured remote tool names. The
  configured server and remote method are bound by the host-created handler; model arguments cannot
  redirect a call. MCP server exposure provides one `run_agent` entry point and does not expose
  registered host handlers directly.
- OpenTelemetry spans contain bounded execution metadata and omit prompts, tool arguments, and
  model output. Signed audit records are an optional local evidence format; LF TRACE export follows
  its declared schema and is not described as proof of model behavior or hardware attestation.
- LangGraph, CrewAI, and n8n integrations call the stateless HTTP/SSE API. They are transport helpers,
  not orchestration engines or zero-configuration deployment integrations.
- The consuming host owns credentials, transport lifecycle, tenant routing, durable state,
  telemetry retention, and shutdown of clients/exporters it creates.

## Consequences

Optional extras keep the base installation small. Integrations can evolve independently while the
core runtime retains one policy and execution boundary. Hosts must opt into and configure each
integration, provide HTTPS outside loopback, and test the actual transport and deployment they use.
The MCP and telemetry adapters do not provide a multi-tenant control plane or a standalone
orchestrator.
