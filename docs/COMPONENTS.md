# Component Catalog

TUI-as-Code requires a stable component vocabulary. Components must be instantiable by `ComponentFactory` from YAML only.

## Layout primitives

### absolute
Position child regions using explicit x/y and width/height constraints.

### box
Container with padding, margin, border, background, min/max dimensions and child content.

### flex
Responsive row/column layout with:
- fixed size
- intrinsic size
- flex grow/shrink
- min/max
- gap
- justify
- align

### layout
Generic layout node and region abstraction used by all other layout components.

### stack
Overlay children in z-order. Required for modal, tooltip, toast and floating panels.

### grid
N-column/N-row layout for wide dashboards.

### spacer
Consumes flexible or fixed empty space.

### center
Centers a child horizontally and/or vertically.

### sized_box
Constrains a child to explicit/min/max size.

### split_view
Two-pane horizontal/vertical layout with proportional or fixed divider.

## Viewport and navigation

### scroll_view
Scrollable clipped region.

### scroll_bars
Visual scroll position.

### list_view
Selectable/virtualized list.

### tabs
Switches between named views.

### pagination
Page navigation for finite datasets.

### navbar
Responsive top/side navigation.

### breadcrumb
Hierarchical navigation path.

## Text and input

### text
Simple immutable text.

### rich_text
Multiple independently styled spans on one line/block.

### text_input
Single-line editor.

### text_area
Multiline editor.

### label_value
Two-style inline label/value rendering, useful for fixed headers.

## Status and data display

### counter
Scalar live value.

### gauge
Bounded scalar.

### progress
Horizontal/vertical progress indicator.

### animated_progress
Separates `actual_value` from `display_value` so visual movement can interpolate without falsifying data.

### spinner
Tick-driven loading animation.

### status
Maps state to icon/style.

### badge
Compact inline status/value.

### table
Rows/columns with optional sorting.

### tree
Hierarchical data.

### key_value
Structured field/value list.

### timeline
Chronological event rendering.

## Observability

### metric
Generic live metric renderer.

### metric_grid
Multiple named metrics.

### log_view
Streaming log list with level styling.

### trace_view
Trace/span hierarchy.

### trace_tree
Hierarchical spans with duration/status.

### histogram
Bucketed values.

### sparkline
Compact historical series.

### event_stream
Chronological event list.

### failure_panel
Error-focused panel.

### failure_window
Temporal window around a failure.

## Feedback and overlays

### alert
Persistent status message.

### modal
Blocking overlay with focus capture.

### toast
Transient notification.

### tooltip
Contextual overlay.

### accordion
Expandable/collapsible section.

## Interaction

### button
Keyboard/mouse action.

### checkbox
Boolean input.

### radio
Single-choice input.

### menu
Selectable actions.

## Rendering foundation required by the TUI engine

These are not YAML components but are runtime prerequisites:

- cell framebuffer
- incremental diff renderer
- Unicode display width
- grapheme segmentation
- clipping
- focus tree
- terminal capability detection
- synchronized output
- resize events
- mouse events
- keyboard protocol abstraction
- RichText/Span rendering
- responsive region measurement

## Component contract

Every component should conceptually implement:

```text
Component
├── init(ComponentConfig)
├── update(ComponentMsg)
├── measure(Constraints) -> Size
├── render(RenderContext, Region)
├── focusable() -> bool
└── subscriptions() -> optional local UI subscriptions
```

Components do not connect directly to NATS/REST/etc. They consume normalized values produced by pipelines.
