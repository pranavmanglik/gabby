-->

# ADR 0128: Optional warm sandbox container pool

- Status: accepted
- Date: 2026-11-12

## Context

Starting a fresh container for every tool run adds startup latency. Reusing containers can reduce
that cost, but reuse must not carry a prior run's workspace, tool payload, process state, or
permissions into the next run.

## Decision

Provide `WarmSandboxPool` as an opt-in adapter for one server-owned Linux sandbox profile. The pool
has a bounded size, rejects workspace mounts, requires a finite process limit, and accepts only
container specs matching its immutable profile with no per-run workspace or tool-input mounts. A
container is leased to at most one run and removed after use; the pool refills in the background.
When no idle container is available, normal container startup is used. The host closes the pool
during server shutdown.

## Consequences

The adapter can reduce startup time for compatible mount-free Linux workloads, but does not promise
zero cold starts or support every engine/platform. It is not enabled by default. Per-run mounted
workspaces and private tool payloads continue to use fresh containers. Operators must size the pool
against their capacity and cleanup requirements and retain ordinary runtime deadlines and resource
limits.
