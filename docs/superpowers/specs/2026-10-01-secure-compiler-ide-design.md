# Canonical Architecture Specification: Secure Compiler IDE

> **Document Version:** 1.0.0 (Frozen Canonical Design)  
> **Date:** 2026-10-01  
> **Status:** APPROVED FOR IMPLEMENTATION  
> **Target System:** Fully Local, Offline, Agentic AI-Assisted Desktop Compiler IDE  

---

## 1. System Vision & Architecture Overview

The **Secure Compiler IDE** is an intelligent, 100% offline desktop development environment for C and C++ programs. It combines real local compiler infrastructure (Clang, LLVM `opt`, `llc`, and static analyzers) with a learned Multi-Agent Hierarchical Reinforcement Learning (MA-HRL) optimization engine under the orchestration of a deterministic **Supervisor Agent**.

The primary product deliverable is a **standalone desktop IDE** providing a familiar folder, project, and code editor workflow (analogous to VS Code / Cursor) with integrated diagnostic sidecars, visual security auditing, live optimization metrics, and dual-level explanations. The core compilation and agent orchestration engine runs locally as an embedded background service (communicating via local loopback IPC/REST on `127.0.0.1`).

```text
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                STANDALONE DESKTOP IDE                                  │
│ ┌──────────────────────┬───────────────────────────────┬─────────────────────────────┐ │
│ │  Project / File Tree │      C / C++ Code Editor      │   Interactive Sidecar Pane  │ │
│ │  - src/main.cpp      │  - Syntax Highlighting        │   - Beginner / Tech Expl.   │ │
│ │  - include/utils.h   │  - Gutter Diagnostics         │   - Security Vulnerabilities│ │
│ │  - Makefile          │  - Read-Only User Control     │   - Delta Comparison Table  │ │
│ └──────────────────────┴───────────────┬───────────────┴─────────────────────────────┘ │
└────────────────────────────────────────┼───────────────────────────────────────────────┘
                                         │ Local IPC / JSON-RPC / REST (127.0.0.1)
                                         ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                               SUPERVISOR AGENT ENGINE                                  │
│             (Observe → Decide → Delegate → Act → Verify → Replan / Finish)             │
│                                                                                        │
│   ┌────────────────────┬────────────────────┬────────────────────┬─────────────────┐   │
│   │   Code Analysis    │   Security Agent   │ Optimization Agent │  Verification   │   │
│   │       Agent        │                    │  (MA-HRL + V8.5)   │      Agent      │   │
│   │                    │                    │                    │                 │   │
│   │ Clang Diagnostics  │ clang --analyze    │ 41 Atomic Passes   │ opt -verify     │   │
│   │ Parser & Structural│ AST Pattern Scanner│ 26 Macro Actions   │ llc codegen     │   │
│   │ Complexity (IR)    │ Security Policy    │ 20 Micro Refiners  │ Smoke & CI95    │   │
│   └─────────┬──────────┴─────────┬──────────┴─────────┬──────────┴────────┬────────┘   │
│             │                    │                    │                   │            │
│             └────────────────────┴──────────┬─────────┴───────────────────┘            │
│                                             ▼                                          │
│                             ┌───────────────────────────────┐                          │
│                             │   Lightweight ProgramState    │                          │
│                             │               +               │                          │
│                             │  Append-Only Transition Log   │                          │
│                             └───────────────┬───────────────┘                          │
│                                             ▼                                          │
│                             ┌───────────────────────────────┐                          │
│                             │ Ephemeral Workspace Artifacts │                          │
│                             │ (.compiler_workspace/session) │                          │
│                             └───────────────────────────────┘                          │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Hard Constraints & Non-Goals

### 2.1 Hard Constraints
1. **100% Local and Offline:** Zero cloud API dependencies, zero external network calls, zero remote telemetry. All compilation, analysis, GNN perception, world-model inference, and verification must run entirely on the local host machine.
2. **No Source-Code Rewriting:** The system **must never automatically mutate or rewrite the student's source code**. Control over source edits remains exclusively with the student. All optimizations are performed on intermediate representations (LLVM IR).
3. **C / C++ Language Scope:** First-class support for `.c`, `.cpp`, `.cc`, and `.cxx` files with appropriate toolchain driver routing (`clang` vs `clang++`) and standard library linkage.
4. **Empirical Evidence Only:** No fabricated metrics or unverified claims. Performance speedups, binary size deltas, and security findings must be backed by real measurements and tool output.
5. **Preservation of MA-HRL Research Stack:** The existing GNN perception, V8.5 World Model (with Residual Scale Correction), Hurdle Meta-Calibrator, and Hierarchical Multi-Agent policy are preserved and integrated as the **Optimization Agent**.

### 2.2 Non-Goals
- Not a generic cloud AI coding chatbot or copilot.
- Not an automated source refactoring or code synthesis engine.
- Not a replacement or ground-up rewrite of the existing MA-HRL reinforcement learning framework.
- Not an "LLM-everywhere" architecture: deterministic algorithms, Clang tooling, and specialized PyTorch neural modules are used where they are strongest.

---

## 3. Section 1: Supervisor Agent, ProgramState & Toolchain Architecture

### 3.1 Supervisor Agent
The Supervisor Agent is the central orchestrator. It is initially implemented as a **deterministic, rule-driven feedback controller** executing an explicit feedback loop:

$$\text{OBSERVE} \longrightarrow \text{DECIDE} \longrightarrow \text{DELEGATE} \longrightarrow \text{ACT} \longrightarrow \text{VERIFY} \longrightarrow \text{REPLAN or FINISH}$$

```text
                 ┌────────────────────────────────────────────────────────┐
                 │                                                        │
                 ▼                                                        │
              [OBSERVE] ◄─── (Inspect source path, diagnostics, metrics)  │
                 │                                                        │
                 ▼                                                        │
              [DECIDE] ◄──── (Evaluate policy gates: compile status,      │
                 │            security verdict, budget, replan count)     │
                 ▼                                                        │
             [DELEGATE] ◄─── (Select Specialist Agent / Tool)             │
                 │                                                        │
                 ▼                                                        │
               [ACT] ◄────── (Invoke Clang, Security Scanner, or MA-HRL)  │
                 │                                                        │
                 ▼                                                        │
              [VERIFY] ◄──── (Collect empirical evidence: IR validity,    │
                 │            binary linking, smoke run, noise checks)    │
                 │                                                        │
        ┌────────┴────────┐                                               │
        ▼                 ▼                                               │
