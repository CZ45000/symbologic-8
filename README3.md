# Ternary ALU & Spatial Coprocessor

A research-oriented hardware architecture exploring **balanced ternary computation, spatial processing, and their combination on conventional binary FPGA/CMOS infrastructure**.

The project investigates three architectural approaches:

1. **System 1 — Ternary Datapath**
2. **System 2 — Spatial Coprocessor**
3. **System 3 — Hybrid Ternary Spatial Coprocessor**

All three systems are evaluated using the same workloads and metrics in order to determine **where ternary encoding helps, where spatial execution helps, and whether combining the two provides a measurable architectural advantage**.

The project does **not** assume that ternary computation is inherently superior to binary computation. Instead, it provides a reproducible simulation framework for measuring the associated trade-offs.

---

# 1. Research Objective

The central research question is:

> **Where does ternary encoding provide an architectural advantage, where does spatial execution provide an advantage, and does combining both approaches produce a measurable improvement?**

The project therefore separates the investigation into three independent but related architectural paths.

```text
                         SAME WORKLOAD
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
       ┌────────────┐  ┌────────────┐  ┌────────────┐
       │  SYSTEM 1  │  │  SYSTEM 2  │  │  SYSTEM 3  │
       │            │  │            │  │            │
       │  Ternary   │  │  Spatial   │  │  Hybrid    │
       │  Datapath  │  │  Coprocess.│  │ Ternary +  │
       │            │  │            │  │  Spatial   │
       └─────┬──────┘  └─────┬──────┘  └─────┬──────┘
             │               │               │
             └───────────────┼───────────────┘
                             ▼
                       SAME METRICS
```

This experimental separation makes it possible to distinguish:

* the effect of ternary encoding;
* the effect of spatial parallelism;
* the combined effect of both.

---

# 2. Core Encoding Concept

The fundamental representation uses **two conventional binary bits to represent one balanced ternary digit (trit)**.

Balanced ternary uses three mathematical states:

```text
-1    0    +1
```

Two binary bits provide four physical states:

```text
00
01
10
11
```

Three states are assigned to valid ternary data and the fourth is reserved.

One possible mapping is:

| Binary encoding |  Meaning | Domain                      |
| --------------- | -------: | --------------------------- |
| `00`            |      `0` | Ternary data                |
| `01`            |     `+1` | Ternary data                |
| `10`            |     `-1` | Ternary data                |
| `11`            | Reserved | Control / exceptional state |

The exact numerical mapping may be changed by the implementation. The architectural principle remains the same:

```text
2 binary bits
     │
     ├── 3 valid states → ternary data
     │
     └── 1 reserved state → control / metadata
```

The `11` state is **not a fourth mathematical value**.

It belongs to a separate control or exceptional domain.

---

# 3. Reserved Fourth State

The unused encoding state provides an architectural extension point.

Possible uses include:

* invalid-state detection;
* fault detection;
* synchronization;
* pipeline markers;
* valid/invalid signalling;
* end-of-stream indication;
* exceptional arithmetic states;
* handshake information;
* local control metadata;
* debugging and observability.

For example:

```text
00 → ternary 0
01 → ternary +1
10 → ternary -1
11 → reserved/control
```

A decoder can therefore distinguish normal data from an exceptional or control condition without treating the reserved state as a valid ternary number.

This mechanism is one of the main architectural hypotheses investigated by the project.

It should **not** automatically be interpreted as a hardware-area or performance reduction. The additional decoding and control logic must be measured.

---

# 4. Three Architectural Systems

The experimental framework contains three systems.

---

## System 1 — Ternary Datapath

The first system evaluates the ternary representation independently from the spatial architecture.

Its purpose is to determine whether a ternary arithmetic pipeline implemented on binary hardware provides useful computational properties.

Conceptually:

```text
Input
  │
  ▼
┌─────────────────┐
│ Ternary Register│
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Ternary ALU /   │
│ Add Tree        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Output Register │
└─────────────────┘
```

### Main questions

* How much latency does the ternary datapath introduce?
* What is the throughput?
* How many binary operations are required per ternary operation?
* What is the storage overhead?
* How useful is the reserved state?
* What is the estimated energy per operation?
* How does the encoding compare with a binary baseline?

System 1 isolates the **encoding and arithmetic effects**.

---

# 5. System 2 — Spatial Coprocessor

The second system evaluates spatial computation independently from the ternary datapath.

The initial target is a:

```text
4 × 4 mesh
= 16 processing tiles
```

Conceptually:

```text
┌────┬────┬────┬────┐
│ T0 │ T1 │ T2 │ T3 │
├────┼────┼────┼────┤
│ T4 │ T5 │ T6 │ T7 │
├────┼────┼────┼────┤
│ T8 │ T9 │T10 │T11 │
├────┼────┼────┼────┤
│T12 │T13 │T14 │T15 │
└────┴────┴────┴────┘
```

Each tile represents a processing element with local computation and communication capabilities.

The purpose is to determine whether **localized parallel execution and mesh-based routing** provide measurable advantages.

System 2 can use a conventional binary datapath.

This is important because it creates a control experiment:

> If the spatial system improves performance, the improvement can be attributed primarily to spatial organization rather than ternary encoding.

---

# 6. System 3 — Hybrid Ternary Spatial Coprocessor

The third system combines the two concepts.

Each spatial tile contains a ternary datapath.

```text
                 HOST
                   │
                   ▼
        ┌─────────────────────┐
        │ Hybrid Coprocessor  │
        │                     │
        │ ┌────┐ ┌────┐       │
        │ │ T  │─│ T  │─ ...  │
        │ └─┬──┘ └─┬──┘       │
        │   │      │          │
        │ ┌─┴──┐ ┌─┴──┐       │
        │ │ T  │─│ T  │       │
        │ └────┘ └────┘       │
        │       ...           │
        └─────────────────────┘
```

The initial configuration is:

```text
4 × 4 = 16 ternary processing tiles
```

This system tests the combined hypothesis:

> **Can ternary datapaths and spatial organization provide complementary benefits when used together?**

---

# 7. Experimental Matrix

The primary experiment consists of:

```text
3 architectures × 3 workloads
```

| Architecture                   | Pattern Matching | Rule Engine | Stream Processing |
| ------------------------------ | ---------------: | ----------: | ----------------: |
| System 1 — Ternary Datapath    |                ✓ |           ✓ |                 ✓ |
| System 2 — Spatial Coprocessor |                ✓ |           ✓ |                 ✓ |
| System 3 — Hybrid              |                ✓ |           ✓ |                 ✓ |

All workloads should use equivalent input sizes and comparable computational requirements wherever possible.

This prevents a workload-specific implementation detail from being mistaken for an architectural advantage.

---

# 8. Workload 1 — Pattern Matching

Pattern matching is used to evaluate highly parallel comparison workloads.

Possible applications include:

* symbolic matching;
* sequence detection;
* low-precision classification;
* local feature comparison;
* pattern recognition.

A conceptual workload is:

```text
Input:
A B C D E F G ...

Pattern:
A B C

Result:
match / no-match
```

The experiment measures:

* comparisons per token;
* latency;
* throughput;
* tile utilization;
* communication volume;
* routing distance;
* operations per token.

---

# 9. Workload 2 — Rule Engine

The rule engine evaluates conditional logic.

Example:

```text
IF A AND B
    THEN ACTION_1

IF C OR D
    THEN ACTION_2

IF E AND NOT F
    THEN ACTION_3
```

This workload is useful for investigating:

* branching;
* control propagation;
* synchronization;
* reserved-state usage;
* localized decision making.

The `11` encoding can be evaluated as a possible control/exception state, but any claimed advantage must be supported by simulation or hardware measurements.

---

# 10. Workload 3 — Stream Processing

Stream processing evaluates a continuous sequence of tokens.

```text
Input Stream
     │
     ▼
┌────┐   ┌────┐   ┌────┐   ┌────┐
│ T0 │ → │ T1 │ → │ T2 │ → │ T3 │
└────┘   └────┘   └────┘   └────┘
     │
     ▼
Output Stream
```

This workload emphasizes:

* sustained throughput;
* pipeline utilization;
* buffering;
* routing;
* backpressure;
* steady-state execution.

Unlike a single-operation benchmark, stream processing makes it possible to evaluate the difference between **latency** and **sustained throughput**.

---

# 11. Metrics

All three systems are evaluated using the same metrics.

## 11.1 Latency

Latency is the number of cycles between input acceptance and result availability.

```text
Latency = output_cycle - input_cycle
```

Latency is reported independently from throughput.

---

## 11.2 Throughput

Throughput measures completed tokens per cycle.

```text
Throughput =
completed_tokens / total_cycles
```

If a clock frequency is assumed:

```text
tokens/s =
tokens/cycle × clock_frequency
```

A frequency-derived result is only a projection unless the frequency has been obtained from synthesis or physical hardware.

---

## 11.3 Operations per Token

This measures the amount of computation required to process one input token.

