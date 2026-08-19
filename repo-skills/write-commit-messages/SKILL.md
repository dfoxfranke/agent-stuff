---
name: write-commit-messages
description: Draft, revise, and audit Git commit messages and the structure of a Git patch series. Use when preparing commits on a branch intended for human-submitted review, rewriting messages, reviewing staged changes, creating review-response fixups, or cleaning history after approval. Produces durable, self-contained commit history while keeping pull-request summaries human-authored.
---

# Write Commit Messages

Use this skill to write exact Git commit messages and to evaluate whether a
branch is organized into reviewable commits. AI-assisted commit-message drafting
is within scope; pull-request communication is not. This skill does not draft
pull-request summaries, descriptions, cover letters, issue text, or review
replies, and it does not submit pull requests. References below to submission or
review concern the branch and its commit history; a human authors and submits
the pull request separately.

## Governing model

A pull request and its commits describe the same change at different levels.

- The human-authored pull request explains the whole change as a unit of review
  and delivery. It emphasizes externally observable behavior, compatibility,
  overall human motivation, issue references, benchmark results, and validation
  performed by the contributor.
- Each commit explains one durable implementation step. It emphasizes the exact
  code-level change, the technical problem or invariant involved, and why that
  implementation was chosen.

Even when a pull request contains one commit, its summary and the commit message
are not interchangeable. Both say what changed, but for different readers and
at different levels of abstraction.

A useful test is:

- Pull request: Why should the project merge this overall change, and what will
  users or API consumers observe?
- Commit: What does this patch change in the implementation, and why is this the
  right technical change?

Commit messages are repository history. They must remain useful in `git log`,
`git blame`, `git bisect`, a downstream fork, or a source archive after the
forge conversation is unavailable.

## Non-negotiable rules

### Never include forge issue or ticket references

Never put issue, ticket, or pull-request references in a commit message. This
includes:

- `#123`, `GH-123`, Jira-style IDs, or similar identifiers;
- `Closes`, `Fixes`, or `Resolves` directives aimed at forge issues;
- links to issues, pull requests, review comments, or project-board items;
- phrases such as “requested in issue 123” or “address review feedback.”

The commit must contain any technical context it needs instead of delegating its
meaning to a forge artifact.

A project-standard reference to a *commit*, such as
`Fixes: <stable-hash> ("subject")`, is not an issue reference. It is permitted
only when the hash satisfies the stability rule below.

### Cite another commit by hash only when the hash is stable

A commit message may name another commit by hash only when that commit is
already on a protected branch and therefore will not be rewritten. Do not cite
hashes from:

- the current patch series;
- an unprotected topic branch;
- a contributor fork whose history may be rebased;
- any branch whose protection status is unknown.

If stability cannot be established, treat the hash as unstable. Describe the
change by name or quote its subject instead. When a stable hash is useful,
include enough of the referenced subject to make the relationship intelligible
without looking it up.

### Keep pull-request material out of commit messages

Do not include:

- issue or ticket metadata;
- benchmark results or benchmark transcripts;
- commands run, manual-testing reports, or test-environment matrices;
- rollout instructions or reviewer instructions;
- product, customer, organizational, or other human motivations for adding a
  feature or intentionally changing non-defective behavior;
- screenshots, review chronology, or discussion summaries.

A commit may describe test code that the patch adds and the behavior that code
covers. It must not turn the commit body into a report of tests the contributor
ran.

A commit may explain an algorithmic property or technical constraint, such as
avoiding an extra scan or maintaining constant-time lookup. Put measured
performance figures in the human-authored pull request, not in the commit.

### Record technical reasoning, not development chronology

Explain root causes, invariants, constraints, tradeoffs, and non-obvious design
choices. Do not narrate the order in which the author discovered or corrected
things. Avoid messages such as:

- “fix typo” when the typo was introduced earlier in the same series;
- “oops” or “fix previous commit”;
- “address review comments”;
- “try another approach”;
- “make tests pass” without explaining the code-level defect.

Before initial review, fold such corrections into the commit that introduced
them. During review, preserve reviewed history and append fixup commits as
described below.

### Do not invent intent or rationale

Ground every statement in the diff, surrounding code, tests, repository
documentation, or explicit human-provided context. Do not manufacture a root
cause, compatibility claim, performance claim, or design rationale merely to
make the message sound complete.

When the change is obvious and no non-obvious rationale is supported, use a
strong subject and omit the body rather than inventing one.

## Patch-set lifecycle

The correct history shape depends on whether review has started.

### Before a human opens a pull request: prepare the branch

