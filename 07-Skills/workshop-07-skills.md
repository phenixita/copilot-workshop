<img src="https://mfitcontent.blob.core.windows.net/static/MF-logo.png" style="height:64px;margin-right:32px"/>


# Workshop 7 — Agent Skills


> **Estimated time: 60 minutes** (4 exercises)
>
> **Difficulty: medium** — this workshop teaches you how to create, use, and share Agent Skills in GitHub Copilot. Agent Skills are portable folders of instructions, scripts, and resources that the AI loads on demand to perform specialized tasks. Unlike custom instructions (which are always active), skills are loaded only when relevant — keeping the context window clean and focused. By the end you will have built a reusable skill from scratch and shared it with your team.

AI-Assisted Development — Skills Module

## Terms and Conditions of Use

This training package is proprietary and confidential and is intended exclusively for the uses described in the training materials. Copying or disclosing all or part of the content and/or software included in these packages is prohibited. The contents of this package are for informational and training purposes only and are provided "as is" without warranties of any kind, express or implied, including, but not limited to, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement. The content of the training package, including URLs and other references to Internet websites, is subject to change without notice. Unless otherwise noted, the companies, organizations, products, domain names, email addresses, logos, people, places, and events depicted herein are fictitious and no association with any real company, organization, product, domain name, email address, logo, person, place, or event is intended or should be inferred.

---

## The Scenario

Your team has conventions that the AI keeps getting wrong. Unit tests should use a specific structure. API endpoints must follow a naming pattern. Commit messages need a particular format. You've been copy-pasting the same instructions into every chat session, and it's getting old.

Agent Skills solve this by packaging instructions, examples, and scripts into a self-contained folder. When the AI encounters a task that matches a skill's description, it loads the skill automatically — or you can reference it explicitly. Skills are an open standard that works across GitHub Copilot in VS Code, Copilot CLI, and the GitHub Copilot coding agent.

> **Remember:** skills are different from custom instructions. Custom instructions are always loaded into every conversation (they consume context window space permanently). Skills are loaded on demand, only when relevant. This makes skills ideal for specialized workflows that you don't need in every interaction.

---

## Exercise 1: Anatomy of an Agent Skill

Before building your own skill, you need to understand the structure. This exercise walks you through exploring an existing skill to learn its conventions, then identifying each component and its purpose.

### Prerequisites

Before starting, make sure you have:

- **Visual Studio Code** installed and updated to the latest stable version
- **GitHub Copilot** activated and working (Workshop 1 completed)
- **Agent mode** enabled (`"chat.agent.enabled": true`)
- A **Git repository** open in VS Code with an existing codebase (your own project or clone [Marvel Zombies Hero](https://github.com/phenixita/marvel-zombies-hero/) as a fallback)

### Objective

1. Understand the folder structure of an Agent Skill
2. Identify the role of `SKILL.md`, scripts, examples, and resources
3. Understand how the AI discovers and loads skills

**Step 1 — Understand the skill folder structure**

An Agent Skill is a folder with a specific structure. At minimum, it contains a `SKILL.md` file that describes what the skill does and how to use it. Here is the general layout:

```
.github/skills/
└── my-skill/
    ├── SKILL.md          # Required: instructions for the AI
    ├── scripts/           # Optional: automation scripts the AI can run
    │   └── validate.sh
    ├── examples/          # Optional: example inputs/outputs for the AI to learn from
    │   ├── good-example.ts
    │   └── bad-example.ts
    └── resources/         # Optional: reference docs, templates, schemas
        └── api-schema.json
```

| Component | Required? | Purpose |
|-----------|-----------|---------|
| `SKILL.md` | **Yes** | The main instruction file. Contains the skill's description, rules, and step-by-step guidance for the AI. This is what the AI reads when the skill is activated. |
| `scripts/` | No | Executable scripts the AI can call during its work. For example, a linter, a test runner, or a code generator. |
| `examples/` | No | Concrete examples of correct (and incorrect) output. These teach the AI by showing, not telling. |
| `resources/` | No | Reference materials: JSON schemas, API specs, templates, or any file the AI might need to consult. |

**Step 2 — Examine the `SKILL.md` format**

The `SKILL.md` file is a standard Markdown file with an optional YAML frontmatter block. Here is a minimal example:

```markdown
---
name: test-generator
description: Generates unit tests following project conventions. Uses Jest with TypeScript, follows AAA pattern (Arrange-Act-Assert), and includes edge cases.
---

# Test Generator Skill

## When to use

Use this skill when the user asks to:
- Generate tests for a function, class, or module
- Add missing test coverage
- Create test scaffolding for a new feature

## Conventions

- Framework: Jest with TypeScript
- Pattern: Arrange-Act-Assert (AAA)
- Naming: `describe('ClassName')` → `it('should [expected behavior] when [condition]')`
- Each test file mirrors the source file path: `src/utils/parser.ts` → `tests/utils/parser.test.ts`

## Process

1. Read the source file to understand the function signatures and behavior
2. Identify edge cases: null inputs, empty arrays, boundary values, error conditions
3. Generate tests following the conventions above
4. Run the tests to verify they compile and execute correctly
```

> **Key insight:** the `description` field in the frontmatter is critical. This is what the AI uses to decide whether to load the skill for a given task. A vague description means the skill will be loaded when it shouldn't be (or worse, never loaded at all). Write the description as if you're explaining to a colleague when to use this skill.

**Step 3 — Understand how skills are discovered**

The AI discovers skills through two mechanisms:

1. **Automatic discovery:** the AI reads the skill's `description` and decides whether it's relevant to the current task. If it is, the AI loads the `SKILL.md` and any referenced files.
2. **Explicit reference:** you can mention a skill directly in your prompt using its name or path. For example: "Use the test-generator skill to create tests for this module."

Skills are discovered from these locations:

| Location | Scope | Best for |
|----------|-------|----------|
| `.github/skills/` in your workspace | Team (version-controlled) | Shared conventions, project-specific workflows |
| User profile skills directory | Personal (all workspaces) | Individual preferences, cross-project tools |
| Installed plugins | Plugin (marketplace) | Community-shared skills |

### Success Criteria

- [ ] You can describe the four components of a skill folder (`SKILL.md`, `scripts/`, `examples/`, `resources/`) and the role of each
- [ ] You understand the difference between automatic discovery (via `description`) and explicit reference
- [ ] You can explain why the `description` field in the frontmatter matters for skill activation

---

## Exercise 2: Skills vs Custom Instructions — When to Use What

Skills and custom instructions both influence the AI's behavior, but they serve different purposes. Choosing the wrong one leads to either a bloated context window (custom instructions for everything) or inconsistent behavior (skills that should always be active). This exercise sharpens your judgment through five real-world scenarios.

### Prerequisites

Before starting, make sure you have:

- Understanding of skill structure (Exercise 1 completed)
- Familiarity with custom instructions from Workshop 1 (the `.github/copilot-instructions.md` file and `settings.json` instructions)

### Objective

1. Understand the key differences between skills and custom instructions
2. Apply a decision framework to five scenarios
3. Justify your choice for each scenario

### Background: The Decision Framework

| Criterion | Custom Instructions | Agent Skills |
|-----------|-------------------|-------------|
| **When loaded** | Always — every conversation | On demand — only when relevant |
| **Context cost** | Permanent (consumes context window in every session) | Temporary (loaded only when needed) |
| **Scope** | Broad rules: coding standards, language, framework | Specialized workflows: testing, debugging, deployment |
| **Contents** | Text only (Markdown instructions) | Folder: instructions + scripts + examples + resources |
| **Portability** | Workspace-specific (`.github/copilot-instructions.md`) or user settings | Cross-tool standard (VS Code, CLI, coding agent) |

**Rule of thumb:** if you want the AI to *always* follow a rule (like "use single quotes in JavaScript"), use custom instructions. If you want the AI to *sometimes* perform a specialized task (like "generate tests following our conventions"), use a skill.

---

**Step 1 — Evaluate five scenarios**

For each scenario, decide: **custom instruction** or **skill**? Write down your reasoning before checking the answer.

**Scenario A:** Your team uses TypeScript with strict null checks. Every file should include `// @ts-strict` at the top. This applies to all code the AI generates, in every conversation.

**Scenario B:** Twice a month, you need to generate a changelog from Git history. The changelog follows a specific format with sections for features, fixes, and breaking changes.

**Scenario C:** Your project follows a REST API naming convention: plural nouns for collections, kebab-case for multi-word resources, consistent use of HTTP verbs. This applies to every API endpoint the AI generates.

**Scenario D:** Your team has a complex database migration workflow: generate a migration file, validate the schema, run a dry-run, and check for backwards compatibility. This involves running three different scripts in sequence.

**Scenario E:** All code the AI generates should include JSDoc comments on public functions. This is a universal rule, not a specialized task.

**Step 2 — Check your answers**

| Scenario | Answer | Reasoning |
|----------|--------|-----------|
| A | **Custom instruction** | Universal rule, applies to every file, no scripts or resources needed — just a one-line directive |
| B | **Skill** | Specialized task, performed occasionally, involves scripts (Git log parsing) and a template (changelog format) |
| C | **Custom instruction** | Universal convention, applies to all API code, pure text rules — no scripts or resources needed |
| D | **Skill** | Complex workflow with scripts, performed only during migrations, would waste context if loaded in every session |
| E | **Custom instruction** | Universal rule for all code, no scripts, should always be active |

**Step 3 — Reflect on edge cases**

Some scenarios are borderline. For example, Scenario C could also be a skill if it includes an OpenAPI schema as a resource and a validation script. The key question is: **does this need scripts, examples, or resources beyond plain text instructions?** If yes, lean towards a skill. If it's just a rule, lean towards custom instructions.

> **Tip:** when in doubt, start with a custom instruction. If you find yourself adding examples, scripts, or reference files, promote it to a skill.

### Success Criteria

- [ ] You have evaluated all five scenarios and written down your reasoning
- [ ] You correctly identified at least 4 out of 5 answers
- [ ] You can explain the decision framework: always-active text rules → custom instructions; on-demand workflows with scripts/resources → skills

---

## Exercise 3: Create a Skill from Scratch

Time to build. In this exercise you will create a complete Agent Skill for generating unit tests following your project's conventions. The skill includes instructions, an example file, and a validation script.

### Prerequisites

Before starting, make sure you have:

- Understanding of skill structure and the decision framework (Exercises 1–2 completed)
- A Git repository open in VS Code with at least a few source files (any language)
- A test framework installed in the project (Jest, pytest, xUnit, or similar)

### Objective

1. Create the skill folder structure
2. Write the `SKILL.md` with clear instructions and conventions
3. Add an example file showing the expected test format
4. Add a validation script that runs the generated tests
5. Test the skill by asking the AI to generate tests

---

**Step 1 — Create the folder structure**

In your project root, create the skill directory:

```
.github/skills/test-generator/
```

You will add three files to this folder in the next steps.

**Step 2 — Write the `SKILL.md`**

Create `.github/skills/test-generator/SKILL.md` with the following content. Adapt the framework and conventions to match your project — the example below uses Jest with TypeScript:

```markdown
---
name: test-generator
description: Generates unit tests following project conventions. Uses Jest with TypeScript, AAA pattern, and includes edge cases. Use when asked to create tests, add test coverage, or scaffold test files.
---

# Test Generator

## When to activate

Activate this skill when the user asks to:
- Generate unit tests for a function, class, or module
- Add missing test coverage
- Create test scaffolding for a new feature
- Write tests for a bug fix (regression tests)

## Conventions

- **Framework:** Jest with TypeScript
- **Pattern:** Arrange-Act-Assert (AAA) — each test has three clearly separated sections
- **File location:** mirror the source path — `src/utils/parser.ts` → `tests/utils/parser.test.ts`
- **Test naming:** `describe('ClassName or functionName')` → `it('should [behavior] when [condition]')`
- **Edge cases:** always include tests for: null/undefined inputs, empty collections, boundary values, error conditions
- **Mocking:** use `jest.mock()` for external dependencies; never mock the unit under test
- **Assertions:** prefer specific matchers (`toEqual`, `toContain`, `toThrow`) over generic ones (`toBeTruthy`)

## Process

1. Read the source file to understand function signatures, types, and behavior
2. Check if a test file already exists — if so, add to it rather than replacing it
3. Identify the happy path and at least 3 edge cases per function
4. Generate tests following the conventions above
5. Run the validation script to verify the tests compile and pass: `bash .github/skills/test-generator/scripts/validate.sh`
 

See `examples/example.test.ts` for a concrete example of the expected format.
```

**Step 3 — Add an example test file**

Create `.github/skills/test-generator/examples/example.test.ts`:

```typescript
import { calculateDiscount } from '../../src/pricing/discount';

describe('calculateDiscount', () => {
  // Happy path
  it('should return the correct discounted price for a valid percentage', () => {
    // Arrange
    const originalPrice = 100;
    const discountPercent = 20;

    // Act
    const result = calculateDiscount(originalPrice, discountPercent);

    // Assert
    expect(result).toEqual(80);
  });

  // Edge case: zero discount
  it('should return the original price when discount is 0%', () => {
    // Arrange
    const originalPrice = 50;
    const discountPercent = 0;

    // Act
    const result = calculateDiscount(originalPrice, discountPercent);

    // Assert
    expect(result).toEqual(50);
  });

  // Edge case: 100% discount
  it('should return 0 when discount is 100%', () => {
    // Arrange
    const originalPrice = 75;
    const discountPercent = 100;

    // Act
    const result = calculateDiscount(originalPrice, discountPercent);

    // Assert
    expect(result).toEqual(0);
  });

  // Error condition: negative price
  it('should throw an error when the original price is negative', () => {
    // Arrange
    const originalPrice = -10;
    const discountPercent = 20;

    // Act & Assert
    expect(() => calculateDiscount(originalPrice, discountPercent)).toThrow(
      'Price must be non-negative'
    );
  });

  // Error condition: discount out of range
  it('should throw an error when discount percentage exceeds 100', () => {
    // Arrange
    const originalPrice = 100;
    const discountPercent = 150;

    // Act & Assert
    expect(() => calculateDiscount(originalPrice, discountPercent)).toThrow(
      'Discount must be between 0 and 100'
    );
  });
});
```

> **Why include an example?** The AI learns better from concrete examples than from abstract rules. By showing a complete test file with AAA comments, specific matchers, and edge cases, you set a clear standard that the AI will replicate.

**Step 4 — Add a validation script**

Create `.github/skills/test-generator/scripts/validate.sh`:

```bash
#!/bin/bash
# validate.sh — Compile and run generated tests to verify they work

set -e

echo "=== Compiling TypeScript ==="
npx tsc --noEmit

echo ""
echo "=== Running tests ==="
npx jest --passWithNoTests --verbose

echo ""
echo "=== All tests passed ==="
```

Make it executable:

```bash
chmod +x .github/skills/test-generator/scripts/validate.sh
```

```powershell
# No direct equivalent for chmod needed on Windows; ensures script is readable/executable by current user
```

> **Note:** adapt this script to your project's toolchain. For Python projects, replace with `pytest -v`. For .NET, use `dotnet test`. The key point is that the skill includes a runnable validation step.

**Step 5 — Test the skill**

Now ask the AI to use the skill. Open the Chat panel and try:

```
Generate unit tests for <component of your choice>. Follow our test-generator conventions.
```

- Observe: the AI should load the `test-generator` skill and produce tests that follow the AAA pattern, use the correct file naming convention, and include edge cases
- Check that the generated tests reference the example style (AAA comments, specific matchers)
- If the agent runs the validation script, verify it passes

If the AI doesn't load the skill automatically, reference it explicitly:

```
Use the test-generator skill to create tests.
```

**Step 6 — Verify the folder structure**

Your final skill folder should look like this:

```
.github/skills/test-generator/
├── SKILL.md
├── examples/
│   └── example.test.ts
└── scripts/
    └── validate.sh
```

### Success Criteria

- [ ] The skill folder `.github/skills/test-generator/` exists with `SKILL.md`, `examples/`, and `scripts/`
- [ ] The `SKILL.md` contains a `description` in the frontmatter, conventions, a process, and a reference to the example
- [ ] The example file demonstrates the exact test format you expect (AAA, naming, matchers)
- [ ] The validation script runs and verifies that generated tests compile and pass
- [ ] The AI generated at least one test file that follows the conventions defined in the skill

---

## Exercise 4: Share a Skill with Your Team

A skill is only valuable if the team can use it. This exercise covers how to commit a skill to your repository, verify it works for other developers, and keep it maintainable over time.

### Prerequisites

Before starting, make sure you have:

- A working skill in `.github/skills/` (Exercise 3 completed)
- A Git repository with at least one remote

### Objective

1. Commit the skill to version control
2. Verify the skill works after a fresh clone
3. Document the skill for your team
4. Understand cross-tool compatibility (VS Code, CLI, coding agent)

**Step 1 — Commit the skill**

Add the skill folder to Git and commit it:

```bash
git add .github/skills/test-generator/
git commit -m "feat(skills): add test-generator skill with Jest/TS conventions"
```

> **Best practice:** treat skills like code. Review them in PRs, version them, and test them before merging. A broken skill can produce incorrect tests for the entire team.

**Step 2 — Verify after a fresh clone**

To confirm the skill works for other team members:

- Clone the repository to a different directory (or ask a colleague to pull):
  ```bash
  git clone <repo-url> /tmp/test-clone
  cd /tmp/test-clone
  ```
- Open the cloned project in VS Code
- Open the Chat panel and ask the AI to generate tests:
  ```
  Generate tests for the main module using our test-generator skill.
  ```
- Verify that the AI finds and loads the skill from `.github/skills/test-generator/`
- Verify that the generated tests follow the conventions

**Step 3 — Add a README for your team**

Create a brief `README.md` inside the skill folder to help teammates understand what the skill does without reading the full `SKILL.md`. This is for humans, not for the AI:

Create `.github/skills/test-generator/README.md`:

```markdown
# Test Generator Skill

Generates unit tests following our project conventions.

## What it does
- Creates Jest/TypeScript tests in AAA (Arrange-Act-Assert) format
- Mirrors source file paths: `src/foo.ts` → `tests/foo.test.ts`
- Includes edge cases: null inputs, empty collections, boundary values
- Runs a validation script to verify tests compile and pass

## How to use
Ask Copilot to generate tests — the skill activates automatically when relevant.
You can also reference it explicitly: "Use the test-generator skill to..."

## Maintaining this skill
- Update `SKILL.md` when test conventions change
- Update `examples/example.test.ts` when the expected format changes
- Test changes by asking the AI to generate a few tests after your update
```

**Step 4 — Understand cross-tool compatibility**

Agent Skills work across multiple GitHub Copilot surfaces:

| Surface | Discovers skills from | Notes |
|---------|----------------------|-------|
| **VS Code (Chat/Agent)** | `.github/skills/` in workspace | Full support: reads `SKILL.md`, runs scripts, loads examples |
| **Copilot CLI** | `.github/skills/` in current directory | Supports `SKILL.md` and scripts; examples referenced by path |
| **GitHub Copilot coding agent** | `.github/skills/` in repository | Runs autonomously on GitHub infrastructure; scripts must be self-contained |

> **Key takeaway:** because skills follow an open standard, a skill you write for VS Code also works in the CLI and on GitHub's coding agent. This makes skills the most portable way to encode team conventions.

### Success Criteria

- [ ] The skill is committed to the repository under `.github/skills/test-generator/`
- [ ] The skill works after a fresh clone (verified in a separate directory or by a colleague)
- [ ] A `README.md` exists inside the skill folder for human documentation
- [ ] You can explain which Copilot surfaces support Agent Skills and any differences between them

---

## Summary: What You Have Built

After this workshop, you have a complete understanding of Agent Skills:

| Component | Status | Notes |
|-----------|--------|-------|
| **Skill anatomy** | ✅ Understood | `SKILL.md`, `scripts/`, `examples/`, `resources/` — and when each is needed |
| **Decision framework** | ✅ Applied | Custom instructions for universal rules; skills for on-demand specialized workflows |
| **Test Generator skill** | ✅ Built | Complete skill with instructions, example, and validation script |
| **Team sharing** | ✅ Done | Skill committed, verified on fresh clone, documented with README |

### The Skill Design Rule

> A good skill is **self-contained**: another developer should be able to understand it from the `SKILL.md` alone, run any scripts without additional setup, and get consistent results across VS Code, CLI, and the coding agent. If a skill requires tribal knowledge to use, it needs better documentation.

---

### Useful Resources

- [Agent Skills overview](https://code.visualstudio.com/docs/copilot/agents/agent-skills)
- [Creating Agent Skills](https://code.visualstudio.com/docs/copilot/agents/agent-skills#_create-a-skill)
- [Skills specification (open standard)](https://github.com/anthropics/agent-skills-spec)
- [Custom instructions reference](https://code.visualstudio.com/docs/copilot/copilot-customization)
- [Copilot CLI — using skills](https://docs.github.com/en/copilot/how-tos/copilot-cli/cli-best-practices)