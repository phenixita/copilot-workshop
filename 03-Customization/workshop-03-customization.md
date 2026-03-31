<img src="https://mfitcontent.blob.core.windows.net/static/MF-logo.png" style="height:64px;margin-right:32px"/>


# Workshop 3 — Customizing GitHub Copilot


> **Estimated time: 60 minutes** (4 exercises)
>
> **Difficulty: low-medium** — this workshop teaches you how to control GitHub Copilot's behavior across a real codebase with multiple bounded contexts. You will use global instructions to set project-wide rules, scoped instructions to enforce module-specific conventions, and reusable prompt templates to automate recurring tasks. By the end you will have a fully customized Copilot setup that your whole team can share via Git.

AI-Assisted Development — Customization Module

## Terms and Conditions of Use

This training package is proprietary and confidential and is intended exclusively for the uses described in the training materials. Copying or disclosing all or part of the content and/or software included in these packages is prohibited. The contents of this package are for informational and training purposes only and are provided "as is" without warranties of any kind, express or implied, including, but not limited to, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement. The content of the training package, including URLs and other references to Internet websites, is subject to change without notice. Unless otherwise noted, the companies, organizations, products, domain names, email addresses, logos, people, places, and events depicted herein are fictitious and no association with any real company, organization, product, domain name, email address, logo, person, place, or event is intended or should be inferred.

---

## The Scenario

You are joining a team that maintains a modular monolith: a C# Vending Machine application split into four bounded contexts — **Cash**, **Inventory**, **Orders**, and **Reporting**. Each module lives in its own project with its own conventions:

- **Cash** enforces strict balance validation and parameterized database queries
- **Inventory** uses a feature-folder structure with MediatR commands and queries
- **Orders** must coordinate across modules without reading other modules' database tables
- **Reporting** consumes domain events and aggregates data for dashboards

Without customization, Copilot treats all these files the same. It might suggest reading Inventory tables directly from Orders code, skip the project's TDD requirement, or use the wrong test infrastructure for L0 vs L1 tests. The result is inconsistent code that breaks the team's conventions.

The project already has a partial customization setup. Your job is to understand what's there, extend it to cover the missing modules, and add reusable prompt templates that make common tasks repeatable.

