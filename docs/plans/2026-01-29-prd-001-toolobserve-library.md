# PRD-001: toolobserve Library Implementation

> **For agents:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a standalone observability library that provides tracing, metrics,
and structured logging for tool execution across the stack.

**Architecture:** A pure instrumentation layer that wraps execution callbacks
with OpenTelemetry spans, metrics, and structured logs. Exporters are configured
by the caller; the library never executes tools or performs network I/O on its
own.

**Tech Stack:** Go 1.24+, OpenTelemetry SDK, OTLP/Jaeger/Prometheus exporters

**Priority:** P1 (Phase 3 in the plan-of-record)

---

## Context and Stack Alignment

toolobserve enables visibility across:
- `toolrun` execution and chaining
- `toolruntime` backend execution
- `metatools-mcp` server pipeline (middleware integration)

It must remain **protocol-agnostic** and **transport-agnostic**. Observability
should not leak into business logic; it is injected via middleware/wrappers.

---

## Scope

### In scope
- Observer configuration and validation
- Tracing setup with deterministic span naming
- Metrics counters/histograms for execution
- Structured logging with tool-scoped fields
- Middleware wrapper for tool execution
- Exporter configuration (OTLP, Jaeger, Prometheus, stdout)
- Unit tests for all exported behavior
- Minimal docs and examples

### Out of scope
- Tool execution logic
- Storage backends for logs/metrics
- Sampling policies beyond OTel defaults
- PII redaction beyond simple field control

---

## Design Principles

1. **Pure instrumentation**: no execution or transport code.
2. **Context-first**: all APIs accept and propagate `context.Context`.
3. **Deterministic naming**: stable span/metric names across runs.
4. **Low overhead**: avoid heavy allocations on hot paths.
5. **Failure isolation**: exporter errors must not crash the caller.

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

## API Shape (Conceptual)

```go
// Config drives observer setup.
type Config struct {
    ServiceName string
    Version     string
    Tracing     TracingConfig
    Metrics     MetricsConfig
    Logging     LoggingConfig
}

// Observer provides tracing/metrics/logging helpers.
type Observer interface {
    Tracer() trace.Tracer
    Meter() metric.Meter
    Logger() Logger
}

// Middleware wraps tool execution with telemetry.
type Middleware struct {
    observer Observer
}
```

---

## Tasks (TDD)

### Task 1 — Config + Observer Core

- Implement `Config` validation (service name, exporter names)
- Implement `NewObserver` returning initialized tracer/meter/logger
- Tests: validation failure, observer initialization

### Task 2 — Tracing

- Create spans with deterministic naming (`tool.<namespace>.<name>`)
- Attach tool metadata attributes (`tool.id`, `tool.name`, `tool.namespace`)
- Tests: span name + attribute presence

### Task 3 — Metrics

- Counters: total executions, error count
- Histogram: duration (ms)
- Tests: metric registration and recording

### Task 4 — Logging

- Structured logger with tool-scoped fields
- Log levels: info, warn, error
- Tests: logger includes tool metadata

### Task 5 — Middleware

- Wrap execution function with tracing/metrics/logging
- Ensure no mutation of input arguments
- Tests: middleware invokes callbacks and records telemetry

### Task 6 — Exporter Setup

- OTLP exporter (grpc/http selectable)
- Jaeger exporter
- Prometheus exporter
- Stdout exporter (dev)
- Tests: invalid exporter config errors

### Task 7 — Docs + Examples

- Update README and docs/index.md with examples
- Add Mermaid flow diagram to user-journey
- Add D2 component diagram in ai-tools-stack

---

## Versioning and Propagation

- **Source of truth**: `ai-tools-stack/go.mod`
- **Version matrix**: `ai-tools-stack/VERSIONS.md` (auto-synced)
- **Propagation**: `ai-tools-stack/scripts/update-version-matrix.sh --apply`
- Tags: `vX.Y.Z` and `toolobserve-vX.Y.Z`

---

## Integration with metatools-mcp

- Wire as middleware in the server pipeline after tool provider selection.
- Span/metric names must match server tool IDs.
- Configuration must be driven via metatools config layer.

---

## Definition of Done

- All TDD tasks complete with tests passing
- `go test -race ./...` succeeds
- Docs include quick start + diagrams
- CI green
- Version matrix updated after first release
