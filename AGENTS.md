# Coding style guidelines for agents

The style guidelines in this file represent the developer's personal
preferences. They are to be applied to all new projects. For existing projects,
these guidelines take lower priority than project-specific instructions or
established and consistent conventions observed by existing code. Code that is
generated, vendored, mechanically translated, or explicitly throwaway does not
need to conform to these guidelines.

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
  instances of itself. Spawning threads from unit tests is okay, but only if
  the test joins on all its created threads before completing.

  This is another place to look out carefully for existing codebase conventions.
  If the codebase already has impure unit tests and it has no integration tests
  at all, then you can presumably add more impure unit tests, and should not
  introduce integration tests just to avoid doing so.

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
