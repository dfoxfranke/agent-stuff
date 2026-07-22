# Coding style guidelines for agents

The style guidelines in this file represent the developer's personal
preferences. They are to be applied to all new projects. For existing projects,
these guidelines take lower priority than project-specific instructions or
established and consistent conventions observed by existing code. Code that is
generated, vendored, mechanically translated, or explicitly throwaway does not
need to conform to these guidelines.

## Comments

- Write doc comments even on private items. In Rust, if the `missing_docs` lint
  would trigger on an item if it were public, then put a doc comment on that
  item whether it's public or not. You don't need to comment items that the
  lint never covers regardless of visibility, such as `impl` blocks. In
  languages other than Rust, follow the spirit of this guideline.

- In Rust, unsafe functions and unsafe traits must have a `# Safety` section in
  their doc comment.

- In Rust, a function or method must have a `# Panics` section when a caller
  permitted to invoke the item can trigger a panic without violating the item’s
  documented safety requirements. For a safe item, consider what callers can do
  using safe Rust. For an unsafe item, assume that callers uphold its `# Safety`
  requirements.

  Evaluate this from the item’s effective visibility boundary. Consider what a
  caller with the access granted by that visibility can do, rather than what
  more-privileged implementation code happens to be able to do.

  Do not document a panic that can occur only after more-private code has
  violated an invariant that it is responsible for maintaining. Such an
  assertion is an internal bug check, not part of the item’s documented panic
  behavior. If the item has no other caller-triggerable panic conditions, omit
  the `# Panics` section.

  For example, suppose a method `foo` contains `assert!(self.x < self.y)`.
  Document this panic if code with permission to call `foo` can use safe Rust to
  make `x >= y`, whether by modifying the fields directly or through
  constructors, setters, callbacks, shared state, or other accessible
  operations. Do not document it if `x < y` is maintained entirely behind a
  more-private boundary, every safe operation available to callers preserves it,
  and the assertion can fail only because the invariant-maintaining
  implementation contains a bug.

  Do not document panics merely propagated from caller-supplied code. Do
  document any distinct panic condition in the function itself that
  caller-supplied code can trigger through its behavior, output, or effects. A
  function that accepts a callback argument does not typically need to document
  "panics if the callback panics".

  Do not conflate panics with aborts. Universal resource failures such as memory
  exhaustion and stack overflow abort the process; # Panics sections do not need
  to document them. Subject to the caller-triggerability and visibility rules
  above, however, actual panic conditions must be documented even if they seem
  "universal-ish". There is no exclusion for debug overflow, out-of-bounds
  indexing, poisoned-lock unwraps, etc.

  Helper functions used by tests should have the same panic documentation as
  anything else, but tests themselves (even those which `#[should_panic]`)
  should not have `# Panics` sections.

- In Rust, for functions that return a `Result`, keep any discussion of error
  conditions specific to the behavior of _that_ method; general discussion of
  the semantics of the error type belongs on the type's documentation and
  shouldn't be repeated. Use a separate `# Errors` section if there are multiple
  error cases or if the explanation is more than a brief sentence. Otherwise,
  fold the error discussion into the main body of the function documentation.

- Doc strings on unit tests should affirmatively state the property asserted by
  that test. Omit "verifies that" and other such modalities.

- There is almost never an excuse for lazy doc comments which merely restate what
  is already stated by the name of the function or type.

  BAD:

  ```rust
  /// Holds the state.
  struct State { … }
  ```

  GOOD:

  ```rust
  /// Holds the pushdown automaton's stack and control state.
  struct State { … }
  ```

  BAD:

  ```rust
  /// Rejects empty usernames.
  #[test]
  fn rejects_empty_usernames() { … }
  ```

  GOOD:

  ```rust
  /// If the `username` field is empty, the request handler returns a 400 error.
  #[test]
  fn rejects_empty_usernames() { … }
  ```

  It is acceptable, when necessary, to bend this rule for truly trivial
  functions, such as field accessors. Even then, include useful context when any
  is available.
  
  BAD:

  ```rust
  impl Session {
      /// Returns the user ID.
      fn user_id(&self) -> &UserId {
          &self.user_id
      }
  }
  ```

  GOOD:

  ```rust
  impl Session {
      /// Returns the session's logged-in user ID.
      fn user_id(&self) -> &UserId {
          &self.user_id
      }
  }
  ```

  In all cases, document only behavior, meaning, invariants, and rationale
  supported by the code or existing project documentation. Do not invent
  semantics merely to make a doc comment sound more informative.