[Evidence Passes]  [Evidence Fails]                                       │
        │                 │                                               │
        │                 ▼                                               │
        │             [REPLAN]                                            │
        │       (Restore last known-good IR, penalize pass,               │
        │        decrement budget, update episode constraints)            │
        │                 │                                               │
        │                 └───────────────────────────────────────────────┘
        ▼
   [FINISH] ───► [EXPLAIN] (Render dual-view presentation & verdict)
```

#### Policy-Driven Decision Logic
1. **Compilation Gate:** If `clang` syntax/semantic compilation fails, the Supervisor halts optimization, delegates to the *Code Analysis Agent* to generate error explanations, and transitions directly to `FINISH`.
2. **Security Policy Gate:** Evaluates security findings through a configurable policy function:
   $$f(\text{severity}, \text{confidence}, \text{evidence\_source})$$
   - High-confidence critical memory vulnerabilities $\to$ block optimization or restrict to conservative safe passes.
   - Low/medium warnings $\to$ proceed with annotated security remarks.
3. **Verification Failure & Replan Protocol:** If the candidate IR fails verification (corrupt IR, linker error, non-zero smoke exit code, or $>3\%$ metric regression outside $CI_{95}$ noise floor):
   - Rollback `current_ir_path` to `last_known_good_ir_path`.
   - Record the failed action in an episode constraint/blacklist.
   - Increment `replan_count` (budget: max 3 replans).
   - Re-enter `DECIDE` to select an alternative pass sequence or terminate gracefully.

---

### 3.2 Lightweight ProgramState & StateTransitionLog
To maintain high responsiveness and avoid memory bloat, `ProgramState` **does not store large IR strings**. All intermediate representations, assembly outputs, and binaries live in an ephemeral workspace directory on disk.

```python
from dataclasses import dataclass, field
from pathlib import Path
from typing import List, Dict, Optional, Any, Literal, Tuple
import time
import uuid

