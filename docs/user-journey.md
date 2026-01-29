# User Journey: Observability for Tool Execution

## Scenario

You want to instrument tool execution with tracing and metrics, exporting to
OTLP for traces and Prometheus for metrics. You also want structured logs for
failed tool calls.

## Step 1: Configure observer

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
    Logging: toolobserve.LoggingConfig{Enabled: true},
}
observer, _ := toolobserve.NewObserver(cfg)
```

## Step 2: Wrap execution

```go
mw := toolobserve.NewMiddleware(observer)
wrapped := mw.Wrap(toolrunExecutor)
_ = wrapped
```

## Step 3: Inspect telemetry

- Spans appear as `tool.mcp.search` with duration and error tags.
- Metrics show execution counts and latency histograms.
- Logs include tool ID, inputs (redacted), and error messages.

## Flow Diagram

```mermaid
flowchart LR
  A[Tool Call] --> B[Middleware]
  B --> C[Observer]
  C --> D[Tracing]
  C --> E[Metrics]
  C --> F[Logging]
  D --> G[OTLP Exporter]
  E --> H[Prometheus]
```