- Comments other than doc comments should be fairly infrequent, but good places
  for them include:

  - Explaining code that executes an inherently complex and
    difficult-to-understand algorithm, for which even the clearest
    implementation cannot stand without exposition. Include citations to
    academic literature, when applicable.

  - Documenting a workaround for a bug in third-party code. Tag these with
    `WORKAROUND:` and include a citation to the bug ticket.

  - `TODO:` and `FIXME:` comments for code that is incomplete or has known issues.
    The comment should identify what, if anything, is blocking it from being
    completed or fixed immediately. However, if there is no blocker and the code
    is incomplete because you were explicitly instructed to leave it that way
    (e.g. while stubbing out a draft API design), do not mention this in the
    comment. Just leave "what's blocking this?" unaddressed in such instances.

  - Comments on `unsafe` blocks and `unsafe` impls in Rust. Always include a
    `SAFETY:` comment above every `unsafe` block and `unsafe` impl, stating the
    invariants that the enclosed code relies upon for its soundness.

## Naming things

- Names of predicates (pure functions with boolean return values) should
  function as adjectives. When possible, accomplish this without a verb:
  `is_noun` needs the "is", but `is_adjective` is redundant and can just be
  `adjective`. Keep the copula if it helps remove part-of-speech ambiguity: for
  example, `empty` might be an adjective or a verb, but `is_empty` makes clear
  that the adjective is intended.

- Names of impure functions should always contain a verb, while names of pure
  functions other than predicates should usually be nouns which describe their
  return value. For example, a method which inverts a matrix in-place should be
  named `invert`, while one which returns an inverted copy should be named
  `inverse`. If a function is technically pure but it's awkward to consider it
  that way, name it like an impure function. For example, some cryptosystems are
  deterministic and some aren't, but either way, a function which generates
  ciphertext from plaintext should be named `encrypt`, not `ciphertext`. In
  general, if it has more the flavor of a command than of a query, use the
  impure naming convention.

- Aside from acronyms, keep the use of abbreviations limited to very common
  ones, such as "msg" for "message". If you do use an abbreviation, use it
  consistently. Don't use "msg" in one place and "message" in another.

## Tests

- In test frameworks that distinguish unit tests from integration tests, keep
  unit tests pure. Unit tests mustn't print to stdout/stderr, access the
  filesystem, create sockets (even loopback sockets), etc. If they mutate global
  state within the process, ensure that they do so in a threadsafe way that
  would be correct even if the test runs in parallel with other tests or other
  instances of itself.

- In Rust, tested rustdoc examples must follow the same purity rules as unit
  tests. If there is tension between purity and writing a good example, the
  latter wins: add `no_run` whenever necessary. Adhere to the spirit of this
  guideline in any other languages which test documentation examples.

- All kinds of tests that run by default as part of `cargo test`, `npm test`,
  etc. must be able to succeed in most any test environment, including CI
  environments that restrict network access. They mustn't assume the existence
  of any system facility (libraries, command-line tools, DBus services, etc.)
  that isn't either explicitly documented as a dev dependency or declared as one
  in the project's package manifest (meaning `Cargo.toml` / `package.json` /
  etc).

- On both success and failure, all tests which touch the filesystem must default
  to cleaning up after themselves before they return, but should include a
  mechanism, such as setting an environment variable, to suppress this cleanup
  (since the files can be useful for debugging problems found by the test).

- If instructed to remove a feature, also remove all tests of that feature. Do
  not retain tests solely to assert that the feature is really gone.

- Favor property-based unit tests (using tools like Rust `proptest`, Python
  `hypothesis`, etc.) whenever practical. Seek opportunities (within the task's
  existing scope) to factor code in ways that make it more amenable to property
  testing.

## Error handling

- Favor API designs that enforce correctness-by-construction, making erroneous
  inputs unrepresentable. When a public function requires an efficiently
  checkable input invariant, prefer a validated wrapper type if the invariant is
  stable, domain-significant, reused across call sites, or otherwise important
  enough to justify a distinct type. Avoid one-off wrapper types whose ceremony
  exceeds their safety benefit.

