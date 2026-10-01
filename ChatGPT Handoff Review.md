# ChatGPT → Claude Code Project Handoff

> **Purpose:** This file is the durable bridge between the architecture/design discussions in ChatGPT and implementation work in Claude Code.
>
> Claude should treat this as the current architectural handoff, together with `CLAUDE.md`, `CONTEXT.md`, `RESEARCH_METHODOLOGY.md`, and task-specific documents.

---

# 1. Project Identity

The project is based on an existing LLVM compiler-optimization research framework (`compiler-opt`) built around a Multi-Agent Hierarchical Reinforcement Learning (MA-HRL) optimizer.

The intended evolution is **not** simply:

> "Put an IDE around the existing optimizer."

The intended direction is a **fully local agentic AI-assisted compiler environment** in which multiple specialized intelligent components cooperate through a Supervisor.

The student-facing system should allow a student to enter C/C++ code and receive:

- compiler diagnostics,
- code-analysis findings,
- security-related findings,
- optimization results,
- verification evidence,
- understandable explanations.

The existing MA-HRL optimization system remains an important research component and should be reused as the **Optimization Agent**, rather than being discarded or unnecessarily replaced.

---

# 2. Core Design Goal

The compiler should behave as an **agentic system**, not merely as a linear collection of scripts.

The key property is the Supervisor's ability to:

```text
OBSERVE
   ↓
REASON
   ↓
DELEGATE
   ↓
ACT
   ↓
VERIFY
   ↓
REPLAN / FINISH
   ↓
EXPLAIN
```

This should be implemented as a feedback loop around compiler and analysis tools.

---

# 3. Target High-Level Architecture

```text
                    ┌─────────────────────┐
                    │       Student       │
                    │      C / C++        │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Supervisor Agent  │
                    │                     │
                    │ Observe             │
                    │ Decide              │
                    │ Delegate            │
                    │ Verify              │
                    │ Re-plan             │
                    └──────────┬──────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
 ┌─────────────────┐  ┌─────────────────┐  ┌──────────────────┐
 │ Code Analysis   │  │ Security Agent  │  │ Optimization     │
 │ Agent           │  │                 │  │ Agent            │
 │                 │  │ Static analysis │  │ Existing MA-HRL  │
 │ Errors          │  │ warnings        │  │ GNN              │
 │ Warnings        │  │ unsafe patterns │  │ World Model      │
 │ Structure       │  │ sanitizer data  │  │ HRL              │
 │ Quality         │  │ evidence        │  │ LLVM passes      │
 └────────┬────────┘  └────────┬────────┘  └────────┬─────────┘
          │                    │                    │
          └────────────────────┼────────────────────┘
                               ▼
                    ┌─────────────────────┐
                    │      Clang/LLVM     │
                    │                     │
                    │ Compile → IR        │
                    │ Optimize → Execute  │
                    │ Measure             │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Verification Agent  │
                    │                     │
                    │ Compilation        │
                    │ Correctness evidence│
                    │ Metrics             │
                    │ Safety checks       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Explanation Layer   │
                    │                     │
                    │ What happened?      │
                    │ Why?                │
                    │ What is the impact? │
                    └─────────────────────┘
```

---

# 4. Existing MA-HRL System

The existing research framework includes a hierarchical architecture with components such as:

- `CompilerOptEnv`
- GNN-based program representation/perception
- a world model
- a hierarchical reinforcement-learning manager
- performance/size/security-oriented specialist policies
- a tactical worker
- LLVM pass execution

The existing optimizer should be treated as the **Optimization Agent** inside the larger agentic compiler.

Do not casually redesign or retrain the existing model stack while implementing the orchestration layer.

First understand the existing implementation and determine the smallest integration surface.

---

# 5. What the New System Adds

The new work is primarily an orchestration and compiler-environment layer around the existing research system.

Major additions/concepts:

