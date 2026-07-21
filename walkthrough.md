# Swades Agent v3.1: The Ultimate Technical Manual & Architecture Guide

Welcome to the definitive guide for **Swades Agent v3.1**. This document is an exhaustive, production-grade technical manual. It details the entire architecture, internal APIs, execution states, system algorithms, configuration parameters, and developer operational workflows of Swades Agent.

---

## 🏗️ Codebase Architecture & File System Map

Swades Agent is written in a modular, zero-build Node.js (ES modules) runtime combined with Python automation helpers. Below is a map of the repository files:

```
workspace/
├── src/
│   ├── index.js             # Entry Point & AI Mode Classifier
│   ├── agent.js             # Core ReAct Loop, Timer & Pressure Injections
│   ├── tools.js             # Shell Spawner, Syntax Guardrails & Auto-Fixers
│   ├── prompts.js           # System Prompts & Tool Schema Definitions
│   ├── llm.js               # OpenAI SSE Token Stream Reconstructor
│   ├── memory.js            # Persistent Session Logger & Context Assembly
│   ├── director.js          # Supervisor Loop (Director Mode)
│   ├── cua.js               # Computer Use Agent (CUA) Desktop Orchestrator
│   ├── cua_helper.py        # Mutter RDP ScreenCast D-Bus Keyboard/Mouse Injector
│   ├── take_portal_screenshot.py # PipeWire Screen Stream Grabber
│   ├── orchestrator.js      # Subtask Decomposer & Worktree Semaphores
│   ├── subagent.js          # Worktree Child Agent Lifecycle Handler
│   └── simulator.js         # Multi-Scenario Sandbox Execution Engine
├── package.json             # App Metadata & Dependencies (openai, dotenv, chalk)
├── .env.example             # Configuration Blueprint
└── walkthrough.md           # This Hyper-Detailed Manual
```

---

## 1. CLI Entry & Mode Auto-Classification (`src/index.js`)