Prepare the branch intended to become a pull request as a clean, LKML-like
ordered patch series. The human will author and submit the pull request
separately. The branch history should express the logical structure of the
solution, not the literal chronology of development.

For each proposed commit, be able to explain why a reviewer benefits from seeing
it separately. Good reasons include:

- isolating a mechanical or preparatory refactor from a behavioral change;
- introducing an invariant or representation before using it;
- separating an independently useful cleanup from the feature it enables;
- keeping an unrelated formatting or generated-file update out of a substantive
  diff;
- separating human-written scaffolding from tool-assisted implementation when
  that division is real and reviewable.

Do not split merely to reduce line count. If two changes require the same
explanation, cannot be reviewed independently, or leave an artificial boundary,
they probably belong in one commit.

Order commits so that prerequisites precede their consumers. Every commit must
keep the build working and must not introduce a regression relative to its
parent. A commit may add a regression test that fails because it exposes a bug
already present on the protected base branch; that expected failure is evidence
of the existing bug, not a regression introduced by the patch. The test commit
must itself build, and the test must fail for the intended reason.

When a bug-fix series includes a regression test, put the regression-test commit
before the fix. Reviewers should be able to see the test fail at that commit and
pass once the fix is applied. Apart from this deliberate test-only exception,
no commit should introduce a new build or test failure that a later commit must
repair. The complete series must build and pass its tests.

A commit need not be a complete standalone product feature, but it should be an
internally coherent unit of review and durable history.

Fold all corrections to an unreviewed patch back into that patch. The initial
series must not contain cleanup commits for mistakes introduced by earlier
commits in the same series. A standalone correction to code already present on
the protected base branch is different and may be a valid commit of its own.

### While code review is in progress: append, do not rewrite

Once review begins, do not rebase, amend, squash, reorder, or otherwise rewrite
the commits reviewers have already seen. Append patches that respond to review
feedback to the tip of the branch so reviewers can see exactly what changed
relative to the reviewed state.

A correction intended to be folded into an earlier commit may use a
`fixup! <target subject>` commit. A genuinely new logical step may use a normal
commit message. Do not disguise substantive new behavior as a fixup.

During this phase, preserving review continuity takes priority over presenting a
perfect final history. Do not perform history-rewriting operations unless the
review process has ended and approval has been granted.

### After approval: restore the clean series without changing the tree

After code review is approved, rebase or autosquash the appended review-response
commits into a clean final patch set. Rewrite final commit messages as needed so
each resulting commit follows this skill.

This cleanup must not change the approved code. Record the approved `HEAD` and
its tree before rewriting:

```sh
approved_head=$(git rev-parse HEAD)
approved_tree=$(git rev-parse 'HEAD^{tree}')
```

After the rebase, verify that the tree at the new `HEAD` is byte-for-byte
identical:

```sh
test "$approved_tree" = "$(git rev-parse 'HEAD^{tree}')"
```

An equivalent empty diff against `$approved_head` may be used as an additional
check. Commit IDs and commit metadata will change during a rebase; the final
Git tree must not. If the tree differs, the operation introduced a content
change rather than merely cleaning history, and that change requires review.

Do not leave `fixup!`, `squash!`, “address review,” “fix typo,” or similar
review-process commits in the final series.

This skill may recommend a reorganization, but it must not rewrite Git history
or create commits unless the human explicitly requests that operation.

## Workflow

### 1. Read repository-specific rules

Before drafting, inspect the repository's contribution guide, commit lint
configuration, existing trailers, and recent protected-branch history. Follow a
clear local convention for component prefixes, capitalization, line wrapping,
and trailer order. Do not copy weak recent messages merely because they exist.

Useful context commands include:

```sh
git status --short
git log -n 30 --format='%h %s'
git diff --cached --stat
git diff --cached
```

For an existing series, identify the intended protected base and inspect commits
in order:

```sh
git log --reverse --format='%H%n%s%n%b%n---' <base>..HEAD
git diff --stat <base>...HEAD
git show --stat --summary <commit>
git show --find-renames <commit>^!
```

Do not infer the base solely from a branch name when that choice affects the
series analysis.

### 2. Establish the review phase

Determine whether the branch is:

- being prepared for a human to submit for initial review;
- currently under review; or
- approved and ready for final history cleanup.

Apply the corresponding lifecycle rules. If the phase is uncertain, do not
silently recommend history rewriting.

### 3. Inspect the full patch, not just filenames

Read the actual diff and relevant surrounding code. Identify:

- the state before the patch;
- the exact state after the patch;
- the technical limitation, defect, or invariant involved;
- the implementation choice made;
- any non-obvious constraints or alternatives;
- tests or documentation added as part of the patch;
- facts supplied by the human that belong only in the pull request.

For a series, inspect both each individual commit and the cumulative diff. A
message that makes sense for the final tree may be wrong for the intermediate
commit that actually carries it.

### 4. Check the commit boundaries

For every commit, answer:

1. What single logical step does this commit perform?
2. Why is that step separate from adjacent commits?
3. Does its diff match its stated purpose without unrelated changes?
4. Does the repository still build at this commit?
5. Does this commit introduce a regression or a new test failure?
6. If it adds an intentionally failing regression test, does that test expose a
   bug already present on the protected base branch and fail for the intended
   reason?
7. In a bug-fix series, does the regression-test commit precede the fix so the
   same test can be observed failing before the fix and passing afterward?
8. Does it rely on a later commit to repair a defect it introduces?
9. Would folding or splitting make review materially clearer?

Do not infer intermediate integrity merely because the final `HEAD` builds and
passes its tests. When the repository and available tooling permit it, validate
the build and relevant tests at each commit in the series. An intentionally
failing regression-test commit is the only exception to the rule against new
test failures.

When boundaries are wrong, report the structural problem before polishing the
messages. Good prose cannot make a poorly organized patch set coherent.

### 5. Separate commit facts from pull-request facts

Use the following allocation rules:

| Information | Commit message | Human-authored pull request |
| --- | --- | --- |
| Exact code-level change in this patch | Yes | At overview level |
| Root cause or violated invariant | Yes | User-visible symptom at most |
| Technical design choice or data-structure rationale | Yes | Only if needed for overview |
| Externally observable behavior of the whole series | Only as needed to orient the patch | Yes, prominently |
| Human or product motivation for a feature | No | Yes |
| Issue, ticket, or pull-request reference | Never | Yes |
| Benchmark results | No | Yes |
| Manual testing and commands run | No | Yes |
| Test code added by the patch | Yes, when useful | At validation level |
| Compatibility, rollout, and reviewer notes | No | Yes |
| `Assisted-by` and other required commit trailers | Yes | No substitute in the PR |

If supplied context belongs only in the pull request, omit it from the commit
message. The skill may report category labels such as “issue reference omitted”
or “manual-test report omitted,” but it must not turn those facts into drafted
pull-request prose.

### 6. Draft the subject

The subject should:

- state the specific action performed by this commit;
- name the affected component when that improves precision;
- use the repository's established style, normally imperative mood;
- be concise enough to scan in `git log`—aim for roughly 50–60 characters and
  avoid exceeding about 72 unless local convention differs;
- omit a trailing period;
- avoid vague verbs and placeholders such as “update,” “changes,” “misc,” “fix
  issue,” or “address feedback” unless followed by a precise object;
- contain no issue reference and no unstable commit hash.

Good subjects distinguish the implementation change:

- `parser: handle empty predicate lists`
- `scheduler: index ready tasks by priority`
- `config: preserve unknown keys during migration`

Weak subjects hide it:

- `Fix #418`
- `Update parser`
- `Address review comments`
- `Misc cleanup`

### 7. Draft the body only when it adds durable value

Separate the body from the subject with a blank line. Use paragraphs to explain
what the diff cannot communicate reliably on its own.

A strong body usually covers some subset of:

1. the relevant prior state or technical limitation;
2. the root cause, violated invariant, or source of ambiguity;
3. the implementation change;
4. why this approach is preferable to an obvious alternative;
5. any consequence that a future maintainer must preserve.

Do not mechanically restate each changed line. Do not repeat the subject in a
longer sentence. Do not add a body merely to satisfy a template.

When referring to another patch in the same series, prefer a conceptual
relationship such as “Now that the parser returns normalized predicates…” or
name the other patch by subject. Avoid patch numbers when reordering would make
them stale, and never use a rewritable hash.

Wrap prose according to repository convention, commonly near 72 columns.
Preserve code identifiers, commands, and literal output where wrapping would
make them misleading.

### 8. Add required trailers

Preserve valid project-required trailers and keep them after a blank line at the
end of the message.

Contributions substantially assisted by AI or another specialized tool require
an `Assisted-by` trailer. Use:

- `harness:vendor/model` for AI systems, using the established `vendor/model`
  identity when available;
- `tool:version` for other specialized tools.

Multiple tools may appear on one trailer line, for example:

```text
Assisted-by: codex-cli:openai/gpt-5.5 cargo-fix:1.96
```