1. **Supervisor Agent**
2. **Code Analysis Agent**
3. **Security Agent**
4. **Optimization Agent integration**
5. **Verification Agent**
6. **Structured ProgramState**
7. **Local compile/analysis interface**
8. **Student-facing explanation**
9. **Real measured results**
10. **Cross-platform compiler/tool discovery**
11. **Improved C/C++ handling**

---

# 6. Supervisor Agent

## Responsibility

The Supervisor is the central orchestration layer.

It does not directly perform every analysis and does not itself replace the compiler or the optimization engine.

It should:

- receive the student's source program,
- inspect current program state,
- decide which specialist should act,
- invoke the appropriate tool/agent,
- collect structured results,
- decide whether another action is necessary,
- trigger verification,
- stop or re-plan when verification fails,
- produce a final student-facing result.

## Initial Implementation

The Supervisor should initially be **deterministic/rule-driven**.

An LLM is not required for the first implementation.

Reason:

- compiler workflows have explicit conditions,
- safety gates should be predictable,
- local/offline operation is required,
- deterministic orchestration is easier to test,
- the architecture can later permit a local model for higher-level reasoning without making the whole system dependent on it.

## Example Decision Flow

```text
Receive source
    ↓
Compile / inspect
    ↓
Compilation failed?
    ├── Yes → Code Analysis Agent
    │             ↓
    │         explain diagnostics
    │             ↓
    │         return to Supervisor
    │
    └── No
         ↓
Security findings?
    ├── High severity → Security Agent
    │                     ↓
    │                 explain findings
    │                     ↓
    │                 stop / require attention
    │
    └── No critical issue
         ↓
Optimization appropriate?
    ├── Yes → Optimization Agent
    │            ↓
    │       collect measurements
    │
    └── No
         ↓
Verification Agent
    ↓
Verification successful?
    ├── No → reject / re-plan
    └── Yes → explanation/result
```

This is an example of the intended architecture. Exact policy thresholds should be defined in implementation documents and tests rather than hardcoded based on this handoff alone.

---

# 7. ProgramState

Agents should communicate through a structured state object rather than arbitrary text.

The Supervisor should maintain something conceptually like:

```text
ProgramState
├── source
├── language
├── compilation_status
├── compiler_diagnostics
├── code_findings
├── security_findings
├── ir_available
├── baseline_metrics
├── optimization_status
├── optimization_results
├── verification_status
├── verification_findings
└── explanation
```

The exact schema can evolve after the existing codebase is inspected.

The important rule is that the state is explicit, inspectable, and testable.

---

# 8. Agent Contracts

Each agent should have:

1. defined inputs,
2. defined tools/models,
3. defined outputs,
4. defined failure behavior.

Agents should return structured evidence.

Avoid designs where one agent emits an unstructured paragraph that another agent must parse.

---

# 9. Code Analysis Agent

## Responsibility

The Code Analysis Agent should help explain:

- compilation errors,
- warnings,
- code-structure problems,
- code-quality observations,
- relevant compiler diagnostics.

Potential evidence sources include local compiler output and static-analysis information.

## Constraint

**The Code Analysis Agent must not rewrite the student's source code.**

The intended role is:

```text
diagnose → reason → explain
```

not:

```text
diagnose → silently edit source
```

The student should retain control over source changes.

---

# 10. Security Agent

## Responsibility

The Security Agent should analyze security-related evidence using local tooling.

Possible evidence sources include:

- compiler diagnostics,
- static-analysis findings,
- unsafe function/pattern checks,
- sanitizer output where supported,
- relevant IR/optimization information.

The agent should report:

```text
Finding
Severity
Evidence
Explanation
```

It must not claim that a program is "completely secure."

Security output should be evidence-based.

---

# 11. Optimization Agent

The Optimization Agent is the existing MA-HRL optimization system.

It should operate on LLVM IR and use the existing research pipeline where possible.

Its output should include information such as:

- selected optimization actions/passes,
- optimization trace,
- baseline metrics,
- optimized metrics,
- observed changes,
- relevant model information where available.

