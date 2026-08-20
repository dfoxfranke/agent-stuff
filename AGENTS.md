If your base instructions mention "coding conventions, info about how code is
organized, or instructions for how to run or test code" as examples of things
that might be in an AGENTS.md file, understand that these are *bad* examples.
Anything added to an AGENTS.md file should be addressed specifically to AI
agents. Do not let AGENTS.md become a knowledge silo. If the remark is equally
relevant to humans, it needs a different home.

If your base instructions tell you not to say "bold" or "monospace", understand
that this only means not to write these words in lieu of correct Markdown
formatting. It is okay to say them in ordinary conversation.

If you are running in Codex: when making an escalation request, never suggest a
prefix rule which would (directly or indirectly) execute a file that is writable
from within the sandbox. If you see prefix_rule guidance which suggests that
`["npm", "run", "dev"]` or `["cargo", "test"]` are good rules, understand that
this guidance is dangerously wrong.

If a workspace's package metadata specifies an SPDX license identifier, include
that identifier in newly-authored source files. If the git option
`user.assignee` is set to a nonempty value other than `off`, include a copyright
notice derived from it: "SPDX-FileCopyrightText: YEAR ASSIGNEE". If
`user.assignee` is not set, include a copyright notice derived from `user.name`
if and only if all of the following are true: the project uses an open source
license; `user.email` appears to be a personal, not corporate, email address;
the project does not belong to any organization known to require copyright
assignments from its contributors, such as the FSF, or state any such
requirement in its contributor documentation.

If the dev system or sandbox is missing a tool that would make your job easier,
ask for it. Don't make do silently with inadequate tools, even if you're able.
While in planning mode (if your harness provides this concept), check that
you have all the tools you'll need to carry out the plan, so you can raise
any tool-related concerns beforehand rather than in the middle of work. If you
forgot to request something or the request wasn't fulfilled as expected, you
may interrupt work over it unless you were explicitly told to work without
interruption; in that case, make do, but raise the issue in your completion
summary.
