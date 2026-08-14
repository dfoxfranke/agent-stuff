---
name: "logging-in-rust-fallback"
description: >-
  FALLBACK ONLY: Do not invoke this skill when $logging-in-rust is available;
  use that repository-scoped skill instead. Use this skill when writing or
  reviewing Rust code that emits log messages or `tracing` events, including
  choosing frameworks and levels, placing events, defining structured fields
  and names, handling authentication events, and controlling log volume. Do
  not use it for log-handling infrastructure that only processes messages
  and emits none of its own.
---

# Logging in Rust

## Select the primary skill first

If `$logging-in-rust` is available, stop using this fallback and use that skill
instead. Never apply both `$logging-in-rust` and `$logging-in-rust-fallback` to
the same task.

The goal is to emit useful operational events at the layer and severity that
reflect their meaning, without duplicating error handling or creating avoidable
production noise.

## Resolve the codebase's logging convention

Before adding, changing, or reviewing an event, determine the convention that
applies to the emitting component.

Use this precedence order:

1.  **Explicit repository instructions.** Follow applicable `AGENTS.md`,
    contributor documentation, style guides, and logging configuration.
2.  **Established framework and tooling.** Inspect Cargo dependencies, feature
    flags, subscriber or logger configuration, and the logging macros already in
    use.
3.  **Existing analogous emitters.** Inspect several comparable call sites,
    preferring code in the same crate or component, at the same application or
    library layer, and with a similar operational purpose.

Do not infer a convention from one isolated call site when more evidence is
available.

Resolve these choices independently:

- **Framework:** continue using `tracing` or `log` when the relevant component
  has already chosen one. If a new Rust project has no logging framework or
  precedent, prefer `tracing` and its ecosystem. Do not introduce a second
  framework merely for an isolated event.
- **Explicit event names:** determine whether the component omits names from
  every event or names `info`, `warn`, and `error` events while leaving `debug`
  and `trace` unnamed. Preserve a clear local policy. If the policy is mixed or
  absent, name the higher-level events when logs may be centrally collected,
  queried, aggregated, alerted on, or consumed programmatically; otherwise omit
  explicit names.
- **Field-key taxonomy:** reuse established keys for the same concepts and
  determine how much context belongs in each key.
- **Severity calibration:** inspect analogous events to understand the
  component's operational boundaries, then apply the level rules below.

If a repository mixes conventions, follow the convention of the relevant
component unless more-specific instructions require consolidation. Do not change
an established framework or naming policy merely to match a general Rust
preference.

## Core rules

### Choose the level from the event's operational meaning

Classify the event by what happened at the emitting layer:

- **`error`:** an intended operation failed because something went wrong.
- **`warn`:** an operation behaved abnormally, may have produced an incorrect
  result, or created a condition that warrants attention before it becomes an
  error.
- **`info`:** a significant but normal state transition occurred.
- **`debug`:** an internal state transition occurred that is useful for
  diagnosis but is not ordinarily significant to a user.
- **`trace`:** the event reports control flow or the current state of execution
  without representing a state transition.

Require an externally visible consequence from the underlying operation for an
`info`, `warn`, or `error` event. For a `debug`, `info`, `warn`, or `error`
event, require a state transition, although it may be internal to the process at
`debug`. Treat an operation completing or failing as a state transition. Use
`trace` for execution-state observations that do not meet that threshold.

Do not choose a level merely because a value has the type `Result` or a message
contains the word "error." Apply the abnormal-condition and authentication rules
below before assigning a severity.

### Add debug and trace events only for a concrete diagnostic need

Before adding a `debug` event, identify a plausible diagnostic question or
recurring failure mode that the event would help resolve. Do not add debug
events merely because a location might someday be interesting.

Add `trace` events only while investigating a concrete problem. Before
completing that investigation, remove the trace events introduced for it. Do not
remove pre-existing trace instrumentation outside the task's scope merely
because it does not appear useful to the current investigation.

After removing investigation-only trace events, identify whether one state
transition would have pointed directly to the problem. If a corresponding
`debug` event is likely to help with the same class of problem again, add that
event.

