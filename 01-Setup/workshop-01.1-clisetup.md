<img src="https://mfitcontent.blob.core.windows.net/static/MF-logo.png" style="height:64px;margin-right:32px"/>


# Workshop 1 — Getting Started with GitHub Copilot CLI

> **Estimated time: 60 minutes** (4 exercises)
>
> **Difficulty: medium** — this workshop walks you through installing, configuring, and using GitHub Copilot CLI directly from the terminal. You will start with the base installation, learn interactive commands, explore non-interactive mode, and finish with practical automation scenarios. By the end you will be able to use Copilot CLI as an AI assistant fully integrated into your daily command-line workflow.

AI-Assisted Development — CLI Module

## Terms and Conditions of Use

This training package is proprietary and confidential and is intended exclusively for the uses described in the training materials. Copying or disclosing all or part of the content and/or software included in such packages is prohibited. The contents of this package are for informational and training purposes only and are provided "as is" without warranties of any kind, express or implied, including but not limited to the implied warranties of merchantability, fitness for a particular purpose, and non-infringement. The content of the training package, including URLs and other references to Internet websites, is subject to change without notice. Unless otherwise noted, the companies, organizations, products, domain names, email addresses, logos, people, places, and events depicted herein are fictitious and no association with any real company, organization, product, domain name, email address, logo, person, place, or event is intended or should be inferred.

---

GitHub Copilot CLI is a terminal-native AI assistant that brings agentic capabilities directly to your command line. Unlike the VS Code extension, Copilot CLI operates entirely in the terminal, allowing you to get answers, generate commands, and automate tasks without ever leaving the shell. In this workshop you will install Copilot CLI, learn to use it in interactive and non-interactive mode, and discover how to integrate it into your automation scripts.

---

## Exercise 1: Installing and Launching Copilot CLI for the First Time

This exercise walks you through installing GitHub Copilot CLI and running your first interactive session, including authentication with your GitHub account.

### Prerequisites

Before you begin, make sure you have:

- A GitHub account with an active Copilot license (Free, Pro, Business, or Enterprise)
- **Node.js 22 or later** installed (required for the npm installation method) — or Homebrew (macOS/Linux) or WinGet (Windows)
- A terminal application (Terminal, iTerm2, Windows Terminal, PowerShell, etc.)
- Internet access

### Objective

1. Install GitHub Copilot CLI on your machine
2. Authenticate with your GitHub account
3. Run your first prompt in interactive mode

**Step 1 — Install Copilot CLI**

Choose the installation method that best fits your operating system:

- **Cross-platform (npm):**
  ```bash
  npm install -g @github/copilot
  ```
- **Windows (WinGet):**
  ```bash
  winget install GitHub.Copilot
  ```
- **macOS/Linux (Homebrew):**
  ```bash
  brew install copilot-cli
  ```

**Step 2 — Start an interactive session**

- Open the terminal and navigate to a project directory (for example an existing Git repository):
  ```bash
  cd ~/my-project
  ```
  ```powershell
  # Cross-platform PowerShell (pwsh):
  Set-Location ~/my-project
  # or
  cd ~/my-project
  ```
- Launch Copilot CLI in interactive mode:
  ```bash
  copilot
  ```

**Step 3 — Authentication**

- In the interactive session, type:
  ```
  /login
  ```
- Follow the on-screen instructions to authenticate with your GitHub account (a browser window will open for the OAuth flow)
- This step is only required the first time you use the CLI

**Step 4 — Trust the directory and ask your first question**

- When prompted, confirm that the files in the current directory are safe for use with an AI tool
- Try asking Copilot a question:
  ```
  Give me an overview of this project.
  ```
- Observe how Copilot analyzes the project context and responds directly in the terminal

### Success Criteria

- [ ] Copilot CLI is installed and the `copilot` command is available in the terminal
- [ ] Authentication with GitHub completed successfully
- [ ] Copilot responds to the first question by showing information about the project

