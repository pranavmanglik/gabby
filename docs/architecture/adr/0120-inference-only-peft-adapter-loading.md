-->

# ADR 0120: Model adapter loading boundary

- Status: superseded
- Date: 2026-10-03

The previously accepted model adapter loading capability has been removed. Gabby does not train
models, modify model weights, or load model adapters. A host configures an agent's requested
character through instructions and optional context; these guide model output but do not guarantee
behavior. Runtime policies enforce tool access and execution boundaries. See
[ADR 0121](0121-runtime-specialization-boundary.md).
