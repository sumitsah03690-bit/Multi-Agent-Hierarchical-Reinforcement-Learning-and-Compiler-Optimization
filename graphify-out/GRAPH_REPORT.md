# Graph Report - src  (2026-10-01)

## Corpus Check
- Corpus is ~21,519 words - fits in a single context window. You may not need a graph.

## Summary
- 450 nodes · 783 edges · 24 communities (9 shown, 15 thin omitted)
- Extraction: 98% EXTRACTED · 2% INFERRED · 0% AMBIGUOUS · INFERRED: 14 edges (avg confidence: 0.9)
- Token cost: 0 input · 0 output

## Community Hubs (Navigation)
- Macro & Micro Action Definitions
- GNN Graph Encoders & MCTS Search
- Hierarchical Multi-Agent V5
- Compiler Environment & Rewards
- Direct Beat-O3 Policy
- IR Scalar Feature Extraction
- Pass History Encoders
- World Model V7 Latent Two-Hot Heads
- Hierarchical Multi-Agent Orchestration
- Compiler Metrics & Correctness Collection
- RGCN Attention-Pooled Encoders
- World Model Base & Two-Hot SymLog
- Stratified Benchmark Sampling
- Carbon & Energy Tracking
- MCTS Node Rollouts
- Calibrated World Model & Two-Stage Gate
- World Model V8.5 Residual Scale Correction
- Foveated IR Graph Extractor V5
- LLVM-MCA Machine Code Analyzer
- IR Context Encoder

## God Nodes (most connected - your core abstractions)
1. `MetricsCollector` - 20 edges
2. `IRParser` - 18 edges
3. `CompilerOptEnv` - 17 edges
4. `NegotiationModule` - 11 edges
5. `WorldModel` - 11 edges
6. `TwoHotSymlogHead` - 11 edges
7. `HierarchicalMultiAgentV6` - 10 edges
8. `DirectBeatO3Policy` - 10 edges
9. `CalibratedWorldModel` - 10 edges
10. `TwoHotSymlogHead` - 10 edges

## Surprising Connections (you probably didn't know these)
- `CompilerOptEnv` --uses--> `IRParser`  [INFERRED]
  env/compiler_env.py → features/ir_parser.py
- `HRLAgent` --uses--> `HRLWorker`  [INFERRED]
  models/hrl_agent.py → models/hrl_worker.py
- `extract_scalar_features()` --calls--> `IRParser`  [EXTRACTED]
  features/feature_vector.py → features/ir_parser.py
- `HierarchicalMultiAgentV6` --uses--> `GRUPassHistoryEncoder`  [INFERRED]
  models/_archive/hrl_agent_v6.py → models/_archive/hrl_agent_v5.py
- `DirectBeatO3Policy` --uses--> `GNNEncoderV6`  [INFERRED]
  models/beat_o3_policy.py → models/graph_encoder.py

## Import Cycles
- None detected.

## Communities (24 total, 15 thin omitted)

### Community 0 - "Macro & Micro Action Definitions"
Cohesion: 0.07
Nodes (9): MicroRefiner, RewardMode, HardwareProfile, HardwareProfiler, create_v6_encoder(), Metrics, compile_to_ir(), PassExecutor (+1 more)

### Community 1 - "GNN Graph Encoders & MCTS Search"
Cohesion: 0.08
Nodes (4): GNNEncoderV6, scatter_mean(), HRLWorker, mission_utility_from_objective()

### Community 2 - "Hierarchical Multi-Agent V5"
Cohesion: 0.08
Nodes (5): GRUPassHistoryEncoder, HierarchicalMultiAgentV5, create_hrl_agent_v6(), HierarchicalMultiAgentV6, WorldModelV6

### Community 3 - "Compiler Environment & Rewards"
Cohesion: 0.10
Nodes (5): CompilerOptEnv, BasicBlock, ControlFlowGraph, IRParser, IRStats

### Community 4 - "Direct Beat-O3 Policy"
Cohesion: 0.08
Nodes (5): DirectBeatO3Policy, create_v6_encoder(), GNNEncoderV6, scatter_mean(), GRUPassHistoryEncoder

### Community 5 - "IR Scalar Feature Extraction"
Cohesion: 0.10
Nodes (8): extract_features(), extract_scalar_features(), normalize(), create_default_encoder(), GNNEncoder, extract_ir_graph(), IRGraphExtractor, extract_ir_graph_v5()

### Community 6 - "Pass History Encoders"
Cohesion: 0.09
Nodes (6): GRUPassHistoryEncoder, HRLAgent, NegotiationModule, make_onehot(), make_onehot(), SpecialistAgent

### Community 8 - "Hierarchical Multi-Agent Orchestration"
Cohesion: 0.14
Nodes (4): create_hrl_agent(), HierarchicalMultiAgent, create_world_model(), WorldModel

### Community 10 - "RGCN Attention-Pooled Encoders"
Cohesion: 0.12
Nodes (4): create_v5_encoder(), GNNEncoderV5, create_hrl_agent_v5(), WorldModelV5

## Knowledge Gaps
- **15 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `MetricsCollector` connect `Compiler Metrics & Correctness Collection` to `Macro & Micro Action Definitions`, `Stratified Benchmark Sampling`?**
  _High betweenness centrality (0.090) - this node is a cross-community bridge._
- **Why does `IRParser` connect `Compiler Environment & Rewards` to `Macro & Micro Action Definitions`, `IR Scalar Feature Extraction`?**
  _High betweenness centrality (0.051) - this node is a cross-community bridge._
- **Should `Macro & Micro Action Definitions` be split into smaller, more focused modules?**
  _Cohesion score 0.06779661016949153 - nodes in this community are weakly interconnected._
- **Should `GNN Graph Encoders & MCTS Search` be split into smaller, more focused modules?**
  _Cohesion score 0.080338266384778 - nodes in this community are weakly interconnected._
- **Should `Hierarchical Multi-Agent V5` be split into smaller, more focused modules?**
  _Cohesion score 0.08412698412698413 - nodes in this community are weakly interconnected._
- **Should `Compiler Environment & Rewards` be split into smaller, more focused modules?**
  _Cohesion score 0.10227272727272728 - nodes in this community are weakly interconnected._
- **Should `Direct Beat-O3 Policy` be split into smaller, more focused modules?**
  _Cohesion score 0.0846774193548387 - nodes in this community are weakly interconnected._