---

## Exercise 2: Commands and Shortcuts in Interactive Mode

This exercise teaches you how to navigate the Copilot CLI interactive session effectively using keyboard shortcuts and slash commands.

### Prerequisites

Before you begin, make sure you have:

- Copilot CLI installed and authenticated (completed in Exercise 1)
- An interactive session open (`copilot`)

### Objective

1. Master the main keyboard shortcuts
2. Use slash commands and the help system
3. Include files as context in prompts using `@`

**Step 1 — Explore keyboard shortcuts**

Try the following key combinations during the interactive session:

| Shortcut | Action |
|----------|--------|
| `Esc` | Cancel the current operation |
| `Ctrl+C` | Cancel if thinking, clear input, or exit |
| `Ctrl+L` | Clear the screen |
| `↑` and `↓` | Navigate the command history |

**Step 2 — Use slash commands**

- Type `/` to display the list of available commands
- Type `?` to open the tabbed help view
- Run the full help command:
  ```
  /help
  ```
- Browse the listed commands and try using at least 2–3 of them

**Step 3 — Include files as context with @**

- Use the `@` symbol to mention a specific file and include it as context in your prompt:
  ```
  @src/index.js Explain what this file does and suggest improvements
  ```
- Try with different files from your project to see how Copilot adjusts its answers based on the context provided

### Success Criteria

- [ ] You can use `Esc` and `Ctrl+C` to cancel operations
- [ ] You have explored the slash command list with `/` and the help view with `?`
- [ ] You have used `@` to include a file as context in a prompt

---

## Exercise 3: Using Copilot CLI in Non-Interactive Mode

This exercise shows you how to use Copilot CLI directly from the command line without starting an interactive session, using the `-p` and `-s` flags.

### Prerequisites

Before you begin, make sure you have:

- Copilot CLI installed and authenticated (completed in Exercise 1)
- A terminal open (you do not need to start an interactive session)

### Objective

1. Use the `-p` flag to send single prompts from the command line
2. Use the `-s` flag to get clean output (response only)
3. Understand the difference between standard output and silent output

**Step 1 — Send a single prompt with `-p`**

- Run a direct command without entering the interactive session:
  ```bash
  copilot -p "In Git, how can I apply a commit from another branch"
  ```
- Observe the output: Copilot's response is followed by additional usage information

**Step 2 — Get only the response with `-s`**

- Combine the `-s` and `-p` flags to get only Copilot's response, without additional information:
  ```bash
  copilot -sp "Explain the difference between git rebase and git merge"
  ```
- Compare the output with the previous command: notice how `-s` removes the extra metadata

**Step 3 — Try different types of prompts**

- Ask Copilot to generate a shell command:
  ```bash
  copilot -sp "Give me the command to find all .log files larger than 100MB in the current directory"
  ```
- Ask for a technical explanation:
  ```bash
  copilot -sp "What does the curl -X POST flag do?"
  ```
- Ask for help with an error:
  ```bash
  copilot -sp "What does 'fatal: not a git repository' mean and how do I fix it?"
  ```

**Step 4 — Explore the command-line help**

- Display all available options:
  ```bash
  copilot help
  ```
- Dig deeper into a specific topic:
  ```bash
  copilot help TOPIC
  ```
  (replace `TOPIC` with one of the topics listed in the `copilot help` output)

### Success Criteria

- [ ] You can use `copilot -p "..."` to send single prompts
- [ ] You can use `copilot -sp "..."` to get clean output without metadata
- [ ] You have explored `copilot help` and understand the available options

---

## Exercise 4: Automation with Non-Interactive Mode

This exercise shows you how to integrate Copilot CLI into scripts and automation workflows, leveraging non-interactive mode to create repeatable tasks.

### Prerequisites

Before you begin, make sure you have:

- Copilot CLI installed and authenticated (completed in Exercise 1)
- Familiarity with the `-sp` flag (completed in Exercise 3)
- Basic knowledge of Bash scripting

### Objective

1. Use Copilot CLI output inside Bash scripts
2. Chain Copilot CLI with other commands using pipes
3. Create a simple automated workflow

**Step 1 — Capture Copilot output in a variable**

- Save Copilot's response into a Bash variable:
  ```bash
  RESPONSE=$(copilot -sp "Write a concise commit message for: added input validation to the login form")
  echo "$RESPONSE"
  ```
  ```powershell
  $RESPONSE = (copilot -sp 'Write a concise commit message for: added input validation to the login form')
  Write-Output $RESPONSE
  ```

**Step 2 — Use Copilot CLI in a pipeline**

- Combine Copilot with other terminal commands using pipes:
  ```bash
  git diff --staged | copilot -sp "Summarize these changes in one sentence"
  ```
- Try generating automatic documentation for a file:
  ```bash
  cat src/utils.js | copilot -sp "Generate JSDoc comments for each function in this file"
  ```
  ```powershell
  Get-Content src/utils.js -Raw | copilot -sp "Generate JSDoc comments for each function in this file"
  ```

**Step 3 — Create an automation script**

- Create a file called `ai-commit.sh` that uses Copilot to generate a commit message:
  ```bash
  #!/bin/bash
  # ai-commit.sh — Generate a commit message with Copilot CLI

  DIFF=$(git diff --staged)

  if [ -z "$DIFF" ]; then
    echo "No staged changes. Run 'git add' first."
    exit 1
  fi

  MSG=$(echo "$DIFF" | copilot -sp "Write a conventional commit message for these changes. Reply with only the commit message, no explanation.")

  echo "Suggested message: $MSG"
  read -p "Use this message? (y/n) " CONFIRM

  if [ "$CONFIRM" = "y" ]; then
    git commit -m "$MSG"
    echo "Commit created!"
  else
    echo "Commit cancelled."
  fi
  ```
  ```powershell
  # ai-commit.ps1 — Generate a commit message with Copilot CLI

  $diff = git diff --staged

  if ([string]::IsNullOrWhiteSpace($diff)) {
    Write-Output "No staged changes. Run 'git add' first."
    exit 1
  }

  $msg = (copilot -sp 'Write a conventional commit message for these changes. Reply with only the commit message, no explanation.') -join "`n"

  Write-Output "Suggested message: $msg"
  $confirm = Read-Host "Use this message? (y/n)"

  if ($confirm -eq 'y') {
    git commit -m "$msg"
    Write-Output "Commit created!"
  } else {
    Write-Output "Commit cancelled."
  }
  ```
- Make the script executable and test it:
  ```bash
  chmod +x ai-commit.sh
  git add .
  ./ai-commit.sh
  ```
  ```powershell
  # Run the PowerShell script (PowerShell Core)
  pwsh ./ai-commit.ps1

  # If using PowerShell interactive session, you can also:
  # Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
  # ./ai-commit.ps1
  ```

### Success Criteria

- [ ] You can capture the output of `copilot -sp` in a Bash variable
- [ ] You have used Copilot CLI in a pipeline with pipes (`|`)
- [ ] You have created and tested a script that integrates Copilot CLI into an automated workflow

---

### Useful Resources

- [About GitHub Copilot CLI](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/about-copilot-cli)
- [Copilot CLI Quickstart](https://docs.github.com/en/copilot/how-tos/copilot-cli/cli-getting-started)
- [Copilot CLI Best Practices](https://docs.github.com/en/copilot/how-tos/copilot-cli/cli-best-practices)
- [Using Copilot CLI Non-Interactively](https://docs.github.com/en/copilot/how-tos/copilot-cli/cli-getting-started#using-github-copilot-cli-non-interactively)
- [Copilot CLI Command Reference](https://docs.github.com/en/copilot/reference/cli-command-reference)