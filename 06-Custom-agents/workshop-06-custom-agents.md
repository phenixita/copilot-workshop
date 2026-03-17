a<img src="https://mfitcontent.blob.core.windows.net/static/MF-logo.png" style="height:64px;margin-right:32px"/>


# Workshop 6 — Custom Agents


> **Estimated time: 75 minutes** (5 exercises)
>
> **Difficulty: medium** — this workshop teaches you how to create, configure, and orchestrate custom agents in GitHub Copilot. You will define agent personas with dedicated tools and models, build sequential workflows with handoffs, and control which agents can be invoked by users versus only by other agents. By the end you will have a working multi-agent setup tailored to your development workflow.

AI-Assisted Development — Custom Agents Module

## Terms and Conditions of Use

This training package is proprietary and confidential and is intended exclusively for the uses described in the training materials. Copying or disclosing all or part of the content and/or software included in these packages is prohibited. The contents of this package are for informational and training purposes only and are provided "as is" without warranties of any kind, express or implied, including, but not limited to, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement. The content of the training package, including URLs and other references to Internet websites, is subject to change without notice. Unless otherwise noted, the companies, organizations, products, domain names, email addresses, logos, people, places, and events depicted herein are fictitious and no association with any real company, organization, product, domain name, email address, logo, person, place, or event is intended or should be inferred.

---

## The Scenario

Your team is growing, and different tasks demand different AI behaviors. A security review should never edit files — it should only read and report. An implementation task needs full editing power but should follow strict coding conventions. A code reviewer should focus on finding issues, not fixing them. Instead of writing a different prompt every time, you want reusable agent personas that encode these behaviors permanently.

Custom agents solve this problem. Each agent is a Markdown file (`.agent.md`) with a frontmatter block that defines its name, description, tools, model, and instructions. You invoke them by name in the Chat panel, and they behave exactly as configured — every time.

> **Remember:** custom agents can be stored in your workspace (shared with the team via version control) or in your user profile (personal, available across all workspaces). This workshop covers both approaches.

---

## Exercise 1: Create a Read-Only Security Reviewer Agent

This exercise walks you through creating your first custom agent: a Security Reviewer that can only read files and search the codebase — it cannot edit anything. This is the safest starting point because a misconfigured read-only agent cannot damage your project.

### Prerequisites

Before starting, make sure you have:

- **Visual Studio Code** installed and updated to the latest stable version
- **GitHub Copilot** activated and working (Workshop 1 completed)
- **Agent mode** enabled (`"chat.agent.enabled": true`)
- A **Git repository** open in VS Code with an existing codebase (your own project or clone [Marvel Zombies Hero](https://github.com/phenixita/marvel-zombies-hero/) as a fallback)

### Objective

1. Understand the structure of an `.agent.md` file
2. Create a Security Reviewer agent with read-only tools
3. Invoke the agent in the Chat panel and observe its behavior
4. Verify that the agent cannot edit files

**Step 1 — Create the agent file**

Custom agents live in a `.github/agents/` directory inside your workspace. Create the directory structure first:

- In your project root, create the folder `.github/agents/` (if it doesn't already exist)
- Inside that folder, create a new file called `security-reviewer.agent.md`

**Step 2 — Write the agent definition**

Open `security-reviewer.agent.md` and add the following content:

```markdown
---
name: Security Reviewer
description: Reviews code for security vulnerabilities, hardcoded secrets, and common attack vectors. Read-only — never edits files.
tools: [read, search/codebase]
---

You are a senior security engineer performing a code review. Your job is to find vulnerabilities, not fix them.

## Rules

- NEVER suggest editing files directly. Report findings only.
- Focus on: injection flaws, hardcoded secrets, missing input validation, insecure dependencies, authentication/authorization gaps.
- For each finding, report: severity (critical/high/medium/low), file and line, description, and remediation guidance.
- If you find no issues, say so explicitly and list what you checked.

## Output format

Use this structure for every review:

### Finding: [title]
- **Severity:** critical | high | medium | low
- **File:** `path/to/file.ext` (line N)
- **Description:** what the vulnerability is and why it matters
- **Remediation:** how to fix it
```

**Understanding the frontmatter:**
    
| Field | Value | Purpose |
|-------|-------|---------|
| `name` | `Security Reviewer` | Display name in the Chat panel and model picker |
| `description` | `Reviews code for security...` | Shown in the agent picker; also used by the AI to decide when to invoke this agent as a subagent |
| `tools` | `['read', 'search/codebase']` | Only these tools are available — no `editFiles`, no `terminalLastCommand`, no `runInTerminal` |

> **Key insight:** by omitting editing tools (`editFiles`, `runInTerminal`), you guarantee the agent can only observe, never modify. This is the principle of least privilege applied to AI agents.

**Step 3 — Invoke the Security Reviewer**

- Open the **Chat** panel (`Ctrl+Shift+I` / `Cmd+Shift+I`)
- Type `@` and look for **Security Reviewer** in the agent picker dropdown
- If it appears, select it and type your prompt:

```
Review the authentication logic in this project. Look for hardcoded secrets, missing input validation, and insecure token handling.
```

- If the agent does not appear, try reloading VS Code (`Ctrl+Shift+P` → `Developer: Reload Window`)

**Step 4 — Verify read-only behavior**

- Observe the agent's response: it should report findings with file paths and line numbers, but it should **never** propose file edits
- If the agent attempts to use an editing tool, it will be blocked because `editFiles` is not in its tool list
- Try explicitly asking the agent to fix something:

```
Fix the hardcoded API key you found in config.js
```

- The agent should decline or explain that it can only report findings, not make changes

**Step 5 — Compare with a regular agent session**

- Start a new chat session (click the `+` icon) without invoking the Security Reviewer
- Ask the same security review question to the default agent
- Compare: the default agent may try to fix issues directly, while the Security Reviewer only reports them

### Success Criteria

- [ ] The file `.github/agents/security-reviewer.agent.md` exists with the correct frontmatter and instructions
- [ ] The Security Reviewer appears in the `@` agent picker in the Chat panel
- [ ] The agent produces a structured security review with severity, file, description, and remediation for each finding
- [ ] The agent does not attempt to edit files, even when explicitly asked to

---

## Exercise 2: Pin a Model to an Agent

Different agents benefit from different models. A security reviewer needs deep reasoning to spot subtle vulnerabilities. A quick helper for formatting or renaming can use a fast, cheap model. This exercise shows you how to pin a specific model to an agent using the `model` property in the frontmatter.

### Prerequisites

Before starting, make sure you have:

- Security Reviewer agent created (Exercise 1 completed)
- Access to multiple models in your Copilot subscription

### Objective

1. Understand the `model` property in the agent frontmatter
2. Pin a reasoning model to the Security Reviewer
3. Create a second agent with a fast model for comparison
4. Observe the difference in output quality and speed

**Step 1 — Add a model to the Security Reviewer**

Open `.github/agents/security-reviewer.agent.md` and add the `model` property to the frontmatter:

```markdown
---
name: Security Reviewer
description: Reviews code for security vulnerabilities, hardcoded secrets, and common attack vectors. Read-only — never edits files.
tools: ['codebase', 'fetch', 'findTestFiles']
model: claude-sonnet-4.6
---
```

> **Why Claude Sonnet 4.6?** Security review requires reasoning about control flow, data flow, and trust boundaries. A reasoning-capable model catches vulnerabilities that a fast model might miss. For the most critical reviews, you could use `claude-opus-4.6` (3x multiplier) — but Sonnet is a good default balance.

**Step 2 — Create a Quick Helper agent with a fast model**

Create a new file `.github/agents/quick-helper.agent.md`:

```markdown
---
name: Quick Helper
description: Fast answers for simple questions — formatting, renaming, syntax lookup, quick explanations. Uses a lightweight model for speed.
tools: ['codebase']
model: gpt-5-mini
---

You are a fast, concise coding assistant. Keep answers short and practical.

## Rules

- Answer in 1–3 sentences when possible
- Show code snippets inline, not in separate files
- If the question requires deep analysis, suggest using a more capable agent instead
- Prefer examples over explanations
```

**Step 3 — Test both agents side by side**

Ask both agents the same question and compare:

- Invoke the Security Reviewer:

```
@Security Reviewer Analyze the error handling in this project. Are there any places where exceptions are caught but not properly logged or re-thrown?
```

- Start a new session and invoke the Quick Helper with a simple task:

```
@Quick Helper How do I convert a JavaScript Date object to an ISO 8601 string?
```

- Observe: the Security Reviewer should give a thorough analysis with findings and recommendations. The Quick Helper should give a fast, concise answer.

**Step 4 — Verify model pinning**

- When the agent responds, check the model indicator in the Chat panel
- The Security Reviewer should use the model specified in its frontmatter (`claude-sonnet-4.6`), regardless of what model is selected in the global model picker
- The Quick Helper should use `gpt-5-mini`

> **Tip:** if you hover over the model name in the chat, you can see the full model identifier and confirm it matches the frontmatter.

### Success Criteria

- [ ] The Security Reviewer's frontmatter includes `model: claude-sonnet-4.6`
- [ ] The Quick Helper agent exists with `model: gpt-5-mini`
- [ ] Each agent uses its pinned model regardless of the global model picker selection
- [ ] You can articulate why different agents benefit from different models

---

## Exercise 3: Build a Handoff Workflow — Researcher to Implementer

Real development tasks often require two distinct phases: first understand the problem, then solve it. Handoffs let you create guided sequential workflows where one agent finishes its work and suggests the next agent to use. This exercise builds a Researcher → Implementer pipeline.

### Prerequisites

Before starting, make sure you have:

- Familiarity with creating `.agent.md` files (Exercises 1–2 completed)
- A project with at least a few source files open in VS Code

### Objective

1. Understand the handoff mechanism between agents
2. Create a Researcher agent with read-only tools
3. Create an Implementer agent with editing tools
4. Test the full Researcher → Implementer handoff flow

**Step 1 — Create the Researcher agent**

Create `.github/agents/researcher.agent.md`:

```markdown
---
name: Researcher
description: Researches codebase patterns, gathers context, and produces a summary of findings. Read-only — never edits files.
tools: ['search/codebase', 'fetch', 'usages']
model: claude-sonnet-4.6
handoffs:
  - agent: Implementer
    description: Hand off to the Implementer when the research is complete and you have a clear picture of what needs to change.
---

You are a senior developer conducting a thorough code review before making changes. Your job is to understand, not to act.

## Process

1. Read the relevant files and understand the current implementation
2. Search for patterns: how similar features are implemented elsewhere in the codebase
3. Identify dependencies and potential side effects
4. Produce a structured research summary

## Output format

### Research Summary: [topic]

**Files analyzed:** list of files you read
**Current implementation:** how it works now
**Patterns found:** existing conventions the implementation should follow
**Dependencies:** what else might be affected
**Recommendation:** what the implementer should do, with specific guidance

When your research is complete, suggest handing off to the Implementer agent.
```

**Step 2 — Create the Implementer agent**

Create `.github/agents/implementer.agent.md`:

```markdown
---
name: Implementer
description: Implements code changes based on research findings or an approved plan. Follows existing code patterns and makes minimal, focused edits.
tools: ['editFiles', 'codebase', 'terminalLastCommand', 'runInTerminal']
model: gpt-5-mini
---

You are a disciplined developer implementing changes based on research or a plan provided to you. You do not freelance — you follow the guidance given.

## Rules

- Follow existing code patterns found in the codebase
- Make minimal, focused edits — do not refactor unrelated code
- After each file edit, verify the change compiles or passes linting
- If you encounter something unexpected, stop and explain rather than guessing
- Reference the research summary or plan when explaining your changes
```

**Step 3 — Test the handoff flow**

Start a session with the Researcher:

```
@Researcher I need to add input validation to the user registration endpoint. Research how validation is done elsewhere in this project and what libraries are used. Then hand off to the Implementer.
```

- Watch the Researcher analyze the codebase and produce a summary
- At the end of its response, the Researcher should suggest handing off to the Implementer (a clickable suggestion will appear in the chat)
- Click the handoff suggestion to continue the conversation with the Implementer
- The Implementer receives the Researcher's context and proceeds to make changes

**Step 4 — Observe the context transfer**

- Notice that the Implementer has access to everything the Researcher found — it doesn't start from scratch
- The Implementer should follow the patterns identified by the Researcher
- If the Implementer deviates from the research, redirect it:

```
The Researcher found that this project uses Joi for validation. Please use Joi, not manual checks.
```

### Success Criteria

- [ ] The Researcher agent exists with read-only tools and a `handoffs` section pointing to the Implementer
- [ ] The Implementer agent exists with editing tools
- [ ] You have completed a full Researcher → Implementer handoff in a single conversation
- [ ] The Implementer followed the patterns and recommendations identified by the Researcher

---

## Exercise 4: Control Agent Invocation — User vs Model

Not all agents should be invocable by users. Some are designed to be called only by a coordinator agent as subagents. For example, you might want a TDD workflow where a coordinator delegates to Red (write failing tests), Green (make tests pass), and Refactor (improve code quality) — but users should never call Red or Green directly. This exercise teaches you how to control who can invoke an agent.

### Prerequisites

Before starting, make sure you have:

- Researcher and Implementer agents created (Exercise 3 completed)
- Subagent support enabled (`"chat.customAgentInSubagent.enabled": true` in `settings.json`)

### Objective

1. Understand the `user-invocable` and `disable-model-invocation` properties
2. Create a coordinator agent that delegates to specialized subagents
3. Restrict subagents so they can only be invoked by the coordinator
4. Test the restrictions

### Background: Two Dimensions of Invocation

Every custom agent has two independent switches:

| Property | Default | Effect when set |
|----------|---------|----------------|
| `user-invocable` | `true` | When `false`, the agent does not appear in the `@` picker — users cannot invoke it directly |
| `disable-model-invocation` | `false` | When `true`, other agents cannot invoke this agent as a subagent |

Combining these gives you four modes:

| `user-invocable` | `disable-model-invocation` | Who can invoke? |
|---|---|---|
| `true` | `false` | Everyone (user and agents) — **default** |
| `true` | `true` | Users only — agents cannot delegate to it |
| `false` | `false` | Agents only — the agent is a pure subagent |
| `false` | `true` | Nobody — effectively disabled |

---

**Step 1 — Create the TDD Coordinator agent**

Create `.github/agents/tdd.agent.md`:

```markdown
---
name: TDD
description: Implements features using test-driven development. Delegates to Red, Green, and Refactor subagents in sequence.
tools: ['agent']
agents: ['Red', 'Green', 'Refactor']
model: claude-sonnet-4.6
---

You are a TDD coach implementing features through the Red-Green-Refactor cycle.

## Process

For each feature or change requested:

1. **Red phase:** Use the Red agent to write failing tests that define the expected behavior.
2. **Green phase:** Use the Green agent to write the minimum code to make the tests pass.
3. **Refactor phase:** Use the Refactor agent to improve code quality without changing behavior.

After each phase, verify the results before moving to the next phase. If tests fail unexpectedly, diagnose the issue before continuing.

Report the outcome of each phase to the user.
```

> **Key frontmatter:** `tools: ['agent']` gives the coordinator the ability to invoke subagents. `agents: ['Red', 'Green', 'Refactor']` restricts which subagents it can call — this prevents the coordinator from accidentally invoking unrelated agents.

**Step 2 — Create the Red, Green, and Refactor subagents**

Create `.github/agents/red.agent.md`:

```markdown
---
name: Red
description: Writes failing tests that define the expected behavior for a feature.
tools: ['editFiles', 'codebase', 'runInTerminal']
user-invocable: false
---

You write tests ONLY. Never write production code. Your tests should:
- Clearly define the expected behavior
- Use the project's existing test framework and conventions
- Fail when run (because the feature is not yet implemented)

After writing the tests, run them and confirm they fail.
```

Create `.github/agents/green.agent.md`:

```markdown
---
name: Green
description: Writes the minimum production code needed to make failing tests pass.
tools: ['editFiles', 'codebase', 'runInTerminal']
user-invocable: false
---

You write the MINIMUM code to make failing tests pass. Do not:
- Add features beyond what the tests require
- Refactor or optimize
- Change the tests

After writing the code, run the tests and confirm they pass.
```

Create `.github/agents/refactor.agent.md`:

```markdown
---
name: Refactor
description: Improves code quality without changing behavior. All tests must continue to pass.
tools: ['editFiles', 'codebase', 'runInTerminal']
user-invocable: false
---

You improve code quality ONLY. Do not:
- Add new features or change behavior
- Modify tests (unless renaming for clarity)

After refactoring, run the tests and confirm they still pass. If any test fails, revert your change.
```

> **Notice:** all three subagents have `user-invocable: false`. They will not appear in the `@` picker — only the TDD coordinator can invoke them.

**Step 3 — Test the coordinator**

Invoke the TDD coordinator:

```
@TDD Add a function called `isPalindrome` that checks if a string is a palindrome. Implement it using the TDD cycle.
```

- Watch the coordinator delegate to Red (failing tests), then Green (implementation), then Refactor (cleanup)
- Each subagent should complete its phase and report back to the coordinator

**Step 4 — Verify the restrictions**

- Type `@` in the Chat panel and look at the agent picker
- You should see **TDD** but NOT Red, Green, or Refactor — they are hidden from the user
- Try typing `@Red` directly — it should not resolve to the Red agent

### Success Criteria

- [ ] The TDD coordinator agent exists with `tools: ['agent']` and `agents: ['Red', 'Green', 'Refactor']`
- [ ] Red, Green, and Refactor agents exist with `user-invocable: false`
- [ ] The TDD coordinator successfully delegates to all three subagents in sequence
- [ ] Red, Green, and Refactor do NOT appear in the `@` agent picker

---

## Exercise 5: Configure Agent File Locations

By default, VS Code looks for `.agent.md` files in `.github/agents/` inside your workspace. But you might want agents in different locations: a shared company repository, a personal dotfiles folder, or a monorepo with agents per package. This exercise shows you how to configure additional locations.

### Prerequisites

Before starting, make sure you have:

- Multiple agents created in `.github/agents/` (Exercises 1–4 completed)
- Familiarity with VS Code settings

### Objective

1. Understand the `chat.agentFilesLocations` setting
2. Add a personal agents directory outside the workspace
3. Add a second workspace location for project-specific agents
4. Verify that agents from all locations are available

**Step 1 — Check your current agents**

Before adding new locations, verify which agents are currently available:

- Open the Chat panel and type `@`
- Note all agents that appear in the picker — these are loaded from `.github/agents/` in your workspace

**Step 2 — Create a personal agents directory**

Create a folder for agents that you want to reuse across all your projects:

- Create a directory on your machine, for example:
  - **macOS/Linux:** `~/.config/copilot-agents/`
  - **Windows:** `%USERPROFILE%\.copilot-agents\`
- Copy the `quick-helper.agent.md` file from Exercise 2 into this new directory — this makes it available in every project

**Step 3 — Configure the setting**

Open your **user** `settings.json` (so it applies to all workspaces):

- Open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
- Type `Preferences: Open User Settings (JSON)` and press Enter
- Add the `chat.agentFilesLocations` setting:

```jsonc
{
    "chat.agentFilesLocations": [
        {
            "path": "~/.config/copilot-agents"
        }
    ]
}
```

On Windows, use the full path:

```jsonc
{
    "chat.agentFilesLocations": [
        {
            "path": "C:\\Users\\<your-username>\\.copilot-agents"
        }
    ]
}
```

> **Note:** the `.github/agents/` workspace directory is always scanned automatically — you don't need to add it to this setting. The setting only adds *additional* locations.

**Step 4 — Add a workspace-level override (optional)**

If you're working in a monorepo or want project-specific agent locations, you can also configure this at the workspace level:

- Open workspace settings: `File > Preferences > Workspace Settings`
- Add a workspace-level path (for example, a `tools/agents/` directory):

```jsonc
{
    "chat.agentFilesLocations": [
        {
            "path": "./tools/agents"
        }
    ]
}
```

- Create the `tools/agents/` directory in your project and move or copy an agent file there

**Step 5 — Verify all locations are loaded**

- Reload VS Code (`Ctrl+Shift+P` → `Developer: Reload Window`)
- Open the Chat panel and type `@`
- You should now see agents from:
  - `.github/agents/` (workspace default)
  - `~/.config/copilot-agents/` (personal directory)
  - `tools/agents/` (workspace override, if configured)

> **Troubleshooting:** if an agent doesn't appear, check that the file has the correct `.agent.md` extension and that the frontmatter is valid YAML. A missing `---` fence or a syntax error in the frontmatter will silently prevent the agent from loading.

### Success Criteria

- [ ] You have created a personal agents directory outside your workspace
- [ ] Your user `settings.json` contains `chat.agentFilesLocations` pointing to the personal directory
- [ ] Agents from both the workspace `.github/agents/` and the personal directory appear in the `@` picker
- [ ] You can explain when to use workspace agents (team-shared) versus personal agents (individual preference)

---

## Summary: What You Have Built

After this workshop, you have a complete multi-agent setup:

| Agent | Type | Model | Tools | Invocable by |
|-------|------|-------|-------|-------------|
| **Security Reviewer** | Read-only reviewer | Claude Sonnet 4.6 | `codebase`, `fetch`, `findTestFiles` | Users and agents |
| **Quick Helper** | Fast assistant | GPT-5 mini | `codebase` | Users and agents |
| **Researcher** | Read-only analyzer | Claude Sonnet 4.6 | `codebase`, `fetch`, `usages` | Users and agents |
| **Implementer** | Code editor | GPT-5 mini | `editFiles`, `codebase`, `terminalLastCommand`, `runInTerminal` | Users and agents |
| **TDD** | Coordinator | Claude Sonnet 4.6 | `agent` | Users only |
| **Red** | Test writer (subagent) | Default | `editFiles`, `codebase`, `runInTerminal` | TDD coordinator only |
| **Green** | Code writer (subagent) | Default | `editFiles`, `codebase`, `runInTerminal` | TDD coordinator only |
| **Refactor** | Code improver (subagent) | Default | `editFiles`, `codebase`, `runInTerminal` | TDD coordinator only |

### The Agent Design Rule

> Give each agent the **minimum tools** it needs and the **best model** for its task. A read-only agent that cannot edit files is always safer than a general-purpose agent with full access. When in doubt, restrict first and expand later.

---

### Useful Resources

- [Custom agents overview](https://code.visualstudio.com/docs/copilot/agents/custom-agents)
- [Agent file format reference](https://code.visualstudio.com/docs/copilot/agents/custom-agents#_agent-file-format)
- [Handoffs between agents](https://code.visualstudio.com/docs/copilot/agents/custom-agents#_handoffs)
- [Subagents](https://code.visualstudio.com/docs/copilot/agents/subagents)
- [Controlling agent invocation](https://code.visualstudio.com/docs/copilot/agents/subagents#_control-subagent-invocation)
- [Agent file locations](https://code.visualstudio.com/docs/copilot/agents/custom-agents#_agent-file-locations)