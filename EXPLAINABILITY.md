# EXPLAINABILITY — OpenCode Autonomous Coding Agent

> **Admissibility & Transparency Report for OpenGAP / Agent Passport**  
> *Agent Name:* OpenCode Autonomous Coding Agent (`opencode-agent-harness`)  
> *Specification:* OpenGAP v0.1.0  
> *Domain:* Developer Tools / Autonomous Coding Agent  

---

## 1. Overview & Operational Purpose

OpenCode Autonomous Coding Agent is an open-source terminal-native software engineering agent engineered to navigate complex codebases, execute targeted refactors, run test suites, and resolve software issues autonomously. Its primary operational purpose is to empower developers with a privacy-respecting, multi-model coding assistant that operates directly in the terminal, integrating Language Server Protocol diagnostics with sandboxed command execution.

By combining AST-aware file patching, reciprocal symbol navigation, and verification-driven test loops, OpenCode eliminates speculative hallucinations, prevents syntax regressions, and provides complete transparency into file edits and command execution.

---

## 2. How the Agent Decides (Decision-Making Logic)

OpenCode Autonomous Coding Agent operates across a deterministic, multi-stage decision pipeline:

```
[Stage 1: Intent & Task Decomposition] ──> [Stage 2: Codebase Context Retrieval] ──> [Stage 3: LSP Symbol Inspection]
                                                                                                    │
                                                                                                    ▼
[Stage 6: Commit Log & Telemetry Audit] <── [Stage 5: Test Execution & Verify]   <── [Stage 4: Surgical Diff Patching]
```

### 2.1 Intent Parsing & Task Decomposition
- **Decision:** The agent parses the developer's request, identifying whether the objective is an exploratory query, bug fix, feature addition, or test execution.
- **Rules:** Decompose complex goals into sequential sub-tasks. If requirements are underspecified, inspect codebase conventions or solicit user clarification before modifying files.

### 2.2 Codebase Context Retrieval
- **Decision:** Identify target files and symbol locations using ripgrep text searches and directory structure maps.
- **Rules:** Restrict search scope to relevant subdirectories. Avoid loading oversized binary files or dependency caches (`node_modules`, `target`, `venv`) into context.

### 2.3 LSP Symbol Inspection & Verification
- **Decision:** Query Language Server Protocol endpoints to resolve symbol definitions, type signatures, and compile diagnostics.
- **Rules:** Confirm type safety and function contract signatures before formulating code edits. Treat compiler warnings and errors as blocking constraints.

### 2.4 Surgical Diff Patching & Test Verification
- **Decision:** Apply localized block replacements and execute project test commands to verify behavioral correctness.
- **Rules:** Reject modifications that produce regressions or failing tests. Automatically revert changes if unexpected build failures occur.

---

## 3. Data Flow & Boundary Privacy

OpenCode operates locally within the terminal workspace, ensuring developer source code and intellectual property remain protected.

| Component / Boundary | Data Received | Processing & Retention | Destination / External Transmission |
|---|---|---|---|
| Terminal UI Gateway | User commands, interactive prompts | Ephemeral session parsing; discarded on exit | Local terminal process |
| Codebase File System | Source code files, project manifests | Local in-memory reading and surgical patching | Local disk filesystem only |
| LSP Server Engine | Symbol definitions, syntax trees, diagnostic errors | In-process IPC communication with local language servers | Local LSP daemon process |
| Model API Dispatcher | Sanitized prompt instructions, targeted code snippets | Transport encryption (TLS 1.3); ephemeral inference only | Configured upstream LLM API |

OpenCode Autonomous Coding Agent complies with operational security and privacy standards:
- **No Cloud Data Exfiltration:** The agent does not transmit user source code to telemetry collectors or unauthorized third-party platforms.
- **Epistemic Isolation:** Subagent processes run with segregated memory contexts, preventing cross-task context leakage.
- **Sanitized Model Payloads:** API keys, secrets, `.env` files, and private credentials are automatically scrubbed from prompt payloads.
- **Data Minimization:** Only code blocks strictly relevant to the active editing task are included in model prompt contexts.

---

## 4. Known Limitations & Failure Modes

Reviewers, auditors, and users should note the following operational constraints:

1. Non-Deterministic Build Environments
   - *Limitation:* External network flakiness or missing system packages during build commands can trigger spurious test failures.
   - *Mitigation:* The agent separates environment setup issues from code errors and advises explicit dependency installation steps.

2. Context Window Contention in Monorepos
   - *Limitation:* Navigating massive enterprise monorepos with hundreds of thousands of files can strain LLM context capacity.
   - *Mitigation:* Employ targeted ripgrep filters and LSP symbol navigation rather than loading whole directories into context.

3. Competing Concurrent File Edits
   - *Limitation:* External file edits made outside the agent while a task is running can cause patch collision conflicts.
   - *Mitigation:* The agent checks file modification timestamps and verifies exact block hashes before writing changes.

4. Terminal Shell Escaping and OS Differences
   - *Limitation:* Command syntax differences between Unix shells and Windows PowerShell can lead to execution syntax errors.
   - *Mitigation:* The agent detects the host operating system and formats command strings with appropriate cross-platform quoting.

---

## 5. Verification, Safety & Human Oversight

OpenCode Autonomous Coding Agent incorporates robust verification, safety gates, and human oversight controls across every layer of execution:

- **Real-Time Human Approval Gate:** Potentially destructive terminal commands (e.g. `rm -rf`, `git push --force`, `git reset --hard`) pause execution and require explicit operator confirmation.
- **Emergency Session Interrupt:** Users can terminate active command loops, background subagents, or model streaming instantly via standard Ctrl+C break signals.
- **Step Quota Guardrails:** Strict session caps (maximum 25 turns) prevent unconstrained automated looping or token budget exhaustion.
- **Structured Audit Logging:** Every executed shell command, file diff, and subagent invocation is logged in structured JSON formats for post-session auditability.