The entry point of Swades Agent is [`src/index.js`](file:///home/soham/reactsystemlearning1/src/index.js). It handles arguments, geolocations, indexes your workspace, and determines how tasks should be executed.

### CLI Flags & Arguments
You can execute Swades Agent using the following CLI syntax:
```bash
node src/index.js "[task prompt]" [flags]
```

| Flag | Shorthand | Target Execution Mode |
|---|---|---|
| `--normal` | `-n` | Normal Mode (Single-run worker ReAct loop) |
| `--autonomous` | `-a` | Autonomous Mode (Director-supervised multi-cycle loop) |
| `--cua` | `-c` | Computer Use Agent (GUI control and portal screenshot automation) |
| `--subagents` | `-s` | Subagents Mode (Run task decomposition directly without sandboxed simulation checks) |
| `--image [path/URL]` | `-i [path/URL]` | Multimodal Mode (Pass local/web images to the agent) |

### AI Mode Classification Flow
If you omit the execution mode flag (e.g. running `node src/index.js "Find memory leaks in database.js"`), `index.js` automatically invokes the AI classifier using a fast LLM request to analyze your prompt:

1. **System Prompt**: Enforces the classifier to output a single word: `"cua"`, `"autonomous"`, or `"normal"`.
2. **Analysis Rules**:
   - Classifies as `"cua"` if the prompt contains GUI, web browsing, click, keyboard shortcut, or desktop app requests (e.g., *"open chrome and check email"*).
   - Classifies as `"autonomous"` if the prompt describes complex coding goals (e.g., *"refactor the database layer and fix all breaking tests"*).
   - Classifies as `"normal"` if the prompt is informational or simple (e.g., *"explain how the git command works"*).
3. **Dispatch**: The program dynamically redirects the prompt to the corresponding execution orchestrator.

---

## 2. ReAct Loop & Core Agent (`src/agent.js` & `src/llm.js`)

The ReAct engine is located in [`src/agent.js`](file:///home/soham/reactsystemlearning1/src/agent.js), acting as the central scheduler.

### Thinking and Tool Call Streaming
Swades Agent uses true Token-by-Token streaming to let developers watch the agent's reasoning process in real time:

- **OpenAI Streaming Connection**: [`src/llm.js`](file:///home/soham/reactsystemlearning1/src/llm.js) requests completion with `stream: true` and the OpenRouter `context-compression` plugin enabled.
- **SSE Chunk Parsing**: As chunks arrive via Server-Sent Events (SSE), the stream parser reconstructs:
  - `delta.content`: Standard text streamed directly to the terminal using a blue thought bubble `💭`.
  - `delta.tool_calls`: Tool call properties (name and arguments) streamed inside a magenta tool indicator `🔧`.
- **Reconstruction**: Chunks are assembled into a clean, final tool call object that matches the standard non-streaming API return format.

---

## 3. Dynamic Timing Countdowns & Urgency Pressure System

To prevent infinite loops and optimize execution speed, Swades Agent v3.1 implements a **Timing Countdowns and Urgency Pressure Manager** directly within [`src/agent.js`](file:///home/soham/reactsystemlearning1/src/agent.js).

### The Deadline Estimator
Before starting a ReAct loop, the agent executes a background estimation call:
- **Prompt**:
  ```text
  You are the time estimator for Swades Agent. Estimate how many seconds this task should take to execute under ordinary circumstances. Respond with ONLY a single integer representing seconds (e.g. 120). Minimum: 30 seconds, Maximum: 600 seconds.
  ```
- **Clamping**: The output is parsed and clamped safely between `30` and `600` seconds, saving the value to `activeDeadline.estimatedSeconds`.

### Visual Countdown Bar & Chalk Colors
At each step of the loop, the agent prints a visual timer bar indicating the remaining duration:
```javascript
const pct = remaining / activeDeadline.estimatedSeconds;
const barWidth = 20;
const filledWidth = Math.max(0, Math.min(barWidth, Math.round(pct * barWidth)));
const barStr = "█".repeat(filledWidth) + "░".repeat(barWidth - filledWidth);
```

Colors update dynamically:
- **CALM** (Green, `remaining > 60%`): Safe workspace, focus on clean and complete code.
- **MEDIUM** (Yellow, `remaining 30%-60%`): Ticking timer, keep edits structured.
- **URGENT** (Red, `remaining 10%-30%`): Time is low, skip non-essential steps.
- **PANIC** (Bold Red, `remaining < 10%`): Finish immediately, skip cleanup.
- **OVERTIME** (Blinking/Inverted Red, `remaining <= 0`): System is running overdue.

### System Prompt Pressure Injection
At the start of each ReAct step, the system dynamically appends the timing context to the system prompt in `messages[0].content`:

```text
## URGENT TIMING AND DEADLINE SYSTEM
- Task Start Time: 2026-07-21T15:00:00Z
- Current Time: 2026-07-21T15:02:00Z
- Total Allocated Duration: 120s
- Elapsed Time: 120s
- Remaining Time: OVERTIME (0s overdue)
- Urgency Level: OVERTIME
- Critical Instruction: 🚨 DEADLINE EXPIRED: You are running in OVERTIME! You MUST wrap up immediately or request an extension.
- GRACE WARNING: You will be forcibly terminated in 3 steps if you do not complete the task or use 'extend_deadline'.
```

### Grace Step Loop Prevention
When the countdown enters `OVERTIME`, a grace counter of `3 steps` begins.
- Each subsequent step decrements the counter.
- If it reaches `0`, the loop is terminated with a loop-prevention error, preventing infinite LLM runs.
- If the agent calls the `extend_deadline` tool, `activeDeadline.estimatedSeconds` is incremented, returning the agent to a calm state and pausing loop prevention.

---

## 4. Self-Healing Linter & Syntax Guardrails (`src/tools.js`)

The file operations in [`src/tools.js`](file:///home/soham/reactsystemlearning1/src/tools.js) feature a self-healing compiler/linter that corrects errors at write-time.

### Indentation Sensitive Languages List
Strict indentation checks are only run for indentation-sensitive files:
```javascript
const isIndentationSensitive = ["py", "yml", "yaml", "gd", "nim", "hs"].includes(ext);
```
All other extensions skip indentation jump and mixed spacing warning alerts.

### 🛠️ Linter Repair Algorithms
When errors are detected on save, the linter applies three auto-repair algorithms:

1. **Unclosed Brackets Stack Repair**:
   - Parses the code character-by-character.
   - Ignores brackets nested inside string literals (`"..."`, `'...'`, `` `...` ``) and matches escape sequences.
   - Pushes open brackets (`{`, `(`, `[`) onto a stack, and pops them when their closing counterparts are encountered.
   - If unmatched items remain on the stack at EOF, it appends the matching closing brackets in reverse order.

2. **JSON Syntax Normalization**:
   - Uses regex to strip trailing commas before closing braces: `content.replace(/,(\s*[\]}])/g, "$1")`.
   - Locates unquoted keys and wraps them in double quotes: `content.replace(/(?<={|,)\s*([a-zA-Z0-9_$]+)\s*:/g, '"$1":')`.
   - Converts single-quoted strings to valid JSON double quotes.

3. **Indentation Mix Conversion**:
   - Scans lines to count space-led vs. tab-led indentation.
   - Detects the dominant type.
   - Re-formats lines by replacing tab characters with spaces (or vice versa) to ensure consistency.

---

## 5. Persistent Shell, Detached Timeout & Peek Terminal

The terminal runner in [`src/tools.js`](file:///home/soham/reactsystemlearning1/src/tools.js) uses an asynchronous, non-blocking execution design to prevent process freezes.

### Process Spawning & Output Buffering
Rather than blocking the main thread using synchronous execution, the agent uses `spawn`:

- **Detached State**: Spawns using `{ shell: true, detached: true, stdio: ["ignore", "pipe", "pipe"] }`. This detaches the process, preventing zombie threads.
- **Log Stream**: Pipes `stdout` and `stderr` streams directly to `.agent_terminal.log` in your workspace using `appendFile`.
- **Interval Check**: A promise-based loop checks if the process has completed.
- **Timeout Detach**: If execution exceeds 30 seconds, the agent detaches the loop and returns a warning. The command continues executing in the background.

### Peek Terminal Buffer Tool
When a command is detached, the agent can call `peek_terminal` at any point:
- **Status Check**: Inspects the background process PID using signal 0 (`process.kill(pid, 0)`).
- **Log Read**: Reads the accumulated content of `.agent_terminal.log` and returns the recent output.
- **Kill Action**: If the task needs to be cancelled, passing `action: "kill"` sends `SIGTERM` to the process group.

---

## 6. GUI Computer Use Agent (CUA) Mode (`src/cua.js`)

CUA mode allows the model to interact directly with GNOME Wayland and X11 desktops.

### Wayland D-Bus Mutter Portals
Wayland restricts global mouse queries and event injection for security reasons. Swades Agent bypasses this by using GNOME's native Mutter remote desktop infrastructure via the Python wrapper [`src/cua_helper.py`](file:///home/soham/reactsystemlearning1/src/cua_helper.py):

1. **Connector Resolution**: Queries the D-Bus display configurations (`org.gnome.Mutter.DisplayConfig`) to find the active monitor connector dynamically.
2. **Session Creation**: Establishes a remote desktop session and links it to a screencast session via Mutter portals.
3. **Screen Grab**: Uses PipeWire streams to grab screenshots, drawing a crosshair at the current mouse coordinates.
4. **Mouse Coordinate Tracking**: Because Wayland blocks coordinate queries, CUA tracks position values internally in `.mouse_position.json`.

### Click Loop Prevention
To prevent the model from getting stuck in click loops:
- **Bounding Box Proximity**: Clicks are mapped. Clicks falling within a **25px horizontal and 15px vertical bounding box** of a previous click are flagged.
- **Rules**:
  - The model cannot click the same coordinate area back-to-back.
  - The model cannot click the same area more than **2 times overall** throughout the task.
  - Clicks exceeding this limit are blocked, returning an error to the LLM to force a course correction.

---

## 7. Isolated Parallel Subagents & Git Worktrees

For high-complexity tasks, the parent orchestrator ([`src/orchestrator.js`](file:///home/soham/reactsystemlearning1/src/orchestrator.js)) decomposes the prompt into discrete subtasks.

### Parallel Workspace Isolation
- **Git Worktree Setup**: Spawns isolated directories under `.swades_worktrees/` via `git worktree add --detach`.
- **Concurrent Execution**: Subagents run parallel workers inside their respective worktree folders.
- **Diff Consolidation**: Diffs are generated and merged back into the main branch. Any git merge conflict is resolved dynamically by a dedicated merge-resolution subagent.

---

## 8. Multi-Scenario Sandbox Simulation Engine

The simulation engine ([`src/simulator.js`](file:///home/soham/reactsystemlearning1/src/simulator.js)) runs alternative implementations inside sandbox directories to choose the best solution.

### The Verification Pipeline
1. **Scenario Generation**: The LLM writes alternative designs (e.g. Scenario 1: Lightweight patch, Scenario 2: Refactored classes).
2. **Sandbox Isolation**: Runs each scenario inside `.swades_sandboxes/scenario_...`.
3. **Build & Test Suite Check**: Runs syntax checks (`node --check`) and testing commands (e.g. `npm test`) inside each sandbox.
4. **Winner Selection**: The LLM evaluates test outputs, select the clean, compilation-passing winner, and promotes the winning diff to the active workspace.
5. **Report Logging**: Writes a telemetry report to `.swades_simulation_report.json` showing simulated outputs vs. final outcomes.

---

## 💡 Developer Best Practices & Troubleshooting

- **Use the Right Flag**: Use `--normal` for single edits and queries, and `--autonomous` for complex development cycles.
- **Ensure Codebase Indexing**: Run `index_codebase` at startup to map imports, exports, functions, and classes across your project.
- **Monitor Terminals**: Use `peek_terminal` to check on background builds. If a compile gets stuck, call `peek_terminal` with `action: "kill"` to stop it.
- **Check Linter Errors**: If the linter reports a syntax warning, let the auto-fixer correct it or use `patch_file` to fix unclosed braces.
- **Manage Deadlines**: If the timer countdown is in `PANIC` or `URGENT` status, let the agent call `extend_deadline` to get more time.
