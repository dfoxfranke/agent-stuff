---
name: "documentation-quality"
description: "Use this skill when writing or reviewing API documentation for functions, methods, subroutines, types, tests, or equivalent program elements."
---

# Documentation quality

The goal is to document the intended contract precisely without adding redundant, incidental, invented, or implementation-specific guarantees.

Language-specific profiles may add requirements to this skill. Apply those requirements in addition to the general rules below.

## Resolve the codebase's documentation convention

Before adding documentation to an item that the codebase might not normally document, determine the convention that applies at that item's scope.

Use this precedence order:

1. **Explicit repository instructions.** Follow applicable `AGENTS.md`, contributor documentation, style guides, lint configuration, or other project-specific instructions.
2. **Clearly enforced tooling.** Treat documentation requirements enforced by the language, compiler, linter, or repository configuration as part of the project's convention.
3. **Existing analogous code.** Inspect several comparable items, preferring examples that are:

   * in the same module, package, crate, or component;
   * of the same kind;
   * at the same visibility level; and
   * recently maintained when that can be determined.

Do not infer a convention from one isolated example when more evidence is available.

Classify the local convention as follows:

* **Consistently documented:** document comparable items you add or modify.
* **Consistently undocumented:** do not introduce documentation solely for completeness.
* **Mixed or inconsistent:** prefer documenting comparable items you add or modify.
* **No useful precedent:** use the defaults supplied by this skill and any applicable language profile.

More-specific project instructions override more-general conventions. A strong local convention may override a common ecosystem convention.

Do not mechanically propagate an obvious documentation omission when explicit project instructions establish a different rule.

## Core rules

### Add information beyond the declaration itself

Documentation should contribute information that is not already adequately conveyed by the item's declaration, signature, or name.

Do not merely expand an identifier into a sentence.

Do not restate facts already evident from a function's type signature unless omitting them would make the prose grammatically awkward or materially harder to understand.

Prefer documenting:

* semantics and meaning;
* caller-relevant constraints;
* effects and state changes;
* invariants;
* relationships between arguments or results;
* intended guarantees;
* rationale that callers need in order to use the API correctly.

For a truly trivial item, such as a field accessor, a small amount of apparent repetition may be unavoidable. Include useful context whenever the code supports it.

### Document only supported semantics

Every behavioral or semantic claim must be supported by the code, an established invariant, or existing project documentation.

Do not invent meaning, rationale, guarantees, or invariants merely to make a comment more informative.

When the available evidence supports only a modest statement, write the modest statement.

### Document observable behavior, not implementation details

Describe behavior that is observable to the relevant caller and intentionally forms part of the item's contract.

Do not describe non-observable implementation details unless one of these exceptions applies:

1. **Didactic explanation.** The discussion explains the broader design rather than specifying the item's contract, and a reasonable reader would not interpret it as an API guarantee.
2. **Performance information.** The detail communicates algorithmic complexity or another important performance characteristic.

Do not introduce implementation details merely to make documentation more explanatory.

When reviewing existing documentation, do not report implementation-detail discussion that clearly falls under one of these exceptions.

### Specify only intended guarantees

Document exactly the observable behavior that callers are intended to rely on.

Do not specify incidental behavior merely because it can currently be observed.

Assume ordinary omissions are intentional unless there is evidence that the omitted detail belongs to the contract. For example, exact wording, formatting, or presentation of human-readable messages is normally not contractual.

Do not request additional specification merely because more detail could be written.

### Express underspecification through omission

When a detail is intentionally unspecified, normally say nothing about it.

Do not add statements such as "the exact format is unspecified" merely to disclaim a guarantee that the documentation never makes.

If maintainers need to record that an omission is deliberate so that future reviewers do not repeatedly request it, use a source comment or other maintainer-facing note rather than user-facing documentation.

### Choose one abstraction level for dependency-backed behavior

When an item relies on another library, service, callback, runtime facility, or dependency, normally use one of these models.

**Model A: document the item's behavior**

Treat the dependency as an implementation detail.

* Do not mention the dependency.
* State the item's observable contract directly.
* Ensure the implementation can uphold that contract whenever the dependency upholds its own contract.

Prefer Model A when there is no reason for callers to know about the dependency.

**Model B: document the interaction with the dependency**

Make the dependency interaction itself part of the documented abstraction.

* State how the dependency is invoked or used.
* State what the item does with the dependency's result.
* Do not separately restate or guarantee the dependency's own behavior.

Do not mix the models by both exposing the dependency interaction and paraphrasing the dependency's contract as though it were an independent guarantee of the current item.

#### Contract-section exception

A language-specific profile may require dedicated documentation for matters such as safety requirements, panics, exceptions, or other failure conditions.

When such a section describes obligations or behavior at the current item's API boundary, write it according to Model A even when the rest of the documentation uses Model B.

State the condition that applies to the current item's caller. Do not substitute statements such as:

* "panics if the dependency panics";
* "is safe if the dependency's safety requirements are met";
* or equivalent delegation of the current item's contract to an implementation dependency.

The dependency may explain why the implementation behaves that way, but the contract section should describe what the caller of the current item must know.

### Keep failure documentation local to the item