Optimization claims must be based on **actual local measurements**.

No fabricated demo metrics.

---

# 12. Verification Agent

The Verification Agent is a gate after meaningful compiler/optimization actions.

It should check evidence such as:

- compilation success,
- correctness-related checks,
- baseline vs optimized measurements,
- available safety checks,
- optimizer execution status.

A successful model decision is not equivalent to successful verification.

The Supervisor should be able to reject a result or re-plan when verification fails.

---

# 13. Explanation Layer

The system is intended to be usable by students, including beginners.

Therefore the final presentation should translate technical compiler output into understandable explanations.

Example:

```text
Compiler:
    use of undeclared identifier 'x'

Student-facing explanation:
    The compiler cannot find a declaration for x before it is used.
    Add a declaration or check whether the variable name is correct.
```

The UI can have two levels:

```text
Beginner view
Technical view
```

The technical view may expose LLVM/compiler terminology when useful.

---

# 14. Not Every Component Needs an AI Model

A major design principle from the architecture discussion:

**Do not put an LLM everywhere.**

Use the strongest local tool for each task.

```text
Clang
    → compilation and diagnostics

LLVM
    → IR generation and optimization infrastructure

Static analysis / sanitizers
    → security and correctness evidence

MA-HRL / GNN / world model / RL
    → optimization decisions

Supervisor
    → orchestration and control logic

Local language model
    → optional natural-language reasoning where it provides real value
```

"Agentic" does not mean every component must be a neural network.

---

# 15. Local/Offline Requirements

These are hard project constraints.

The system should be:

- fully local,
- offline-capable,
- independent of cloud APIs,
- free of telemetry,
- free of hidden network dependencies.

Do not add cloud LLM calls, cloud inference, or online services as required dependencies.

A local model can be considered later, but it must not violate the offline architecture.

---

# 16. Source-Rewriting Constraint

The project currently has a deliberate non-goal:

**Do not implement automatic source-code rewriting.**

The system may:

- diagnose,
- explain,
- compare,
- recommend what to inspect,
- optimize compiler IR using the existing optimization pipeline.

It should not automatically rewrite the student's C/C++ source as part of the core workflow.

---

# 17. C/C++ Scope

Initial source-language scope:

- `.c`
- `.cpp`
- `.cc`
- `.cxx`

C++ compilation/linking should use the appropriate local compiler driver (`clang++`) when required.

The implementation should avoid assumptions that only one file extension or one compiler invocation is valid.

---

# 18. Existing Compiler Environment Work

Important engineering work already identified includes:

- removing hardcoded Windows LLVM paths,
- supporting compiler discovery through environment variables / PATH,
- robust C++ compilation and linking,
- improving portability,
- replacing fixed/demo output with measured output.

Before changing these areas, inspect the current implementation and tests.

---

# 19. Local Compile Interface

A local compile API is part of the intended architecture.

Conceptual interface:

```http
POST /compile
```

Conceptual input:

```json
{
  "language": "cpp",
  "source": "..."
}
```

Conceptual response should be structured and may include:

- compiler status,
- diagnostics,
- baseline O0/O2/O3 information,
- optimization results,
- metrics,
- findings,
- explanation,
- verification status.

The exact schema should be finalized after inspecting the current backend/frontend structure.

---

# 20. Existing Helper Components

The design discussion included:

- Explainer
- Comparer
- Coach
- Verdict

These may be retained as presentation/analysis services.

They should not automatically be treated as four additional autonomous neural agents.

A useful distinction is:

```text
Core agents:
- Supervisor
- Code Analysis
- Security
- Optimization
- Verification

Presentation/analysis services:
- Explainer
- Comparer
- Coach
- Verdict
```

The implementation can change this classification later if there is a strong architectural reason.

---

# 21. Important Existing-Project Context

The repository already contains project/research material, including:

