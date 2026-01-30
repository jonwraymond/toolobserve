# Changelog

## [0.2.0] - 2026-01-29

### Added

- **Config + Observer Core** (`observe.go`)
  - `Config` struct with `TracingConfig`, `MetricsConfig`, `LoggingConfig`
  - `Observer` interface with `Tracer()`, `Meter()`, `Logger()`, `Shutdown()`
  - `NewObserver()` factory with config validation
  - Support for service name, version, sample percentage, log levels

- **Tracing** (`tracer.go`)
  - `ToolMeta` struct for tool metadata
  - `SpanName()` returns deterministic `tool.exec.<namespace>.<name>`
  - `Tracer` interface with `StartSpan()` / `EndSpan()`
  - Span attributes: tool.id, tool.namespace, tool.name, tool.version, tool.category, tool.tags, tool.error

- **Metrics** (`metrics.go`)
  - `Metrics` interface with `RecordExecution()`
  - `tool.exec.total` counter for all executions
  - `tool.exec.errors` counter for failed executions
  - `tool.exec.duration_ms` histogram for execution duration

- **Logging** (`logger.go`)
  - `Logger` interface with `Info()`, `Warn()`, `Error()`, `Debug()`, `WithTool()`
  - Structured JSON output with tool context fields
  - Log level filtering (debug/info/warn/error)
  - Automatic input redaction for sensitive fields

- **Middleware** (`middleware.go`)
  - `Middleware` struct combining tracing, metrics, logging
  - `Wrap()` method to instrument `ExecuteFunc`
  - `MiddlewareFromObserver()` convenience factory

- **Exporters** (`exporters/`)
  - `NewTracingExporter()`: stdout, otlp, jaeger, none
  - `NewMetricsReader()`: stdout, otlp, prometheus, none
  - OTLP requires `OTEL_EXPORTER_OTLP_ENDPOINT` environment variable
  - Jaeger uses native OTLP support

## [0.1.0] - 2026-01-29

- Initial scaffolding: module, docs, CI, and planning artifacts.
