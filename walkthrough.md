# Swades Agent v3.1: The Ultimate Operational Manual & Developer Tutorial

Welcome to the definitive user manual and operational handbook for **Swades Agent v3.1**. This document is written specifically for software developers, system administrators, and quality assurance engineers. It details exactly how to install, configure, command, interact with, and troubleshoot the agent, helping you leverage every advanced capability to its absolute limit.

---

## Table of Contents
1. [Core Philosophy & Operational Flow](#1-core-philosophy--operational-flow)
2. [Installing and Configuring the Agent Environment](#2-installing-and-configuring-the-agent-environment)
3. [The Command Interface (Modes & Syntax)](#3-the-command-interface-modes--syntax)
4. [Mastering the Countdown Timer & Urgency Pressure System](#4-mastering-the-countdown-timer--urgency-pressure-system)
5. [Operating the Self-Healing Linter & Indentation Rules](#5-operating-the-self-healing-linter--indentation-rules)
6. [Managing Persistent Terminal Processes (`peek_terminal`)](#6-managing-persistent-terminal-processes-peek_terminal)
7. [Wayland Native GUI (CUA Mode) Installation & Execution](#7-wayland-native-gui-cua-mode-installation--execution)
8. [Decomposing Complex Code (Subagents & Sandboxes)](#8-decomposing-complex-code-subagents--sandboxes)
9. [Advanced Prompting Recipes & Operational Troubleshooting](#9-advanced-prompting-recipes--operational-troubleshooting)

---

## 1. Core Philosophy & Operational Flow

Swades Agent v3.1 is designed to operate as a terminal-native AI software engineer. Instead of using a complex visual interface or locked cloud sandboxes, the agent operates directly on your local system, reading your codebase, making surgical file edits, running shell commands, and compiling logs.

### The Developer-in-the-Loop Cycle
The agent interacts with you using a structured execution pipeline:
1. **Goal Submission**: You submit a task via the command line or an interactive dashboard.
2. **Codebase Indexing**: The agent scans your codebase structure to find imports, functions, and classes.
3. **Execution Plan**: The agent lists its thoughts, schedules tool runs, and monitors time constraints.
4. **Tool Execution**: Tools run locally. If a command runs too long, the system detaches it to prevent shell freezes.
5. **Syntax Correction**: The linter reviews every written file, automatically fixing bracket errors, formatting issues, or JSON mistakes before the code is compiled.
6. **Goal Achievement**: The agent presents the final code modifications and copy-pasteable verification scripts.

---

## 2. Installing and Configuring the Agent Environment

Follow these steps to set up Swades Agent inside your project workspace.

### Step 1: Install Node.js & System Prerequisites
Ensure you have **Node.js v18 or later** installed (v22 is highly recommended for optimal performance). Verify your installation:
```bash
node -v
npm -v
```

### Step 2: Install Project Dependencies
Run the installation command inside the agent's root directory:
```bash
npm install
```
This command installs the three core NPM modules required by the runtime:
- `openai`: Manages communication with OpenAI-compatible API endpoints and parses SSE token streams.
- `dotenv`: Loads environment configurations from your local `.env` file into `process.env`.
- `chalk`: Formats and color-codes terminal logs (e.g. countdown bars, tool indicators, errors).

### Step 3: Setup the `.env` Configuration File
Copy the example template to create your active configuration:
```bash
cp .env.example .env
```
Open `.env` in your text editor. Below is a detailed breakdown of every environment variable you can customize:

```env
# ============================================================
# SWADES AGENT ENVIRONMENT CONFIGURATION
# ============================================================

# 1. API Credentials & Endpoints
# Your LLM provider API key (e.g., OpenRouter, OpenAI, Groq, etc.)
API_KEY=sk-or-v1-your-key-here

# The base URL of your model provider (change for Groq, Ollama, etc.)
BASE_URL=https://openrouter.ai/api/v1

# 2. Model Selections
# The primary model used to handle task planning and code editing
MODEL=openrouter/free

# The vision-capable model used to analyze screen grabs in GUI CUA mode
CUA_MODEL=openrouter/free

# 3. Execution Safety Caps
# Maximum step iterations allowed per agent run before a forced stop
MAX_STEPS=30

# Maximum character length for tool outputs returned to the agent
MAX_OUTPUT_LENGTH=10000

# 4. Scope & Directories
# Target workspace directory (defaults to the agent folder)
# Set to "../" if the agent is installed as a subfolder in your project
WORKDIR=process.cwd()

# 5. Connectors
# Optional API key for Google Maps directions queries
GOOGLE_MAPS_API_KEY=
```

### Setup Templates for LLM Providers

#### Template A: OpenRouter Setup (Default)
```env
API_KEY=sk-or-v1-your-openrouter-key-here
BASE_URL=https://openrouter.ai/api/v1
MODEL=meta-llama/llama-3.3-70b-instruct
```

#### Template B: OpenAI Setup
```env
API_KEY=sk-proj-your-openai-key-here
BASE_URL=https://api.openai.com/v1
MODEL=gpt-4o
```

#### Template C: Groq Setup
```env
API_KEY=gsk_your-groq-key-here
BASE_URL=https://api.groq.com/openai/v1
MODEL=llama-3.3-70b-versatile
```

#### Template D: Local Ollama Setup
```env
API_KEY=ollama
BASE_URL=http://localhost:11434/v1
MODEL=qwen2.5-coder:7b
```

---

## 3. The Command Interface (Modes & Syntax)

You can launch tasks interactively or pass prompts directly as command-line arguments.

### Interactive Dashboard Setup
Run the dashboard script:
```bash
npm start
```
The terminal will display the configuration menu:
1. **Task →**: Enter your coding task (e.g. *"add input validation to the register form in src/auth.js"*).
2. **Image path/URL (optional) →**: Provide a path or web URL to a layout image/mockup if you are using a vision-capable model.
3. **Mode? →**: Select your execution mode:
   - `c` (or `cua`): Desktop GUI automation.
   - `a` (or `autonomous`): Director-supervised multi-cycle development loops.
   - `s` (or `subagents`): Runs task decomposition in parallel workspaces.
   - `n` (or `normal`): Single-run worker loop.
   - Press **Enter** to let the AI auto-classify task complexity.

### CLI Flag Specification
Trigger tasks directly by passing CLI flags:
```bash
# Run a simple check query
node src/index.js "Check git status and list modified files" --normal

# Launch a complex feature implementation with tests
node src/index.js "Implement a DB caching mechanism and run npm test" --autonomous

# Spawn Wayland GUI mode to open a web browser
node src/index.js "Open Chrome and search for Node.js docs" --cua
```

---

## 4. Mastering the Countdown Timer & Urgency Pressure System

Swades Agent v3.1 features a countdown deadline manager to keep the agent focused.

### Understanding the Visual Timer
At the start of the task, the agent estimates a duration budget in seconds. At each step, a countdown status bar is rendered to the terminal. Watch the bar colors to monitor progress:

* **[CALM]** (Green Bar): > 60% time remaining. The agent operates normally, focusing on clean code writing and validating edits.
* **[MEDIUM]** (Yellow Bar): 30% to 60% time remaining. The agent works efficiently, avoiding redundant commands.
* **[URGENT]** (Red Bar): 10% to 30% time remaining. The agent prioritizes completing the task, running quick compiler syntax validation checks.
* **[PANIC]** (Bold Red Bar): < 10% time remaining. The agent focuses strictly on the core solution, omitting non-essential checks.
* **[OVERTIME]** (Inverted Red Bar): Deadline has run out. The agent is in overtime and must finish immediately.

### Overtime Grace Ticks & Forced Termination
If the agent enters **OVERTIME**, a warning is injected into its prompt:
`🚨 DEADLINE EXPIRED: You are running in OVERTIME! Wrap up or request an extension.`
- The agent gets a strict **limit of 3 steps** in overtime.
- If it doesn't finish or request an extension, the system triggers a **forced loop-prevention termination** to save token costs.

### How to Tell the Agent to Extend the Timer
If you anticipate that a task is complex or requires installing software, you can prompt the agent to manage its time budget directly:
> *"Implement feature X and run the build tests. If you run low on time, make sure to call the extend_deadline tool to add 120 seconds."*

The agent will invoke the tool:
```json
{
  "name": "extend_deadline",
  "arguments": {
    "additional_seconds": 120,
    "reason": "Compiling libraries is taking longer than expected"
  }
}
```
This resets the timers and returns the agent to a CALM state.

---

## 5. Operating the Self-Healing Linter & Indentation Rules

When the agent writes files, syntax guardrails prevent code corruption.

### Read Linter Reports
On every save or patch, the linter prints validation outputs to the console:
- **`✅ File written successfully`**: Indicates zero syntax or indentation errors.
- **`❌ WARNING: SYNTAX ERRORS DETECTED`**: Lists JavaScript compile errors (`node --check`), bracket mismatch lines, or JSON parser errors.
- **`⚠️ INDENTATION WARNINGS`**: Informs the agent of inconsistent tab/space patterns or sudden indentation jumps (jumps > 4 spaces without a block opening character like `{`, `(`, `[`, or `:`).

### Self-Healing Logic
You do not need to intervene if the agent makes a bracket error. The linter's auto-fixer will:
1. Parse bracket structures and append missing closing brackets at EOF.
2. Format mixed spaces/tabs automatically.
3. Clean JSON trailing commas and wrap unquoted keys in double quotes.

If successful, the linter writes the corrected content and prints:
`✅ File written and automatically auto-fixed syntax errors.`

### Strict vs Cosmetic Indentation Languages
Indentation warnings are only enabled for indentation-sensitive files:
- **Strict Indentation**: Python (`.py`) and YAML (`.yml`, `.yaml`).
- **Cosmetic Indentation**: JavaScript, CSS, HTML, Markdown, and configuration files skip indentation warnings.

---

## 6. Managing Persistent Terminal Processes (`peek_terminal`)

If a command takes longer than 30 seconds (e.g. running a compile step or dev server), the agent detaches the task to run in the background.

### Checking Background Process Logs
If a task detaches, you can instruct the agent to poll the output using `peek_terminal`:
> *"Run npm run build. If it detaches, wait 10 seconds and peek at the terminal log to confirm it compiled successfully."*

The agent calls `peek_terminal` with the default `peek` action:
- **`[STATUS: RUNNING]`**: The background task is still executing. The agent reads the tail of `.agent_terminal.log` to view recent output.
- **`[STATUS: COMPLETED]`**: The background process finished. The agent reads the final logs and handles success/error codes.

### Terminating Stuck Processes
If a background server or test loop gets stuck:
1. Prompt the agent: *"Kill the background process immediately using peek_terminal with the kill action."*
2. The agent executes `peek_terminal` with `action: "kill"`.
3. This sends `SIGTERM` to the process group, releasing system resources.

---

## 7. Wayland Native GUI (CUA Mode) Installation & Execution

CUA mode allows the agent to control your desktop. For safety under modern Linux installations, set up the following:

### Step 1: Install Introspector Packages
Wayland CUA requires system GObject libraries to communicate over the Mutter remote desktop bus:

* **Ubuntu / Debian**:
  ```bash
  sudo apt update && sudo apt install -y python3-gi python3-gi-cairo
  ```
* **Fedora**:
  ```bash
  sudo dnf install -y python3-gobject python3-gobject-base
  ```
* **Arch Linux**:
  ```bash
  sudo pacman -S --noconfirm python-gobject
  ```

### Step 2: Enable Remote Sharing
In your GNOME desktop settings, navigate to:
**Settings** -> **Sharing** -> **Remote Desktop**
Turn on **Remote Desktop** and **Screen Sharing**.

### Interactive Click Proximity Protection
If CUA mode clicks a frozen UI element, the system protects against loops:
- If the agent clicks within **25px horizontally and 15px vertically** of its previous click consecutively, the orchestrator blocks the call.
- The agent is blocked if it clicks the same coordinate area more than **2 times overall**.
- The tool returns a declined message, forcing the model to scroll, type, or navigate elsewhere.

---

## 8. Decomposing Complex Code (Subagents & Sandboxes)

For complex refactoring, Swades Agent orchestrates multiple tasks in isolated workspaces.

### Monitoring Parallel Subagents
When a task is classified as high-complexity:
1. The parent orchestrator breaks down the main prompt into subtasks.
2. It sets up git worktrees under `.swades_worktrees/`.
3. Subagents run parallel modifications. You can watch progress outputs from all subagents concurrently.
4. Git merge conflicts are resolved dynamically by a merge-resolution agent.

### Sandbox Simulation Engine
Before applying modifications to your live workspace files, the engine generates alternative scenarios:
1. It copies project code to scenario sandboxes under `.swades_sandboxes/`.
2. It compiles the code and runs test suites inside each sandbox.
3. The LLM evaluates the results, selects the winner, and promotes the winning diff to your workspace.
4. Telemetry logs containing comparisons are saved to `.swades_simulation_report.json`.

---

## 9. Advanced Prompting Recipes & Operational Troubleshooting

Optimize agent behavior using the following structured prompts:

### Caching Refactor Recipe
> *"Scan the repository using index_codebase. Refactor src/db.js to include a cache layer. Write the cache configuration in config.json. Run node --check to verify compile status. If the step timer runs low, extend the deadline by 120 seconds."*

### Background Test Compilation Recipe
> *"Execute npm run build. If the build command takes longer than 30s and detaches, wait 15 seconds, then call peek_terminal with action='peek' to read the compilation completion logs."*

### Troubleshooting FAQ

#### Q: The agent gets stuck in a loop trying to patch a file.
* **Solution**: Ensure your target block matches the file content exactly. You can read the target file using `read_file` with a line range to confirm its indentation.

#### Q: I hit a timeout when installing packages.
* **Solution**: Detached package installations run in the background. Prompt the agent: *"Call peek_terminal to verify the package installation completed."*

#### Q: How do I change the default time limit?
* **Solution**: Set `MAX_STEPS` in your `.env` file to limit the steps, or use `extend_deadline` to adjust timing.

#### Q: The agent fails to connect to the model.
* **Solution**: Double-check your `API_KEY` and `BASE_URL` settings in `.env`. If using local models, ensure Ollama is running (`ollama run qwen2.5-coder:7b`).
