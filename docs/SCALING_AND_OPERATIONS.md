# Scaling and operational tradeoffs

This guide maps common production concerns to Gabby's current controls, limits, and deployment
choices. It distinguishes behavior implemented in Gabby from decisions and infrastructure owned by
the host. A passing local test is not evidence for a different provider, operating system, container
engine, or hosted deployment.

## Synchronous extension handlers

Synchronous callbacks run in Gabby's process through a dedicated bounded worker pool. The current
pool has eight daemon workers and a queue capped at 256 waiting callbacks. This prevents unbounded
thread creation and waiting work; when both workers and queue are occupied, new tool callbacks are
rejected with the stable `capacity_exhausted` tool error. It does not make a synchronous callback
interruptible. If a callback
hangs after starting, it continues occupying one worker even when the caller times out or cancels.

For callbacks that must be forcibly stopped, use a sandboxed/container-backed tool or a separate
process/service with an enforceable deadline. Keep synchronous host handlers short, make blocking
I/O use explicit timeouts, and cooperate with `ToolContext.cancellation` when possible. The callback
pool is process-global and its worker/queue sizing is not currently a public per-agent setting;
scale service processes only after accounting for per-process model and storage resources.

## Vector search and hybrid indexing

`SQLiteVectorStore` performs exact cosine ranking by scanning stored vectors. It is suitable for
local and modest corpora where predictable setup and exact results matter. There is no universal
chunk-count cutoff: measure latency and memory with representative vector dimensions, filters,
concurrency, and hardware before choosing a backend.

For larger or shared deployments, Gabby includes `PostgresVectorStore` using pgvector's HNSW cosine
index. Install the `postgres` extra, apply the vector migration, and use a host-owned PostgreSQL
pool. For coordinated lexical and vector reindexing across hosts, use the PostgreSQL lexical store,
PostgreSQL vector store, and `PostgresGenerationManifestStore` in the same consistency domain. The
manifest coordinates durable generations and fencing; it does not make local SQLite indexes
shared or manage the host's pool, credentials, TLS, migration, or backup lifecycle.

See the [PostgreSQL hybrid indexing example](README_GUIDE.md#knowledge-retrieval),
[vector store reference](EXTENSIONS.md), and [integration checks](../tests/integration/README.md).

## Streaming and local Transformers

Hosted providers that implement Gabby's streaming contract can emit text and tool progress through
SSE. The local `TransformersProvider` emits bounded token deltas while generating when no tools are
available. If tools are available, it buffers and parses the complete response because tool-control
tokens are model-specific; this avoids exposing them as user-facing text. Tool-enabled local runs can
therefore still have a pause before output. For interactive tool-enabled local inference, use a local
inference server with a compatible streaming API (for example an OpenAI-compatible endpoint), or a
hosted provider with native streaming.

Live provider latency and compatibility are service- and model-dependent. The opt-in checks in
[`tests/integration/README.md`](../tests/integration/README.md) exercise completion and streaming
against configured endpoints. They verify basic contract compatibility, not service-level latency,
long-duration reliability, model quality, or schema stability. Run them using protected environment
secrets before changing providers or models; retain production monitoring and bounded retries at the
host boundary.

## Container engine and platform support

Gabby deliberately fails closed for native Windows Docker containers unless the Docker Engine
server is at least 29.1.4. Earlier versions have a known failure when starting containers with
networking disabled. Gabby does not fall back to enabled networking. Upgrade the engine or choose a
supported deployment boundary; do not bypass the version check. The requirement is Windows-specific
and does not apply to Linux containers.

Container support varies by host, engine, API/CLI adapter, image, and OS. The [sandbox guide in the
user documentation](README_GUIDE.md#container-sandbox-tools) and opt-in live tests describe the
validated combinations. Require target-host acceptance checks for network isolation, mounts,
resource limits, Hyper-V isolation where applicable, and cleanup before relying on sandboxing.

## Tool schemas and prototyping

Tool input and output schemas are required runtime contracts. Gabby validates handler output before
adding it to model context, and applies the configured result-size limit. There is no warn-only mode:
letting invalid handler output pass can make model context and downstream behavior unpredictable.
For fast prototyping, begin with a broad JSON-only contract and narrow it as the handler stabilizes:

```python
from gabby import Tool

prototype_tool = Tool(
    name="lookup",
    description="Prototype lookup result shape.",
    parameters={"type": "object", "additionalProperties": True},
    output_schema={"type": ["object", "array", "string", "number", "boolean", "null"]},
    handler=lookup,
)
```

The broad output schema still rejects non-JSON values and does not disable byte limits. Before
shipping, replace it with the smallest useful schema, including required fields and property bounds
where the consuming application depends on the result shape. See the [tool contract](EXTENSIONS.md)
and [structured output guide](STRUCTURED_OUTPUT.md).

## Evidence checklist

Before scaling a Gabby deployment, record evidence for the actual configuration:

- Run the regular local suite and focused provider, PostgreSQL, and container acceptance checks that
  match the target deployment.
- Measure end-to-end latency, throughput, worker saturation, request/response sizes, and resource
  use with representative workloads.
- Test cancellation and timeout behavior for each custom provider, tool, retriever, verifier, and
  tracer; synchronous callbacks may continue after cancellation.
- Confirm the selected vector and manifest backends share the intended database consistency domain.
- Verify TLS, authentication, authorization, process supervision, secrets, backups, and ingress
  controls in the host deployment.
- Treat untested provider/OS/engine combinations as unverified, not as supported by analogy.

## Concern-by-concern disposition

| Concern | Current disposition |
|---|---|
| Stuck synchronous callbacks | Worker threads and the waiting queue are bounded; new tool calls receive `capacity_exhausted` after saturation. A running Python callback still cannot be forcibly stopped. Use a container or separate process when hard termination is required. |
| Linear SQLite vector search | This remains an exact local backend with linear scan cost. Gabby makes no universal chunk threshold claim. PostgreSQL/pgvector HNSW is the included larger/shared-index option. |
| Multi-host manifest setup | `PostgresGenerationManifestStore`, `PostgresKnowledgeStore`, and `PostgresVectorStore` are implemented and compose through `HybridIndexCoordinator`. The host still applies three migrations and supplies one pool/consistency domain; the guide has a complete constructor example. |
| Local Transformers streaming | Tool-free calls now stream bounded token deltas. Calls where tools are available still buffer for safe model-specific tool parsing. |
| Mocked provider acceptance | Opt-in real completion/stream checks are documented for Ollama, Hugging Face, Anthropic, and Gemini. They require credentials or a local service and are not run in ordinary CI; they do not guarantee provider uptime or schema stability. |
| Docker version floor | The Windows Docker floor remains fail-closed because older daemons can fail when networking is disabled. There is no unsafe fallback that enables networking. Linux containers do not use this Windows-specific floor. |
| Strict tool schemas | Runtime validation remains mandatory. Developers can start with a broad JSON-only schema and refine it before release; Gabby does not silently pass invalid values through. |
