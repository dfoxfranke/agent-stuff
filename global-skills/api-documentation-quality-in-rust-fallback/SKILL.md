---
name: "api-documentation-quality-in-rust-fallback"
description: >-
  FALLBACK ONLY: Do not invoke this skill when
  $api-documentation-quality-in-rust is available; use that repository-scoped
  skill instead. Use this skill when writing or reviewing API documentation
  for Rust crates. Use $api-documentation-quality when available; otherwise
  use $api-documentation-quality-fallback, never both.
---

# Rust documentation profile

## Select the primary skill first

If `$api-documentation-quality-in-rust` is available, stop using this fallback
and use that skill instead. Never apply both
`$api-documentation-quality-in-rust` and
`$api-documentation-quality-in-rust-fallback` to the same task.

Use `$api-documentation-quality` when available; otherwise use
`$api-documentation-quality-fallback`. Never apply both. Apply this profile
together with the selected general API-documentation skill when working on Rust
code.

When this profile calls for Rust error-handling guidance, use
`$error-handling-in-rust` when available; otherwise use
`$error-handling-in-rust-fallback`. Never apply both.

These rules supplement the general skill. Where this profile establishes a
Rust-specific documentation requirement, follow it even when the same concept
has no equivalent requirement in other languages.

## Private items

First apply the general convention-resolution procedure to determine whether
private items are documented in the current codebase.

Do not assume either that private Rust items should be documented or that they
should be left undocumented merely because one choice is more common in the
ecosystem.

Use these rules after resolving the local convention:

- If comparable private items are consistently undocumented, follow that
  convention.
- If comparable private items are consistently documented, document items you
  add or modify.
- If private-item documentation is mixed or inconsistent, document applicable
  items you add or modify.
- If there is no useful precedent and no more-specific project instruction,
  default to documenting applicable private items.

These rules govern Rust doc comments and generated documentation coverage, not
every maintainer-facing source annotation. When a project consistently omits
private rustdoc, a narrowly necessary ordinary `//` comment or other non-rustdoc
record may still preserve a supported private invariant for an in-scope change.
Use the least anomalous local form; do not convert it to `///`, treat it as a
public guarantee, or use it as precedent to document peer items.

When the resolved convention calls for documenting private items, use the
`missing_docs` lint as the boundary for which kinds of items need doc comments:

> If `missing_docs` would require documentation for the item if it were public,
> give the item a doc comment even when it is private.

Do not add doc comments solely because of this rule to kinds of items that
`missing_docs` does not cover regardless of visibility, such as `impl` blocks.

The convention resolved here also controls whether the `# Safety`, `# Panics`,
and `# Errors` requirements below require rustdoc on a private item. Apply every
applicable section whenever the item is documented; require documentation on an
otherwise undocumented private item only when explicit repository policy,
enforced tooling, or the resolved convention calls for it. Public items remain
subject to every applicable section.

## `# Safety`

Every unsafe function and unsafe trait must have a `# Safety` section.

Write `# Safety` at the API boundary of the item being documented.

State the requirements that callers or implementors of this item must uphold. Do
not merely refer them to the safety requirements of an implementation
dependency.

Even when the rest of the comment uses dependency Model B, treat `# Safety` as
Model A.

For example, avoid:

> # Safety
>
> The caller must satisfy the safety requirements of `dependency_operation`.

Instead, state the actual conditions that make calling or implementing the
current item sound.

Use terminology from another documented abstraction when that terminology is
itself part of the public API, but do not outsource the current item's safety
contract to an implementation detail.

## `# Panics`

A function or method must have a `# Panics` section when a caller permitted to
invoke the item can trigger a panic without violating the item's documented
safety requirements. Also document an unavoidable third-party panic permitted by
the selected Rust error-handling skill, even when the caller does not control
its triggering condition.

Evaluate panic behavior using the procedure below.

### 1. Establish the caller boundary

Determine the item's effective visibility.

Reason about callers that possess the access granted by that visibility, not
about more-privileged implementation code that happens to exist elsewhere in the
module or crate.

For a safe item, consider what such callers can do using safe Rust.

For an unsafe item, assume that callers satisfy the requirements documented in
`# Safety`.

### 2. Identify caller-triggerable panic conditions

Consider the states, inputs, outputs, callbacks, shared state, and other effects
that permitted callers can cause.

Document a panic when such a caller can cause the panic while respecting the
item's contract and, for unsafe items, its safety requirements.

Describe the triggering condition at the boundary of the current function or
method.

Do not describe the implementation mechanism when a behavioral condition can be
stated instead.

For example, prefer:

> Panics if `index` is outside the current buffer.

over:

> Panics if the indexing operation used internally panics.

`# Panics` follows dependency Model A even when other parts of the same doc
comment use Model B.

### 3. Exclude failures of more-private invariants

Do not document a panic that can occur only after more-private implementation
code violates an invariant that it is responsible for maintaining.

Such a panic is an internal bug check rather than part of the item's documented
panic behavior.

For example, suppose a method contains:

``` rust
assert!(self.x < self.y);
```

Document the panic if a caller with permission to invoke the method can use safe
operations available at that visibility boundary to make `x >= y`, whether
through direct field access, constructors, setters, callbacks, shared state, or
other accessible operations.

Do not document the panic if:

- `x < y` is maintained entirely behind a more-private boundary;
- every operation available to the caller preserves that invariant; and
- the assertion can fail only because the invariant-maintaining implementation
  is buggy.

If no other panic condition requiring documentation under this section exists,
omit `# Panics`.