Include AI models, static analyzers, fuzzers, `cargo fix`, and similarly
substantive assistance. Do not include routine Rust tooling such as `rustfmt`,
`cargo clippy`, or `rust-analyzer` merely because it was used normally.

Generating only the English subject line with AI does not by itself require
disclosure. Broader drafting or substantive assistance may. Apply the project's
substance threshold to each commit rather than treating tool use as microscopic
contagion.

Never guess a tool identity or version. Use information provided by the runtime,
repository, or human. If a required identity is unknown, flag that the message
cannot be finalized until it is supplied; do not emit a fake identity or an
unresolved placeholder as a final trailer.

### 9. Audit the result

Before returning a message or series, verify:

- the subject accurately describes this commit's diff;
- the body explains only supported technical facts;
- no issue, ticket, PR number, or forge URL appears anywhere;
- every commit-hash reference points to a commit known to be on a protected
  branch;
- no benchmark result, manual-test report, rollout note, or human feature
  motivation appears;
- no development or review chronology remains in the final series;
- required `Assisted-by` trailers are present and correctly spelled;
- the initial or final series is logically ordered;
- every commit keeps the build working;
- no commit introduces a regression or new test failure, except for a
  regression-test commit that exposes a bug already present on the protected
  base branch;
- in a bug-fix series, the regression test precedes the fix, fails for the
  intended reason before the fix, and passes after the fix;
- an in-review branch has not been told to rewrite reviewed commits;
- a post-approval cleanup preserves the approved tree exactly.

## Message patterns

### Bug-fix series with a regression test

When a branch contributes both a regression test and its fix, order the series
so the test comes first and the fix comes second. At the test commit, the build
must succeed and the new test must fail because it exposes the bug already
present on the protected base branch. After the fix commit, the same test must
pass. Do not put the test after the fix merely to keep every intermediate test
run green.

A test commit can explain the pre-existing failure it captures:

```text
tests: reproduce empty-filter panic

An explicitly empty filter reaches build_conjunction with no child predicates.
Add a regression case for this path so the existing panic is visible before the
implementation changes.
```

The following fix commit should explain the implementation failure and the
invariant restored, as in the next pattern.

### Defect fix

State the implementation failure and the invariant restored. Do not rely on an
issue report to explain the bug.

```text
parser: handle empty predicate lists

An explicitly empty filter reaches build_conjunction with no child
predicates. The builder assumes at least one child and indexes the first
element, so the empty form panics after normalization.

Return the identity predicate for an empty list. Keeping the empty case in
the builder preserves one invariant for every caller and avoids duplicated
special cases.
```

The human-authored pull request may separately describe the observed failure,
link the issue, and report manual validation. This skill must not draft that
text.

### Internal refactor

Say which internal structure changes and why the new structure is easier or
safer to maintain. Do not invent a user-facing effect.

```text
scheduler: index ready tasks by priority

The ready queue stores tasks in insertion order and every selection scans the
full queue to find the highest-priority runnable task. That also spreads the
tie-breaking rule across the insertion and selection paths.

Store ready tasks in a priority-keyed map and keep FIFO order within each key.
This makes the ordering invariant explicit and centralizes task selection in
one operation.
```

Measured benchmark results belong in the pull request, not in this message.

### Preparatory commit in a series

Explain the independent technical step and the invariant it establishes. Do not
use an unstable hash for the later consumer.

```text
config: separate parsing from normalization

The parser currently applies defaults while it is still decoding input. That
prevents callers from inspecting which values were explicit and makes format
migration depend on parser control flow.

Return the decoded representation unchanged and move default application into
a normalization pass. Later behavior changes can then operate on one explicit
normalized form without duplicating decode logic.
```

### Trivial standalone correction

A body is unnecessary when the subject completely explains a small correction
to code already present on the protected base branch.

```text
docs: correct the Environment heading
```

If that misspelling was introduced by an earlier unreviewed commit in the same
series, amend that commit instead of creating this one.

## Output contract

When asked for one commit message, return the exact message in a plain-text code
block. Add concise diagnostic notes outside the block only when there is a
structural problem, missing required identity, or PR-only material that was
omitted.

When asked for a series, present commits in order. For each commit, identify the
commit or proposed position and provide the exact message. Report any required
split, fold, reorder, fixup, or post-approval cleanup separately from the message
text.

When asked to audit existing messages, identify each violated rule and provide a
corrected message grounded in the corresponding diff.

Never generate a pull-request summary. When PR-only facts are encountered, name
only the omitted categories so the human can write the summary independently.
