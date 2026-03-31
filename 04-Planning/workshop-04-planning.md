<img src="https://mfitcontent.blob.core.windows.net/static/MF-logo.png" style="height:64px;margin-right:32px"/>


# Workshop 4 — Customize Planning


> **Estimated time: 60 minutes** (4 exercises)
>
> **Difficulty: medium** — this workshop teaches you how to use and customize the Plan agent in GitHub Copilot. You will learn to separate thinking from doing: first research and plan, then implement. By the end you will be able to configure dedicated models for planning and implementation, add extra tools to the planning phase, and run a full plan-then-implement cycle on a real feature request.

AI-Assisted Development — Planning Module

## Terms and Conditions of Use

This training package is proprietary and confidential and is intended exclusively for the uses described in the training materials. Copying or disclosing all or part of the content and/or software included in these packages is prohibited. The contents of this package are for informational and training purposes only and are provided "as is" without warranties of any kind, express or implied, including, but not limited to, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement. The content of the training package, including URLs and other references to Internet websites, is subject to change without notice. Unless otherwise noted, the companies, organizations, products, domain names, email addresses, logos, people, places, and events depicted herein are fictitious and no association with any real company, organization, product, domain name, email address, logo, person, place, or event is intended or should be inferred.

---

## The Scenario

You are the lead developer on a web application project. Your team has learned the hard way that jumping straight into code generation leads to incomplete implementations and wrong architectural decisions. A new feature request just landed: "Add a three-state theme switch (Light / Dark / System) with full light and dark theme support". The app already has some dark mode infrastructure but no user-facing toggle. Instead of letting the AI start editing files immediately, you want to research, plan, get alignment, and only then implement. The Plan agent is the tool for this job.

> **Remember:** the Plan agent is a built-in Copilot agent designed to collaborate with you on creating detailed implementation plans *before* any code changes are made. It uses read-only tools during the research phase so it cannot accidentally modify your codebase while thinking.

---

## Exercise 1: The Plan Agent and Its 4 Phases

This exercise introduces the Plan agent and walks you through its four phases — Discovery, Alignment, Design, and Refinement — on a real feature request.

### Prerequisites

Before starting, make sure you have:

- **Visual Studio Code** installed and updated to the latest stable version
- **GitHub Copilot** activated and working (Workshop 1 completed)
- Comfort with starting and scoping chat sessions, using context references, choosing a model, and reviewing proposed edits (Getting Started module completed)
- **Git** installed and configured
- Clone the **[Marvel Zombies Hero](https://github.com/phenixita/marvel-zombies-hero/)** repository to use as your practice codebase:
  ```bash
  git clone https://github.com/phenixita/marvel-zombies-hero/
  ```
- Open the cloned repository in VS Code
- **Agent mode** enabled in Chat settings (`"chat.agent.enabled": true`)

### Objective

1. Understand the four phases of the Plan agent: Discovery, Alignment, Design, Refinement
2. Start a planning session from the Chat panel
3. Observe how the Plan agent researches the codebase before proposing changes
4. Interact with the plan: ask questions, request changes, approve the final version

**Step 1 — Open a planning session**

- Open the **Chat** panel in VS Code (`Ctrl+Shift+I` / `Cmd+Shift+I`)
- In the model picker dropdown at the top of the chat, make sure an agent-capable model is selected (e.g., Claude Sonnet 4.6, GPT-5.2, or Auto)
- Type a feature request that requires planning across multiple files:

```
/plan Add a three-state theme toggle component (Light, Dark, System) to this application. The System state should read the user's OS preference using prefers-color-scheme. The preference should be persisted across reloads. The toggle should accurately display the current state and provide smooth transitions between themes.
```

- Press Enter and observe the Plan agent's behavior

> **Note:** the `/plan` mention explicitly invokes the Plan agent. If you don't see `/plan` as an option, make sure agent mode is enabled in your settings.

**Step 2 — Observe the Discovery phase**

- Watch the Plan agent's first actions: it will use **read-only tools** to explore your codebase
- You should see tool calls like: reading the project structure, opening key files (routes, components, configuration), searching for existing patterns
- The agent is building a mental model of your project — it will not edit any files during this phase
- Take note of which files and patterns the agent identifies as relevant

**Step 3 — Engage during the Alignment phase**

- After the initial research, the Plan agent may ask you **clarifying questions** to resolve ambiguities
- Answer these questions thoughtfully — they shape the implementation plan. For example:
  - *"Should the theme toggle use a dropdown, a segmented control, or icon buttons?"* → Answer based on your project's existing patterns
  - *"Is there an existing theme provider or CSS variable system I should build on?"* → Point the agent to the right file
- If the agent doesn't ask questions, prompt it yourself:

```
Before you design the plan, what assumptions are you making about the tech stack and existing patterns in this project?
```

**Step 4 — Review the Design and Refinement phases**

- The Plan agent will produce a **structured implementation plan** with:
  - Files to create or modify
  - The order of changes
  - Key decisions and trade-offs
- Review the plan carefully. Push back on anything you disagree with:

```
I'd prefer the toggle to be a segmented control with icons (sun/moon/monitor) rather than a dropdown menu. Can you update the plan?
```

- The agent will refine the plan based on your feedback
- Continue iterating until you are satisfied with the plan

**Step 5 — Approve and transition to implementation**

- When the plan is final, the Plan agent will present a todo list with the implementation steps
- You can now choose to proceed to implementation (the agent will switch to edit mode) or save the plan for later
- For this exercise, **do not implement yet** — we will customize the planning configuration first in the next exercises

### Success Criteria

- [ ] You have started a planning session using `/plan` in the Chat panel
- [ ] You observed the Plan agent using read-only tools to research your codebase (Discovery phase)
- [ ] You answered at least one clarifying question or prompted the agent to share its assumptions (Alignment phase)
- [ ] The Plan agent produced a structured implementation plan (Design phase)
- [ ] You requested at least one change to the plan and the agent refined it (Refinement phase)

---

## Exercise 2: Configure Separate Models for Planning and Implementation

Different phases benefit from different models. Planning requires deep reasoning — understanding architecture, weighing trade-offs, anticipating edge cases. Implementation requires fast, accurate code generation. This exercise teaches you how to assign the right model to each phase.

### Prerequisites

Before starting, make sure you have:

- Plan agent working (Exercise 1 completed)
- Access to multiple models in your Copilot subscription (check the model picker dropdown)

### Objective

1. Understand why separating planning and implementation models improves results
2. Configure `chat.planAgent.defaultModel` for the planning phase
3. Configure `github.copilot.chat.implementAgent.model` for the implementation phase
4. Test the configuration with a planning session

### Background: Why Two Models?

The Plan agent operates in two distinct phases that have different requirements:

| Phase | What it does | Best model characteristics |
|-------|-------------|---------------------------|
| **Planning** | Reads code, reasons about architecture, identifies edge cases, produces a structured plan | Strong reasoning, large context window, slower is acceptable |
| **Implementation** | Writes code, edits files, runs terminal commands, iterates on test failures | Fast code generation, accurate edits, tool use proficiency |

Using a powerful reasoning model (like Claude Opus or GPT-5.2) for planning and a fast general-purpose model (like Claude Sonnet or GPT-5 mini) for implementation gives you the best of both worlds: thorough planning without paying the speed cost during code generation.

---

**Step 1 — Open your user `settings.json`**

- Open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
- Type `Preferences: Open User Settings (JSON)` and press Enter

**Step 2 — Add the planning model setting**

Add the following setting to assign a reasoning-capable model to the Plan agent:

```jsonc
{
    "chat.planAgent.defaultModel": "GPT-5.4 (copilot)"
}
```
> **Model choice:** choose a model that excels at reasoning and has a large context window. Check the [model comparison](https://docs.github.com/en/copilot/reference/ai-models/model-comparison) to see which models are best for planning tasks.

**Step 3 — Add the implementation model setting**

Add a second setting to assign a fast model to the implementation phase:

```jsonc
{
    "chat.planAgent.defaultModel": "GPT-5.4 (copilot)",
    "github.copilot.chat.implementAgent.model": "GPT-5.3 Codex (copilot)"
}
```

> **Why a different model?** The implementation agent executes a plan that's already been approved. It doesn't need deep reasoning — it needs speed, accurate edits, and reliable tool use. A fast model reduces wait times during the edit-test-fix loop.

**Step 4 — Test the configuration**

- Open the Chat panel and start a new planning session:

```
/plan Refactor the existing color system to use CSS custom properties for all theme tokens, ensuring light and dark variants are defined in a single source of truth. All components should reference these tokens instead of hardcoded colors.
```

- Observe the model indicator in the Chat panel: during the planning phase, it should show the model you configured in `chat.planAgent.defaultModel`
- When the plan is approved and implementation begins, the model should switch to the one configured in `github.copilot.chat.implementAgent.model`

**Step 5 — Experiment with different model combinations**

Try at least two different combinations and compare the results:

| Configuration | Planning model | Implementation model | Observe |
|--------------|----------------|---------------------|---------|
| **A — Balanced** | `claude-sonnet-4.6` | `gpt-5-mini` | Good plan quality, fast implementation |
| **B — Maximum reasoning** | `claude-opus-4.6` | `claude-sonnet-4.6` | Deeper analysis, higher cost |

> **Tip:** the model picker dropdown in the Chat panel shows which model is active. You can also check the model name in the tool call details (expand the collapsed entries in the chat).

### Success Criteria

- [ ] Your `settings.json` contains `chat.planAgent.defaultModel` with a model ID
- [ ] Your `settings.json` contains `github.copilot.chat.implementAgent.model` with a different model ID
- [ ] You can explain why using a reasoning model for planning and a fast model for implementation is beneficial
- [ ] You have tested at least two different model combinations and observed the difference in output quality and speed

---

## Exercise 3: Add Extra Tools to the Plan Agent

By default, the Plan agent uses read-only tools: it reads files, searches the codebase, and analyzes structure. But sometimes planning requires more — checking a database schema, reading an API specification from a URL, or querying an internal documentation system. This exercise shows you how to extend the Plan agent's capabilities with additional tools.

### Prerequisites

Before starting, make sure you have:

- Plan agent configured with a dedicated model (Exercise 2 completed)
- Familiarity with VS Code settings

### Objective

1. Understand what `github.copilot.chat.planAgent.additionalTools` does
2. Add the `fetch` tool so the Plan agent can read external URLs during research
3. Test the extended Plan agent with a prompt that requires external information
4. Understand the security implications of giving extra tools to the planning phase

### Background: Extending the Plan Agent's Reach

The `github.copilot.chat.planAgent.additionalTools` setting accepts an array of tool identifiers. These tools become available to the Plan agent during the Discovery phase, expanding what it can research before proposing a plan.

Common tools you might want to add:

| Tool | What it provides | Use case |
|------|-----------------|----------|
| `fetch` | Read content from URLs | External API docs, RFCs, blog posts with architecture patterns |
| `usages` | Find all usages of a symbol | Impact analysis before refactoring |
| `githubRepo` | Read files from a GitHub repository | Reference implementations, upstream library code |
| MCP server tools | Custom data sources | Internal wikis, database schemas, Jira tickets |

> **Important:** this setting is marked as **experimental**. The tool identifiers may change in future VS Code releases.

---

**Step 1 — Open your user `settings.json`**

- Open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
- Type `Preferences: Open User Settings (JSON)` and press Enter

**Step 2 — Add the additional tools setting**

Add the `fetch` tool to the Plan agent so it can read external URLs during the research phase:

```jsonc
{
    "chat.planAgent.defaultModel": "claude-sonnet-4.6",
    "github.copilot.chat.implementAgent.model": "gpt-5-mini",
    "github.copilot.chat.planAgent.additionalTools": [
        "fetch"
    ]
}
```

> **Why `fetch`?** Many planning tasks benefit from reading external documentation. For example, if you're implementing an OAuth flow, the Plan agent can read the OAuth 2.0 specification directly instead of relying on its training data (which may be outdated).

**Step 3 — Test with a prompt that requires external information**

Start a new planning session that explicitly references an external URL:

```
/plan I need to implement system theme detection using the prefers-color-scheme media query. Read this reference first: https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme — then design a plan to implement a ThemeProvider component that reacts to OS-level theme changes in real-time.
```

- Observe: the Plan agent should use the `fetch` tool to retrieve the Mozilla Developer Network page content
- The resulting plan should incorporate specific details from the fetched page (checking media queries, adding event listeners, etc.)
- Compare this to what would happen without the `fetch` tool — the agent would rely solely on its training data

**Step 4 — Add more tools (optional)**

If your project uses specific tools, you can add them:

```jsonc
{
    "github.copilot.chat.planAgent.additionalTools": [
        "fetch",
        "usages",
        "githubRepo"
    ]
}
```

- Test with a refactoring prompt to see `usages` in action:

```
/plan I want to replace all hardcoded color values (hex, rgb, hsl) across the codebase with CSS custom property references. Research all usages first and create a safe migration plan.
```

**Step 5 — Understand the security trade-off**

Adding tools to the Plan agent expands its capabilities but also its attack surface:

- The `fetch` tool can retrieve content from any URL — including pages with malicious prompt injection
- External content should always be treated as untrusted input
- Review the Plan agent's proposals carefully when external data influenced the plan

> **Best practice:** only add tools that the Plan agent genuinely needs. Keep the planning phase as read-only as possible to minimize risk.

### Success Criteria

- [ ] Your `settings.json` contains `github.copilot.chat.planAgent.additionalTools` with at least the `fetch` tool
- [ ] You have tested a planning session where the Plan agent used the `fetch` tool to read external content
- [ ] The resulting plan incorporated specific information from the fetched URL
- [ ] You can explain the security implications of giving the Plan agent access to tools like `fetch`

---

## Exercise 4: Full Plan-Then-Implement Cycle

Time to put it all together. In this exercise you will run a complete plan-then-implement cycle: research, plan, review, approve, implement, and validate. This is the workflow you should adopt for any non-trivial feature or refactoring task.

### Prerequisites

Before starting, make sure you have:

- Planning and implementation models configured (Exercise 2 completed)
- Additional tools configured (Exercise 3 completed)
- A Git repository with an existing codebase open in VS Code
- At least one pending feature request or refactoring task for your project

### Objective

1. Execute a full planning cycle from feature request to approved plan
2. Transition from plan to implementation and observe the model switch
3. Use the todo list to track implementation progress
4. Validate the implementation against the original plan

---

**Step 1 — Choose a real task**

Pick a task that is meaningful for your project. It should touch at least 3 files and require some architectural thinking. Examples:

- Add a three-state theme toggle component (Light / Dark / System) with persistence
- Implement smooth CSS transitions when switching themes
- Add theme-aware illustrations or hero images that change with the theme
- Create an accessible theme indicator in the navigation bar

Write the feature request clearly:

```
/plan [Your feature request here. Be specific about requirements, constraints, and expected behavior.]
```

**Step 2 — Drive the planning phase**

- Let the Plan agent research your codebase (Discovery)
- Answer any clarifying questions (Alignment)
- Review the structured plan (Design)
- Request at least one refinement (Refinement)
- When satisfied, approve the plan

> **Tip:** if the plan is complex, ask the Plan agent to break it down into smaller, independently testable chunks:

```
Can you break this plan into smaller steps where each step produces a working, testable state? I want to be able to verify each step before moving to the next.
```

**Step 3 — Transition to implementation**

- After approving the plan, tell the agent to proceed with implementation
- Observe:
  - The model switches from your planning model to your implementation model
  - The agent follows the plan's todo list, checking off items as it completes them
  - The agent uses editing tools (not just read-only) to modify files

**Step 4 — Monitor progress with the todo list**

- As the agent works through the implementation, the todo list in the chat updates
- If the agent deviates from the plan, redirect it:

```
You're drifting from the plan. Step 2 says to implement the ThemeProvider before building the toggle UI. Please follow the original order.
```

- If you discover an issue during implementation, you can pause and adjust:

```
Wait — I just realized we also need to handle the case where the user's OS doesn't support prefers-color-scheme. Can you update the plan to include a fallback?
```

**Step 5 — Validate the implementation**

After the agent completes all steps:

- Review the changes in the **Source Control** panel
- Run the project's test suite (if available) to check for regressions
- Manually verify that the implementation matches the approved plan
- If there are discrepancies, ask the agent to fix them:

```
The plan specified that the theme preference should persist in localStorage, but the implementation is using sessionStorage. Please fix this to match the plan.
```

### Success Criteria

- [ ] You have completed a full plan-then-implement cycle on a real task
- [ ] The Plan agent researched, proposed, and refined a plan before any code was edited
- [ ] The implementation model was different from the planning model (verified in the Chat panel)
- [ ] You used the todo list to track and verify implementation progress
- [ ] The final implementation matches the approved plan (or deviations were intentionally approved by you)

---

## Summary: What You Have Configured

After this workshop, your planning workflow is fully customized:

| Component | Status | Notes |
|-----------|--------|-------|
| **Plan Agent** | ✅ Working | Research → Align → Design → Refine cycle mastered |
| **Planning Model** | ✅ Configured | `chat.planAgent.defaultModel` set to a reasoning model |
| **Implementation Model** | ✅ Configured | `github.copilot.chat.implementAgent.model` set to a fast model |
| **Additional Tools** | ✅ Extended | `fetch` (and optionally `usages`, `githubRepo`) available during planning |
| **Full Cycle** | ✅ Tested | End-to-end plan-then-implement workflow completed |

### The Planning Rule

> For any change that touches more than 2 files or requires architectural decisions, **always plan first**. The time invested in planning is always less than the time lost fixing a wrong implementation.

---

### Useful Resources

- [Plan agent overview](https://code.visualstudio.com/docs/copilot/agents/plan-agent)
- [Customize the agent loop](https://code.visualstudio.com/docs/copilot/agents/customize-agent-loop)
- [VS Code AI core concepts](https://code.visualstudio.com/docs/copilot/core-concepts)
- [Model comparison](https://docs.github.com/en/copilot/reference/ai-models/model-comparison)
- [Model multipliers and billing](https://docs.github.com/en/copilot/concepts/billing/copilot-requests#model-multipliers)