- `CONTEXT.md`
- `RESEARCH_METHODOLOGY.md`
- `README.md`
- `.claude/`
- benchmarks/data/models/logs/results
- source and scripts
- report/research material
- a TUI

Do **not** create unnecessary duplicates of this information.

Before creating new documentation:

1. inspect existing docs,
2. identify what already exists,
3. preserve useful existing material,
4. add only the minimum new documentation required,
5. avoid contradictory sources of truth.

---

# 22. Documentation Architecture

Recommended project memory:

```text
CLAUDE.md
    ↓
Permanent rules + identity

CONTEXT.md
    ↓
Existing project/codebase context

RESEARCH_METHODOLOGY.md
    ↓
Existing optimizer research/methodology

docs/CHATGPT_HANDOFF.md
    ↓
Current architecture bridge from design discussions

docs/ARCHITECTURE.md
    ↓
Canonical system architecture

docs/AGENTS.md
    ↓
Agent contracts

docs/SUPERVISOR.md
    ↓
Supervisor behavior

docs/OPTIMIZATION_AGENT.md
    ↓
MA-HRL integration

docs/SECURITY_AGENT.md
    ↓
Security architecture

docs/VERIFICATION_AGENT.md
    ↓
Verification architecture

docs/CURRENT_TASK.md
    ↓
Only the task currently being implemented

docs/SESSION_HANDOFF.md
    ↓
What was done in the previous Claude session

docs/DECISIONS.md
    ↓
Architecture decisions that should not be casually reversed

docs/KNOWN_ISSUES.md
    ↓
Known technical problems and gaps

docs/ROADMAP.md
    ↓
Future work
```

The files should not duplicate one another.

---

# 23. Context/Token Strategy

Claude should **not** reread the entire project and conversation for every task.

Preferred context-loading flow:

```text
CLAUDE.md
   ↓
CURRENT_TASK.md
   ↓
Relevant architecture document(s)
   ↓
Relevant source files
   ↓
Relevant tests
```

For example, security work should not require loading the entire ML/research methodology.

This layered documentation strategy is intended to reduce context waste while preserving project continuity.

---

# 24. Claude Code Workflow

For each meaningful feature:

```text
1. Read CLAUDE.md
2. Read CURRENT_TASK.md
3. Read only relevant architecture docs
4. Inspect existing implementation
5. Identify integration points
6. Create a plan
7. Implement incrementally
8. Add/update tests
9. Run tests
10. Debug failures systematically
11. Review the changes
12. Update docs
13. Update SESSION_HANDOFF.md
```

Use the project's existing Claude/Superpowers workflows where applicable, especially:

- brainstorming,
- writing plans,
- executing plans,
- test-driven development,
- systematic debugging,
- verification-before-completion,
- code review,
- isolated worktrees where appropriate.

---

# 25. Git as Project Memory

Git should be the source of truth for project state and architectural evolution.

Use Git to preserve:

- code,
- architecture docs,
- decisions,
- tests,
- implementation history.

Do not rely on an external notes application as the only record of important engineering decisions.

Obsidian is optional and can remain a personal research knowledge base, but it should not be required for Claude to understand this repository.

---

# 26. Architectural Principles

The implementation should follow these principles:

### Reuse before replacement

Understand existing components and reuse them where appropriate.

### Evidence before claims

Do not claim correctness, security, performance improvement, or successful optimization without corresponding evidence.

### Structured communication

Agents communicate through defined data contracts.

### Local first

All required behavior should work locally.

### Human control

The student remains in control of source changes.

### Small, testable steps

New orchestration should be introduced incrementally rather than as one giant rewrite.

### Clear separation of responsibilities

The Supervisor coordinates; specialists analyze/decide within their domains; Clang/LLVM performs compilation and optimization infrastructure; Verification checks outcomes.

---

# 27. Known Conceptual Non-Goals

The project should not drift into:

- a generic AI coding tutor,
- a source-rewriting copilot,
- a cloud AI service,
- an "LLM everywhere" architecture,
- a replacement for the existing MA-HRL research system,
- a collection of disconnected AI buttons with no orchestration loop.

