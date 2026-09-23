# TUI-as-Code

Declarative, event-driven Terminal User Interfaces defined entirely in YAML.

TUI-as-Code turns a YAML dashboard definition into a compiled `WatchPlan` that subscribes only to the required events, extracts and aggregates values, and renders them through a TUI engine such as ChappieTUI.

## Core idea

> The dashboard does not live in application code. The dashboard lives in YAML.

The runtime provides only reusable factories and primitives:

```text
configs/allas.server-watch.yml
              │
              ▼
      ServerWatchCompiler
              │
       ┌──────┴───────┐
       ▼              ▼
 ChannelFactory    ViewFactory
       │              │
 ┌─────┼─────┐    ChappieTUI
 │     │     │
stdio REST  WS NATS QUIC
       │
       ▼
 selective listen
       │
       ▼
 normalized events
```

## Architecture

```text
TUI-as-Code
├── Config DSL (YAML)
├── ServerWatchCompiler
│   ├── parser
│   ├── validator
│   ├── subscription planner
│   ├── pipeline compiler
│   ├── layout compiler
│   └── component compiler
├── WatchPlan
│   ├── ChannelPlan[]
│   ├── SubscriptionPlan[]
│   ├── PipelinePlan[]
│   ├── ComponentPlan[]
│   └── LayoutPlan
├── ChannelFactory
│   ├── stdio
│   ├── rest
│   ├── ws
│   ├── nats
│   └── quic
├── Pipeline
│   ├── selector
│   ├── filter
│   ├── extractor
│   ├── aggregator
│   └── formatter
└── ComponentFactory
    ├── layout primitives
    ├── data components
    ├── navigation components
    ├── overlays
    └── observability components
```

## Compilation model

```text
YAML
 ↓
Parse
 ↓
Validate
 ↓
Resolve channels/components
 ↓
Plan minimal subscriptions
 ↓
Compile pipelines
 ↓
Compile responsive layout
 ↓
WatchPlan
 ↓
Runtime execution
```

The YAML is parsed once. Runtime hot paths operate from the compiled `WatchPlan`, not from repeated YAML interpretation.

## Event pipeline

Every visible value follows the same model:

```text
Channel
   ↓
Selector
   ↓
Filter
   ↓
Extractor
   ↓
Aggregator
   ↓
Formatter
   ↓
Component
```

## Selective listening

A source-level `listen` is the coarse subscription filter. A block-level `listen` is the fine filter.

Example:

```yaml
sources:
  distributed:
    channel: nats
    listen:
      - Metrics.Payment.*
      - Logs.Payment.*
      - Traces.Payment.*

blocks:
  - id: failures
    source: distributed
    listen:
      - Logs.Payment.Error
```

The compiler must calculate the smallest safe transport subscription instead of listening to every event and dropping most of them afterwards.

## Config concepts

- `listen`: which events are accepted.
- `where`: predicates applied after subscription.
- `value.path`: which field is extracted.
- `aggregate`: optional time/window aggregation.
- `format`: optional presentation formatting.
- `component`: how the value is rendered.
- `layout`: where the component appears.
- `retention`: how much local state is retained.
- `actions`: keyboard/mouse interactions.
- `style`: declarative visual styling.

## Dynamic modules

Dynamic modules are optional and most useful for channels and advanced components.

Recommended split:

```text
built-in:
  stdio
  counter
  text
  list
  progress
  box
  flex

dynamic:
  nats
  quic
  websocket
  trace-tree
  flamegraph
  histogram
  image
```

A minimal server should not load transports or widgets it does not use.

## AllasCode use case

`allas watch` can load:

```text
configs/allas.server-watch.yml
```

and render a live observability dashboard fed by the in-memory broker, NATS, stdio, REST, WebSocket or QUIC.

See:
- `docs/SPEC.md`
- `docs/COMPONENTS.md`
- `docs/CHANNELS.md`
- `docs/WATCHPLAN.md`
- `configs/allas.server-watch.yml`