@dataclass
class DiagnosticEntry:
    """Parsed Clang compiler diagnostic."""
    severity: Literal["error", "warning", "note", "remark"]
    file: str
    line: int
    column: int
    message: str
    code: Optional[str] = None
    suggested_fix: Optional[str] = None  # Explanatory suggestion; no auto-mutation

@dataclass
class SecurityFinding:
    """Static security finding with provenance."""
    check_id: str
    title: str
    severity: Literal["critical", "high", "medium", "low", "info"]
    confidence: Literal["high", "medium", "low"]
    evidence_source: Literal["clang-static-analyzer", "pattern-scanner", "clang-warning", "sanitizer"]
    cwe: Optional[str]
    description: str
    location: Optional[Tuple[int, int]]  # (line, column)
    evidence: str

@dataclass
class OptimizationStep:
    """Record of a single pass sequence application."""
    action_type: Literal["atomic", "macro"]
    action_id: int
    pass_sequence: List[str]
    predicted_delta: Dict[str, float]  # V8.5 World Model predictions
    measured_delta: Dict[str, float]   # Real local measurements
    gate_decision: bool                # Hurdle Meta-Calibrator state

@dataclass
class VerificationEvidence:
    """Empirical verification evidence collected post-action."""
    ir_syntactically_valid: bool
    assembly_succeeded: bool
    binary_compiles: bool
    smoke_test_executed: bool = False
    smoke_test_exit_code: Optional[int] = None
    smoke_test_output_matched: Optional[bool] = None
    metric_regression_detected: bool = False
    measured_speedup: Optional[float] = None
    evidence_notes: List[str] = field(default_factory=list)
    passed: bool = False

@dataclass
class StateTransition:
    """Immutable audit record of a single step in the Supervisor loop."""
    transition_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    timestamp: float = field(default_factory=time.time)
    stage: Literal["observe", "decide", "delegate", "act", "verify", "replan", "finish"]
    agent: Optional[str] = None
    action_taken: Optional[str] = None
    evidence_summary: Optional[str] = None
    state_delta: Dict[str, Any] = field(default_factory=dict)

@dataclass
class ProgramState:
    """Lightweight mutable working state tracking workspace artifact references."""
    session_id: str
    source_path: Path
    language: Literal["c", "cpp"]
    
    # Workspace & Artifact References
    workspace_dir: Path
    baseline_ir_path: Optional[Path] = None
    current_ir_path: Optional[Path] = None
    last_known_good_ir_path: Optional[Path] = None
    candidate_binary_path: Optional[Path] = None
    
    # Toolchain & Compilation
    compilation_succeeded: bool = False
    diagnostics: List[DiagnosticEntry] = field(default_factory=list)
    security_findings: List[SecurityFinding] = field(default_factory=list)
    
    # Real Empirical Metrics
    baseline_metrics: Dict[str, float] = field(default_factory=dict)
    current_metrics: Dict[str, float] = field(default_factory=dict)
    optimization_trace: List[OptimizationStep] = field(default_factory=list)
    
    # Verification & Replan State
    latest_verification: Optional[VerificationEvidence] = None
    replan_count: int = 0
    max_replan_budget: int = 3
    
    # Append-Only Audit History
    transition_log: List[StateTransition] = field(default_factory=list)

    def log_transition(self, stage: str, agent: Optional[str], action: Optional[str], 
                       evidence: Optional[str], delta: Dict[str, Any]) -> None:
        self.transition_log.append(StateTransition(
            stage=stage,
            agent=agent,
            action_taken=action,
            evidence_summary=evidence,
            state_delta=delta
        ))
