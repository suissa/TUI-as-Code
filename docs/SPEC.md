# TUI-as-Code YAML Specification

## 1. Root schema

```yaml
version: 1

dashboard:
  id: server-watch
  title: AllasCode Server Watch
  refresh: 100ms

sources: {}
layout: {}
blocks: []
footer: {}
```

## 2. Dashboard

```yaml
dashboard:
  id: string
  title: string
  refresh: duration
  retention:
    max_events: integer
    max_age: duration
  pause:
    buffer_while_paused: true
```

## 3. Sources

A source defines transport and coarse event selection.

```yaml
sources:
  runtime:
    channel: stdio
    listen:
      - Runtime.*

  distributed:
    channel: nats
    connection: runtime
    listen:
      - Metrics.Payment.*
      - Logs.Payment.*
      - Traces.Payment.*

  gateway:
    channel: ws
    endpoint: /observability
    listen:
      - Gateway.*

  api:
    channel: rest
    endpoint: http://127.0.0.1:8080/metrics
    poll: 1s

  edge:
    channel: quic
    endpoint: 127.0.0.1:8443
    listen:
      - Edge.*
```

## 4. Block

```yaml
blocks:
  - id: metrics
    title: METRICS
    source: distributed
    component: grid

    listen:
      - Metrics.Payment.*

    where:
      payload.environment: development

    values:
      - label: Requests/s
        event: Metrics.Payment.RequestRate
        path: payload.value
        aggregate:
          type: rate
          window: 10s
        component: counter
```

## 5. Event matching

Patterns must support exact and wildcard labels.

```yaml
listen:
  - Payment.CreatePix.Ok
  - Payment.*.Error
  - Metrics.>
```

The compiler should lower patterns into the most efficient selector supported by the channel.

## 6. Filtering

```yaml
where:
  payload.currency: BRL
  payload.environment: development
```

Future extension:

```yaml
where:
  all:
    - path: payload.duration
      op: gt
      value: 100
    - path: payload.status
      op: eq
      value: error
```

Required operators:
- eq
- neq
- gt
- gte
- lt
- lte
- contains
- starts_with
- ends_with
- exists

## 7. Extraction

```yaml
value:
  label: Amount
  path: payload.amount
```

Multiple values:

```yaml
values:
  - label: Action
    path: payload.action

  - label: Duration
    path: payload.duration
```

## 8. Aggregation

Required aggregators:

```yaml
aggregate:
  type: count | sum | min | max | avg | rate | percentile | latest | distinct
  window: 60s
  value: 95
```

`value` is used when required, for example percentile 95.

## 9. Formatting

```yaml
format:
  type: number | duration | bytes | percentage | currency | text
  precision: 2
  prefix: ""
  suffix: ""
```

## 10. Style

```yaml
style:
  foreground: white
  background: black
  bold: true
  italic: false
  underline: false

  header:
    foreground: black
    background: yellow
```

State-aware styling:

```yaml
states:
  running:
    foreground: yellow
  success:
    foreground: green
  failure:
    foreground: red
```

## 11. Responsive layout

Breakpoints are based on terminal cells, not pixels.

```yaml
layout:
  type: responsive

  breakpoints:
    compact: 0
    medium: 120
    wide: 180

  compact:
    type: flex
    direction: column

  medium:
    type: grid
    columns: 2

  wide:
    type: grid
    columns: 3
```

## 12. Fixed regions

```yaml
header:
  fixed: true

footer:
  fixed: true

progress:
  fixed: true
```

Only the content viewport should scroll.

## 13. Actions

```yaml
footer:
  actions:
    - key: enter
      action: details

    - key: f
      action: filter

    - key: p
      action: pause

    - key: q
      action: quit
```

Actions should emit messages into the host MVU/runtime instead of executing arbitrary application code.

## 14. Retention

```yaml
retention:
  mode: ring
  max_events: 10000
  max_age: 5m
```

For failure reconstruction:

```yaml
failure_window:
  before: 30s
  after: 10s
```

## 15. Security

Configuration MUST NOT provide arbitrary shell execution.

Channels, formatters, aggregators and components must be resolved from registered factories/modules.

Secrets must be referenced indirectly:

```yaml
connection:
  secret_ref: env:NATS_TOKEN
```

Secret values must never be rendered unless a component explicitly receives already-redacted data.
