# toolobserve

Observability primitives for tool execution: tracing, metrics, and structured logs.

## Overview

toolobserve provides OpenTelemetry-based instrumentation that can be attached to
`toolrun`, `toolruntime`, and `metatools-mcp` via middleware. It is pure
observability: **no execution, no I/O, no network** beyond exporter setup.

## Design Goals

1. **Deterministic telemetry**: stable span/metric names and attributes
2. **Context propagation**: instrumentation preserves `context.Context`
3. **Low overhead**: opt-in exporters and sampling support
4. **Thread safety**: safe under concurrent tool execution
5. **Minimal dependencies**: OTel SDK only; no server/transport coupling

## Position in the Stack

```
toolrun/toolruntime --> toolobserve --> exporters (OTLP/Jaeger/Prometheus)
                          \
                           -> metatools-mcp middleware
```

## API Reference

### Core Types

| Type | Purpose |
|------|---------|
| `Config` | Observability configuration (exporters, sampling) |
| `Observer` | Central entry point for tracing/metrics/logs |
| `ToolMeta` | Tool metadata for telemetry attribution |
| `Middleware` | Wraps execution with tracing, metrics, logging |
| `Tracer` | Span creation and context propagation |
| `Metrics` | Counters and histograms for tool execution |
| `Logger` | Structured log emission with tool context |

### Config

```go
type Config struct {
    ServiceName string         // Required: service name for telemetry
    Version     string         // Optional: service version
    Tracing     TracingConfig  // Tracing configuration
    Metrics     MetricsConfig  // Metrics configuration
    Logging     LoggingConfig  // Logging configuration
}

type TracingConfig struct {
    Enabled   bool    // Enable tracing
    Exporter  string  // stdout|otlp|jaeger|none
    SamplePct float64 // 0.0-1.0 sampling rate
}

type MetricsConfig struct {
    Enabled  bool   // Enable metrics
    Exporter string // stdout|otlp|prometheus|none
}

type LoggingConfig struct {
    Enabled bool   // Enable logging
    Level   string // debug|info|warn|error
}
```

### Observer

```go
type Observer interface {
    Tracer() trace.Tracer     // OpenTelemetry tracer
    Meter() metric.Meter      // OpenTelemetry meter
    Logger() Logger           // Structured logger
    Shutdown(ctx context.Context) error
}

// Create an observer
observer, err := toolobserve.NewObserver(ctx, cfg)
```

### ToolMeta

```go
type ToolMeta struct {
    ID        string   // Fully qualified tool ID (optional, derived if empty)
    Namespace string   // Tool namespace (e.g., "github")
    Name      string   // Tool name (required)
    Version   string   // Tool version
    Tags      []string // Tags for discovery
    Category  string   // Tool category
}

// Methods
meta.SpanName() string  // Returns "tool.exec.<namespace>.<name>"
meta.ToolID() string    // Returns "<namespace>.<name>" or just "<name>"
```

### Middleware

```go
type ExecuteFunc func(ctx context.Context, tool ToolMeta, input any) (any, error)

type Middleware struct {
    // ...
}

// Create middleware from components
mw := toolobserve.NewMiddleware(tracer, metrics, logger)

// Or create from observer
mw, err := toolobserve.MiddlewareFromObserver(observer)

// Wrap execution function
wrapped := mw.Wrap(myExecuteFunc)

// Execute with full observability
result, err := wrapped(ctx, meta, input)
```

### Logger

```go
type Logger interface {
    Info(ctx context.Context, msg string, fields ...Field)
    Warn(ctx context.Context, msg string, fields ...Field)
    Error(ctx context.Context, msg string, fields ...Field)
    Debug(ctx context.Context, msg string, fields ...Field)
    WithTool(meta ToolMeta) Logger
}

type Field struct {
    Key   string
    Value any
}
```

### Interface Contracts

- **Observer**: thread-safe; `Shutdown` honors context and is idempotent.
- **Tracer**: thread-safe; `StartSpan` honors context; `EndSpan` is best-effort.
- **Metrics**: thread-safe; `RecordExecution` is best-effort and must not panic.
- **Logger**: thread-safe; logging is best-effort and must not panic; `WithTool` returns a non-nil logger.

## Usage Examples

### Basic Setup

```go
cfg := toolobserve.Config{
    ServiceName: "metatools",
    Version:     "1.0.0",
    Tracing: toolobserve.TracingConfig{
        Enabled:   true,
        Exporter:  "stdout",
        SamplePct: 1.0,
    },
    Metrics: toolobserve.MetricsConfig{
        Enabled:  true,
        Exporter: "prometheus",
    },
    Logging: toolobserve.LoggingConfig{
        Enabled: true,
        Level:   "info",
    },
}

observer, err := toolobserve.NewObserver(ctx, cfg)
if err != nil {
    log.Fatal(err)
}
defer observer.Shutdown(ctx)
```

### With OTLP Backend

```go
// Set environment variable
os.Setenv("OTEL_EXPORTER_OTLP_ENDPOINT", "http://localhost:4317")

cfg := toolobserve.Config{
    ServiceName: "metatools",
    Tracing: toolobserve.TracingConfig{
        Enabled:   true,
        Exporter:  "otlp",
        SamplePct: 0.1, // 10% sampling
    },
    Metrics: toolobserve.MetricsConfig{
        Enabled:  true,
        Exporter: "otlp",
    },
}
```

### Integration with toolrun

```go
// Wrap a toolrun executor
executor := func(ctx context.Context, tool toolobserve.ToolMeta, input any) (any, error) {
    // Call toolrun.Run() here
    return toolrun.Run(ctx, toolID, args)
}

mw, _ := toolobserve.MiddlewareFromObserver(observer)
instrumentedExecutor := mw.Wrap(executor)

// All executions now emit spans, metrics, and logs
result, err := instrumentedExecutor(ctx, meta, input)
```

## Telemetry Output

### Spans

```
Span: tool.exec.github.create_issue
  Attributes:
    tool.id: github.create_issue
    tool.namespace: github
    tool.name: create_issue
    tool.version: 1.0.0
    tool.error: false
  Duration: 150ms
```

### Metrics

```
tool.exec.total{tool.id="github.create_issue"} 42
tool.exec.errors{tool.id="github.create_issue"} 3
tool.exec.duration_ms{tool.id="github.create_issue"} histogram
```

### Logs

```json
{
  "timestamp": "2026-01-29T12:00:00Z",
  "level": "info",
  "msg": "tool execution completed",
  "tool.id": "github.create_issue",
  "tool.namespace": "github",
  "tool.name": "create_issue",
  "duration_ms": 150.5
}
```

## Versioning

toolobserve follows semantic versioning aligned with the stack. The source of
truth is `ai-tools-stack/go.mod`, and `VERSIONS.md` is synchronized across repos.

## Next Steps

- [Design Notes](design-notes.md)
- [User Journey](user-journey.md)
- [Plans](plans/README.md)
