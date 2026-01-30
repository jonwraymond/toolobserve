# toolobserve

Observability library for tool execution: tracing, metrics, and structured logs.

## Overview

toolobserve provides OpenTelemetry-based instrumentation that can be wired into
`toolrun`, `toolruntime`, and `metatools-mcp` via middleware. It is pure
observability: **no execution, no network access, no transport**. Exporters are
configured by the caller.

## Installation

```bash
go get github.com/jonwraymond/toolobserve
```

## Quick Start

```go
package main

import (
    "context"
    "log"

    "github.com/jonwraymond/toolobserve"
)

func main() {
    ctx := context.Background()

    // Configure observability
    cfg := toolobserve.Config{
        ServiceName: "my-service",
        Version:     "1.0.0",
        Tracing: toolobserve.TracingConfig{
            Enabled:   true,
            Exporter:  "stdout",  // or "otlp", "jaeger", "none"
            SamplePct: 1.0,       // 100% sampling
        },
        Metrics: toolobserve.MetricsConfig{
            Enabled:  true,
            Exporter: "stdout",  // or "otlp", "prometheus", "none"
        },
        Logging: toolobserve.LoggingConfig{
            Enabled: true,
            Level:   "info",  // debug, info, warn, error
        },
    }

    // Create observer
    observer, err := toolobserve.NewObserver(ctx, cfg)
    if err != nil {
        log.Fatal(err)
    }
    defer observer.Shutdown(ctx)

    // Create middleware from observer
    mw, err := toolobserve.MiddlewareFromObserver(observer)
    if err != nil {
        log.Fatal(err)
    }

    // Wrap your tool execution function
    wrapped := mw.Wrap(func(ctx context.Context, tool toolobserve.ToolMeta, input any) (any, error) {
        // Your tool execution logic here
        return "result", nil
    })

    // Execute with full observability
    result, err := wrapped(ctx, toolobserve.ToolMeta{
        Namespace: "example",
        Name:      "my_tool",
        Version:   "1.0.0",
    }, map[string]any{"key": "value"})

    log.Printf("Result: %v, Error: %v", result, err)
}
```

## Design Goals

1. **Deterministic span/metric naming** - Span names follow `tool.exec.<namespace>.<name>`
2. **Low overhead** - Opt-in exporters with configurable sampling
3. **Thread-safe** - Context-aware instrumentation safe for concurrent use
4. **Structured logs** - Tool-scoped fields with automatic input redaction
5. **Minimal dependencies** - OpenTelemetry SDK only; no server/transport coupling

## Telemetry Semantics

### Span Naming

```
tool.exec.<namespace>.<name>   # If namespace is set
tool.exec.<name>               # If no namespace
```

### Span Attributes

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `tool.id` | string | Yes | Fully qualified tool ID |
| `tool.namespace` | string | No | Tool namespace |
| `tool.name` | string | Yes | Tool name |
| `tool.version` | string | No | Tool version |
| `tool.category` | string | No | Tool category |
| `tool.tags` | string[] | No | Tool tags |
| `tool.error` | bool | Yes | Whether execution failed |

### Metrics

| Metric | Type | Unit | Description |
|--------|------|------|-------------|
| `tool.exec.total` | Counter | `{call}` | Total executions |
| `tool.exec.errors` | Counter | `{error}` | Failed executions |
| `tool.exec.duration_ms` | Histogram | `ms` | Execution duration |

### Logging

Structured JSON logs with tool context fields. Sensitive inputs are automatically redacted.

## Position in the Stack

```
toolrun/toolruntime --> toolobserve --> exporters (OTLP/Jaeger/Prometheus)
                           \
                            -> metatools-mcp middleware
```

## Exporters

### Tracing Exporters

| Name | Description | Environment Variables |
|------|-------------|----------------------|
| `stdout` | Print spans to stdout | None |
| `otlp` | Send to OTLP-compatible backend | `OTEL_EXPORTER_OTLP_ENDPOINT` |
| `jaeger` | Send to Jaeger (via OTLP) | `OTEL_EXPORTER_JAEGER_ENDPOINT` |
| `none` | No-op exporter | None |

### Metrics Exporters

| Name | Description | Environment Variables |
|------|-------------|----------------------|
| `stdout` | Print metrics to stdout | None |
| `otlp` | Send to OTLP-compatible backend | `OTEL_EXPORTER_OTLP_ENDPOINT` |
| `prometheus` | Expose Prometheus endpoint | None |
| `none` | No-op exporter | None |

## Versioning

toolobserve follows semantic versioning aligned with the stack. The source of
truth is `ai-tools-stack/go.mod`, and `VERSIONS.md` is synchronized across repos.

## Documentation

- See `docs/index.md` for detailed usage and API reference
- See `docs/design-notes.md` for architectural decisions
- See `docs/user-journey.md` for integration examples
- PRD and execution plan in `docs/plans/`

## License

MIT