When documenting errors, exceptions, panics, or equivalent failure behavior, describe conditions specific to the item being documented. Do not repeat general semantics that belong to the documentation of an error type, exception type, dependency, or other reusable abstraction.

Document failure behavior that callers are expected to account for or are entitled to rely on. Do not document assertion failures for "impossible" conditions that callers cannot trigger without some more-private invariant or implementation assumption already having been violated.

### Prefer plain language

Use ordinary language when it is as precise as specialized terminology.

Use jargon only when it improves precision or is the natural established terminology for the concept.

### Use terminology the reader can reasonably know

Specialized terminology is appropriate when the expected reader can reasonably be assumed to know it.

Typical sources of acceptable terminology include:

* elementary theoretical computer science;
* the specification or reference manual of the programming language being used;
* standards directly relevant to the library's domain.

Avoid:

* Borrowing terminology from unrelated or obscure domains unless it is necessary and explained.
* Borrowing terminology coined by dependencies when the use of those dependencies is intended to be a private implementation detail.
* Treating a library's users and maintainers as the same audience. Jargon necessary and appropriate for documenting library internals may nonetheless be inappropriate in user-facing documentation.

### Do not invent terminology unnecessarily

Prefer established terminology or a plain description over a newly coined term.

If a new term is genuinely necessary, define it clearly in an obvious location before or when readers first need it.

### Document tests as propositions

Follow local conventions for how and whether to document tests. Use the guidance in this section as a default when local conventions are unclear, inconsistent, or not established.

When a test has user- or maintainer-facing documentation, state the property that the test establishes.

Write the proposition directly rather than describing the act of testing it.

Prefer:

> If the username is empty, the request handler returns a 400 error.

over:

> Verifies that empty usernames are rejected.

The test's name already communicates that it is a test; the documentation should communicate the behavior or invariant being asserted.

Do not add a separate doc comment when the test framework already requires or conventionally uses a human-readable proposition as part of the test declaration. In such frameworks, that proposition serves the same purpose and additional documentation would normally be redundant.

For example, Ruby's RSpec commonly expresses tests as:


```ruby
it "returns a 400 error when the username is empty" do
  # ...
end
```

Here, the string passed to it already states the property being asserted, so a doc comment repeating that proposition is unnecessary.

## Author procedure

When writing or revising documentation:

1. Resolve the applicable codebase convention before deciding which items require documentation.
2. Read the item's declaration, implementation, relevant invariants, and existing public contract.
3. Identify the behavior, meaning, constraints, effects, and guarantees that callers are intended to rely on.
4. Remove information already adequately communicated by the declaration or name.
5. Remove claims that are merely incidental behavior.
6. Remove unsupported semantics or rationale.
7. Remove non-observable implementation details unless they serve a permitted didactic or performance purpose.
8. For dependency-backed behavior, choose Model A or Model B and use it consistently, except for contract sections covered by the contract-section exception.
9. Document item-specific failure behavior without repeating general semantics defined elsewhere.
10. Use plain language and established terminology where possible.
11. Remove statements whose only purpose is to announce that an undocumented detail is unspecified.
12. Re-read the result as a set of promises. Remove any promise the implementation is not intended to preserve.

When uncertain whether a detail belongs in the contract, prefer omission unless callers need the detail to use the API correctly or there is clear evidence that it is intended as a guarantee.

When uncertain whether an item should receive documentation at all, use the convention-resolution procedure rather than assuming either maximal documentation or ecosystem defaults.

## Reviewer procedure

When reviewing documentation, first resolve the same codebase convention an author should have used.

Then report a finding when documentation:

* fails an applicable project or language-specific documentation requirement;
* merely restates the item's name or declaration without adding useful context;
* exposes non-observable implementation details outside the allowed exceptions;
* invents semantics, rationale, guarantees, or invariants unsupported by the code or project documentation;
* promises incidental behavior that is not intended to be contractual;
* omits behavior that is clearly necessary to express the intended public contract;
* explicitly announces underspecification that should instead be expressed through silence;
* inconsistently mixes direct behavioral guarantees with claims about a dependency's behavior;
* delegates a required contract section to the behavior or requirements of an implementation dependency;
* repeats general failure semantics that belong elsewhere;
* externally documents assertion failures that should never be externally triggerable;
* uses avoidable jargon;
* assumes terminology the expected reader cannot reasonably be expected to know;
* introduces unnecessary or undefined terminology; or
* describes what a test "verifies" instead of stating the property it establishes, unless local conventions dictate this style.

Be conservative about findings for missing detail. Do not ask for additional specification merely because additional detail is possible.

Do not treat a repository's unusual documentation convention as an error merely because it differs from common ecosystem practice.

## Editing stance

For both writing and review:

* Prefer useful semantics over paraphrases of names and signatures.
* Prefer fewer, stronger guarantees over exhaustive description.
* Prefer observable behavior over implementation explanation.
* Prefer omission over disclaimers about unspecified behavior.
* Prefer the current item's contract over paraphrases of dependency contracts.
* Prefer plain language over unnecessary terminology.
* Treat every specific behavioral claim as a promise the implementation is intended to preserve.
