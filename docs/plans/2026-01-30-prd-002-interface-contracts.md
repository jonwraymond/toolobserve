# PRD-002 Interface Contracts — toolobserve

**Status:** Draft
**Date:** 2026-01-30


## Overview
Define explicit interface contracts (GoDoc + documented semantics) for all interfaces in this repo. Contracts must state concurrency guarantees, error semantics, ownership of inputs/outputs, and context handling.


## Goals
- Every interface has explicit GoDoc describing behavioral contract.
- Contract behavior is codified in tests (contract tests).
- Docs/README updated where behavior is user-facing.


## Non-Goals
- No API shape changes unless required to satisfy the contract tests.
- No new features beyond contract clarity and tests.


## Interface Inventory
| Interface | File | Methods |
| --- | --- | --- |
| `Metrics` | `toolobserve/metrics.go:12` | RecordExecution(ctx context.Context, meta ToolMeta, duration time.Duration, err error) |
| `ExtendedLogger` | `toolobserve/logger.go:182` | Logger<br/>WithTool(meta ToolMeta) Logger |
| `Observer` | `toolobserve/observe.go:106` | Tracer() trace.Tracer<br/>Meter() metric.Meter<br/>Logger() Logger<br/>Shutdown(ctx context.Context) error |
| `Logger` | `toolobserve/observe.go:121` | Info(ctx context.Context, msg string, fields ...Field)<br/>Warn(ctx context.Context, msg string, fields ...Field)<br/>Error(ctx context.Context, msg string, fields ...Field)<br/>Debug(ctx context.Context, msg string, fields ...Field)<br/>WithTool(meta ToolMeta) Logger |
| `Tracer` | `toolobserve/tracer.go:44` | StartSpan(ctx context.Context, meta ToolMeta) (context.Context, trace.Span)<br/>EndSpan(span trace.Span, err error) |

## Contract Template (apply per interface)
- **Thread-safety:** explicitly state if safe for concurrent use.
- **Context:** cancellation/deadline handling (if context is a parameter).
- **Errors:** classification, retryability, and wrapping expectations.
- **Ownership:** who owns/allocates inputs/outputs; mutation expectations.
- **Determinism/order:** ordering guarantees for returned slices/maps/streams.
- **Nil/zero handling:** behavior for nil inputs or empty values.


## Acceptance Criteria
- All interfaces have GoDoc with explicit behavioral contract.
- Contract tests exist and pass.
- No interface contract contradictions across repos.