```

---

### 3.3 Decoupled Toolchain Architecture
The toolchain layer is strictly separated into two distinct components:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                           TOOLCHAIN RESOLVER                           │
│  (Discovers and validates binary paths across 4 fallback layers)       │
│                                                                        │
│  Tier 1: Explicit Configuration (User config file / UI settings)       │
│      │ (fallback)                                                      │
│  Tier 2: Environment Variables ($LLVM_HOME, $LLVM_DIR, $CLANG_PATH)    │
│      │ (fallback)                                                      │
│  Tier 3: System PATH Resolution (shutil.which for clang, opt, llc)     │
│      │ (fallback)                                                      │
│  Tier 4: Platform Fallbacks (macOS Homebrew, Linux /usr/lib, Win MSYS2)│
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Returns ResolvedToolchain dict
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                            COMPILER DRIVER                             │
│  (Invokes tools, manages flags, formats diagnostics, handles stdlib)   │
│                                                                        │
│  - Language Routing: .c -> clang (c17), .cpp/.cc/.cxx -> clang++ (c++20)│
│  - Target Stdlib Linkage: -lc++ (macOS) vs -lstdc++ (Linux/Windows)    │
│  - Emits JSON/structured stderr diagnostics                            │
│  - Compiles to Bitcode / LLVM IR (-S -emit-llvm)                       │
│  - Assembles Object Files (llc) & Links Test Binaries                  │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Section 2: Code Analysis & Security Specialist Agents

### 4.1 Code Analysis Agent (Diagnostic Reasoner)
- **Primary Responsibility:** Parse Clang compiler diagnostics (`-fsyntax-only`) and map raw error/warning output into structured `DiagnosticEntry` objects.
- **Diagnostic Engine:** Implements a dedicated Clang diagnostic parser supporting both `-fdiagnostics-format=json` and traditional regex fallback (`<file>:<line>:<col>: <severity>: <message>`).
- **Structural Complexity Analysis:** Reuses `src/features/ir_parser.py` (`IRParser`, `IRStats`, `ControlFlowGraph`) to calculate cyclomatic complexity, loop nesting depth, basic block counts, and function call graph topology for valid IR.
- **Beginner Translation Layer:** Translates obscure compiler messages into actionable plain-English explanations (e.g. missing semicolons, type mismatches, use of undeclared identifiers, uninitialized variables).
- **Hard Invariant:** Read-only / Explanatory. Never alters source code.

---

### 4.2 Security Agent (Multi-Tier Static Auditor)
- **Primary Responsibility:** Collect evidence-based security findings, tag them with CWE classifications and provenance metadata, and evaluate security risk before optimization.
- **Evidence Collection Tiers:**
  1. **Tier 1 — Unsafe Pattern Scanner:** AST and regex token matching for dangerous legacy C APIs (`gets`, `strcpy`, `sprintf`, `strcat`, `scanf("%s")`).
  2. **Tier 2 — Clang Static Analyzer:** Subprocess execution of `clang --analyze` with Core, Security, and Unix checker suites enabled:
     - Null pointer dereferences (`core.NullDereference`)
     - Buffer overflows and out-of-bounds access (`alpha.security.ArrayBound`)
     - Memory leaks and use-after-free (`unix.Malloc`)
  3. **Tier 3 — Undefined Behavior & Warning Analysis:** Captures compiler remarks regarding signed integer overflow (`-Wstrict-overflow`), implicit conversions (`-Wconversion`), and unaligned pointer casting.
- **Supervisor Policy Gating:** The Supervisor evaluates findings to determine whether to proceed, annotate warnings, or block aggressive optimization.
- **Non-Claim Invariant:** Explicitly reports findings as *static tool observations*. Never issues blanket claims that code is "100% secure".

---

### 4.3 Security Evidence in Optimization Rewards
In reinforcement learning optimization, compiler passes cannot be assumed to be inherently secure or insecure. Instead, security-aware optimization uses **measurable security evidence**:
- **Sanitizer Retention:** Checks whether optimization passes preserve compiler-inserted stack protector canaries (`__stack_chk_fail`) and bounds checks.
- **UB Generation Detection:** Penalizes pass sequences that trigger new undefined behavior remarks or signed overflow warnings during IR transformation.
- **Dead Code Elimination Integrity:** Rewards the clean removal of unreachable basic blocks and dead functions without breaking security invariant assertions.

---

## 5. Section 3: Optimization Agent & Verification Gate

### 5.1 Optimization Agent (MA-HRL Stack Integration)
The Optimization Agent encapsulates the active research stack without breaking or retraining its learned weights:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        OPTIMIZATION AGENT HUB                          │
│                                                                        │
│  1. Foveated Perception (src/features/ir_graph_extractor_v5.py)        │
│     - Dual-fidelity graph: 1:1 hotspot nodes + N:1 HBC background      │
│     - 64 scalar features + 64 GNN embeddings = 128-dim state           │
│                                                                        │
│  2. GNN Backbone (src/models/graph_encoder.py)                         │
│     - GNNEncoderV6 with GATv2 relational attention pooling             │
│                                                                        │
│  3. V8.5 World Model + RSC (src/models/world_model.py)                 │
│     - Two-Hot SymLog metric heads predicting percentage deltas         │
│     - Residual Scale Correction (RSC) gated-affine multiplier/shift    │
│                                                                        │
│  4. Two-Stage Hurdle Meta-Calibrator (src/models/meta_calibrator.py)   │
│     - Gate Classifier: Filters out ~65% no-op passes                   │
│     - Conditional Regressor: Predicts magnitude of active passes       │
│                                                                        │
│  5. Hierarchical Policy Stack (src/models/hrl_agent.py)                │
│     - Strategic Manager: Multi-specialist negotiation lookahead        │
│     - Tactical Worker: Micro-action refinement tuning                  │
│                                                                        │
│  6. Pass Execution Engine (src/passes/pass_executor.py)                │
│     - Applies candidate sequences via local `opt` binary to workspace  │
└────────────────────────────────────────────────────────────────────────┘
```