### 4. Preserve trait panic contracts

A trait implementation must not introduce a panic condition that the trait's
documented contract does not permit. This rule concerns behavior reachable while
the implementation and its dependencies satisfy their contracts; it does not
turn failures of private invariants into documented panic conditions. Adding
documentation only to the implementation does not ordinarily repair that
contract violation.

If the codebase controls the trait, either add the intended panic condition to
the trait's contract or make the implementation non-panicking.

For a third-party trait, follow the exception in the selected Rust
error-handling skill when clear evidence establishes that the defining crate
does not attempt to document all such panics. When that exception applies, add a
`# Panics` section to the implementation and identify the affected method when
the condition does not apply to every method. This documentation is required
even though implementation blocks do not otherwise need doc comments merely for
completeness.

### 5. Document unavoidable third-party panics as possibilities

When the selected Rust error-handling skill permits an unavoidable panic
originating in third-party code, document the condition at the current API
boundary using `may panic`. That language is sufficient to avoid guaranteeing
that the panic will occur; do not add a longer disclaimer.

For example:

> # Panics
>
> May panic if the platform cannot provide the current system time.

This does not prohibit a concise note about concrete intended API evolution. For
example, if a function already returns Result but may also panic for a condition
better reported as an error, its documentation may note an intention to report
that condition as an error in a future version.

Describe the caller-visible condition rather than saying that the item may panic
if a named dependency panics. This remains dependency Model A even when the rest
of the documentation uses Model B.

### 6. Do not attribute propagated panics to caller-supplied code

Do not document a panic merely because caller-supplied code may itself panic.

For example, a function accepting a closure ordinarily should not say:

> Panics if the callback panics.

A panic originating in caller-supplied code is not, by itself, a distinct panic
condition of the function being documented.

However, caller-supplied code can produce a value, state change, or other effect
that triggers a distinct panic condition in the function itself. Document that
condition when it is caller-triggerable.

Describe the function's own condition rather than saying that some dependency or
callback panics.

### 7. Distinguish panics from aborts

Do not document process aborts as panics.

Universal resource failures such as memory exhaustion and stack overflow do not
belong in `# Panics`.

Do not generalize this exclusion to actual Rust panic conditions. Subject to the
caller-triggerability and visibility rules above, document real panic conditions
even when they appear common or implementation-adjacent.

Examples can include:

- debug arithmetic overflow;
- out-of-bounds indexing;
- unwrapping a poisoned lock; and
- other genuine panic paths.

Outside the unavoidable-third-party exception, the relevant question is whether
a permitted caller can trigger the panic, not whether the panic mechanism feels
sufficiently unusual to mention.

### 8. Apply the rule to helpers, not test bodies

Test helper functions follow the same panic-documentation rules as other
functions.

Do not add `# Panics` sections to test functions themselves, including tests
annotated with `#[should_panic]`.

### Reviewer checks

Report a finding when:

- a trait implementation introduces a panic, reachable while it and its
  dependencies satisfy their contracts, that its trait contract does not permit
  and the third-party exception does not apply;
- incomplete third-party panic documentation is inferred from silence or
  isolated omissions rather than established by the evidence required by the
  selected Rust error-handling skill;
- an accepted trait exception lacks implementation-level `# Panics`
  documentation;
- an accepted third-party panic is undocumented at an affected API boundary;
- documentation promises that an accepted third-party panic will occur instead
  of saying it may occur; or
- documentation delegates the condition to a dependency instead of describing it
  at the current API boundary.

Do not report a correctly documented panic merely because it relies on the
established third-party exception.

## `# Errors`

For a function or method returning `Result`, document error conditions specific
to that item.

Do not repeat general semantics of the error type. Put those on the error type's
own documentation.

Use a separate `# Errors` section when:

- there are multiple distinct error cases; or
- explaining the error behavior requires more than a brief sentence.

For a single simple error condition, the explanation may be folded into the main
description.

When an error is ultimately produced by a dependency, follow the general
dependency-abstraction rules. Describe the current function's error contract
rather than paraphrasing the dependency's general error semantics unless the
dependency interaction itself is intentionally exposed.

## Test doc comments

When a unit test has a doc comment, state the property asserted by the test.

Do not write modalities such as "verifies that", "checks that", or "tests that".

Prefer:

``` rust
/// If the `username` field is empty, the request handler returns a 400 error.
#[test]
fn rejects_empty_usernames() { /* ... */ }
```

over:

``` rust
/// Rejects empty usernames.
#[test]
fn rejects_empty_usernames() { /* ... */ }
```

The test name identifies the scenario. The doc comment should make the expected
behavior precise.

Tests themselves do not receive `# Panics` sections.

## Avoid name-only comments

A Rust doc comment should normally add information beyond the item's identifier.

Avoid:

``` rust
/// Holds the state.
struct State { /* ... */ }
```

Prefer:

``` rust
/// Holds the pushdown automaton's stack and control state.
struct State { /* ... */ }
```

For trivial functions such as field accessors, some overlap with the function
name may be unavoidable. Add supported context when any is available.

Avoid:

``` rust
impl Session {
    /// Returns the user ID.
    fn user_id(&self) -> &UserId {
        &self.user_id
    }
}
```

Prefer:

``` rust
impl Session {
    /// Returns the session's logged-in user ID.
    fn user_id(&self) -> &UserId {
        &self.user_id
    }
}
```

Do not invent additional semantics merely to avoid a short or apparently
redundant comment. If the code and existing project documentation support no
richer claim, document only what they support.
