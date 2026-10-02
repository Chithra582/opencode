# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **OpenCode Autonomous Software Engineering Harness** (`opencode`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** OpenCode Autonomous Software Engineering Harness (`opencode`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Developer Tools / Open Autonomous Coding & Agent Harness  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

OpenCode Autonomous Software Engineering Harness is an open-source terminal-driven agent runtime engineered to inspect codebases, execute targeted refactors, automate package operations, and verify software patches with native developer toolchains. Its primary operational purpose is to provide software engineers with an unconstrained, vendor-independent AI pair programmer capable of driving terminal tools, reading multi-file context, and applying verified diffs with complete local privacy.

### 1. Decision Architecture

The issue intake, AST codebase exploration, patch drafting, and test execution pipeline operates across a deterministic, five-stage architecture:

```
Developer Directive / Issue Trigger (Task Prompt / Stack Trace / Refactoring Goal)
    │
    ▼
[Stage 1: Intent & Task Decomposition]
    │  - Evaluates user engineering instructions and project constraints
    │  - Decomposes complex refactor objectives into discrete file-level tasks
    │  - Initializes local workspace exploration boundaries
    ▼
[Stage 2: Codebase Navigation & Symbol Indexing]
    │  - Executes ripgrep symbol queries, AST parsing, and directory walks
    │  - Bounds file reading to focused line windows (<150 lines) to avoid context saturation
    │  - Maps dependencies and identifying test suites verifying target modules
    ▼
[Stage 3: Surgical Diff Synthesis & Patching]
    │  - Synthesizes atomic line-replacement diffs targeting only the affected logic blocks
    │  - Preserves surrounding indentation, style conventions, and type annotations
    │  - Validates patch syntax with local linters prior to disk commitment
    ▼
[Stage 4: Subprocess Execution & Test Verification]
    │  - Executes test suites in bounded terminal subprocesses (pytest, npm test, cargo test)
    │  - Parses test failures and compiler error tracebacks
    │  - Iterates through corrective patch loops if assertions fail (up to 3 cycles)
    ▼
[Stage 5: Git Patch Packaging & Trajectory Archive]
    │  - Packages clean unified git diffs ready for developer code review
    │  - Scrubs private user tokens, environment variables, and local paths
    │  - Emits structured decision traces to local workspace for auditing
    ▼
Validated Software Engineering Patch & Auditable Harness Trajectory Record
```

### 2. Decision Logic & Patch Verification Formulations

OpenCode evaluates code localization, patch safety, and test confidence using deterministic mathematical models:

1. **Symbol Relevance Affinity Score ($S_{\text{symbol}}$)**:
   $$S_{\text{symbol}} = (w_f \cdot F_{\text{frequency}}) + (w_d \cdot D_{\text{docstring}}) + (w_c \cdot C_{\text{callgraph}})$$
   where:
   - $F_{\text{frequency}} \in [0, 1]$ represents BM25 keyword frequency in the target module.
   - $D_{\text{docstring}} \in [0, 1]$ represents semantic alignment with module docstrings.
   - $C_{\text{callgraph}} \in [0, 1]$ represents static call graph proximity to the failing line.
   - Weights: $w_f = 0.40, w_d = 0.30, w_c = 0.30$ ($\sum w_i = 1.0$).

2. **Patch Confidence Index ($I_{\text{conf}}$)**:
   $$I_{\text{conf}} = \frac{1}{3} \left( T_{\text{pass}} + L_{\text{lint}} + M_{\text{diff}} \right)$$
   where $T_{\text{pass}} \in \{0, 1\}$ indicates test suite success, $L_{\text{lint}} \in [0, 1]$ measures zero new linter warnings, and $M_{\text{diff}} \in [0, 1]$ rewards minimal surgical line modifications. Delivery requires $I_{\text{conf}} \ge 0.90$.

### 3. Thresholding & Refusal Decision Criteria

OpenCode Autonomous Software Engineering Harness enforces strict operational safety and integrity boundaries:
- **Refusal of Destructive Host Terminal Commands**: Arbitrary host system shell commands (`rm -rf /`, formatting drives, killing unrelated processes) are deterministically rejected with code `ERR_DESTRUCTIVE_COMMAND_PROHIBITED`.
- **Refusal to Commit Hardcoded Credentials**: Patches introducing plaintext API tokens, private SSH keys, or database credentials trigger immediate rejection (`ERR_CREDENTIAL_LEAK_PREVENTED`).
- **Turn Ceiling Enforcement**: Coding iterations enforce a hard ceiling of `max_turns: 25` to eliminate runaway token exhaustion (`WARN_TURN_BUDGET_REACHED`).
- **Local Repository Isolation**: File modifications are restricted strictly to the workspace directory; modifications targeting system root or parent directories are blocked (`ERR_UNAUTHORIZED_DIRECTORY_TRAVERSAL`).

### 4. Fallback Decision Mechanism

