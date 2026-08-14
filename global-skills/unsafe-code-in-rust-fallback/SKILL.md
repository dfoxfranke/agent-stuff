---
name: "unsafe-code-in-rust-fallback"
description: >-
  FALLBACK ONLY: Do not invoke this skill when $unsafe-code-in-rust is
  available; use that repository-scoped skill instead. Decide whether unsafe
  Rust is justified and review its contracts and proof obligations. Use this
  skill when adding, changing, or reviewing Rust code involving unsafe blocks,
  unsafe functions or traits, unsafe impls, FFI, raw pointers, unchecked
  operations, low-level systems code, or performance-motivated unsafe code, even
  when the task does not explicitly ask about safety comments or documentation.
---

# Unsafe code in Rust

## Select the primary skill first

If `$unsafe-code-in-rust` is available, stop using this fallback and use that
skill instead. Never apply both `$unsafe-code-in-rust` and
`$unsafe-code-in-rust-fallback` to the same task.

Use unsafe Rust only when its necessity or measured benefit justifies the
additional proof burden. Make every caller obligation explicit and make every
implementation-side safety claim both true and sufficient.

## Resolve the applicable unsafe-code policy

Before introducing or accepting unsafe code, inspect the applicable repository
instructions and compiler or lint configuration for a stricter policy.

Follow a stricter policy, including a prohibition on unsafe code.

Do not treat a permissive repository policy, widespread existing unsafe code, or
the absence of an explicit restriction as permission to weaken this skill's
defaults. Depart from the eligibility rules below only when an explicit
instruction scoped to the current task directs otherwise.

An instruction to use unsafe code relaxes only the eligibility rules. It does
not waive Rust's soundness requirements, caller-facing safety documentation, or
implementation-side `SAFETY:` comments.

## Decide whether unsafe code is justified

Prefer a safe Rust implementation whenever one can provide the required
behavior.

Use unsafe code when the required behavior cannot be implemented in safe Rust.
Examples can include FFI boundaries and genuinely low-level systems components
such as kernels, allocators, and language runtimes. These domains do not grant
blanket permission: establish the need for each use of unsafe code.

Unsafe code may also be justified in performance-critical code. Unless an
explicit task-scoped instruction directs otherwise, retain it in a completed
change only when both forms of evidence exist:

1.  Profile evidence identifies the affected code as relevant to an important
    performance objective.
2.  A comparative benchmark shows that the unsafe implementation makes an
    important difference relative to the safe implementation.

An experimental unsafe implementation may be written to collect the comparison.
When relying on performance evidence rather than a task-scoped override, remove
it from the completed change if the evidence does not satisfy both conditions.
Report the profile finding, benchmark setup and results, and why the measured
difference matters to the stated performance objective. A merely measurable
improvement is not enough.

## Document caller-facing safety contracts

Give every unsafe function and unsafe trait a `# Safety` section in its API
documentation.

For an unsafe function, state the requirements that every caller must uphold.
For an unsafe trait, state the requirements that every implementation must
uphold. Express the actual contract at the current item's boundary; do not
outsource it to the safety requirements of an implementation dependency.

For the related general API-documentation guidance, use
`$api-documentation-quality` when available; otherwise use
`$api-documentation-quality-fallback`. Never apply both. Also use
`$api-documentation-quality-in-rust` when available; otherwise use
`$api-documentation-quality-in-rust-fallback`. Never apply both.

## Write `SAFETY:` comments as soundness arguments

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
argument must hold unconditionally for callers using that interface. Do not
assume an obligation that the safe interface cannot require its callers to
uphold.

When the enclosing function is unsafe, the argument may assume that its caller
has satisfied the function's documented `# Safety` contract. Connect any such
assumption to that contract explicitly, and do not rely on an undocumented
caller obligation.

For an unsafe impl, state why the implementing type and implementation uphold
the unsafe trait's contract. The comment must establish that contract for every
use permitted by the safe interfaces involved, not merely assert that the impl
is safe or that its method bodies use unsafe operations correctly.

## Author procedure

When writing or revising unsafe Rust:

1.  Resolve any applicable unsafe-code policy and follow the strictest relevant
    requirement.
2.  Establish that safe Rust cannot provide the required behavior, collect the
    required profile and comparative benchmark evidence, or identify the
    explicit task-scoped instruction that authorizes the use.
3.  Identify the caller or implementor obligations of every unsafe function or
    trait and document them in `# Safety`.
4.  Identify the safety contract of every unsafe operation used in each unsafe
    block.
5.  Establish the surrounding facts that discharge every requirement, checking
    those facts against the safe or unsafe boundary of the enclosing function.
6.  Write a `SAFETY:` argument immediately above each unsafe block and unsafe
    impl.
7.  Re-read each argument as a proof: verify that every premise is true and that
    the premises collectively imply the complete safety contract.
8.  For performance-motivated unsafe code that does not rely on a task-scoped
    override, report the profile and benchmark evidence before completing the
    change.

## Reviewer procedure

Perform the same policy, eligibility, contract, and proof analysis an author
should have performed.

Report a finding when code:

- uses unsafe despite a safe Rust implementation that provides the required
  behavior, absent an explicit task-scoped instruction;
- retains performance-motivated unsafe code without both profile and comparative
  benchmark evidence of an important benefit or a task-scoped instruction that
  explicitly authorizes doing so;
- omits or incompletely states the `# Safety` contract of an unsafe function or
  unsafe trait;
- omits a `SAFETY:` comment from an unsafe block or unsafe impl;
- makes a `SAFETY:` claim that is not true of the surrounding code;
- fails to cover every unsafe operation in its block;
- gives premises that are true but not logically sufficient for the operations'
  complete safety requirements;
- assumes an undocumented caller obligation, especially across a safe function
  boundary; or
- asserts that an unsafe impl is safe without establishing the unsafe trait's
  contract.

Do not accept avoidable unsafe code merely because its `SAFETY:` comment is
well-written. Do not reject necessary, explicitly task-directed, or
evidence-backed unsafe code when its contracts are complete and its
implementation-side arguments are sound.
