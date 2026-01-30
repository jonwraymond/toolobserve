# Changelog

## [0.2.0](https://github.com/jonwraymond/toolobserve/compare/toolobserve-v0.1.0...toolobserve-v0.2.0) (2026-01-30)


### Features

* **toolobserve:** add config validation and observer core ([e717465](https://github.com/jonwraymond/toolobserve/commit/e717465e21e075eadfd04df68b8547074280e2e5))
* **toolobserve:** add execution middleware ([bfde9bf](https://github.com/jonwraymond/toolobserve/commit/bfde9bf35f6c8969bb81ea145064f08718b0b0f3))
* **toolobserve:** add exporter configuration ([f2697e8](https://github.com/jonwraymond/toolobserve/commit/f2697e80c4c71ad5c6bae055957712fade01da3c))
* **toolobserve:** add metrics ([1869b3f](https://github.com/jonwraymond/toolobserve/commit/1869b3f72af0d4f4e024a65483f497624c691cfc))
* **toolobserve:** add structured logger ([c6e651c](https://github.com/jonwraymond/toolobserve/commit/c6e651cd314f1a999c5cf3f782fcdc0aa4119d51))
* **toolobserve:** add tracing ([437cac2](https://github.com/jonwraymond/toolobserve/commit/437cac2eeafca9903b41928801f6812ea3a413de))


### Bug Fixes

* **toolobserve:** wire exporters and structured logger ([0e6213a](https://github.com/jonwraymond/toolobserve/commit/0e6213a408188f5236243e5b12362719fed25ea1))

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