#### Action Space Taxonomy (Verified from Active Source)
- **41 Atomic LLVM Passes (`LLVM_PASSES` in `src/config.py`):**
  - Loop optimizations: `loop-unroll`, `loop-vectorize`, `loop-rotate`, `licm`, `loop-deletion`, `indvars`, `loop-idiom`, `loop-interchange`, `simple-loop-unswitch`.
  - Vectorization: `slp-vectorizer`.
  - Inlining & IPO: `inline`, `argpromotion`, `ipsccp`, `globalopt`, `tailcallelim`, `callsite-splitting`.
  - Scalar & Memory: `mem2reg`, `sroa`, `instcombine`, `aggressive-instcombine`, `reassociate`, `gvn`, `sccp`, `dce`, `adce`, `dse`, `memcpyopt`, `bdce`.
  - Control Flow: `simplifycfg`, `jump-threading`, `correlated-propagation`, `lower-expect`.
  - Type & Cleanup: `float2int`, `lower-constant-intrinsics`, `early-cse`, `newgvn`, `sink`, `mergereturn`, `constmerge`, `deadargelim`, `strip-dead-prototypes`.
- **26 Macro Actions (`MACRO_ACTIONS` in `src/actions/macro_actions.py`):**
  - Actions 0–24: 25 empirically discovered high-rank n-gram pass pairs (e.g. `[sroa, simplifycfg]`, `[mem2reg, sroa]`).
  - Action 25: Explicit `["TERMINATE"]` action.
- **20 Micro Refinements (`MicroRefiner` in `src/actions/micro_actions.py`):**
  - 0–5: Tactical pass prefix insertion (`dce`, `instcombine`, `simplifycfg`, `early-cse`, `reassociate`, `sccp`).
  - 6–11: Tactical pass suffix insertion.
  - 12–14: Loop unroll factor tuning ($2\times, 4\times, 8\times$).
  - 15–17: Inline threshold tuning ($50, 250, 500$).
  - 18: Sequence reversal.
  - 19: Identity (no-op).

---

### 5.2 Verification Agent (Empirical Evidence Gate)
The Verification Agent does not attempt mathematical formal semantic equivalence proofs; it gathers **concrete empirical evidence** to determine whether candidate IR is sound, compilable, and beneficial.

