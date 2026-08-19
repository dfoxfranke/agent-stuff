# Style guide

This guide is a simplified gloss on requirements covered in much greater detail
by AI agent skills installed at `.agents/skills`. The skills are a tedious read,
and human contributors aren't expected to go through them all. This version is
much shorter because it assumes you will apply common sense rather than trying
to explicitly articulate every detail, edge case, and exception. Read this guide
before you write code, and then afterward, ask for a code review from an AI that
has the skills loaded.

## Documentation

**Documentation is a contract.**

Every function and every type declaration — even small, private helpers —
**must** be accompanied by API documentation specifying its contract and
invariants. Callers may rely upon precisely what is specified by the
documentation, and nothing beyond that.

Documentation should add meaning that is not already clear from an item’s name
or declaration. Explain the semantics, constraints, effects, invariants, and
relationships that readers need in order to use the API correctly. Don't repeat
information that's already there in the type signature.

Treat every behavioral claim as a promise the implementation is intended to
preserve. Describe supported, caller-observable behavior, not incidental
behavior that merely happens to be visible today. Implementation details belong
in API documentation only when they provide a genuinely useful design
explanation or an important performance property.

When a function calls out to a dependency for some of its functionality, its API
documentation should not make claims about how the dependency behaves. You have
two models to choose from:

1. Specify your function's observable behavior directly, leaving the dependency
   as a hidden implementation detail, or
2. Make the manner of interaction with the dependency an explicit part of your
   function's contract. Specify what it passes to the dependency and how it
   handles what's returned, but not what happens inside the dependency.

When in doubt, pick model 1. *Always* pick model 1 when specifying conditions
that can lead to an exception/panic/crash or undefined behavior; don't write
things to the effect of "it panics if the dependency panics".

When code relies on the behavior of third-party dependencies that aren't
documented up to our standards, it may be necessary to make some subjective
inferences about what the author really had in mind regarding its intended
contract. State these inferences near the call site which depends on them.

Exception/panic conditions should have a dedicated section in API documentation.
In languages which support ad-hoc polymorphism through some sort of interface
definition (Rust traits, Go interfaces, Haskell typeclasses, etc.),
implementations should not except/panic under any conditions outside of those
specified by the interface contract. Bug-check conditions which assert an
internal invariant that it should be impossible for outside callers to violate
don't count and shouldn't be mentioned in API documentation.

## Tests

**Tests verify contracts.**

The purpose of a unit test is to verify the behavior promised by the tested
function's contract as thoroughly as practical — but nothing beyond that. A test
which asserts things beyond what is stating in the contract should be considered
broken, even if those assertions happen to be true of the function's present
implementation.

Every test should include a string or doc comment which affirmatively states the
proposition that the test is checking. Omit modalities such as "verifies that…".

Sometimes practicality dictates that a function's internal contract should be
stronger than its public one. For example, a function whose purpose is to print
a colored diff should probably not commit in its public documentation to what
shade of green will be used for added lines. Nonetheless, its private
documentation should specify a specific RGB value so that tests can check for
that rather than have to invent some "what qualifies as green?" criteria. Put
private documentation of this nature with the function, not with the test.

It's okay for tests to make reasonable assumptions about their execution
environment and to fail if those assumptions are violated. For example, a test
which creates some temporary files in a standard system location and then
represents their path as a string is allowed to fail someone has set `$TMPDIR`
to a path containing invalid Unicode. When it is non-obvious that an assumption
is required, document it with the test.

Property tests are usually preferable to tests which just check a few
hand-chosen inputs. Seek to factor code in ways that make it more amenable to
property testing.

In languages which make a formal distinction between unit and integration tests,
keep unit tests observationally pure. Any test which touches the filesystem,
binds sockets, or spawns processes should be an integration test.

Correctness properties whose evaluation requires human judgement should be
checked with acceptance tests. Provide the tester with detailed, repeatable
instructions on what to verify and how. Even if the acceptance criteria are
vague (e.g. "output looks correctly formatted"), the procedure for getting to
where they can be evaluated should not be.

