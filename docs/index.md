<<<<<<< Updated upstream
<!--
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

Copyright 2026-present Gabby Contributors.
-->
=======
---
title: Gabby documentation
description: Guides and references for Gabby's agent runtime and governance framework.
hide:
  - toc
---
>>>>>>> Stashed changes

<div class="gabby-docs-home" markdown>

<div class="docs-home-hero" markdown>

<div class="docs-home-eyebrow">AGENT RUNTIME · GOVERNED EXECUTION</div>

# Build agents with clear boundaries.

Gabby gives applications a place to define agent capabilities, govern tool access and run tasks
within explicit limits. These docs cover how to use it, how its runtime is put together, and what
to check before operating it.

<div class="docs-home-actions" markdown>

[Start with the user guide](README_GUIDE.md){ .md-button .md-button--primary }
[Understand the architecture](ARCHITECTURE.md){ .md-button }

</div>
</div>

## Find the right guide

<div class="grid cards" markdown>

-   **Get started**

    ---

    Install Gabby, configure an agent and run it through the Python or HTTP interface.

    [User guide](README_GUIDE.md) · [HTTP client](CLIENT.md)

-   **Understand the runtime**

    ---

    Follow the request lifecycle, state ownership, policy checks and public contracts.

    [Architecture](ARCHITECTURE.md) · [API stability](API_STABILITY.md)

-   **Add capabilities**

    ---

    Extend providers and tools, configure approvals, and work with skills and structured output.

    [Extensions](EXTENSIONS.md) · [Skill registry](SKILL_REGISTRY.md)

-   **Operate with care**

    ---

    Review the threat model, deployment boundaries and operational responsibilities.

    [Threat model](THREAT_MODEL.md) · [Operations](OPERATIONS.md)

</div>

## Project status

!!! warning "Pre-1.0 beta"
    Gabby is beta software (`0.1.0b5`), and public extension APIs may change before 1.0. Tests and
    static checks cover only the behaviors they exercise. Review the [API stability policy](API_STABILITY.md),
    [threat model](THREAT_MODEL.md) and [readiness scorecard](PROJECT_SCORECARD.md) when evaluating
    Gabby for a deployment.

## More to explore

- [Examples](https://github.com/undreamt-oss/gabby/tree/main/examples) and [starter bundles](https://github.com/undreamt-oss/gabby/tree/main/bundles)
- [Runtime integrations](INTEGRATIONS.md) and [host-orchestrated pipelines](HOST_ORCHESTRATED_PIPELINES.md)
- [Contributing guide](https://github.com/undreamt-oss/gabby/blob/main/CONTRIBUTING.md) and [architecture decisions](architecture/adr/README.md)
- [Build or preview this documentation](BUILDING_DOCS.md)

</div>