#### Reused Verification Engine: `src/passes/metrics.py` (`MetricsCollector`)
- **IR Assembly Check:** Runs `opt -passes=verify -S <candidate.ll> -o /dev/null` to ensure pass transformations did not violate LLVM IR well-formedness invariants.
- **Object Lowering & Sizing:** Runs `llc -filetype=obj <candidate.ll>` to measure exact binary object code size in bytes.
- **Executable Linking & Smoke Test:** Invokes `clang <candidate.ll> -o <binary> -lm` and executes the test binary with a strict timeout (default: 5.0s) to confirm zero runtime crashes/segfaults.
- **Noise-Aware Latency Profiling ($CI_{95}$):** Executes 5 timed iterations, filters IQR outliers, calculates median runtime and 95% confidence intervals ($CI_{95}$). Performance speedups are only accepted if the delta exceeds the noise floor ($>3\%$).
- **Output Correctness Comparison:** Runs `MetricsCollector.compare(baseline_ir, opt_ir)` to verify stdout output parity between baseline and optimized builds.

---

## 6. Section 4: Explanation Layer, Desktop IDE & Offline Architecture

### 6.1 Presentation Services & Dual-View Explanations
The explanation system is evidence-grounded and structured via deterministic presentation engines:

1. **Explainer Service:** Translates Clang compiler diagnostics and static analyzer warnings into plain-language diagnostic cards with file, line, and column references.
2. **Comparer Service:** Builds structured comparison tables contrasting baseline ($O_0, O_2, O_3$) against the MA-HRL optimized candidate (instruction count delta %, binary size change %, cycle execution latency).
3. **Coach Service:** Provides educational programming insights explaining *why* certain compiler optimizations occurred (e.g., explaining loop unrolling trade-offs or why struct alignment affected SROA).
4. **Verdict Service:** Generates the final session verdict card: Compilation Status, Security Policy Evaluation, Measured Speedup, and Verification Outcome.

#### Dual-View Presentation Modes
- **Beginner View:** Plain English explanations, visual status indicators (`[PASSED]`, `[WARNING]`, `[BLOCKED]`), and beginner-focused guidance.
- **Technical View:** Raw Clang stderr output, intermediate LLVM IR diff views, GNN foveation metrics, and exact pass pipeline traces.

---

### 6.2 Standalone Desktop IDE Architecture
The primary student-facing deliverable is a standalone desktop application featuring:
- **Project / File Tree Pane:** Workspace folder navigation supporting multi-file C/C++ projects.
- **Editor Pane:** Syntax-highlighted code editor with line numbers, gutter diagnostic markers, and read-only source preservation guarantees.
- **Action Toolbar:** One-click actions: `Compile & Check`, `Audit Security`, `Run Agent Optimization`, and `Verify`.
- **Interactive Sidecar Pane:** Collapsible split-view rendering Beginner/Technical explanations, security finding cards, and optimization delta charts.
- **Internal IPC / REST Backend:** The desktop GUI communicates with the local Python engine via loopback HTTP/IPC (`127.0.0.1:8765`), keeping the UI lightweight and fully decoupled from the compiler/ML processes.

---

## 7. Agent Contracts & Communication Interfaces

| Agent | Input Contract | Tools & Models Used | Output Contract | Failure Handling |
| :--- | :--- | :--- | :--- | :--- |
| **Supervisor Agent** | `source_path`, `session_id`, user goals | Deterministic State Machine, Policy Engine | Updated `ProgramState`, `transition_log`, final verdict | Rolls back to last known-good state; triggers replan or stops with explanation |
| **Code Analysis Agent** | `source_path`, `workspace_dir` | `CompilerDriver` (`clang -fsyntax-only`), `IRParser` | `List[DiagnosticEntry]`, `IRStats`, beginner translations | Reports parse errors; marks `compilation_succeeded = False` |
| **Security Agent** | `source_path`, `current_ir_path` | `CompilerDriver` (`clang --analyze`), AST pattern scanner | `List[SecurityFinding]`, Security Policy verdict | Logs tool execution errors; reports findings with provenance |
| **Optimization Agent** | `current_ir_path`, `workspace_dir` | `HRLAgent`, `WorldModelV8.5`, `MetaCalibrator`, `PassExecutor` | Candidate `.ll` path, predicted deltas, `OptimizationStep` | Emits empty pass sequence; leaves IR unchanged |
| **Verification Agent** | `baseline_ir`, `candidate_ir`, `workspace_dir` | `opt -verify`, `llc`, `MetricsCollector` (`clang`, smoke runner) | `VerificationEvidence` (`passed`, deltas, speedup, $CI_{95}$) | Sets `passed = False` with detailed `rejection_reason` |

