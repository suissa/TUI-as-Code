# Channel Factory

Channels are transport adapters. They do not own dashboard semantics.

## Canonical contract

```text
Channel
├── open(ChannelConfig)
├── listen(SelectorSet) -> EventStream
├── poll() -> optional EventStream
├── close()
├── capabilities()
└── health()
```

Normalized event:

```text
NormalizedEvent
├── source_id
├── channel
├── label
├── timestamp
├── correlation_id?
├── causation_id?
├── trace_id?
├── span_id?
└── payload
```

## stdio

Use for child process/runtime streaming.

Required:
- line mode
- NDJSON mode
- bounded buffering
- backpressure policy
- malformed-frame handling

## REST

REST is normally polling rather than listen.

Required:
- GET endpoint
- poll interval
- timeout
- headers
- auth secret references
- JSON decoding
- conditional requests where available

## WebSocket

Required:
- reconnect
- exponential backoff
- ping/pong
- bounded queue
- selector/filter lowering where the remote endpoint supports it

## NATS

Required:
- subject-level selective subscriptions
- wildcard lowering
- reconnect
- queue/backpressure
- optional JetStream mode later

The compiler must prefer minimal subscriptions.

Example requested events:

```text
Metrics.Payment.*
Action.*.Error
```

must not become a global `>` subscription unless unavoidable.

## QUIC

Required:
- connection lifecycle
- stream mapping
- mTLS support where configured
- reconnect
- bounded streams/queues
- event framing

QUIC may transport normalized NDJSON, binary frames or an AllasCode-specific event protocol.

## Dynamic module model

Channels may be statically linked or loaded as dynamic modules.

Suggested interface boundary:

```text
ChannelModule
├── metadata()
├── create(allocator, config) -> Channel
└── destroy(Channel)
```

Dynamic loading is particularly useful for NATS/QUIC/WS so a minimal binary does not include every transport.