The differentiating feature is **agentic compiler orchestration around real compiler tooling and the existing learned optimization engine**.

---

# 28. Suggested Implementation Order

The exact order can change after code inspection, but a sensible implementation progression is:

```text
Phase 1
Understand and stabilize existing compiler/tool discovery

Phase 2
Robust C/C++ local compilation + diagnostics

Phase 3
Define ProgramState and agent interfaces

Phase 4
Implement deterministic Supervisor

Phase 5
Integrate Code Analysis Agent

Phase 6
Integrate Security Agent

Phase 7
Integrate existing MA-HRL as Optimization Agent

Phase 8
Implement Verification Agent / gates

Phase 9
Implement local compile/API and result aggregation

Phase 10
Student-facing IDE/result presentation

Phase 11
Improve explanation layer

Phase 12
Optional local-model reasoning where it adds real value
```

Do not jump directly to a large UI or LLM integration before the orchestration and contracts are sound.

---

# 29. Definition of "Agent"

For this project, use a practical definition:

> An agent is a specialized component that receives a defined state/input, uses defined tools/models, makes or supports a domain-specific decision, and returns structured output that another component can consume.

A component does not need to be a large language model to qualify as an agent.

The Supervisor's decision loop is what creates the system-level agentic behavior.

---

# 30. Implementation Discipline

Before making architectural changes, Claude should answer:

- What already exists?
- Which existing component owns this responsibility?
- What new interface is required?
- What is the smallest safe change?
- How will the behavior be tested?
- What evidence will prove it works?

Avoid speculative rewrites.

Never report "implemented" merely because files were edited.

The final report for a task should distinguish:

```text
Implemented
Tested
Observed
Not yet verified
Known limitation
```

---

# 31. Current Handoff Instruction to Claude

When starting work from this handoff, begin with:

```text
Read:
- CLAUDE.md
- docs/CHATGPT_HANDOFF.md
- CONTEXT.md
- RESEARCH_METHODOLOGY.md
- docs/CURRENT_TASK.md (when present)

Then inspect the existing implementation.

Do not implement yet.

First explain:
1. How the current repository maps onto this architecture.
2. Which components already exist.
3. Which components are missing.
4. Which assumptions in this handoff require confirmation from the code.
5. The smallest implementation plan for the current task.

Do not invent APIs, trained-model behavior, or benchmark results.
Do not rewrite source code.
Do not introduce cloud dependencies.
```

This is the starting bridge between architecture work and Claude Code implementation.

---

# 32. Source of Truth Policy

When documents disagree:

```text
Current code + tests
        ↓
accepted architecture decisions
        ↓
current task
        ↓
general suggestions
```

However, if the current code contradicts an accepted architecture decision, **do not silently "fix" the architecture**.

Raise the discrepancy and ask for a deliberate decision.

---

# 33. Final Project Vision

The end goal is a local compiler environment where a student can enter ordinary C/C++ code and the system can:

```text
Understand the program state
        ↓
Diagnose compiler/code issues
        ↓
Inspect security evidence
        ↓
Decide whether optimization is appropriate
        ↓
Use the existing learned MA-HRL optimizer
        ↓
Measure actual results
        ↓
Verify the outcome
        ↓
Explain what happened
        ↓
Present understandable results to the student
```

The software should feel like an **intelligent compiler system**, while remaining grounded in real compiler behavior, measurable results, explicit agent contracts, and local/offline execution.

---

# 34. Important Note

This document is a distilled architectural handoff from design discussions. It is **not a substitute for inspecting the repository**.

Before implementing any claim in this document, Claude should verify:

- the corresponding existing code,
- the current model interfaces,
- the available compiler/toolchain,
- the current test coverage,
- platform-specific assumptions,
- and the actual runtime behavior.

Where the repository differs from this handoff, stop and surface the discrepancy before making a large architectural change.