### Log abnormal conditions at the layer that disposes of them

Choose one layer to decide the disposition of an abnormal condition. If a
function returns or propagates that condition as an error, the caller retains
responsibility for its disposition; do not also emit an `info`, `warn`, or
`error` event for the same condition. Wrapping the error before returning it
does not change this rule.

A function may add `debug` or `trace` context and still return the error because
those levels do not dispose of the condition.

Do not assume every `Err` is abnormal in the application that receives it.
Reusable library code that lacks enough application context to determine an
error's operational significance must leave `info`, `warn`, and `error`
disposition to its caller. Such code may emit `debug` or `trace` events when
they meet the other rules in this skill. When no evidence establishes that a
library owns higher-level operational events, default it to `debug` and `trace`
and leave higher levels to the application.

### Do not duplicate panics in logs

When code must panic, perform any cleanup or invariant restoration required for
panic safety and then panic. Do not emit a log event immediately beforehand that
duplicates the panic message or reports the same failure a second time.

### Put dynamic semantic data in structured fields

When using `tracing`, record every dynamic fact with semantic significance as a
structured field. A human-readable message may repeat or format those fields,
but it must not be the only place where the information appears.

A presentation-only value need not have its own field when it is determined
entirely by a recorded field. For example:

``` rust
let s = if number == 1 { "" } else { "s" };
info!(name: "blegs_frobnicated", number, "frobnicated {number} bleg{s}");
```

Here `number` carries the semantic data. The unrecorded `s` only makes the
message grammatical and introduces no independent information.

### Apply one explicit-event-name policy consistently

Absent a more-specific established local policy, use one of these policies
throughout a component:

1.  Leave `trace` and `debug` events unnamed, and assign explicit names to every
    `info`, `warn`, and `error` event.
2.  Do not assign explicit names to any events.

Use the first policy when higher-level events need stable identities for
centralized collection, queries, aggregation, alerting, or other programmatic
consumption. Use the second when logs serve only as local, human-readable
diagnostics. Follow an established local policy when one exists.

### Name fields for consistent queries

Reuse the same key for the same concept throughout the relevant logging domain.
Start with the least-qualified key that remains unambiguous in the emitting
component, and add at most one contextual qualifier when necessary.

For example, a general-purpose filesystem component may use `file`, while a
configuration parser may need `config_file`. Do not repeat context already
implied by the crate's purpose: Rust-specific source tooling should prefer
`src_file` to `rust_src_file`.

Represent independent dimensions as separate fields instead of adding multiple
qualifiers to one key. If source tooling later supports multiple languages, keep
`src_file` and add a separate field such as `language = "rust"`.

### Decide logging separately from user-interface feedback

Decide independently whether a condition requires immediate user feedback and
whether it is operationally useful to log.

Do not log invalid input that the user interface handles completely without
taking an action. For example, let `clap` report a command-line parse failure
without adding a log event for it.

Never rely on a log message to provide required user feedback. Giving the user
feedback and logging the same underlying condition is acceptable when each
communication has an independent purpose; duplication between the two is not
itself a defect.

### Classify authentication from the emitter's perspective

On a client, treat an authentication failure as an `error` when it prevents the
requested operation and the emitting layer owns the failure's disposition.

On a server, successfully rejecting invalid credentials means the server behaved
as designed. If an individual rejection has diagnostic or administrative value,
record it at `debug` or `info` according to that value; otherwise omit it. Never
record a correct rejection at `warn` or `error`.

Record every accepted server authentication at least as severely as a rejected
authentication. Use `info` for routine successful logins. Use `warn` when a
successful login is itself abnormal enough to demand attention, such as access
to an administrator account that should be used rarely.

Distinguish a correct credential rejection from a failure of the authentication
system, such as an unavailable identity provider. Classify a system failure
under the ordinary level rules, and emit its disposition event only at the layer
that stops returning or propagating it as an error.

### Bound production log volume

Before adding an event on a request, input item, retry, or other potentially
unbounded path, evaluate how many events attacker-controlled or worst-case
traffic can produce. Treat the change from no per-item event to any per-item
event as the first scaling boundary, then account for the number and size of
events each item produces.

