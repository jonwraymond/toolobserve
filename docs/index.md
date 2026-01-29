# toolobserve

Observability primitives for tool execution: tracing, metrics, and structured logs.

## Overview

toolobserve provides OpenTelemetry-based instrumentation that can be attached to
`toolrun`, `toolruntime`, and `metatools-mcp` via middleware. It is pure
observability: **no execution, no I/O, no network** beyond exporter setup.

## Design Goals

1. **Deterministic telemetry**: stable span/metric names and attributes
2. **Context propagation**: instrumentation must preserve `context.Context`
3. **Low overhead**: opt-in exporters and sampling support
4. **Thread safety**: safe under concurrent tool execution
5. **Minimal dependencies**: OTel SDK only; no server/transport coupling

## Position in the Stack

```
toolrun/toolruntime --> toolobserve --> exporters (OTLP/Jaeger/Prometheus)
                          \
                           -> metatools-mcp middleware
```

## Core Types

| Type | Purpose |
|------|---------|
| `Observer` | Central entry point for tracing/metrics/logs |
| `Config` | Observability configuration (exporters, sampling) |
| `Tracing` | Span creation and context propagation |
| `Metrics` | Counters, histograms, gauges for tool execution |
| `Logger` | Structured log emission with tool context |
| `Middleware` | Wrapping execution with telemetry |

## Quick Start

```go
cfg := toolobserve.Config{
    ServiceName: "metatools",
    Version:     "0.1.0",
    Tracing: toolobserve.TracingConfig{
        Enabled:  true,
        Exporter: "otlp",
    },
    Metrics: toolobserve.MetricsConfig{
        Enabled:  true,
        Exporter: "prometheus",
    },
}

observer, err := toolobserve.NewObserver(cfg)
if err != nil {
    log.Fatal(err)
}

wrapped := toolobserve.NewMiddleware(observer).Wrap(toolrunExecutor)
_ = wrapped
```

## Versioning

toolobserve follows semantic versioning aligned with the stack. The source of
truth is `ai-tools-stack/go.mod`, and `VERSIONS.md` is synchronized across repos.

## Next Steps

- [Design Notes](design-notes.md)
- [User Journey](user-journey.md)
- [Plans](plans/README.md)
