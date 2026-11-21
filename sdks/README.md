-->

# Gabby client SDKs

Small HTTP and Server-Sent Events clients for TypeScript/Node.js, Go, and Rust are generated from
the checked-in templates. Regenerate them from the repository root with:

```sh
python scripts/generate_client_sdks.py
```

Tagged project releases attach versioned TypeScript, Go, and Rust source archives to the GitHub
release. The clients are distributed as source archives and repository packages; they are not
separately published to npm, crates.io, or a Go module proxy.

The clients target `/v1/agents/{agent_name}/run` and `/stream`, support bearer authentication, and
preserve the SSE idempotency and resume headers. They do not retain conversation history. Pass
context, memory, metadata, and host context explicitly for each run. Use HTTPS outside loopback and
read API tokens from your deployment's secret store.

## TypeScript / Node.js

```ts
import { GabbyClient } from "@undreamt-oss/gabby-client";

const client = new GabbyClient({
  baseUrl: "https://agents.example.com",
  agentName: "support-agent",
  bearerToken: process.env.GABBY_API_TOKEN,
});
const result = await client.run({ input: "Triage this ticket", include_trace: false });
console.log(result.output);

for await (const event of client.stream({ input: "Summarize this report" })) {
  if (event.type === "text_delta") process.stdout.write(String(event.data.text ?? ""));
}
```

## Go

```go
client, err := gabby.New("https://agents.example.com", "support-agent")
if err != nil { panic(err) }
client.Token = os.Getenv("GABBY_API_TOKEN")
result, err := client.Run(ctx, gabby.Request{Input: "Triage this ticket"})
if err != nil { return err }
fmt.Println(result.Output)
```

## Rust

```rust
let client = gabby_client::Client::new("https://agents.example.com", "support-agent")?
    .with_bearer_token(std::env::var("GABBY_API_TOKEN")?);
let result = client.run(&gabby_client::RunRequest {
    input: "Triage this ticket".into(),
    ..Default::default()
}).await?;
println!("{}", result.output);
```

SSE clients yield the API's typed JSON event envelopes. Resume a disconnected stream by reusing the
same idempotency key and passing the last event ID received. The server's documented retention and
same-worker routing limits still apply.
