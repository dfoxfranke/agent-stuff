---
name: logging-in-rust
description: How to craft log messages and tracing events when working in Rust. Use when writing or reviewing Rust code that generates log messages. Not applicable for work on log-handling infrastructure which processes log messages but does not produce any of its own.
---

# Logging in Rust

## Framework choice

When adding logging to a new Rust project that doesn't have any yet, prefer the
`tracing` crate and its surrounding ecosystem over `log`. If `log` is already in
use, continue using that instead of `tracing`, but continue to apply the rest of
these guidelines insofar as they are still applicable.

## Choosing a log level

Log levels should be understood as follows:

* `error`: Something should have happened, but it didn't, because something went
  wrong in the attempt.
* `warn`: Something happened abnormally, may have happened incorrectly, or
  created a condition that warrants attention before it becomes an error.
* `info`: Something significant, but normal, happened.
* `debug`: Something happened, but probably not of any significance to the user
  unless the user is trying to debug something.
* `trace`: Does not represent that anything has happened; we're reporting on the
  state of program execution and nothing more.

Anything reported at a level of `info` or higher should have some kind of
visible consequence from outside the executing process.

Anything reported at a level of `debug` or higher should represent some kind of
state change, but not necessarily one visible outside the process. For example,
writing to a channel or updating a state variable can be reported as a debug
event.

## Debug and trace logging

`debug` events should be added sparingly, only when there is good reason to
anticipate that they will eventually be useful. `trace` events should never
be added proactively: wait until there is a concrete problem that you are using
them to investigate.

Think of `trace` logging as a more civilized form of printf-debugging. Unlike
the cruder form, it is acceptable to commit, but still should not be a permanent
feature of the codebase. Remove trace events introduced for the current
investigation once they are no longer needed. Do not remove pre-existing trace
instrumentation outside the task’s scope merely because the surrounding code
appears well tested. After removing them, consider whether there is some single
debug event, meeting the "something happened" threshold, which would have
immediately pointed to the problem that you were using the trace events to
discover if it had been present. If such a debug event is likely to be useful
again in the future, add it.

## Logging abnormal conditions

Functions should either dispose locally of abnormal conditions, or indicate them
through an error return — never both. An error return means it's up to the
caller to decide the disposition of the error. Logging an error *is* its
disposition, or at least a part of it. However, logging at `debug` or `trace` is
not considered dispositive: functions may log at one of these levels and also
provide an error return.

When code needs to panic, do not log the failure immediately beforehand. Perform
any cleanup or invariant restoration required for panic safety, then panic
without duplicating the panic message in a log event.

Not every `Err` result is truly an abnormal condition. Crates must not log
errors if they lack sufficient context to know whether or not the `Err` is
really abnormal. For example, it would be very bad for a library crate to log
every connection failure as an error, if the application using that crate turned
out to be a port scanner. The safest policy is for library crates to log only at
`debug` or `trace` level, and leave `info`/`warn`/`error` to the application.

## Structured logging

When using `tracing`, take full advantage of its support for structured logging.
All dynamic semantic data associated with an event must be recorded in
structured fields. Values substituted into the human-readable message must carry
no information that is absent from those fields. This, however, is fine:

```rust
let s = if number == 1 { "" } else { "s" };
info!(name: "blegs_frobnicated", number, "frobnicated {number} bleg{s}");
```

The `s` is just a grammatical affordance whose value is already a function of
`number`, so you wouldn't want or need to add a structured `plural` key.

When you don't supply a `name` argument to a `tracing` macro, it defaults to the
file name and line number of the macro-invocation site. Codebases should
consistently follow one or the other of these two policies:

1. Never supply a `name` for `trace` and `debug` events, but always supply one
   for `info`, `warn`, and `error` events.

2. Never supply a `name` at all.

As discussed above, library crates won't usually be logging at
`info`/`warn`/`error`, in which case these policies collapse to the same thing.
For application development, use policy 1 when events may be centrally
collected, queried, aggregated, alerted on, or consumed programmatically. Use
policy 2 when logs are intended only for local, human-readable diagnostics.

## Naming event keys

The naming of keys in `tracing` events should seek to adhere to a consistent
taxonomy that facilitates their use in search queries. The specificity used in
naming the key should be commensurate with the specificity-of-purpose of the
code which emits the event. For example, a general-purpose filesystem facility
could use a key just named `file`, but a configuration parser ought to use
`config_file` instead. However, do not add qualifiers that are already implied
by the overall purpose of the crate. For example, Rust-specific tooling that
reads a source file should just use `src_file` rather than `rust_src_file`.
Names of keys should not carry more than one qualifier. This makes
`rust_src_file` bad for a second reason: if the same hypothetical crate adds
support for additional languages, it should still just use `src_file`, and add
`language=rust` as a separate key/value pair.

## Log messages vs. UI feedback

Log messages and messages to the user interface are separate magisteria. Errors
in user input that are handled within the UI and do not result in any action
being taken, should not be logged. For example, if a tool can't parse its
command line, just let clap report that to the user in its usual way; do not log
it.

Do not assume that any log message will ever be read by the user. Redundancy
between log messages and UI feedback is completely acceptable and frequently
appropriate.

## Logging authentication events

A client which fails at authenticating to a server will in most circumstances
consider that an error. A server failing to authenticate a client is **not** an
error: if a server receives a misauthenticated request and rejects it, then the
server is working exactly as designed and nothing has gone wrong. The event is
usually not even noteworthy to the server administrator: password-guessers and
similar pest traffic on the public Internet are just routine background noise.
Application context may inform whether it is more appropriate for a server to
log authentication failures as `info` or `debug`, but `error` and `warn` are
always inappropriate.

Accepted authentications are the dangerous kind, and (on the server) must be
logged at least at the same severity as rejected ones. Routine logins should be
`info`, but consider using `warn` for administrator accounts that should not be
logging in very often.

## Managing log volume

For high-volume servers where logging can pose a performance concern, the
asymptotes of log volume are more interesting than the coefficients. It probably
doesn't matter whether a network request or some other IO event produces one log
line or five: the interesting threshold is from zero to one. Before adding a log
event, think about how it will affect log volume during a DDoS attack. If the
event you are recording is something only interesting in aggregate rather than
individually significant, consider recording it through the `metrics` crate
rather than through `tracing`, and then logging only aggregate data at slow,
regular intervals. You could also continue to log the individual events at
`debug` level, anticipating that debug logging in production will be turned off,
while logging the aggregate data at `info`.
