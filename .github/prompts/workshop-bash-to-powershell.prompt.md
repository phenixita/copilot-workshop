---
agent: agent
description: Review a workshop Markdown file and add PowerShell equivalents only when the Bash commands materially differ.
---

You are updating a workshop document, usually a Markdown file, that contains command-line examples.

Your job is to inspect Bash command examples and add a matching PowerShell example only when a distinct PowerShell version is actually needed.

Arguments from the user:

${input:Target file, selection, or extra constraints}

Scope and intent:

- Focus on Markdown workshop content.
- Look for fenced code blocks labeled `bash`, `sh`, or `shell`.
- Treat the surrounding prose as part of the contract: preserve tone, ordering, headings, and learning flow.
- Prefer editing the current file in place unless the user explicitly asks for a separate output.

Decision rule:

- Add a `powershell` code block immediately after the Bash block only when the PowerShell version would differ in syntax, semantics, or required conventions.
- Do not duplicate commands that are already valid and effectively identical in PowerShell, such as many `git`, `node`, `npm`, `npx`, `docker`, or `copilot` invocations.
- Do not add a PowerShell block just to restate the same command with cosmetic changes.
- If a block already has a PowerShell equivalent, improve it only if it is clearly incomplete or incorrect.

What usually requires conversion:

- Variable assignment or expansion.
- Environment variables.
- Command substitution.
- Pipelines that rely on Unix text tools or stream behavior.
- File and directory commands that differ materially.
- Shell control flow such as `if`, loops, exit handling, and tests.
- Scripts that are written for Bash syntax.

Conversion rules:

- Prefer idiomatic PowerShell 7 syntax that works in `pwsh`.
- Preserve the intent of the original command, not its exact token-by-token shape.
- When a Bash snippet is multi-line, provide a complete PowerShell alternative for the whole snippet.
- Keep explanations inside the code block minimal and only when they prevent confusion.
- Preserve existing Markdown formatting and keep changes tightly local to the relevant section.
- If a reliable PowerShell translation would be misleading or too speculative, leave the Bash block as-is and briefly explain why outside the code block.

Quality bar:

- Avoid shallow alias-based rewrites when PowerShell needs a different approach.
- Avoid changing unrelated prose.
- Avoid adding Windows-only assumptions unless the workshop explicitly targets Windows.
- Prefer cross-platform PowerShell behavior when possible.

Response format:

1. Update the Markdown file.
2. Summarize which Bash blocks received a PowerShell companion.
3. Call out any blocks intentionally left unchanged because the commands are effectively the same.
4. Mention any ambiguous cases where a human review would still be useful.