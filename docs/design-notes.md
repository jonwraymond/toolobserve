# Design Notes: Observability Boundaries

## Core Principles

- **No execution logic**: toolobserve never executes tools; it only observes.
- **Pure instrumentation**: create spans/metrics/logs around provided callbacks.
- **Context-first**: all APIs accept `context.Context` and propagate it.
- **Zero side effects by default**: exporters are opt-in and configurable.

## Telemetry Naming

- Spans follow `tool.<namespace>.<name>` naming.
- Attributes include `tool.id`, `tool.namespace`, `tool.name`, `tool.version`.
- Metrics use stable names (`tool.exec.duration_ms`, `tool.exec.errors_total`).

## Error Semantics

- Errors are recorded on spans and emitted as structured log fields.
- No panics; errors are returned to callers and logged with context.

## Exporter Strategy

- Exporter config is declarative and validated at startup.
- Multiple exporters can be enabled simultaneously.
- Exporter failures do **not** crash the process; they are logged.

## Concurrency

- Observer and metric registries are safe under concurrent use.
- Middleware must be re-entrant and avoid shared mutable state.

## Integration Points

- `toolrun`: wrap `RunTool` and `RunChain` execution.
- `toolruntime`: wrap backend execution calls.
- `metatools-mcp`: wire middleware in server pipeline.
