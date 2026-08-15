---
name: "unicode-punctuation-fallback"
description: >-
  FALLBACK ONLY: Do not invoke this skill when $unicode-punctuation is
  available; use that repository-scoped skill instead. Choose and write
  Unicode punctuation and typographic symbols in standalone documentation,
  ordinary code comments, and documentation comments. Use when creating or
  revising prose that may benefit from dashes, quotation marks, apostrophes,
  ellipses, section, copyright, trademark, or similar non-ASCII marks while
  respecting source safety, rendering, and machine-consumed text.
---

# Unicode punctuation

If `$unicode-punctuation` is available, stop using this fallback and use that
skill instead. Never apply both `$unicode-punctuation` and
`$unicode-punctuation-fallback` to the same task.

Use Unicode punctuation and typographic symbols in new or directly revised
human-readable prose when they are safe and semantically appropriate. Do not
normalize unrelated passages merely because they use different punctuation.

## Resolve applicable constraints

Apply constraints in this order:

1.  Follow applicable explicit instructions and style requirements.
2.  Follow enforced compiler, formatter, linter, parser, or renderer
    requirements.
3.  Follow a clear implicit convention only when it is evident in the exact file
    being modified.
4.  Otherwise, apply this skill's defaults.

Inspect project configuration or instructions when needed to identify explicit
or enforced requirements. Do not search neighboring files or the broader
codebase merely to discover implicit punctuation precedent. Treat an implicit
convention that is absent or ambiguous in the target file as no convention.

## Decide how to encode the character

Before inserting literal non-ASCII text, use `$unicode-source-safety` when
available; otherwise use `$unicode-source-safety-fallback`. Never apply both.
Apply the selected skill to the exact target file and supported processing path.

- If the result is `Safe`, use the literal character when it is otherwise
  appropriate.
- If the result is `Unsafe` and the exact parsed documentation context defines
  an escape that renders the intended character there, use that escape.
- Otherwise, use conventional ASCII punctuation or rewrite the prose without the
  character.

Use a documentation escape only where the applicable parser interprets it. Never
put one where it will display literally, such as an ordinary source comment, a
verbatim region, or a code span. Treat a documentation comment as a parsed
context only when its actual documentation tool expands the escape at that exact
position.

## Choose punctuation by meaning

Choose each character for its actual prose meaning rather than as decoration.
Distinguish hyphens, minus signs, en dashes, and em dashes; distinguish straight
machine delimiters from prose quotation marks and apostrophes. Use ellipses,
`§`, `©`, `®`, `™`, and comparable symbols only when they convey their
conventional meaning in the sentence.

When an em dash is likely to be rendered in a monospaced font, surround it with
ordinary spaces: `word — word`. This normally includes ordinary source comments,
plain-text or terminal documentation, and monospaced rendered blocks. When an
escape renders the dash, put the spaces outside the escape. Do not infer
monospaced output solely because the source is edited in a monospaced editor. In
other contexts, follow the applicable spacing convention.

Preserve machine-consumed or exact text, including code spans and blocks,
identifiers, commands, paths, URLs, regular expressions, protocol values,
directives, serialized data, fixtures, and quoted source text. Do not replace
their ASCII characters with typographic lookalikes.

When punctuation is part of a person's preferred name, use
`$represent-person-names` when available; otherwise use
`$represent-person-names-fallback`. Never apply both. Defer to the selected
skill instead of normalizing the name under this skill.

During review, do not report conventional ASCII punctuation solely because a
Unicode alternative is available. Report it only when it violates a controlling
requirement or is part of prose already being revised where the Unicode form is
both safe and appropriate.