- In languages such as Rust and Go which don't have exceptions but do have a
  "panic" concept, for every error condition consider whether it could possibly
  reflect a problem originating from the filesystem, the network, the user,
  other processes, etc.; or if it could only possibly reflect a bug, either in
  your own code or in other code executing in the same process as your code. You
  should panic in exactly the second case. That is, every panic should be
  understood as some kind of assertion failure, regardless of whether the
  assertion happens to be spelled `assert!`, `.expect()`, etc.

  Standard practice in the Go world tells you that library code should never
  panic, and instead return an error even in situations where it is completely
  clear that a bug has arisen. Here I am deliberately flouting "standard
  practice": panic anyway, unless you are contributing to an established
  codebase which already documents or consistently observes a "never panic"
  policy.

- Library code should return structured error details, but remember that all
  these details are API commitments. Target the finest detail level that will
  plausibly be useful to calling code in determining the disposition of the
  error, without exposing implementation details that would be burdensome to
  guarantee will remain unchanged in future versions. It's okay, however, for
  `Debug` impls to expose unstable implementation details, because the format of
  a debug string is broadly understood not to be a part of a crate's API contract.

- In Rust, prefer using `thiserror` for deriving `Display` and `Error` impls for
  your structured errors, but it is okay to implement them manually if doing so is
  necessary for providing a clearer error message.

- For non-structured errors (in application code where you know you'll always
  just be reporting the error to the user rather than handling it in some other
  way), use `anyhow`. For plugin/middleware/extension APIs which need a generic,
  open-ended error type, prefer `anyhow::Error` over `Box<dyn Error>` or
  similar.

- Your error's `Display` impl should never write anything that would be
  redundant with what is written by the error's `source()`'s `Display` impl.
  Unless existing codebase conventions clearly establish otherwise, assume that
  the code responsible for reporting the error to the user will traverse source
  chains.

  PREFERRED:

  ```rust
  #[derive(Debug, Error)]
  #[error("failed to load {path}")]
  struct LoadError {
      path: PathBuf,
      #[source]
      reason: IoError
  }
  ```

  DISPREFERRED BUT ACCEPTABLE:

  ```rust
  #[derive(Debug, Error)]
  #[error("failed to load {path}: {reason}")]
  struct LoadError {
      path: PathBuf,
      reason: IoError
  }
  ```

  NEVER ACCEPTABLE:

  ```rust
  #[derive(Debug, Error)]
  #[error("failed to load {path}: {reason}")]
  struct LoadError {
      path: PathBuf,
      #[source]
      reason: IoError
  }
  ```

## Dependencies

- Even if it's something recommended by these guidelines (such as `proptest` or
  `thiserror`), don't add new dependencies to a project without asking
  permission. However, be proactive about suggesting them. If you know of an
  existing project that already does a better job of doing something you've
  been asked to implement, point it out.

- The same goes for dev tools. If the dev system or sandbox is missing a tool
  that would make your job easier, ask for it. Don't make do silently with
  inadequate tools, even if you're able. While in planning mode, check that you
  have all the tools you'll need to carry out the plan, so you can raise any
  tool-related concerns beforehand rather than in the middle of work. If you
  forgot to request something or the request wasn't fulfilled as expected, you
  may interrupt work over it unless you were explicitly told to work without
  interruption; in that case, make do, but raise the issue in your completion
  summary.

## Use of special characters

Outside of languages such as Julia whose established conventions routinely use
Unicode identifiers, keep identifier names restricted to printable ASCII.

In languages with an adequately defined Unicode source-text model, Unicode may
be used freely in strings and comments and is encouraged when typographically
appropriate. This applies primarily to human-readable prose. Do not replace
characters in machine-consumed text (such as protocol elements, command-line
syntax, paths, regular expressions, serialized formats, or exact test fixtures)
with typographic lookalikes.

Adequate support requires a language specification that defines source text in
terms of Unicode or gives source files a well-defined Unicode encoding. C fails
this test, so keep C source files themselves within printable ASCII. JavaScript
can be regarded as passing. Rust, Go, and Python 3 straightforwardly pass.

In Unicode-supporting source files, render the names of persons mentioned in
comments faithfully in their preferred orthography when that orthography is
known. Preserve the applicable letters and diacritics; do not invent a spelling
or substitute typographic presentation ligatures.

For names normally written in a non-Latin script, give both the preferred-script
and Latin-script forms on the first substantive mention in the file, and use the
Latin-script form thereafter. Prefer the person’s own Latin-script spelling when
one is available; otherwise use an established transliteration consistently.
When the preferred spelling is not known, preserve the spelling supplied by the
user, cited source, or existing project documentation rather than guessing.

In comments and in strings likely to be rendered in a monospaced font, surround
em-dashes with whitespace.
