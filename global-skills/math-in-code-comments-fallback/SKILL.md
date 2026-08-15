---
name: "math-in-code-comments-fallback"
description: >-
  FALLBACK ONLY: Do not invoke this skill when $math-in-code-comments is
  available; use that repository-scoped skill instead. Use this skill when
  writing or reviewing mathematical formulas in ordinary source-code comments or
  documentation comments, including choosing among established local notation,
  documentation-generator math markup, UnicodeMath, conventional ASCII
  arithmetic, and TeX. When source safety matters, use $unicode-source-safety
  when available; otherwise use $unicode-source-safety-fallback, never both.
---

# Math in code comments

If `$math-in-code-comments` is available, stop using this fallback and use that
skill instead. Never apply both `$math-in-code-comments` and
`$math-in-code-comments-fallback` to the same task.

Use `$unicode-source-safety` when available; otherwise use
`$unicode-source-safety-fallback`. Never apply both. References below to the
selected Unicode source-safety skill mean the one chosen here.

Represent formulas precisely and readably without assuming that every source
file or documentation renderer accepts the same notation.

## Choose the representation

Apply the first matching rule:

1.  Follow an established codebase convention for formulas in the same kind of
    comment.
2.  For a documentation comment that a standardized or configured documentation
    generator actually parses, use the generator's native mathematical markup
    when it has one.
3.  Otherwise, apply the selected Unicode source-safety skill to the exact
    target file. If it concludes `Safe`, use UnicodeMath.
4.  If it concludes `Unsafe` and the formula has a simple, conventional ASCII
    representation, use ASCII.
5.  Otherwise, use TeX.

Do not apply documentation-generator markup to an ordinary implementation
comment that the generator does not parse.

## Resolve the codebase convention

Determine the convention at the scope of the comment before selecting a
fallback. Use this precedence order:

1.  **Explicit repository instructions.** Follow applicable agent instructions,
    contributor documentation, style guides, encoding policies, and
    documentation rules.

2.  **Clearly enforced tooling.** Inspect documentation-generator configuration,
    build dependencies, lint and formatter settings, source-encoding
    declarations, and CI documentation builds.

3.  **Existing analogous comments.** Inspect several formulas, preferring
    comments that are:

    - in the same module, package, crate, or component;
    - ordinary or documentation comments of the same kind;
    - written in the same source language;
    - intended for the same documentation output; and
    - recently maintained when that can be determined.

Do not infer a convention from one isolated formula when more evidence is
available. Resolve ordinary-comment and documentation-comment conventions
separately.

Classify the result:

- **Consistent:** use the established representation and delimiters.
- **Mixed:** follow the most analogous internally consistent group; if no group
  controls, continue with this skill's decision procedure.
- **No useful precedent:** continue with this skill's decision procedure.

Let a more-specific instruction override a general convention. Resolve formula
notation precedent here, but defer source-encoding evidence and classification
to the selected Unicode source-safety skill when the decision procedure reaches
that branch.

## Use native documentation markup

Take this branch only when the target is a documentation comment and the
repository will feed that comment to the relevant generator. Confirm the
generator and version from configuration, dependencies, or documentation build
commands.

Treat math support as native only when the generator's official documentation
defines mathematical input syntax. Do not count generic Markdown support,
generic raw-HTML passthrough, or the mere availability of a third-party plugin.
A configured extension and established local syntax can still control under the
codebase-convention rule.

Use these known profiles:

- **Julia Markdown and Documenter:** infer this profile for Julia docstrings.
  Put inline LaTeX formulas between two backticks on each side and display
  formulas in fenced blocks tagged `math`. Use `@doc raw"""..."""` or escape
  backslashes required by Julia string syntax. Follow the [Julia documentation
  manual](https://docs.julialang.org/en/v1/manual/documentation/).

- **R Rd and roxygen2:** infer this profile for R documentation comments. Use
  `\eqn{tex}{ascii}` inline and `\deqn{tex}{ascii}` for display math. Supply a
  useful ASCII rendering in the second argument when the TeX form is not already
  readable in text output. In roxygen2 `#'` comments, write a single backslash.
  Follow [Writing R
  Extensions](https://stat.ethz.ch/R-manual/R-devel/doc/manual/R-exts.html#Mathematics).

- **Sphinx:** require repository evidence that Sphinx consumes the Python
  docstring and that math rendering is enabled. Use the inline role shown below
  and `.. math::` for display math. Use a raw Python docstring or double the TeX
  backslashes. Do not substitute raw MathJax delimiters. Follow the [Sphinx math
  documentation](https://www.sphinx-doc.org/en/master/usage/extensions/math.html).

  ``` text
  :math:`...`
  ```

- **Doxygen:** require repository evidence that Doxygen consumes the comment.
  Use `\f$...\f$` or `\f(...\f)` inline and `\f[...\f]` for display math. Follow
  the [Doxygen formula
  documentation](https://www.doxygen.nl/manual/formulas.html).

- **TypeDoc:** require repository evidence for a TypeDoc version and output that
  supports its documented MathML. Use `<math>...</math>` rather than inventing
  TeX delimiters. Follow the [TypeDoc Markdown
  showcase](https://typedoc.org/example/documents/Markdown_Showcase.html#mathml).

- **DocFX:** require repository evidence that the modern DocFX template
  processes the comment. Use `$...$` inline and `$$...$$` for display math,
  while also satisfying C# XML escaping. Follow the [DocFX math
  documentation](https://dotnet.github.io/docfx/docs/markdown.html#math-expressions).

- **MATLAB `publish`:** use `$...$` inline and `$$...$$` for display math only
  in publishable comments immediately following a section break. Do not apply
  this syntax to arbitrary help or implementation comments. Follow the [MATLAB
  publishing markup
  documentation](https://www.mathworks.com/help/matlab/matlab_prog/marking-up-matlab-comments-for-publishing.html).

Treat rustdoc, Javadoc, Go doc, DocC, Dokka, JSDoc, phpDocumentor, RDoc, YARD,
Scaladoc, and dartdoc as having no documented first-class mathematical markup.
For their comments, continue to the selected Unicode source-safety skill unless
a higher-priority local convention applies. Do not assume that `$...$` renders
merely because the generator accepts Markdown.

## Determine source-text safety

Apply the selected Unicode source-safety skill to the exact target file and
every supported source-processing path. Use its binary `Safe` or `Unsafe`
conclusion; do not independently reclassify the language in this skill. That
conclusion concerns only whether the source can reliably carry literal non-ASCII
text. This skill still determines which mathematical representation is
appropriate.

## Write UnicodeMath

Use UnicodeMath whenever the source-text branch is safe, even when the formula
could also be written using only ASCII. The ASCII rule is a fallback for unsafe
source, not a preference over UnicodeMath.

For a nontrivial formula or uncertain syntax, read [the UnicodeMath
reference](references/unicode-math.md) and consult its linked version 3.3 PDF.
In particular:

- use actual Unicode mathematical characters in the final comment;
- treat backslash control words as input shortcuts, not normally as final
  output;
- use UnicodeMath's linear `/`, `_`, and `^` syntax and explicit parentheses to
  preserve grouping;
- distinguish mathematical minus, multiplication, differential, and other
  symbols when the distinction carries meaning; and
- preserve identifiers from the code or define any intentional mapping between
  code identifiers and mathematical variables.

For example:

``` rust
// The weighted mean is μ = (∑_(i=1)^n w_i x_i)/(∑_(i=1)^n w_i).
```

Do not insert decorative mathematical glyphs that change or obscure the
formula's semantics.

## Write simple ASCII arithmetic

Take this branch only when non-ASCII source is unsafe and every semantic element
has an unambiguous, conventional ASCII spelling.

Use:

- ASCII identifiers and numerals;
- parentheses or brackets for grouping;
- operators such as `+`, `-`, `*`, `/`, `%`, `=`, `!=`, `<`, `<=`, `>`, and
  `>=`; and
- `^` or `**` for exponentiation only when the surrounding convention makes that
  meaning unambiguous. Otherwise write repeated multiplication when simple or
  use TeX.

Write multiplication explicitly with `*` rather than relying on juxtaposition.
Add parentheses whenever precedence could be misread.

For example:

``` c
/* The rectangle's area is area = width * height. */
```

Do not invent ASCII art or lossy transliterations for roots, sums, integrals,
Greek variables, decorated symbols, matrices, cases, or other mathematical
structure. Do not replace `∑` with an invented `sum(...)` notation unless that
function or notation is already part of the code's vocabulary.

## Write TeX

When non-ASCII source is unsafe and simple ASCII cannot preserve the formula,
write TeX markup. Use `$...$` for an inline formula and `\[...\]` for a display
formula unless a higher-priority local convention specifies other delimiters.

Prefer widely supported core TeX or LaTeX math commands. Do not depend on
undeclared packages or project-specific macros. Escape backslashes or delimiters
only when the host language's doc-comment representation requires it; ordinary
lexical comments generally do not.

Keep every TeX control sequence inside math delimiters. Write surrounding prose
in ordinary ASCII, or delimit a side condition separately when it also needs TeX
notation.

For example:

``` c
/* TeX: For $a \ne 0$, the roots are $x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$. */
```

Label literal TeX when readers might otherwise mistake it for rendered
documentation.

## Check the result

For writing:

1.  Resolve the local convention.
2.  Verify the formula against the implementation, cited source, or established
    invariant before reformatting it.
3.  Determine whether the target is an ordinary or parsed documentation comment.
4.  Apply the native-markup and source-safety decisions in order.
5.  Preserve mathematical grouping, symbol meanings, and the mapping to code
    identifiers.
6.  Apply host-language escaping and local line-wrapping conventions.
7.  Build or preview generated documentation when native markup is used and the
    project provides a normal documentation check.
8.  Re-read the raw source as well as any rendered output.

For review, report a problem when a formula:

- violates a clear local convention;
- uses documentation markup in a comment that will not be parsed;
- assumes undocumented renderer support;
- introduces non-ASCII into an unsafe source path;
- uses ASCII when doing so loses mathematical structure;
- uses malformed UnicodeMath, native markup, or TeX;
- changes the formula's meaning through grouping, operator, or symbol choices;
  or
- disagrees with the code or documented invariant it purports to explain.

Do not request a notation change merely because another valid representation is
personally preferable.