## Error handling

Libraries should signal errors through structured error types, but remember that
every exposed detail is an API commitment. Choose the finest detail level
plausibly useful to calling code when deciding how to handle the error, without
exposing implementation details that would be burdensome to preserve.

When the same error type is used by multiple functions, common semantics should
belong to the type documentation, not the function documentation. The function
documentation's discussion of error conditions should remain specific to the
behavior of that function and avoid rehashing what the type already documents.

### Rust-specific considerations for error handling

Never use `Option::unwrap()`. When `None` would necessarily indicate a bug, use
`Option::expect()` and state why the value should be `Some`. For `Result`, use
`expect()` instead of `unwrap()` if and only if the message can add informative
context that the `Err` does not already carry. Otherwise use `unwrap()` rather
than adding a redundant message.

When creating a new structured error, normally use `thiserror` to derive
`Display` and `Error` implementations, but implement them manually when it's
necessary for providing a clearer error message.

Do not make an error's `Display` implementation repeat text emitted by its
source's `Display` implementation. refer an outer message that adds context
while exposing the underlying error through `source()`:

``` rust
#[derive(Debug, thiserror::Error)]
#[error("failed to load {path}")]
struct LoadError {
    path: std::path::PathBuf,
    #[source]
    reason: std::io::Error,
}
```

Interpolating the reason without exposing it as a source is dispreferred but
acceptable:

``` rust
#[derive(Debug, thiserror::Error)]
#[error("failed to load {path}: {reason}")]
struct LoadError {
    path: std::path::PathBuf,
    reason: std::io::Error,
}
```

Never both interpolate the reason and expose the same value as a source:

``` rust
#[derive(Debug, thiserror::Error)]
#[error("failed to load {path}: {reason}")]
struct LoadError {
    path: std::path::PathBuf,
    #[source]
    reason: std::io::Error,
}
```

Use `anyhow` for non-structured application errors when the application will
report them rather than inspect them programmatically. For plugin, middleware,
and extension APIs that require a generic open-ended error type, prefer
`anyhow::Error` over `Box<dyn Error>` or a similar erased trait object.

## Logging

Log at five levels:

- `error`: an intended operation failed because something went wrong.
- `warn`: an operation behaved abnormally, may have produced an incorrect
  result, or created a condition that warrants attention before it becomes an
  error.
- `info`: a significant but normal event occurred.
- `debug`: an event of some kind occurred, but not one that is typically of any
  operational significance.
- `trace`: reports on the current state of execution, not necessarily
  representing that anything at all "happened".

Everything logged at `info` or higher needs to have some sort of observable
consequence outside the process. `debug` still needs to represent some kind of
state transition, but it could be a purely internal one, such as a message being
written to a channel. `trace` could simply be a dump of a function's local
variables at some point in its execution.

If you're working with a logging framework that doesn't have a `trace` level,
then log `trace` events as `debug`, but retain the distinction mentally.

`debug` events should be added sparingly, only when there is good reason to
anticipate that they will be eventually be useful. `trace` events should never
be added proactively: wait until there is a concrete problem that you are using
them to investigate.

Think of `trace` logging as a more civilized form of printf-debugging. Unlike the
cruder form, it is acceptable to commit, but still should not be a permanent
feature of the codebase. Once you are confident that a function or module works
correctly and have verified this with solid test coverage, you should remove its
`trace`-level logging. After removing it, consider whether there is some single
`debug` event, meeting the "something happened" threshold, which would have
immediately pointed to the problem that you were using the `trace` events to
discover if it had been present. If such a `debug` event is likely to be useful
again in the future, you should add it.

