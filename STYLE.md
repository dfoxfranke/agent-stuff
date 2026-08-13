# Style guide

This guide states the project’s conventions for API documentation, Rust error
handling and logging, and human-readable text in source and documentation. It
is addressed to contributors: use the principles here to communicate the
program’s intent clearly, preserve useful abstraction boundaries, and avoid
turning incidental behavior into permanent promises.

## Documentation is a contract

Documentation should add meaning that is not already clear from an item’s name
or declaration. Explain the semantics, constraints, effects, invariants, and
relationships that readers need in order to use the API correctly. A short
comment that says only what the signature already says is usually noise; a
modest but useful statement is better than invented detail.

Treat every behavioral claim as a promise the implementation is intended to
preserve. Describe supported, caller-observable behavior, not incidental
behavior that merely happens to be visible today. Leave intentionally
unspecified details unmentioned instead of adding disclaimers about them.
Implementation details belong in API documentation only when they provide a
genuinely useful design explanation or an important performance property.

Keep a coherent abstraction level when an implementation delegates to another
component. Either describe this API’s own observable behavior, or make its
interaction with the dependency part of the interface. Do not expose the
interaction and then restate the dependency’s contract as a second guarantee.
Likewise, describe errors, exceptions, and other failures in terms of the
current item’s boundary rather than repeating general behavior documented by an
error type or dependency.

Prefer plain language and established terms that the intended reader can
reasonably know. Introduce a new term only when it improves precision, and
define it where readers first need it.

When a test has explanatory prose, state the property it establishes directly:
“If the username is empty, the handler returns 400,” not “Verifies that empty
usernames are rejected.” Do not repeat the proposition when the test
declaration already expresses it clearly.

## Rust API documentation

Document private Rust items of the kinds that `missing_docs` would cover if the
items were public. These comments are still contracts within their visibility
boundary, so they should contribute supported semantic information rather than
expand an identifier into a sentence. `impl` blocks and other item kinds that
the lint does not cover do not need comments merely for completeness.

Every unsafe function and unsafe trait has a `# Safety` section. State the
conditions that callers or implementors of that item must uphold; do not
outsource its safety contract to a dependency.

A function or method has a `# Panics` section when a caller permitted by its
visibility can trigger a panic while honoring its contract and, for an unsafe
item, its safety requirements. Describe the caller-visible condition, not the
internal operation that panics. Assertions that can fail only after private
code breaks its own invariant are bug checks rather than public panic
conditions. Panics originating solely in caller-supplied code, process aborts,
memory exhaustion, and stack overflow are also outside this section. Test
bodies do not receive `# Panics` sections, although test helpers follow the
ordinary rule.

For every function or method returning `Result`, document the error conditions
specific to that item. A single simple condition may fit in the main
description; multiple cases or a substantial explanation belong in an
`# Errors` section. General semantics of the error type belong on the type
itself.

Documentation records a panic contract; it does not expand one. A trait
implementation must still respect the trait’s documented panic behavior.
Where a genuinely unavoidable panic from correctly used third-party code is
accepted, describe the condition at the affected API boundary as one under
which the item “may panic.”

## Rust errors and panics

Make invalid states difficult to construct. A validated type is worthwhile
when it captures an efficiently checked invariant that is stable,
domain-significant, or reused widely enough to justify the added type and API
surface. Do not introduce a wrapper whose ceremony outweighs its safety value.

Failures that may originate in user input, the filesystem, the network,
another process, or another external system are recoverable errors. Panics are
for programmer bugs and broken in-process invariants. Trait implementations
remain subject to their trait’s panic contract. Propagating an externally
triggered panic from third-party code is a narrow exception: the dependency
must be used according to its contract, and there must be no reasonable
fallible way to preserve the required behavior.

Panic messages should explain the invariant being asserted. Do not use
`Option::unwrap`; when `None` necessarily means a bug, use `expect` to explain
why the option must be `Some`. For `Result`, use `expect` only when its message
adds invariant context that the error itself does not carry; otherwise
`unwrap` avoids a redundant message. Either is appropriate only after
establishing that `Err` necessarily indicates a bug.

Library errors should expose the finest distinctions and data that callers can
plausibly use, without committing to incidental implementation details. Prefer
`thiserror` for new structured errors. Use `anyhow` for application failures
that will be reported rather than inspected, and prefer `anyhow::Error` at
open-ended plugin, middleware, or extension boundaries. Preserve a suitable
existing error representation instead of changing it merely to adopt these
libraries. Unstable diagnostic detail belongs in `Debug`, not in public error
structure or stable display text.

Preserve error source chains. Each outer error’s `Display` text should add
context without repeating the text produced by its source. An error should not
both interpolate a source’s message and expose that same value through
`source()`; reporters commonly traverse the chain themselves.

## Operational logging in Rust

Use `tracing` for Rust logging and instrumentation. Choose levels by the
event’s meaning at the layer that emits it:

- `error` means an intended operation failed because something went wrong.
- `warn` means behavior was abnormal, may have produced a wrong result, or
  warrants attention before it becomes an error.