---

## 8. Five-Phase Implementation Roadmap

```text
┌────────────────────────────────────────────────────────────────────────────┐
│ Phase 1: Core Foundation, Toolchain & ProgramState                         │
│ - Implement layered ToolchainResolver (Explicit → Env → PATH → Fallback)   │
│ - Implement CompilerDriver (.c/.cpp routing, stdlib flags, IR emission)   │
│ - Implement lightweight ProgramState & append-only StateTransitionLog      │
│ - Implement ephemeral workspace manager (.compiler_workspace/session_<id>) │
│ - Unit test suite with mocked toolchains for zero-dependency CI testing    │
└─────────────────────────────────────┬──────────────────────────────────────┘
                                      │
                                      ▼
┌────────────────────────────────────────────────────────────────────────────┐
│ Phase 2: Code Analysis & Security Specialist Agents                        │
│ - Implement Clang diagnostic parser (JSON & regex formats)                 │
│ - Integrate IRParser for CFG, cyclomatic complexity & instruction counts   │
│ - Implement Security Agent (AST pattern scanner + clang --analyze)        │
│ - Implement Supervisor Security Policy Gating (Proceed / Warn / Block)     │
│ - Unit tests on diagnostic translation and security vulnerability fixtures │
└─────────────────────────────────────┬──────────────────────────────────────┘
                                      │
                                      ▼
┌────────────────────────────────────────────────────────────────────────────┐
│ Phase 3: Optimization Agent & Empirical Verification Gate                  │
│ - Build Optimization Agent adapter wrapping HRLAgent, V8.5 World Model,   │
│   Hurdle MetaCalibrator, and PassExecutor                                  │
│ - Wire PassExecutor to dynamic ToolchainResolver paths                     │
│ - Build Verification Agent encapsulating MetricsCollector (IR verification,│
│   object sizing via llc, smoke testing, and CI95 noise-aware timing)       │
│ - Implement Supervisor Feedback Controller with Replan & Rollback mechanics│
│ - Unit tests on pass application, verification rejection, and replan loop  │
└─────────────────────────────────────┬──────────────────────────────────────┘
                                      │
                                      ▼
┌────────────────────────────────────────────────────────────────────────────┐
│ Phase 4: Explanation Services & Local Compile Backend                      │
│ - Implement Explainer, Comparer, Coach, and Verdict presentation engines  │
│ - Implement Beginner vs. Technical dual-view response builder              │
│ - Implement local REST/IPC API schema (POST /compile, POST /optimize)      │
│ - Integration tests validating end-to-end orchestration on sample kernels  │
└─────────────────────────────────────┬──────────────────────────────────────┘
                                      │
                                      ▼
┌────────────────────────────────────────────────────────────────────────────┐
│ Phase 5: Desktop IDE Application & Standalone Offline Packaging            │
│ - Build standalone Desktop IDE UI (Editor, Project Tree, Sidecar Pane)     │
│ - Connect Desktop UI to local backend IPC service (127.0.0.1)              │
│ - Package application for 100% offline local execution                     │
│ - Full end-to-end verification across benchmark suites                     │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Verification & Acceptance Criteria

1. **Toolchain Portability:** Unit tests pass with both real system LLVM and mocked toolchains on macOS, Linux, and Windows environments.
2. **Deterministic Replan Test:** When candidate IR is deliberately corrupted or causes a runtime crash, the Supervisor successfully catches the failure, restores `last_known_good_ir_path`, decrements budget, and attempts an alternative pass.
3. **Security Provenance Test:** All security findings carry explicit `evidence_source` tags, and the policy engine correctly blocks or permits optimization based on severity thresholds.
4. **Metric Integrity Test:** Optimization speedup claims are verified by $CI_{95}$ multi-run timing with outlier filtering; no speedup is claimed if within the noise floor ($\le 3\%$).
5. **Offline Invariant Test:** The entire test suite executes with network interfaces disabled, confirming zero external calls.