Functions must either dispose locally of abnormal conditions or propagate them
to the caller — never both. An error return (or a thrown exception, in languages
where that's an idiomatic substitute for error returns) means that it's up to
the caller to decide the disposition of the error. Logging an error *is* its
disposition, or at least part of it. However, logging at `debug` or `trace`
isn't considered dispositive: it's okay to log at one of these levels and also
return an error.

Not every error means something has actually gone wrong; don't log above `debug`
if you don't have enough context to be sure. It would be very bad, for example,
for a library to log every connection failure as an error if the application
using it turned out to be a port scanner.

A client which fails at authenticating to a server will in most circumstances
consider that an error. A server failing to authenticate a client is not an
error: if a server receives a misauthenticated request and rejects it, then the
server is working exactly as designed and nothing has gone wrong. The event is
usually not even noteworthy to the server administrator: password-guessers and
similar pest traffic on the public Internet are routine background noise.
Application context may inform whether it is more appropriate for a server to
log authentication failures as `info` or `debug`, but `error` and `warn` are
always inappropriate.

Accepted authentications are the dangerous kind, and (on the server) should be
logged at least at the same severity as rejected ones. Routine logins should be
`info`, but consider using `warn` for administrator accounts that should not be
logging in very often.

For high-volume servers where logging can pose a performance concern, the
asymptotics of log volume are more interesting than the coefficients. It
probably doesn't matter whether a network request or some other IO event
produces one log line or five: the interesting threshold is from zero to one.
Before adding a log event, think about how it will affect log volume during a
DDoS attack. If the event you are recording is something only interesting in
aggregate rather than individually significant, then only record it in
aggregate. You could also continue to log the individual events at `debug`
level, anticipating that debug logging in production will be turned off, while
logging the aggregate data at `info`.

## Unicode

In languages which cleanly support Unicode, put it to use in strings,
documentation, and comments. "Clean support" means that the language either
specifies a Unicode encoding for its source files, or has its lexical syntax
defined terms of Unicode characters. This includes Rust, JavaScript/TypeScript,
Go, Python 3, Ruby 2+, Java, C#, Kotlin, Swift, Dart, Haskell, and Scala. It
excludes C, C++, Objective C, PHP, R, POSIX shell, and Lua.

Good uses for Unicode include:

* Personal names
* Mathematical formulas
* General punctuation such as dashes, ellipses, qutotation marks, etc.

For personal names that use a non-Latin script, include both the preferred form
and a transliteration with the name's first substantive use in a file or document,
and use the transliterated form thereafter.

For mathematical formulas, use [UnicodeMath](https://www.unicode.org/notes/tn28/).

## Unsafe Rust

It is appropriate to write unsafe Rust:

1. When it is unavoidable, such as at FFI boundaries and in low-level systems
   code such as kernel, allocators, and language runtimes.

2. In performance-critical code, but only after its benefit has been
   quantitatively demonstrated through profiling and benchmarks.

Every unsafe function and unsafe trait must have a `# Safety` section in its API
documentation. For an unsafe function, state the requirements that every caller
must uphold. For an unsafe trait, state the requirements that every
implementation must uphold.

Place a `SAFETY:` comment immediately above every unsafe block and every unsafe
impl.

For an unsafe block, the comment must:

1.  State facts that are true of the surrounding code.
2.  Cover the safety requirements of every unsafe function or operation used in
    the block.
3.  Be logically sufficient to show that those requirements hold.

A shared argument may cover multiple unsafe operations only when it is
sufficient for all of them. Do not substitute a vague assurance, a description
of what the block does, or a restatement of an operation's safety contract for
the facts that establish the contract.

For example, avoid:

``` rust
// SAFETY: The unchecked access is safe.
unsafe { slice.get_unchecked(index) }
```

Prefer a locally established fact that discharges the actual requirement:

``` rust
// SAFETY: The guard above established that `index < slice.len()`.
unsafe { slice.get_unchecked(index) }
```

When the enclosing function has a safe interface, every fact in the `SAFETY:`
argument must hold unconditionally, even when interface is misused by the
caller.

When the enclosing function is unsafe, the argument may assume that its caller
has satisfied the function's documented `# Safety` contract. Connect any such
assumption to that contract explicitly, and do not rely on an undocumented
caller obligation.

For an unsafe impl, state why the implementing type and implementation uphold
the unsafe trait's contract. The comment must establish that contract for every
use permitted by the safe interfaces involved, not merely assert that the impl
is safe or that its method bodies use unsafe operations correctly.
