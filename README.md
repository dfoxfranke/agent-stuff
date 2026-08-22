# Agent Stuff

This is my personal collection of AI agent prompts, skills, and
policies.

Everything in the skill collection pertains to coding style, with
design-by-contract as the unifying theme. It is almost all about tests and
documentation: document all items, even small private helpers; the documentation
is the contract; tests exercise that contract and nothing beyond it. But, the
discpline of conforming to this style tends to also to lead to elegant, readable
code even though the style doesn't have much to say about the code itself.

For humans, [STYLE.md](STYLE.md) provides a more concise gloss of what the
skills say.

In `prompts/`, there is a sample prompt telling an agent to bring the entire
codebase up to that standard. I have found it to be shockingly effective: AI
slop goes in, beautiful code comes out.

The skills have two versions:

* `global-skills/` contains user-wide fallback skills. Install or symlink its
  individual child directories into `$HOME/.agents/skills`.
* `repo-skills/` contains repository-authoritative skills. Install or symlink
  the desired child directories into `<repo>/.agents/skills` and commit them
  with that repository when appropriate.

Each global skill has a `-fallback` name. Its description and instructions defer
to the corresponding unsuffixed repository skill whenever that skill is
available, and the two versions must never be applied together. This is a
semantic deduplication mechanism: the fallback remains available in repositories
that do not install the authoritative version without requiring an installer or
per-repository global configuration. However, the skills are *not* merely
identical but-for the fallback instruction. The repository skills impose a
particular coding style as policy, while the fallback skills say to defer to 
repository conventions.

Be aware that imposing this style on your codebase may be somewhat discouraging
to would-be human contributors. It is quite pleasant to read and review, but
rather labor-intensive to write.

[AI_POLICY.md](AI_POLICY.md) outlines a policy around AI-assisted contributions
that I use for my projects. In short, they're welcome, but: 1. Humans must be
the face of all communication with project maintainers and assume final
responsibility for all contributions.
2. Tool-assisted contributions must include commit trailers noting what tools
   were used.

[AGENTS.md](AGENTS.md) is meant to be installed globally, and mostly corrects
some nonsense that's in Codex's system prompts. If you don't use Codex as a
harness, you probably don't need it. Do take note of the paragraph about license
and copyright identifiers, as the instructions there might not fit everyone.
