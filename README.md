# Agent Stuff

This is my personal collection of AI agent instructions, skills, and
policies. They're mostly Rust-centric but written to be usable anywhere.

## Skills

The skill sources are split by installation scope:

* `global-skills/` contains user-wide fallback skills. Install or symlink its
  individual child directories into `$HOME/.agents/skills`.
* `repo-skills/` contains repository-authoritative skills. Install or symlink
  the desired child directories into `<repo>/.agents/skills` and commit them
  with that repository when appropriate.

Codex scans repository skill directories from the current working directory up
to the repository root, as well as the user-wide directory. See the official
[Build skills](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills)
documentation for the full discovery rules and supported locations.

Each global skill has a `-fallback` name. Its description and instructions
defer to the corresponding unsuffixed repository skill whenever that skill is
available, and the two versions must never be applied together. This is a
semantic deduplication mechanism: the fallback remains available in repositories
that do not install the authoritative version without requiring an installer or
per-repository global configuration.

The repository-only `work-in-vendored-forks` skill preserves upstream
conventions when a task changes third-party source maintained as a downstream
fork. It does not apply merely because first-party code interfaces with a
third-party dependency.
