# PRD-001: toolobserve Library Implementation

> **For agents:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a standalone observability library providing tracing, metrics,
and structured logging for tool execution across the stack.

**Architecture:** Pure instrumentation layer that wraps execution callbacks with
OpenTelemetry spans, metrics, and structured logs. Exporters are configured by
callers. The library never executes tools or performs network I/O on its own.

**Tech Stack:** Go 1.24+, OpenTelemetry SDK, OTLP/Jaeger/Prometheus exporters.

**Priority:** P1 (Phase 3 in the plan-of-record)

---

## Context and Stack Alignment

toolobserve provides consistent telemetry for:
- `toolrun` execution and chaining
- `toolruntime` backend execution
- `metatools-mcp` server middleware

It must remain **protocol-agnostic** and **transport-agnostic**. Observability
must not leak into business logic; it is injected via middleware/wrappers.

---

## Requirements

### Functional

1. Configurable tracing, metrics, and logging.
2. Deterministic span naming and metric naming.
3. Structured logs with tool-scoped fields.
4. Middleware wrapper for execution callbacks.
5. Exporter configuration for OTLP/Jaeger/Prometheus/Stdout.

### Non-functional

- Thread-safe under concurrent use.
- No global mutable state beyond internal OTel providers.
- No panics on exporter failure (errors are returned/logged).
- Context propagation for every API surface.
- Deterministic output for tests.

---

## API Model (Target)

```go
// Config drives observer setup.
type Config struct {
    ServiceName string
    Version     string
    Tracing     TracingConfig
    Metrics     MetricsConfig
    Logging     LoggingConfig
}

type TracingConfig struct {
    Enabled   bool
    Exporter  string // otlp|jaeger|stdout|none
    SamplePct float64
}

type MetricsConfig struct {
    Enabled  bool
    Exporter string // otlp|prometheus|stdout|none
}

type LoggingConfig struct {
    Enabled bool
    Level   string // debug|info|warn|error
}

// Observer provides tracing/metrics/logging helpers.
type Observer interface {
    Tracer() trace.Tracer
    Meter() metric.Meter
    Logger() Logger
}

// ToolMeta is the minimal tool identity.
type ToolMeta struct {
    ID        string
    Namespace string
    Name      string
    Version   string
    Tags      []string
    Category  string
}

// Middleware wraps execution with telemetry.
type Middleware struct { observer Observer }

// ExecuteFunc is a generic execution signature.
type ExecuteFunc func(ctx context.Context, tool ToolMeta, input any) (any, error)
```

---

## Telemetry Semantics

### Span Naming

- `tool.exec.<namespace>.<name>`
- If namespace is empty: `tool.exec.<name>`

### Span Attributes

- `tool.id`, `tool.namespace`, `tool.name`, `tool.version`, `tool.category`
- `tool.tags` (string list)
- `tool.error` (bool)

### Metrics

- Counter: `tool.exec.total` (unit: `{call}`)
- Counter: `tool.exec.errors` (unit: `{error}`)
- Histogram: `tool.exec.duration_ms` (unit: `ms`)

### Logging Fields

- `tool.id`, `tool.namespace`, `tool.name`, `tool.version`
- `duration_ms`, `error`
- Inputs should be **redacted** or hashed by default (no raw inputs)

---

## Directory Structure

```
toolobserve/
├── observe.go
├── observe_test.go
├── tracer.go
├── tracer_test.go
├── metrics.go
├── metrics_test.go
├── logger.go
├── logger_test.go
├── middleware.go
├── middleware_test.go
├── exporters/
│   ├── otlp.go
│   ├── jaeger.go
│   ├── prometheus.go
│   └── stdout.go
├── doc.go
├── README.md
├── go.mod
└── go.sum
```

---

## TDD Task Breakdown (Detailed)

### Task 1 — Config + Observer Core

**Files:** `observe.go`, `observe_test.go`

**Tests:**
- `TestConfigValidate_Valid`
- `TestConfigValidate_MissingServiceName`
- `TestConfigValidate_UnknownTracingExporter`
- `TestConfigValidate_UnknownMetricsExporter`
- `TestNewObserver_DisabledNoop`
- `TestNewObserver_ReturnsTracerAndMeter`

**Acceptance:** Config validation rejects unknown exporters and empty service
name. `NewObserver` returns a no-op implementation when disabled.

**Commit:** `feat(toolobserve): add config validation and observer core`

---

### Task 2 — Tracing

**Files:** `tracer.go`, `tracer_test.go`

**Tests:**
- `TestTracer_SpanNameDeterministic`
- `TestTracer_SpanAttributes`
- `TestTracer_ContextPropagation`

**Acceptance:** Span names and attributes match the semantic contract, and
span context is propagated.

**Commit:** `feat(toolobserve): add tracing`

---

### Task 3 — Metrics

**Files:** `metrics.go`, `metrics_test.go`

**Tests:**
- `TestMetrics_CountersIncrement`
- `TestMetrics_DurationHistogram`

**Acceptance:** Counters and histograms record execution and error events.

**Commit:** `feat(toolobserve): add metrics`

---

### Task 4 — Logging

**Files:** `logger.go`, `logger_test.go`

**Tests:**
- `TestLogger_IncludesToolFields`
- `TestLogger_ErrorLevel`

**Acceptance:** Logs include tool context and error information.

**Commit:** `feat(toolobserve): add structured logger`

---

### Task 5 — Middleware

**Files:** `middleware.go`, `middleware_test.go`

**Tests:**
- `TestMiddleware_SuccessPath`
- `TestMiddleware_ErrorPath`
- `TestMiddleware_DoesNotMutateInput`

**Acceptance:** Middleware wraps execution with tracing/metrics/logging and
preserves inputs.

**Commit:** `feat(toolobserve): add execution middleware`

---

### Task 6 — Exporter Setup

**Files:** `exporters/*.go`, `exporters/*_test.go`

**Tests:**
- `TestExporter_InvalidName`
- `TestExporter_Stdout`
- `TestExporter_OtlpConfigValidation`
- `TestExporter_PrometheusConfigValidation`

**Acceptance:** Exporter configuration errors are returned, not panicked.

**Commit:** `feat(toolobserve): add exporter configuration`

---

### Task 7 — Docs + Examples

**Files:** `README.md`, `docs/index.md`, `docs/user-journey.md`

**Acceptance:** Quick start examples exist, Mermaid diagram is present, and
ai-tools-stack includes a D2 diagram.

**Commit:** `docs(toolobserve): finalize documentation`

---

## PR Process

1. Create branch: `feat/toolobserve-<task>`
2. Implement TDD task in isolation
3. Run: `go test -race ./...`
4. Commit with scoped message from task
5. Open PR against `main`
6. Merge after CI green

---

## Versioning and Propagation

- **Source of truth:** `ai-tools-stack/go.mod`
- **Matrix:** `ai-tools-stack/VERSIONS.md` (auto-synced)
- **Propagation:** `ai-tools-stack/scripts/update-version-matrix.sh --apply`
- Tags: `vX.Y.Z` and `toolobserve-vX.Y.Z`

---

## Integration with metatools-mcp

- Wire middleware after tool provider selection.
- Span/metric names must match tool IDs emitted by the server.
- Configuration driven through `metatools-mcp` config layer.

---

## Definition of Done

- All tasks complete with tests passing
- `go test -race ./...` succeeds
- Docs + diagrams updated in ai-tools-stack
- CI green
- Version matrix updated after first release
