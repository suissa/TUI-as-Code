# WatchPlan

`WatchPlan` is the compiled intermediate representation of a TUI-as-Code YAML dashboard.

## Why it exists

YAML is authoring syntax, not runtime state.

The compiler should validate and lower configuration before starting event consumption.

```text
YAML
 ↓
ServerWatchCompiler
 ↓
WatchPlan
 ↓
Runtime
```

## Structure

```text
WatchPlan
├── dashboard
├── channels[]
├── subscriptions[]
├── pipelines[]
├── components[]
├── layout
├── actions[]
├── retention
└── capabilities
```

## ChannelPlan

```text
ChannelPlan
├── source_id
├── module
├── config
├── selector_set
├── queue_capacity
└── backpressure_policy
```

## SubscriptionPlan

Represents the minimal transport subscription calculated from all consumers.

```text
SubscriptionPlan
├── source_id
├── selectors[]
└── consumers[]
```

## PipelinePlan

```text
PipelinePlan
├── source_id
├── selectors
├── filters
├── extractors
├── aggregators
├── formatter
└── target_component
```

## ComponentPlan

```text
ComponentPlan
├── id
├── type
├── props
├── styles
├── state_styles
└── bindings[]
```

## LayoutPlan

```text
LayoutPlan
├── fixed_header?
├── viewport
├── fixed_progress?
├── fixed_footer?
└── breakpoints[]
```

Each breakpoint contains a prevalidated layout tree.

## Runtime loop

```text
channels
   ↓
bounded event queues
   ↓
normalize
   ↓
pipeline routing
   ↓
aggregators/state
   ↓
component update messages
   ↓
MVU update
   ↓
render
```

## Backpressure

Every channel must have a bounded queue.

Policies:
- `drop_oldest`
- `drop_newest`
- `sample`
- `block` only when safe for that transport

Observability defaults should prefer preserving errors and dropping redundant high-frequency success samples.

## Pause

UI pause must not necessarily stop ingestion.

Recommended behavior:

```text
[p] pause
   ↓
freeze displayed snapshot
   ↓
continue ingesting into bounded ring buffer
```

Resume reconciles to current state.

## Failure windows

When an error event occurs, retain a configurable window around it.

```text
              Error
                ▼
───────┬────────●────────┬───────
      -30s              +10s
```

Metrics, logs, traces and events within this window may be correlated by:
- correlation_id
- causation_id
- trace_id
- agent/action label
- timestamp

## Determinism

A WatchPlan should be serializable/hashable so the same configuration yields the same structural plan.