- `info` marks a significant, normal state transition.
- `debug` marks an internal transition that answers a concrete diagnostic
  question.
- `trace` observes control flow or execution state without a transition.

Higher-level events should have an externally meaningful consequence. Keep
control-flow tracing temporary to an active investigation; if the same class
of problem is likely to recur, retain a focused state-transition event at
`debug` instead.

Record an abnormal condition once, at the layer that decides its disposition.
Code that returns or propagates an error has not disposed of it and should not
also emit an `info`, `warn`, or `error` event for the same condition. Reusable
libraries generally lack the application context to own those levels and
should leave them to the handling boundary, though purposeful `debug` or
`trace` context may still be useful. Do not duplicate a panic with a log
immediately before it.

Put every dynamic, semantically meaningful fact in a structured field; the
human-readable message may repeat those facts for readability. Use the same,
minimally qualified field key for the same concept, and represent independent
dimensions as separate fields. Give `info`, `warn`, and `error` events stable
names when they will be queried, aggregated, or alerted on centrally.
Otherwise leave events unnamed; `debug` and `trace` events remain unnamed in
either case.

Logging and user feedback serve different audiences. Neither substitutes for
the other, although both may describe the same condition when each has an
independent purpose. Input rejected completely by the user interface without
attempting an operation usually needs no log event.

Classify authentication from the emitter’s perspective. A client whose
requested operation is blocked may own an authentication error. A server that
correctly rejects invalid credentials has behaved normally, so record the
rejection at `debug`, `info`, or not at all—not at `warn` or `error`. A failure
of the authentication system is an operational failure, while a successful
server authentication should be at least as prominent as a rejected one.

Keep production volume bounded. Before adding per-request, per-item, or
per-retry events, consider worst-case and attacker-controlled traffic. Facts
that matter chiefly in aggregate belong in metrics or periodic summaries;
individual diagnostic events may remain at levels normally disabled in
production.

## Unicode and human-readable text

Use literal Unicode only when the source format and every supported tool that
reads or rewrites it preserve a defined Unicode encoding. Embedded
documentation inherits the encoding constraints of its outer source language.
Rust and CommonMark, for example, define Unicode source text; C, C++, and shell
source require an explicit end-to-end encoding guarantee. Existing non-ASCII
text, editor defaults, or one successful compiler invocation are not by
themselves such a guarantee.

When literal Unicode is unsafe, use an escape only in documentation syntax that
actually renders it at that location. Otherwise use conventional ASCII or
rewrite the prose. Source safety establishes only that characters survive the
toolchain; grammar, semantics, typography, normalization, and security remain
separate concerns.

In new or substantively revised prose, use Unicode punctuation when it is safe
and expresses the intended meaning. Distinguish a hyphen from a minus sign, en
dash, or em dash, and prose quotation marks and apostrophes from machine
delimiters. In monospaced output, ordinarily space an em dash as `word — word`.
Do not normalize unrelated passages merely for typographic consistency.

Exact and machine-consumed text takes precedence over typography. Preserve
code, identifiers, commands, paths, URLs, regular expressions, protocol values,
directives, serialized data, fixtures, and quoted source text rather than
substituting typographic lookalikes.

### People’s names

Represent each person’s name faithfully, including genuine letters,
diacritics, and orthographic marks. Prefer the person’s own spelling; when it
is unknown, retain the form supplied by the person, a cited source, or existing
documentation. Do not guess, expand initials, strip marks to manufacture ASCII,
or invent a transliteration. Preserve genuine letters such as `Æ`, but do not
introduce decorative presentation ligatures such as `ﬁ`.

For a name normally written in a non-Latin script, introduce the first
substantive prose mention as `preferred-script form (Latin-script form)`, then
use the Latin-script form consistently. Prefer the person’s own Latin-script
spelling; otherwise use one established transliteration. Bylines, citation
metadata, identifiers, paths, URLs, and similar structured text are exact data,
not prose to normalize.

If the source path cannot carry the faithful literal spelling, use a rendering
escape where the documentation format supports one, or use a known preferred
or established safe form. Do not manufacture a lossy substitute.

## Mathematics in comments

Write formulas so their mathematical meaning and relationship to the code are
unambiguous. Preserve grouping, operator meanings, and symbol distinctions, and
either retain code identifiers or explain their correspondence to conventional
mathematical variables.

In documentation processed by a renderer with native mathematical markup, use
that markup. In other Unicode-safe source, use UnicodeMath, including actual
mathematical characters and its structural `/`, `_`, and `^` notation. Use
parentheses wherever the linear form could obscure grouping. Renderer-specific
markup belongs only in comments that the renderer actually parses.

Where literal Unicode is unsafe, conventional ASCII is appropriate for simple
arithmetic: spell multiplication explicitly with `*` and make grouping clear
with parentheses. Do not invent lossy ASCII approximations for sums, roots,
integrals, matrices, decorated symbols, or similar structure. Use plain TeX for
such formulas, with `$...$` inline and `\[...\]` for display math unless the
documentation format defines its own notation. Label literal TeX when readers
might otherwise mistake it for rendered output.
