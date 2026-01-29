# toolobserve

Observability library for tool execution: tracing, metrics, and structured logs.

## Overview

toolobserve provides OpenTelemetry-based instrumentation that can be wired into
`toolrun`, `toolruntime`, and `metatools-mcp` via middleware. It is pure
observability: **no execution, no network access, no transport**. Exporters are
configured by the caller.

## Design Goals

1. Deterministic span/metric naming
2. Low overhead with opt-in exporters
3. Thread-safe, context-aware instrumentation
4. Structured logs with tool-scoped fields
5. Minimal dependencies and clean seams

## Position in the Stack

```
toolrun/toolruntime --> toolobserve --> exporters (OTLP/Jaeger/Prometheus)
```

## Versioning

toolobserve follows semantic versioning aligned with the stack. The source of
truth is `ai-tools-stack/go.mod`, and `VERSIONS.md` is synchronized across repos.

## Next Steps

- See `docs/index.md` for usage and design notes.
- PRD and execution plan live in `docs/plans/`.
