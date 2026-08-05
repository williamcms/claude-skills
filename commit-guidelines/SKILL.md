---
name: commit-guidelines
description: Apply when writing, proposing, or running a git commit message. Covers the mandatory approve-the-exact-message-before-committing workflow, message formatting (line-length limits, dash-bullet complements, tense, tone), verifying against git history before describing a change as moved or refactored rather than newly added, and never attributing an LLM as commit co-author without prior confirmation. Use whenever composing a commit message, about to run `git commit`, or asked to "create a commit" versus just "create a commit message".
---

# Commit guidelines

## Before running `git commit`

- **Never run `git commit` without the user approving the exact message first.** "Create a commit" means present the message, wait for approval, then commit. "Create a commit message" means the message only, no git commands.
- **Never attribute an LLM/AI assistant as commit co-author** (a `Co-Authored-By:` trailer for the assistant) unless the user has explicitly confirmed, for this commit or as a standing preference, that they want it. A commit message suggestion alone is not proof that an LLM/AI is actively working on the repo. Default: ask first.
- **Before describing a change as moved, renamed, or refactored instead of newly added, verify it against git history**, not against what the working tree or staging area currently looks like. Run `git log` and `git show <past-ref>:<path>` on the affected file; only call something a "move" if a past commit actually contained the prior state. Content that only ever existed uncommitted (working tree or index) was never "moved" from anywhere, it is a first-time addition.

## Message format

- First line: max 50 characters, concise summary. Keep any provided prefixes (e.g., `feat:`, `fix:`, `refactor:`), and adjust wording to meet the 50-char limit.
- If a prefix was not provided, ask if the user wants one added, and suggest one based on the Conventional Commits spec (https://www.conventionalcommits.org/en/v1.0.0/).
- After a blank line, one or more complement lines may follow, each max 70 characters.
- Prefix each complement line with "- " (a dash bullet).
- Each complement bullet is one self-contained statement on a single line, within the 70-char limit. Never wrap a statement onto a continuation line; if it doesn't fit, rephrase it as two or more shorter bullets instead.
- Keep component or module names only if relevant or included by the user.
- Use past tense in descriptions (e.g., "adjusted" instead of "adjust").
- No markdown formatting in commit messages (no backticks, no code formatting) except for lists.
- Match tone (technical or casual) to the original request unless told otherwise.
- Emphasize clarity, improved logic, and UI or behavioral context when rewriting a message.
- Present all commit message responses as plain code blocks for easy copying.
