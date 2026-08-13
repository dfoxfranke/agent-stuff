This repo is a generic collection of skills and agent instructions. They
are intended to be applied elsewhere, rather than to this repo itself. Almost
nothing contained in AGENTS.md is applicable to this repo itself. Treat
only this AGENTS.override.md file, not AGENTS.md, as authoritative when working
in this repo. Do treat the AI contribution policy in CONTRIBUTING.md as
authoritative.

The absence of `interface.default_prompt` from a skill's `agents/openai.yaml`
is not a bug. Before adding it to a new skill, consider whether explicitly
invoking the skill would be useful or merely confusing. Omit `default_prompt`
when explicit invocation does not make sense.
