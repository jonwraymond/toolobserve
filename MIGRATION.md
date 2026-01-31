# Migration Guide: toolobserve to toolops/observe

This guide helps you migrate from `github.com/jonwraymond/toolobserve` to `github.com/jonwraymond/toolops/observe`.

## Import Path Changes

Update your imports from:

```go
import "github.com/jonwraymond/toolobserve"
```

To:

```go
import "github.com/jonwraymond/toolops/observe"
```

## Package Name Change

The package name changes from `toolobserve` to `observe`. Update all references accordingly:

| Before | After |
|--------|-------|
| `toolobserve.Config` | `observe.Config` |
| `toolobserve.NewObserver` | `observe.NewObserver` |
| `toolobserve.TracingConfig` | `observe.TracingConfig` |
| `toolobserve.MetricsConfig` | `observe.MetricsConfig` |
| `toolobserve.LoggingConfig` | `observe.LoggingConfig` |
| `toolobserve.MiddlewareFromObserver` | `observe.MiddlewareFromObserver` |
| `toolobserve.ToolMeta` | `observe.ToolMeta` |

## go.mod Update

Replace the dependency in your `go.mod`:

```bash
# Remove old dependency
go mod edit -droprequire github.com/jonwraymond/toolobserve

# Add new dependency
go get github.com/jonwraymond/toolops/observe@latest
```

## Quick Migration Script

For larger codebases, you can use this script to automate the migration:

```bash
#!/bin/bash
# migrate-toolobserve.sh

# Find and replace import paths
find . -name "*.go" -type f -exec sed -i '' \
    's|github.com/jonwraymond/toolobserve|github.com/jonwraymond/toolops/observe|g' {} +

# Find and replace package references
find . -name "*.go" -type f -exec sed -i '' \
    's|toolobserve\.|observe.|g' {} +

# Update go.mod
go mod edit -droprequire github.com/jonwraymond/toolobserve
go get github.com/jonwraymond/toolops/observe@latest
go mod tidy
```

## Example: Before and After

### Before

```go
package main

import (
    "context"
    "log"

    "github.com/jonwraymond/toolobserve"
)

func main() {
    ctx := context.Background()

    cfg := toolobserve.Config{
        ServiceName: "my-service",
        Version:     "1.0.0",
        Tracing: toolobserve.TracingConfig{
            Enabled:  true,
            Exporter: "stdout",
        },
    }

    observer, err := toolobserve.NewObserver(ctx, cfg)
    if err != nil {
        log.Fatal(err)
    }
    defer observer.Shutdown(ctx)
}
```

### After

```go
package main

import (
    "context"
    "log"

    "github.com/jonwraymond/toolops/observe"
)

func main() {
    ctx := context.Background()

    cfg := observe.Config{
        ServiceName: "my-service",
        Version:     "1.0.0",
        Tracing: observe.TracingConfig{
            Enabled:  true,
            Exporter: "stdout",
        },
    }

    observer, err := observe.NewObserver(ctx, cfg)
    if err != nil {
        log.Fatal(err)
    }
    defer observer.Shutdown(ctx)
}
```

## API Compatibility

The `toolops/observe` package maintains full API compatibility with `toolobserve`. All types, functions, and methods have the same signatures; only the import path and package name have changed.

## Getting Help

If you encounter issues during migration, please open an issue on the [toolops repository](https://github.com/jonwraymond/toolops/issues).
