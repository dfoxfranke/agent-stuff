Comphrehensively review and improve all first-party tests, API documentation,
and source comments in this repo, using the following skills:

* $api-documentation-quality
* $document-code-caveats
* $math-in-code-comments
* $represent-person-names
* $unicode-punctuation
* $test-quality

and if the repository includes Rust code, then also use:

* $api-documentation-quality-in-rust
* $unsafe-code-in-rust, insofar as it pertains to the quality of safety
  comments. Eliminating unnecessary-but-sound use of unsafe code is out
  of scope for this task.
* $error-handling-in-rust, and treat improvements to error handling as in
  scope for this task, insofar as those improvements do not lead to
  semver-breaking changes in any public APIs.

If the repo contains any vendored forks of third-party code, those are out
of scope. Do not modify them, and do not report findings regarding the quality
of their tests, comments, or documentation. If they contain substantive flaws
that impact other work within the scope of this task, you may report findings
which recommend fixing those.

If existing tests and documentation disagree on an intended variant, or tests
and documentation for a function are both absent, resolve the intent
holistically and then close the gap accordingly.

If newly created-or-improved tests discover a genuine bug, fix that bug if and
only if the fix is clean, simple, and low-risk. Otherwine, retain the defective
behavior, leave the test failing, and report the finding.

If in the course of constructing a safety argument you discover a soundness
bug, fix that bug if and only if the fix is clean, simple, and low-risk.
Otherwise, report the finding and, if possible, construct a test which crashes
in MIRI. 

The refactoring-for-testability that $test-quality describes as presumptively
authorized is, in fact, authorized as part of this task.

Report residual findings in an untracked Markdown file at the root of the repo.
Name it `FINDINGS.md` unless a file by that name already exists.

You are finished when:

1. All comments and API documentation outside of vendored forks satsify the
   requirements of the above-enumerated skills.

2. All tests outside of vendored forks satsify $test-quality up to its
   convergence criteria, and remaining coverage gaps are reported in priority
   order in an untracked Markdown file at the root of the repo.

3. All Rust code (if any) outside of vendored forks satisfies as much of
   $error-handling-in-rust as can be accomplished without introducing
   semver-breaking changes.

4. All tests and documentation outside of vendored forks test and document
   the correct API contract, as inferred holistically from the present
   codebase. 

5. All tests either:
   - Pass, or
   - Fail because they expose genuine bugs for which there is no simple, clean,
     and low-risk fix; the reason for this is documented as a finding at
     highest priority.