```text
Operations per Token =
total elementary operations / processed tokens
```

This metric allows workload-normalized comparisons.

---

## 11.4 Memory Cost

The simulator estimates:

* local state;
* input buffers;
* output buffers;
* intermediate storage;
* metadata;
* control state.

The distinction between:

```text
logical memory requirement
```

and:

```text
physical FPGA/ASIC memory utilization
```

must be maintained.

Physical resource usage requires synthesis.

---

## 11.5 Routing Cost

For the spatial systems, routing is characterized using:

* number of hops;
* average Manhattan distance;
* maximum distance;
* communication volume;
* link utilization;
* congestion indicators.

For two tiles:

```text
distance =
|x1 - x2| + |y1 - y2|
```

This provides a deterministic baseline for mesh communication.

---

## 11.6 Parallelism

The simulator reports:

* active tiles;
* tile utilization;
* parallel operations;
* speedup;
* parallel efficiency.

Parallel efficiency is defined as:

```text
Efficiency(N) =
Speedup(N) / N
```

where `N` is the number of processing elements.

---

# 12. Scalability

The initial benchmark is:

```text
4 × 4
```

but the simulator should support larger configurations:

```text
2 × 2
4 × 4
8 × 8
16 × 16
```

where computationally practical.

This makes it possible to investigate:

* scaling efficiency;
* communication overhead;
* routing growth;
* utilization;
* memory growth;
* energy growth.

The goal is not to assume linear scaling.

Instead, the simulator should expose where communication and control overhead begin to dominate.

---

# 13. Parametric Energy Model

Energy is initially modeled parametrically.

The simulator does **not** claim to measure physical silicon power.

A conceptual model is:

```text
E_total =
    E_compute
  + E_memory
  + E_routing
  + E_control
```

For example:

```text
E_compute =
    N_compute × E_compute_op

E_memory =
    N_memory_access × E_memory_access

E_routing =
    N_hops × E_hop

E_control =
    N_control_events × E_control_event
```

The model parameters can be swept to perform sensitivity analysis.

For example:

```text
E_compute_op
E_memory_access
E_hop
E_control_event
```

can be varied independently.

This produces **architectural energy estimates**, not measured silicon power.

---

# 14. Binary Baseline

Where possible, the experimental framework should also include a conventional binary baseline.

This provides an additional reference point:

```text
Binary baseline
       │
       ├── System 1: Ternary datapath
       │
       ├── System 2: Spatial binary datapath
       │
       └── System 3: Hybrid ternary spatial datapath
```

The purpose is to answer two different questions:

### Question A

Does ternary encoding improve the datapath?

```text
Binary datapath
      vs
Ternary datapath
```

### Question B

Does spatial organization improve execution?

```text
Single datapath
      vs
Spatial mesh
```

### Question C

Do the two advantages combine?

```text
Ternary
   +
Spatial
   =
Hybrid
```

This separation is essential for meaningful conclusions.

---

# 15. Reproducible Simulation

The Python simulator should be deterministic.

Every experiment records:

```text
architecture
mesh dimensions
encoding
workload
token count
random seed
clock assumption
energy parameters
software version
```

Example:

```bash
python run_benchmarks.py \
    --system hybrid \
    --mesh 4x4 \
    --workload pattern_matching \
    --tokens 100000 \
    --seed 42
```

A complete benchmark may be executed with:

```bash
python run_benchmarks.py \
    --mesh 4x4 \
    --all-systems \
    --all-workloads \
    --tokens 100000 \
    --seed 42
```

The simulator should export machine-readable results such as:

```text
CSV
JSON
```

and automatically generate plots.

---

# 16. Expected Output

The simulation framework should generate comparative figures such as:

### Latency

```text
Latency
  │
  │       █
  │   █   █
  │   █   █       █
  └──────────────────
      S1  S2      S3
```

### Throughput

Comparison of tokens/cycle across the three architectures.

### Operations per Token

Comparison of computational work independent of clock frequency.

### Memory Cost

Logical storage requirements per token and per tile.

### Routing Cost

Average and maximum number of hops.

### Tile Utilization

Percentage of active processing resources.

### Scaling Efficiency

Performance as the mesh grows from:

```text
2×2 → 4×4 → 8×8 → 16×16
```

### Energy

Parametric energy/token under different technology assumptions.

---

# 17. Results Classification

The project explicitly separates experimental evidence into three categories.

## A. Simulated Results

These are directly generated by the Python/RTL simulation.

Examples:

* latency;
* simulated throughput;
* operations/token;
* tile utilization;
* routing hops;
* simulated scaling;
* workload execution statistics.

These can be reproduced from the benchmark configuration.

---

## B. Theoretical / Parametric Results

These are derived from explicit architectural assumptions.

Examples:

* energy/token;
* projected tokens/second at a hypothetical clock;
* projected scaling;
* logical memory estimates;
* sensitivity to routing energy.

These should always be labeled as estimates or projections.

---

## C. Hardware Results

These require actual FPGA/ASIC implementation.

Examples:

* LUT usage;
* flip-flop usage;
* BRAM usage;
* DSP usage;
* actual Fmax;
* timing closure;
* routing congestion;
* physical power;
* measured energy/token;
* ASIC area;
* physical performance.

These values must not be inferred from the Python model.

---

# 18. Validation Status

The repository should maintain an explicit validation table.

| Metric / Feature           | Status                                 |
| -------------------------- | -------------------------------------- |
| 2-bit/trit representation  | Architectural                          |
| Three valid ternary states | Architectural                          |
| Reserved fourth state      | Architectural                          |
| Ternary datapath           | RTL / Simulation                       |
| 4×4 spatial mesh           | Simulation target                      |
| Pattern matching           | Simulation                             |
| Rule engine                | Simulation                             |
| Stream processing          | Simulation                             |
| Latency                    | Simulated                              |
| Throughput                 | Simulated                              |
| Operations/token           | Simulated                              |
| Routing hops               | Simulated                              |
| Tile utilization           | Simulated                              |
| Parametric energy          | Estimated                              |
| Binary baseline            | Experimental comparison                |
| FPGA LUT/FF usage          | Requires synthesis                     |
| FPGA Fmax                  | Requires timing analysis               |
| FPGA routing congestion    | Requires implementation                |
| FPGA power                 | Requires power analysis or measurement |
| ASIC area                  | Requires ASIC synthesis                |
| Physical energy/token      | Requires hardware measurement          |

---

# 19. Proposed Experimental Sequence

The recommended evaluation sequence is:

```text
STEP 1
Validate ternary encoding
        │
        ▼
STEP 2
Validate ternary datapath
        │
        ▼
STEP 3
Validate spatial mesh
        │
        ▼
STEP 4
Run System 1
        │
        ▼
STEP 5
Run System 2
        │
        ▼
STEP 6
Run System 3
        │
        ▼
STEP 7
Compare all systems
        │
        ▼
STEP 8
Scale mesh
        │
        ▼
STEP 9
Run energy sensitivity analysis
        │
        ▼
STEP 10
Validate promising configurations on FPGA
```

This prevents the hybrid architecture from being evaluated without understanding the contribution of its individual components.

---

# 20. Recommended Repository Structure

```text
.
├── src/
│   ├── ternary_system.v
│   ├── ternary_alu.v
│   ├── ternary_add_tree.v
│   └── ...
│
├── sim/
│   ├── testbench/
│   └── ...
│
├── python/
│   ├── ternary_model.py
│   ├── binary_model.py
│   ├── tile.py
│   ├── mesh.py
│   ├── metrics.py
│   ├── energy.py
│   ├── experiments.py
│   ├── comparison.py
│   │
│   └── workloads/
│       ├── pattern_matching.py
│       ├── rule_engine.py
│       └── stream_processing.py
│
├── results/
│   ├── raw/
│   ├── csv/
│   ├── json/
│   └── figures/
│
├── reports/
│   └── technical_report.md
│
├── requirements.txt
├── run_benchmarks.py
└── README.md
```

---

# 21. Research Questions

The experimental framework is designed to answer:

### RQ1 — Ternary Representation

Can a balanced ternary datapath encoded using two binary bits provide useful computational characteristics on conventional binary hardware?

### RQ2 — Reserved State

Can the fourth encoding state be effectively used for control, synchronization, fault detection, or metadata?

### RQ3 — Spatial Execution

Does a localized tile architecture improve throughput or scalability for suitable workloads?

### RQ4 — Hybrid Architecture

Does combining ternary computation with spatial execution produce an advantage beyond either approach independently?

### RQ5 — Workload Dependence

Which workloads benefit most from each architecture?

### RQ6 — Scaling

At what mesh size does communication overhead begin to dominate computational parallelism?

### RQ7 — Energy

Under realistic parameter ranges, does the hybrid architecture show a potential energy advantage?

---

# 22. Architectural Hypothesis

The project can be summarized by the following hypothesis:

```text
                  TERNARY ENCODING
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
       Compact data/control    Ternary arithmetic
              │                     │
              └──────────┬──────────┘
                         │
                         ▼
                 TERNARY TILE
                         │
                         ▼
                  SPATIAL MESH
                         │
                         ▼
                HYBRID PROCESSOR
```

However, the hypothesis is intentionally open-ended.

The experiments may demonstrate that:

* ternary encoding helps;
* spatial organization helps;
* both help independently;
* the hybrid architecture helps;
* only specific workloads benefit;
* or the additional encoding/control overhead eliminates the expected advantage.

All of these are valid research outcomes.

---

# 23. Limitations

The use of two binary bits per trit does **not** imply a direct 2× performance improvement.

The encoding may introduce:

* decoder logic;
* additional control;
* storage overhead;
* conversion overhead;
* invalid-state handling;
* additional switching activity.

Likewise, a spatial mesh does not automatically reduce routing cost.

Its effectiveness depends on:

* workload locality;
* communication patterns;
* tile utilization;
* topology;
* buffering;
* congestion;
* synthesis;
* physical implementation.

The project therefore treats performance and energy improvements as **testable hypotheses rather than predetermined conclusions**.

---

# 24. Hardware Validation Roadmap

After software simulation, promising configurations should be implemented on FPGA.

A possible validation sequence is:

```text
Python model
     │
     ▼
RTL simulation
     │
     ▼
Logic synthesis
     │
     ▼
Place & Route
     │
     ▼
Timing analysis
     │
     ▼
FPGA implementation
     │
     ▼
Power estimation
     │
     ▼
Physical measurement
```

The final stage should be used to determine whether the architectural trends predicted by simulation survive real hardware constraints.

---

# 25. What This Project Does Not Claim

This project does not claim that:

* ternary logic is universally superior to binary logic;
* two-bit ternary encoding automatically reduces area;
* the reserved fourth state automatically reduces hardware cost;
* a spatial mesh automatically improves performance;
* simulated energy equals physical energy;
* theoretical frequency equals achievable FPGA frequency.

Such claims require appropriate experimental evidence.

---

# 26. Research Contribution

The primary contribution of the project is the creation of a reproducible experimental framework that compares:

```text
                ┌──────────────────────┐
                │ Binary Baseline      │
                └──────────┬───────────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        Ternary        Spatial        Hybrid
        Datapath       Coprocessor     System
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                  Common Workloads
                           │
                           ▼
                 Common Measurements
                           │
                           ▼
               Reproducible Comparison
```

The central experimental question is not simply:

> "Is ternary better?"

but rather:

> **Under which workloads and architectural conditions does ternary encoding, spatial processing, or their combination provide a measurable advantage over a conventional binary baseline?**

---

# 27. Current Scope

The initial experimental configuration is:

```text
Encoding:
    2 bits / trit

Valid states:
    3 ternary states

Reserved state:
    1 control/exception state

Spatial configuration:
    4 × 4 tiles

Workloads:
    Pattern Matching
    Rule Engine
    Stream Processing

Architectures:
    System 1 — Ternary Datapath
    System 2 — Spatial Coprocessor
    System 3 — Hybrid

Metrics:
    Latency
    Throughput
    Operations/token
    Memory cost
    Routing cost
    Parallelism
    Scalability
    Parametric energy
```

The architecture and simulator should remain parameterized so that larger meshes, additional workloads, and alternative encodings can be evaluated later.

---

# 28. Conclusion

This project explores a practical path toward ternary-inspired computing without requiring a custom ternary semiconductor process.

The fundamental abstraction is:

```text
       2 binary bits
             │
       ┌─────┴─────┐
       │           │
   3 data states   1 reserved state
       │           │
       ▼           ▼
   Ternary data   Control /
                  metadata /
                  fault state
```

This abstraction can be evaluated at two architectural levels:

```text
Level 1:
Ternary Datapath

Level 2:
Spatial Processing Fabric
```

and ultimately combined:

```text
Ternary Datapath
        +
Spatial Mesh
        =
Hybrid Ternary Spatial Coprocessor
```

The three-system experimental methodology makes it possible to determine whether any observed improvement originates from:

1. ternary representation;
2. spatial organization;
3. or the interaction between both.

The project therefore focuses on **measurable, reproducible architectural evidence**, with a strict distinction between simulated results, theoretical estimates, and hardware results that remain to be validated.

---

## Status

**Research / Experimental**

The architecture is under investigation. Performance, energy, area, and scaling claims should be considered provisional until supported by reproducible simulation, RTL verification, FPGA synthesis/implementation, or physical hardware measurements.