> **Note:** all exercises use the [modular-monolith](https://github.com/phenixita/modular-monolith) repository as the target project. Clone it before starting Exercise 1.

---

## Exercise 1: Global Instructions and `/init`

This exercise walks you through the project-wide instruction layer — the rules that apply to every Copilot interaction regardless of which file is open. You will use `/init` to understand how Copilot bootstraps a project's instructions, then inspect and test the existing global rules in the modular-monolith.

### Prerequisites

Before starting, make sure you have:

- **Visual Studio Code** installed and updated to the latest stable version
- **GitHub Copilot** activated and working (Workshop 1 completed)
- **Agent mode** enabled (`"chat.agent.enabled": true`)
- The **modular-monolith** repository cloned: `git clone https://github.com/phenixita/modular-monolith.git`
- The cloned repository open as the current workspace in VS Code

### Objective

1. Understand what `/init` does and when to use it
2. Read and explain the project's global instruction file
3. Verify that Copilot respects project-wide rules on different modules
4. Test boundary enforcement between bounded contexts

**Step 1 — Run `/init` and inspect the output**

Open the Chat panel (`Ctrl+Shift+I` / `Cmd+Shift+I`) and switch to **Agent** mode. Type:

```
/init
```

Copilot scans the repository and generates a scaffolded `copilot-instructions.md`. It reads the project structure, tech stack, and existing patterns to propose content.

> **Note:** the modular-monolith already has a hand-crafted instruction file at `.github/instructions/copilot-instructions.md`. When `/init` produces its output, compare the two side by side. Notice what `/init` gets right from static analysis versus what the team had to add manually (TDD flow, module boundaries, test level rules).

**Step 2 — Inspect the existing global instructions**

Open `.github/instructions/copilot-instructions.md`. Read each section:

| Section                  | What it enforces                                                                                                                        |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Architecture**         | All source code lives under `src/`; each module has a dedicated project, path, and database schema                                      |
| **Definition of module** | No reading or editing data from other modules' tables; cross-module communication via services, events, or shared libraries             |
| **Testing guidelines**   | L0 = in-memory unit tests, no external dependencies, execution under 100ms; L1 = integration tests with real infra (database), under 1s |
| **Developer flow**       | TDD is non-negotiable: failing test first, then implement, then repeat                                                                  |
| **Test method naming**   | `Given_ValidInput_When_CallingMethod_Then_ReturnsExpectedResult` pattern                                                                |

These rules apply to every Copilot interaction — regardless of which file is open, the model is always aware of them.

**Step 3 — Test global rules on the Cash module**

Open `src/VendingMachine.Cash/CashRegisterService.cs`. Open the Chat panel and ask:

```
Add a method to deduct a specific amount from the balance. The method should be called DeductAmount and take a decimal parameter.
```

Observe the response:
- Does Copilot suggest writing a test first? (TDD rule)
- Does the generated code use `LoggingHelper.ExecuteWithLoggingAsync` to wrap the operation? (logging pattern from the existing service)
- Is the method placed correctly within the service and registered via MediatR?

**Step 4 — Test global rules on the Reporting module**

Open `src/VendingMachine.Reporting/ReportingService.cs`. Ask:

```
Add a method to get the top 3 selling products based on order confirmed events.
```

Observe that the same global conventions apply — TDD suggestion, logging pattern, Given_When_Then test names — even though this is a completely different bounded context file.

**Step 5 — Test boundary enforcement**

Open `src/VendingMachine.Orders/PlaceOrder/PlaceOrderHandler.cs`. Ask:

```
I need to check available stock. Read the inventory directly from the database tables.
```

The global instructions explicitly forbid reading other modules' database tables. Observe whether Copilot:
- Declines the direct table approach
- Suggests using `IInventoryService` instead (the correct cross-module communication pattern)
- References the module isolation rule

This is the key power of global instructions: the rule is encoded once, and Copilot applies it consistently across the entire project.

### Success Criteria

- [ ] `/init` has been run and its output compared to the existing `.github/instructions/copilot-instructions.md`
- [ ] Each section of the global instruction file can be explained (architecture, module definition, test levels, TDD flow, naming)
- [ ] Copilot respects the logging pattern and TDD suggestion when working in the Cash module
- [ ] Copilot applies the same conventions in the Reporting module without any extra prompting
- [ ] Copilot refuses direct database access across modules and suggests the correct service-based pattern

---

## Exercise 2: Scoped Instructions per Bounded Context

One instruction file is not enough for a codebase where each module has different infrastructure, test patterns, and naming conventions. This exercise introduces `.instructions.md` files — scoped instructions that activate only when matching files are open in context. You will inspect two existing scoped instructions, create a new one for the Orders module, and add a cross-cutting instruction for test files.

### Prerequisites

Before starting, make sure you have:

- Exercise 1 completed and the modular-monolith workspace open
- Familiarity with the four bounded contexts: Cash, Inventory, Orders, Reporting

### Objective

1. Understand the YAML frontmatter format of `.instructions.md` files
2. Distinguish between `applyTo` (explicit scoping) and description-only (on-demand discovery)
3. Inspect the existing scoped instructions for Cash and Inventory
4. Create a new scoped instruction for the Orders module
5. Create a cross-cutting instruction for L0 test projects
6. Create an on-demand instruction and attach it manually

**Step 1 — Understand the frontmatter format**

A `.instructions.md` file uses YAML frontmatter to control when it is loaded:

```yaml
---
description: "Use when..."    # Required for on-demand discovery — keyword-rich phrase
applyTo: "src/MyModule/**"    # Optional — auto-attach when matching files are in context
---

# Your instructions here
```

The two activation modes are:

| Mode                             | How it works                                        | When to use                                           |
| -------------------------------- | --------------------------------------------------- | ----------------------------------------------------- |
| **Explicit** (`applyTo`)         | Auto-loaded when matching files are open            | Module-specific rules you always want active          |
| **On-demand** (description only) | Copilot loads it when the task semantically matches | Specialized workflows not needed on every interaction |
| **Manual**                       | User selects via Add Context → Instructions         | Ad-hoc, for one-off tasks                             |

**Step 2 — Inspect the Cash scoped instruction**

Open `.github/instructions/vendingmachine-cash.instructions.md`. Notice:

```yaml
---
description: Use when working on cash domain implementations (ICashrepository),
             Postgres/InMemory cash repository, or cash persistence for balance.
applyTo: 'src/VendingMachine.Cash.Infrastructure/**'
---
```

The `applyTo` targets only the infrastructure project — not `VendingMachine.Cash` (the domain) or `VendingMachine.Cash.Abstractions`. This is intentional: the rules in this file are about repository implementations (parameterized queries, `EnsureCreated`, sealed classes) and are irrelevant when working on pure domain logic.

**Step 3 — Inspect the Inventory scoped instruction**

Open `.github/instructions/vendingmachine-inventory.instructions.md`. Notice:

```yaml
---
description: Rules and coding standards for VendingMachine.Inventory module
applyTo: 'src/VendingMachine.Inventory/**'
---
```

This one covers the entire Inventory module — not just infrastructure. Compare the scope choice: Cash instructions apply to `Cash.Infrastructure/**` because the domain logic is simple; Inventory instructions apply to `Inventory/**` because the whole module uses a consistent feature-folder structure (`CreateProduct/`, `UpdateProduct/`, `Stock/Add/`, etc.) that Copilot should know about everywhere.

**Step 4 — Create a scoped instruction for the Orders module**

The Orders module has no `.instructions.md` yet. Create `.github/instructions/vendingmachine-orders.instructions.md` with the following content:

```markdown
---
description: Rules and coding standards for VendingMachine.Orders module — order placement, cross-module validation, and event emission
applyTo: 'src/VendingMachine.Orders/**'
---

# VendingMachine.Orders Standards

## Cross-module coordination
- NEVER read from Inventory or Cash database tables directly.
- Validate product availability by calling `IInventoryService.GetProduct` before placing an order.
- Validate sufficient balance by calling `ICashRegisterService.GetBalance` before placing an order.
- If validation fails, throw a domain exception with a descriptive message — do not return null.

## CQRS pattern
- All service operations use MediatR commands and queries.
- Commands go in dedicated subfolders: `PlaceOrder/PlaceOrderCommand.cs` and `PlaceOrder/PlaceOrderHandler.cs`.
- `OrderService` is a thin facade — it delegates to MediatR, it does not contain business logic.

## Event emission
- After a successful order, emit `OrderConfirmed` (defined in `VendingMachine.Orders.Abstractions`).
- The Reporting module consumes `OrderConfirmed` via `OrderConfirmedHandler` — do not call Reporting services directly.

## Logging
- Wrap all service methods with `LoggingHelper.ExecuteWithLoggingAsync`.
- Use operation names in the format `OrderService.<MethodName>`.
```

**Step 5 — Create a cross-cutting instruction for L0 tests**

L0 tests exist in every module (`VendingMachine.Cash.Tests.L0`, `VendingMachine.Inventory.Tests.L0`, etc.) and always follow the same pattern. Create `.github/instructions/testing-l0.instructions.md`:

```markdown
---
description: Standards for L0 in-memory unit tests across all modules
applyTo: 'src/**/*.Tests.L0/**'
---

# L0 Test Standards

## Scope
- L0 tests are pure unit tests — no external dependencies, no database, no network.
- Always use the InMemory repository implementations (e.g., `InMemoryCashRepository`, `InMemoryInventoryRepository`).
- Target execution time: under 100ms per test.

## Markers
- Decorate every test class with `[Trait("Level", "L0")]`.
- Do not mix L0 and L1 tests in the same project.

## Structure
- Follow Arrange / Act / Assert within each test method, with blank-line separation between sections.
- Method names follow the Given_When_Then pattern:
  `Given_SufficientBalance_When_PlacingOrder_Then_OrderIsConfirmed`

## What not to do
- Do NOT inject infrastructure fixtures (`InfrastructureFixture`) in L0 tests.
- Do NOT use `[Collection]` with infrastructure fixtures in L0 tests.
- Do NOT hit the file system, environment variables, or any external process.
```

**Step 6 — Create an on-demand instruction for database migrations**

Create `.github/instructions/postgres-migration.instructions.md` with no `applyTo` field — only a description:

```markdown
---
description: Use when creating or modifying PostgreSQL schemas, EnsureCreated patterns, migration scripts, or database table setup for any VendingMachine module.
---

# PostgreSQL Migration Standards

## EnsureCreated pattern
- Each repository must call `EnsureCreated` inside `GetBalance`/`GetStock`/equivalent read methods.
- `EnsureCreated` must create the schema, table, and seed a default row if they do not exist.
- Keep `EnsureCreated` idempotent — safe to call on every read.

## Parameterized queries
- NEVER use string interpolation in SQL — always use parameterized queries with `@param` syntax.
- Parameterize all values that come from method arguments, including connection strings assembled at runtime.

## Schema isolation
- Each module owns its own schema (e.g., `cash`, `inventory`, `orders`, `reporting`).
- Do NOT create tables in the `public` schema for module-owned data.
- Cross-schema queries are forbidden — read through services, not SQL joins.
```

This instruction will not attach automatically. Copilot will load it on-demand when you ask about database schema changes, or you can attach it manually.

**Step 7 — Verify scoping**

Test that scoped instructions activate correctly:

1. Open `src/VendingMachine.Orders/PlaceOrder/PlaceOrderHandler.cs`. Ask: *"Add balance validation before placing the order."* Verify Copilot references `ICashRegisterService` and the domain exception pattern from your Orders instruction.

2. Open `src/VendingMachine.Cash/CashRegisterService.cs`. Ask the same question. Verify the **Cash** instruction applies instead (balance validation, logging pattern) — the Orders instruction should not appear.

3. Open `src/VendingMachine.Cash.Tests.L0/CashRegisterTests.cs`. Ask: *"Add a test for the DeductAmount method."* Verify Copilot applies the L0 instruction: `[Trait("Level", "L0")]`, InMemory repository, Given_When_Then naming, under 100ms.

4. Open `src/VendingMachine.Reporting/ReportingService.cs`. Click **Add Context** → **Instructions** in the Chat panel and select `postgres-migration`. Ask: *"Add a new table to store daily sales totals."* Observe the on-demand instruction being used.

### Success Criteria

- [ ] The YAML frontmatter format is understood — `applyTo` vs description-only vs manual
- [ ] The difference in scope between `Cash.Infrastructure/**` and `Inventory/**` can be explained
- [ ] `vendingmachine-orders.instructions.md` exists with cross-module coordination rules and CQRS pattern
- [ ] `testing-l0.instructions.md` exists with `applyTo: 'src/**/*.Tests.L0/**'` and correct standards
- [ ] `postgres-migration.instructions.md` exists with description only (no `applyTo`) and migration rules
- [ ] Copilot applies Orders rules when working in the Orders module and Cash rules in the Cash module
- [ ] The on-demand instruction can be manually attached to any conversation

---

## Exercise 3: Reusable Prompt Templates

Custom instructions tell Copilot *how to behave*. Prompt templates tell it *what task to perform* in a repeatable, parameterized way. This exercise introduces `.prompt.md` files — slash-command templates that package task instructions, tool requirements, and argument hints into a single file you can invoke with `/`.

### Prerequisites

Before starting, make sure you have:

- The modular-monolith workspace open in VS Code
- Agent mode enabled

### Objective

1. Understand the `.prompt.md` frontmatter format
2. Create a prompt that scaffolds a new CQRS feature in any module
3. Create a prompt that generates L0 tests for a given service or handler
4. Create a personal user-level prompt that travels across all workspaces

**Step 1 — Understand the prompt frontmatter format**

A `.prompt.md` file uses the same YAML frontmatter approach as instructions, but with task-specific fields:

```yaml
---
description: "What this prompt does — shown in the / picker"
agent: "agent"               # ask | agent | plan — determines mode
argument-hint: "..."         # Text shown in the chat input as a hint
tools: [search, read, edit]  # Tool access this prompt needs
---

Your task instructions here.
```

Prompts live in `.github/prompts/` (shared with the team) or in your VS Code user profile (personal). You invoke them by typing `/` in the Chat panel and selecting the prompt name.

**Step 2 — Create the `.github/prompts/` directory**

Create the folder `.github/prompts/` in the modular-monolith project root.

**Step 3 — Create a "New Feature" prompt**

Create `.github/prompts/new-feature.prompt.md`:

```markdown
---
description: "Scaffold a new CQRS feature (Command/Query + Handler) in a bounded context module"
agent: "agent"
argument-hint: "Describe the feature and which module it belongs to (Cash, Inventory, Orders, Reporting)"
tools: [search, read, edit]
---

Scaffold a new CQRS feature in the modular-monolith project.

## Steps

1. Identify the target module from the user's description.
2. Determine whether this is a Command (mutates state) or a Query (reads state).
3. Create a new feature subfolder inside the module following this structure:
   - `src/VendingMachine.<Module>/<FeatureName>/<FeatureName>Command.cs` (or `Query.cs`)
   - `src/VendingMachine.<Module>/<FeatureName>/<FeatureName>Handler.cs`
4. Follow the exact pattern used in `src/VendingMachine.Orders/PlaceOrder/`:
   - The Command/Query is a C# `record` with the input parameters.
   - The Handler is a class implementing `IRequestHandler<TCommand, TResult>`.
   - The Handler injects only the interfaces it needs from `VendingMachine.<Module>.Abstractions`.
5. Register the Handler in `src/VendingMachine.<Module>/ServiceCollectionExtensions.cs` using `AddMediatR`.
6. Do NOT add the endpoint — only the domain-layer feature. Endpoints are in `VendingMachine.Api`.

## Quality checks
- Confirm the new feature folder follows the naming convention of existing features in the module.
- Remind the user to write an L0 test before implementing (TDD rule from global instructions).
```

**Step 4 — Invoke the new-feature prompt**

Open the Chat panel. Type `/` and look for `new-feature` in the picker. Select it and enter:

```
Add a GetOrderHistory query to the Orders module that returns the last N orders for a given session
```

Observe:
- Copilot creates `src/VendingMachine.Orders/GetOrderHistory/` with a `GetOrderHistoryQuery.cs` record and `GetOrderHistoryHandler.cs`
- The Handler injects `IOrderService` or the appropriate repository interface from Abstractions
- `ServiceCollectionExtensions.cs` in the Orders module is updated with `AddMediatR`
- Copilot reminds you to write an L0 test first

**Step 5 — Create an "Add L0 Tests" prompt**

Create `.github/prompts/add-l0-tests.prompt.md`:

```markdown
---
description: "Generate L0 unit tests for a service or handler using in-memory repositories"
agent: "agent"
argument-hint: "Name of the service or handler class to test (e.g., CashRegisterService, PlaceOrderHandler)"
tools: [search, read, edit]
---

Generate L0 in-memory unit tests for the specified class.

## Steps

1. Locate the target class in the `src/` tree.
2. Identify the corresponding `Tests.L0` project (e.g., `VendingMachine.Cash.Tests.L0` for a Cash class).
3. Create a new test file (or add to existing) following the structure of `src/VendingMachine.Inventory.Tests.L0/ProductManagementTests.cs`:
   - `[Trait("Level", "L0")]` on the test class
   - InMemory repository implementations only — never infrastructure fixtures
   - Arrange / Act / Assert sections with blank-line separation
   - Given_When_Then method names: `Given_<state>_When_<action>_Then_<outcome>`
4. Cover at minimum:
   - The happy path
   - At least one validation failure (invalid input or business rule violation)
   - Any edge case visible from the method signature or documentation

## Quality checks
- Confirm no external dependencies are injected.
- Confirm execution would complete well under 100ms.
- Confirm `[Trait("Level", "L0")]` is present.
```

**Step 6 — Invoke the add-l0-tests prompt**

Open Chat, type `/`, select `add-l0-tests`, and enter:

```
ReportingService
```

Observe that Copilot:
- Locates `src/VendingMachine.Reporting/ReportingService.cs`
- Creates tests in `src/VendingMachine.Reporting.Tests.L0/`
- Uses `InMemoryReportingRepository` (not a Postgres fixture)
- Follows Given_When_Then naming and applies `[Trait("Level", "L0")]`

**Step 7 — Create a personal user-level prompt**

Workspace prompts in `.github/prompts/` are shared with the team via Git. You can also create personal prompts that travel with your VS Code profile across all workspaces.

Create the file `explain-code.prompt.md` in your VS Code user prompts folder:

- **Windows:** `%APPDATA%\Code\User\prompts\explain-code.prompt.md`
- **macOS:** `~/Library/Application Support/Code/User/prompts/explain-code.prompt.md`
- **Linux:** `~/.config/Code/User/prompts/explain-code.prompt.md`

```markdown
---
description: "Explain selected code in plain language, suitable for onboarding or documentation"
agent: "ask"
argument-hint: "Optional: specify the audience (junior dev, architect, business stakeholder)"
---

Explain the selected code in plain language.

- Start with one sentence summarizing what the code does.
- Describe the main steps or responsibilities in order.
- Highlight any non-obvious design decisions or patterns.
- If the audience is specified, adjust the vocabulary and depth accordingly.
- Do not restate the code itself — explain its purpose and behavior.
```

Open Chat, type `/`, and verify `explain-code` appears alongside the workspace prompts. This prompt is now available in every project you open on this machine.

### Success Criteria

- [ ] `.github/prompts/new-feature.prompt.md` exists with correct frontmatter and instructions
- [ ] `new-feature` prompt appears in the `/` picker and scaffolds a CQRS feature in the correct module folder
- [ ] `.github/prompts/add-l0-tests.prompt.md` exists and generates tests in the correct `Tests.L0` project
- [ ] `add-l0-tests` generates tests with `[Trait("Level", "L0")]`, InMemory repos, and Given_When_Then names
- [ ] A personal `explain-code.prompt.md` exists in the user prompts folder and appears in the `/` picker

---

## Exercise 4: All Three Layers Working Together

Custom instructions, scoped instructions, and prompt templates are more powerful together than individually. This exercise runs an end-to-end scenario that exercises all three layers at once, using the Orders bounded context as the stage.

### Prerequisites

Before starting, make sure you have:

- Exercises 1, 2, and 3 completed
- The following files in place:
  - `.github/instructions/copilot-instructions.md` (global, existing)
  - `.github/instructions/vendingmachine-orders.instructions.md` (scoped, created in Exercise 2)
  - `.github/instructions/testing-l0.instructions.md` (scoped, created in Exercise 2)
  - `.github/prompts/new-feature.prompt.md` (created in Exercise 3)
  - `.github/prompts/add-l0-tests.prompt.md` (created in Exercise 3)

### Objective

1. Experience all three customization layers composing in a single workflow
2. Understand which layer is responsible for which constraint in the generated code
3. Observe commit message conventions as a fourth customization layer
4. Understand the team sharing model for each file type

**Step 1 — Review the customization hierarchy**

Before starting the scenario, map the layers:

| Layer             | File                                    | Activates                                      | Responsibility in this project                                               |
| ----------------- | --------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------------------- |
| **Global**        | `copilot-instructions.md`               | Always, every interaction                      | TDD requirement, module isolation, L0/L1 test levels, Given_When_Then naming |
| **Module-scoped** | `vendingmachine-orders.instructions.md` | When Orders files are in context               | Cross-module validation via services, CQRS pattern, event emission, logging  |
| **Cross-cutting** | `testing-l0.instructions.md`            | When any `Tests.L0` project file is in context | `[Trait]`, InMemory repos, Arrange/Act/Assert, execution time                |
| **Task**          | `new-feature.prompt.md`                 | When user types `/new-feature`                 | Scaffolds CQRS feature folder, reminds about TDD                             |
| **Task**          | `add-l0-tests.prompt.md`                | When user types `/add-l0-tests`                | Generates test file with correct structure and markers                       |

**Step 2 — Scaffold a new feature using the prompt**

Open Chat, type `/new-feature`, and enter:

```
Add a CancelOrder command to the Orders module that refunds the balance and restores stock
```

Watch which layers activate and what constraint each enforces:

- **Global instructions:** Copilot reminds you to write a failing test first (TDD rule)
- **Orders instructions:** the generated `CancelOrderHandler` calls `ICashRegisterService` to refund and `IInventoryService` to restore stock — not touching their databases directly
- **Prompt template:** the feature is scaffolded into `src/VendingMachine.Orders/CancelOrder/` with `CancelOrderCommand.cs` and `CancelOrderHandler.cs`

**Step 3 — Generate tests with layered constraints**

Open Chat, type `/add-l0-tests`, and enter:

```
CancelOrderHandler
```

Watch which layers activate:

- **Global instructions:** Given_When_Then method naming
- **L0 test instructions** (auto-attached because the target is in `Tests.L0/**`): `[Trait("Level", "L0")]`, `InMemoryInventoryRepository` and `InMemoryCashRepository` instead of real infrastructure, execution time goal
- **Prompt template:** test file created in `src/VendingMachine.Orders.Tests.L0/` with Arrange/Act/Assert structure

**Step 4 — Observe the commit message layer**

Stage the new files (`CancelOrder/` folder and the test file). Open the Source Control panel and use Copilot to generate a commit message.

The repository also has `.github/instructions/copilot-commit-message-instructions.md` — a Conventional Commits specification. Observe that the generated message follows the format:

```
feat(orders): add CancelOrder command with balance refund and stock restore
```

This is a fourth customization layer you did not write for this exercise — it came with the project and applies automatically to commit message generation.

**Step 5 — Understand the team sharing model**

Look at the complete `.github/` directory you have built across all four exercises:

```
.github/
├── instructions/
│   ├── copilot-instructions.md          ← global, always active
│   ├── copilot-commit-message-instructions.md  ← commit messages
│   ├── vendingmachine-cash.instructions.md     ← Cash infra only
│   ├── vendingmachine-inventory.instructions.md ← Inventory module
│   ├── vendingmachine-orders.instructions.md   ← Orders module (new)
│   ├── testing-l0.instructions.md              ← all L0 test projects (new)
│   └── postgres-migration.instructions.md      ← on-demand (new)
├── prompts/
│   ├── new-feature.prompt.md            ← team prompt (new)
│   └── add-l0-tests.prompt.md           ← team prompt (new)
└── skills/
    └── code-cleanup/SKILL.md            ← existing skill
```

Every file in `.github/` is version-controlled and shared with the team. A new developer who clones the repository gets the full Copilot customization from day one — no personal setup required.

The only customization that does **not** travel via Git is user-level files (`%APPDATA%\Code\User\prompts\`, `%APPDATA%\Code\User\agents\`, etc.), which are personal and follow your VS Code profile across machines.

### Success Criteria

- [ ] The `CancelOrder` feature is scaffolded in `src/VendingMachine.Orders/CancelOrder/` with Command and Handler files
- [ ] The Handler uses `ICashRegisterService` and `IInventoryService` — not direct database access
- [ ] L0 tests are generated in `src/VendingMachine.Orders.Tests.L0/` with correct markers and InMemory repos
- [ ] The Copilot-generated commit message follows the `feat(orders): ...` Conventional Commits format
- [ ] The final `.github/` tree contains all instruction and prompt files and the difference between team-shared and user-level customization is understood

---

## Summary: What You Have Built

After this workshop, you have a complete Copilot customization stack for the modular-monolith:

| Layer                     | File                                       | Scope                    | What it does                                                              |
| ------------------------- | ------------------------------------------ | ------------------------ | ------------------------------------------------------------------------- |
| **Global instructions**   | `copilot-instructions.md`                  | Every interaction        | Enforces TDD, module isolation, L0/L1 test levels, Given_When_Then naming |
| **Cash scoped**           | `vendingmachine-cash.instructions.md`      | `Cash.Infrastructure/**` | Balance safety, EnsureCreated, parameterized queries                      |
| **Inventory scoped**      | `vendingmachine-inventory.instructions.md` | `Inventory/**`           | Feature-folder structure, MediatR pattern, logging                        |
| **Orders scoped**         | `vendingmachine-orders.instructions.md`    | `Orders/**`              | Cross-module coordination via services, CQRS, event emission              |
| **L0 test cross-cutting** | `testing-l0.instructions.md`               | All `Tests.L0/**`        | Trait markers, InMemory repos, Arrange/Act/Assert                         |
| **Postgres on-demand**    | `postgres-migration.instructions.md`       | Manual or on-demand      | EnsureCreated patterns, parameterized queries, schema isolation           |
| **Commit messages**       | `copilot-commit-message-instructions.md`   | Commit generation        | Conventional Commits format                                               |
| **New feature prompt**    | `new-feature.prompt.md`                    | `/new-feature` command   | Scaffolds CQRS feature folder in any module                               |
| **L0 test prompt**        | `add-l0-tests.prompt.md`                   | `/add-l0-tests` command  | Generates L0 tests with correct structure and markers                     |

### The Customization Design Principle

> Put a rule at the **lowest scope** that captures it correctly. A rule that applies to every file belongs in `copilot-instructions.md`. A rule that applies to one module belongs in that module's `.instructions.md`. A rule that applies to a specific task belongs in a `.prompt.md`. Never repeat the same rule at multiple levels — it creates maintenance debt and conflicting signals.

---

### Useful Resources

- [Copilot customization overview](https://code.visualstudio.com/docs/copilot/copilot-customization)
- [Custom instructions reference](https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilot-instructionsmd-file)
- [Instruction files with applyTo](https://code.visualstudio.com/docs/copilot/copilot-customization#_instruction-files)
- [Prompt files reference](https://code.visualstudio.com/docs/copilot/copilot-customization#_prompt-files)
- [modular-monolith sample repository](https://github.com/phenixita/modular-monolith)