If a fact is useful only in aggregate, prefer recording it as a metric, using
the project's metric system or the `metrics` crate where appropriate. When
human-readable reporting is also useful, consider logging aggregate summaries at
slow, regular intervals. Individual events may remain at `debug` when production
deployments normally disable that level.

## Author procedure

When writing or revising Rust logging:

1.  Resolve the component's framework, event-name policy, field-key taxonomy,
    and severity convention.
2.  Decide separately whether the condition belongs in a log, in user-interface
    feedback, in a metric, or in none of them.
3.  Identify the state transition or execution-state observation that the event
    represents. Treat an operation completing or failing as a state transition.
4.  For an abnormal condition, identify the layer that owns its disposition. Do
    not emit an `info`, `warn`, or `error` event from a function that returns or
    propagates the same condition as an error.
5.  Choose the level from the event's operational meaning and visibility, not
    from the vocabulary or Rust type used at the call site.
6.  Evaluate event volume under worst-case and attacker-controlled inputs.
    Prefer metrics and periodic summaries to per-item events for aggregate-only
    information.
7.  When using `tracing`, move every dynamic semantic fact into a structured
    field and apply the established field-key taxonomy.
8.  Apply the component's explicit-event-name policy consistently.
9.  For authentication events, classify the outcome from the client or server's
    perspective and ensure successful server authentications are recorded at
    least as severely as rejections.
10. Remove trace events introduced for a completed investigation, then consider
    whether one durable debug event would make the same class of problem easier
    to diagnose.
11. Re-read the affected call path and remove duplicate disposition logs,
    duplicate pre-panic logs, and events that no longer meet their level's
    threshold.

## Reviewer procedure

When reviewing Rust logging, first resolve the same codebase convention an
author should have used.

Then report a finding when code:

- introduces a logging framework or explicit-event-name policy inconsistent with
  the relevant component without a project-specific reason;
- assigns a level that does not match the event's operational meaning;
- emits at `info`, `warn`, or `error` without an externally visible consequence;
- emits at `debug`, `info`, `warn`, or `error` without a state transition,
  including an operation completing or failing;
- adds a debug event without a plausible diagnostic use or leaves
  investigation-only trace events in the completed change;
- emits at `info`, `warn`, or `error` for an abnormal condition and also returns
  or propagates that condition as an error;
- duplicates a panic in a log event immediately before panicking;
- emits at `info`, `warn`, or `error` from reusable library code that lacks the
  context to determine the condition's application-level significance;
- places dynamic semantic information only in a `tracing` message instead of a
  structured field;
- violates the component's explicit-event-name policy or established field-key
  taxonomy;
- encodes multiple independent dimensions as qualifiers in one field key;
- logs invalid input that the user interface handles completely without taking
  an action;
- relies on a log event to provide required user feedback;
- records an individual server-side authentication rejection at `warn` or
  `error`;
- treats an authentication-system failure as a credential rejection or emits an
  `info`, `warn`, or `error` event before returning or propagating it;
- omits an accepted server authentication or records it less severely than a
  rejection; or
- adds potentially unbounded production logging without accounting for
  worst-case volume, especially when the information is meaningful only in
  aggregate.

Be conservative about requests to add logging. Do not require an event merely
because an `Err` exists or an event might occasionally be useful.

Do not report a finding merely because user-interface feedback and a log event
describe the same condition, an explicit event name is absent under the unnamed
policy, reusable library code limits itself to `debug` and `trace`, or
pre-existing trace instrumentation exists outside the change's scope.

## Logging stance

For both writing and review:

- Prefer the established local framework over logging-framework churn.
- Prefer one disposition event over repeated logs along an error's propagation
  path.
- Prefer structured fields over message-only semantic data.
- Prefer a durable state-transition debug event over permanent control-flow
  tracing.
- Prefer metrics over high-volume events that matter only in aggregate.
- Prefer the emitter's operational perspective over severity inferred from
  generic labels such as "error" or "authentication failure."