Continuous engineering problem-solving is maintained through multi-tier fault recovery:
- **Model Cascade Failover**: When the primary foundation model experiences latency spikes or HTTP 429 rate limits, the orchestrator cascades automatically between `claude-3-5-sonnet`, `gpt-4o`, and `gemini-2.0-flash`.
- **Deterministic Static AST Fallback**: If LLM-guided exploration fails to localize a defect, the system falls back to static AST call-hierarchy trees and git-blame history.
- **Graceful Patch Reversion**: If a synthesized patch introduces regression failures that cannot be resolved within 3 turns, the agent automatically reverts the diff to a clean working state.

### 5. Human-in-the-Loop Governance

Human software engineers retain full oversight and final commit authority:
- **Explicit Operator Approval Gates**: Creating git commits, pushing branches, or modifying build configurations requires explicit human confirmation.
- **Immediate Execution Cancellation**: Operators can halt agent execution loops instantly via `Ctrl+C` interrupt signals.
- **Inspectable Unified Diff Review**: Every file modification is formatted as a standard git diff for line-by-line developer review before disk application.

---

## The Data It Uses

OpenCode operates under strict privacy, data minimization, and local workspace isolation standards.

### 1. Ingested Input Data

The agent processes only operational assets necessary to fulfill software engineering tasks:
- **Task Directives**: Natural language instructions, issue tickets, and refactoring goals.
- **Repository Files**: Local source code files, unit test suites, and package dependency manifests.
- **Terminal Execution Outputs**: Standard output and error streams generated by compiler runs and test suites.

### 2. Configuration & Reference Data

- **Harness Tool Specs**: Command syntax definitions for ripgrep search, AST parsing, and line replacement editors.
- **Language Linter Schemas**: Pre-configured rulesets for flake8, black, ESLint, TypeScript, and pytest.
- **Workspace Toolchain State**: Read-only detection of package managers and language runtimes installed locally.

### 3. Base Model & Inference Lineage

- **Deterministic Algorithmic Engines**: Ripgrep symbol indexing, AST parsing, subprocess sandbox execution, and git diff formatters executed natively (100% deterministic with zero LLM variance).
- **Foundation LLMs**: High-capability frontier models (`claude-3-5-sonnet`, `gpt-4o`, `gemini-2.0-flash`) utilized for complex code reasoning, traceback deduction, and surgical patch generation.
- **Zero Training on Proprietary Codebases**: User source code, local repositories, and debugging transcripts are never transmitted to external cloud training corpora.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Systematically protected against prompt injection, insecure output handling, and excessive authority.
- **Local-Only Working Storage**: All execution traces, step summaries, and diff artifacts reside exclusively on the user's filesystem.
- **Automated PII & Secret Scrubbing**: Environment variables, authentication keys, and user credentials are scrubbed from generation logs.
- **Zero Commercial Monetization**: Developer repositories, issue descriptions, and patch histories are never monetized, aggregated, or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of OpenCode is essential for effective engineering deployment.

### 1. Large-Scale Multi-Repo Distributed Refactors
- **Limitation**: While highly proficient at surgical fixes within a single repository, coordinating simultaneous cross-repo architectural refactors exceeds single-session bounds.
- **Mitigation**: The agent focuses on intra-repo modularity and outputs standardized interface contracts for dependent repositories.

### 2. Graphical UI and Visual Rendering Defects
- **Limitation**: The agent operates on headless code, ASTs, and terminal outputs, lacking visual rendering pipelines to detect subtle CSS pixel misalignment.
- **Mitigation**: OpenCode relies on DOM snapshot assertions, visual regression test suites, and human developer review for visual interface polish.

### 3. Proprietary Binary Dependencies & Custom Hardware
- **Limitation**: Codebases dependent on closed-source binary drivers or proprietary hardware accelerators cannot be executed in standard headless test environments.
- **Mitigation**: The agent authors isolated mock interfaces and stubs to test algorithmic logic independently of physical hardware.

### 4. Flaky and Non-Deterministic Test Suites
- **Limitation**: Test suites exhibiting intermittent timing failures or network flakiness can provide confusing signals to automated patch validation loops.
- **Mitigation**: OpenCode runs regression tests multiple times and isolates known flaky test cases from patch verification criteria.

### 5. Ambiguous or Contradictory Requirements
- **Limitation**: Bug reports with contradictory descriptions can lead the agent to explore multiple plausible but mutually exclusive hypotheses.
- **Mitigation**: The agent explicitly lists conflicting interpretations and pauses execution to request developer clarification.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & patch verification formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested task directives, repository files & terminal logs | Section 1 | Verified |
| - Configuration, tool specs & linter schemas | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Large-scale multi-repo distributed refactors | Section 1 | Verified |
| - Graphical UI and visual rendering defects | Section 2 | Verified |
| - Proprietary binary dependencies & custom hardware | Section 3 | Verified |
| - Flaky and non-deterministic test suites | Section 4 | Verified |
| - Ambiguous or contradictory requirements | Section 5 | Verified |
