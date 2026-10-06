# Transient Data, Lightweight Tokens, and Spatial Processing

## An Architectural Research Direction Beyond Ternary Computing

### Abstract

Conventional computer architectures generally treat data storage, data movement, and computation as closely related physical problems. A bit that must be preserved in a memory cell, a bit held temporarily in a pipeline, and a bit travelling between processing elements may be represented using substantially different physical mechanisms, yet architectural discussions often treat them as instances of the same abstract storage problem.

This document proposes a broader architectural research direction:

> **Can transient information be represented and transported through a chip using a physically lighter mechanism than conventional static storage, while more complex fixed structures perform the computation?**

The hypothesis is not specific to ternary computing.

It can be investigated in conventional binary processors, accelerators, spatial architectures, FPGA-like fabrics, ternary systems, and hybrid architectures.

The central idea is to distinguish three physical functions:

1. **persistent information storage;**
2. **transient information transport;**
3. **localized computation and control.**

Instead of giving every piece of information the physical characteristics of a conventional memory cell, an architecture may attempt to optimize temporary information specifically for movement.

The resulting architecture can be understood as a separation between:

```text
DATA
  │
  │ lightweight and transient
  ▼
SPATIAL MOVEMENT
  │
  ▼
FIXED PROCESSING STRUCTURES
  │
  │ persistent rules and control
  ▼
TRANSFORMED DATA
```

This is an architectural hypothesis, not a claim of proven efficiency.

The purpose of this research direction is to determine whether such a separation can reduce area, energy, latency, memory traffic, or other costs for specific workloads while remaining compatible with conventional semiconductor fabrication technologies.

---

# 1. The Question Behind the Architecture

A conventional digital system usually asks:

> How do we store and process a bit reliably?

A spatial architecture can ask a different question:

> How do we move information from one computational location to another with the minimum necessary physical overhead?

These are not necessarily the same problem.

A memory cell is designed to preserve information.

A pipeline stage may only need to preserve information for a short period.

A data token moving between neighboring processing elements may need to remain valid only until the receiving element captures or transforms it.

This difference suggests that the physical implementation of transient information does not necessarily need to be identical to the physical implementation of persistent information.

The research direction proposed here is therefore based on a simple principle:

> **Temporary information does not necessarily need the same physical storage structure as persistent information.**

This principle is independent of the logical representation used by the system.

The information may be binary, ternary, multi-valued, symbolic, numerical, or application-specific.

---

# 2. The Physical Cost of a Bit

The phrase "one bit" describes an information state, not a unique physical structure.

Different technologies and architectural roles use different physical mechanisms.

For example, a conventional SRAM bit cell commonly uses six transistors in a static CMOS implementation.

Dynamic memories can use substantially fewer active devices per stored bit, but rely on capacitive charge storage, sensing, addressing, refresh, and associated circuitry.

Registers and flip-flops generally require several transistors per stored bit because they must provide stable static behavior.

Other storage technologies use different physical mechanisms again.

Therefore:

```text
logical bit
    ≠
fixed transistor count
```

The physical cost of a bit depends on:

- retention requirements;
- access requirements;
- speed;
- stability;
- noise margins;
- leakage;
- refresh requirements;
- read/write mechanisms;
- control;
- interconnect;
- technology;
- architecture.

This observation leads to a more specific question.

Instead of asking:

> How many transistors are required to store one bit?

the proposed architecture asks:

> **How many physical resources are actually necessary to transport a temporary bit through a controlled computational pipeline?**

That question is more directly relevant to a spatial processor.

---

# 3. Static Storage Versus Transient Transport

Consider two situations.

### Static information

A memory cell may need to preserve its value for a relatively long period:

```text
             ┌───────────────┐
             │ MEMORY CELL   │
             │               │
             │     DATA      │
             │               │
             └───────────────┘
                    │
             retain indefinitely
```

### Transient information

A token may instead move through a sequence of processing stages:

```text
TOKEN
  │
  ▼
[PE A]
  │
  ▼
[PE B]
  │
  ▼
[PE C]
  │
  ▼
[PE D]
  │
  ▼
RESULT
```

At each stage, the information may only need to survive long enough to be transferred or transformed.

This does not mean that the token has no physical storage.

It means that its storage requirement may be fundamentally different from that of a long-lived memory cell.

This distinction creates the possibility of specialized transient storage and transport structures.

---

# 4. Lightweight Data as an Architectural Object

The architecture proposed here introduces the concept of a **lightweight data token**.

A token is a compact unit of information that moves through a computational fabric.

For example:

```text
+----------------+
|  DATA TOKEN    |
+----------------+
| payload        |
| metadata       |
| control        |
+----------------+
```

The token could contain:

- an integer;
- a character;
- a byte;
- a vector element;
- a symbolic value;
- a ternary value;
- a state-machine symbol;
- an intermediate computation result;
- a small control field.

The token is not necessarily intended to be stored permanently.

It is intended to **flow**.

This leads to a distinction:

```text
Persistent information
    └── optimized for retention

Transient information
    └── optimized for movement
```

This is the central architectural concept.

---

# 5. Data and Computation as Separate Physical Layers

A spatial architecture can separate the physical complexity of the data from the physical complexity of the computation.

One possible organization is:

```text
                 DATA FLOW

        TOKEN ───► TOKEN ───► TOKEN
          │          │          │
          ▼          ▼          ▼

       ┌────────┐ ┌────────┐ ┌────────┐
       │ RULE A │ │ RULE B │ │ RULE C │
       │        │ │        │ │        │
       │ LOGIC  │ │ LOGIC  │ │ LOGIC  │
       └────────┘ └────────┘ └────────┘

          │          │          │
          ▼          ▼          ▼

        TOKEN ───► TOKEN ───► RESULT
```

The data moves.

The rules remain localized.

The processing structures can therefore be more complex than the data representation itself.

This produces a useful architectural asymmetry:

```text
DATA
    lightweight
    transient
    mobile

PROCESSING
    comparatively complex
    persistent
    localized
```

The objective is not to make the entire chip physically simple.

The objective is to place physical complexity where it provides the most computational value.

---

# 6. Fixed Rule Structures

A processing tile can contain relatively complex logic that remains physically stationary.

A simplified tile could be represented as:

```text
+--------------------------------+
|        PROCESSING TILE         |
|                                |
|  +--------------------------+  |
|  | Rule / LUT / Logic       |  |
|  +--------------------------+  |
|              |                 |
|              v                 |
|  +--------------------------+  |
|  | Local Transformation     |  |
|  +--------------------------+  |
|              |                 |
|              v                 |
|  +--------------------------+  |
|  | Routing / Selection      |  |
|  +--------------------------+  |
|                                |
+--------------------------------+
```

A token arriving at the tile is transformed according to the local rule.

The transformed token then moves to another tile.

The rule does not need to move with the data.

This can potentially create substantial rule reuse.

For example:

```text
TOKEN A ──► RULE A ──► TOKEN A'
TOKEN B ──► RULE A ──► TOKEN B'
TOKEN C ──► RULE A ──► TOKEN C'
TOKEN D ──► RULE A ──► TOKEN D'
```

The same physical rule structure may process many tokens.

This is analogous to moving data through a fixed computational landscape rather than repeatedly moving both data and computation through the system.

---

# 7. Spatial Computation

The architecture naturally leads to a spatial processing fabric.

A simple fabric could look like:

```text
+------+     +------+     +------+     +------+
| PE 0 |────►| PE 1 |────►| PE 2 |────►| PE 3 |
+------+     +------+     +------+     +------+
   │            │            │            │
   ▼            ▼            ▼            ▼
+------+     +------+     +------+     +------+
| PE 4 |────►| PE 5 |────►| PE 6 |────►| PE 7 |
+------+     +------+     +------+     +------+
   │            │            │            │
   ▼            ▼            ▼            ▼
+------+     +------+     +------+     +------+
| PE 8 |────►| PE 9 |────►| PE10 |────►| PE11 |
+------+     +------+     +------+     +------+
```

The exact topology is not fixed.

Possible structures include:

- linear pipelines;
- 1D fabrics;
- 2D meshes;
- trees;
- rings;
- systolic arrays;
- networks-on-chip;
- application-specific fabrics;
- dynamically configured spatial networks.

The key property is that **computation is associated with physical location**.

---

# 8. The 1-Transistor-Per-Bit Question

An extreme version of the research question is:

> Could a transient 8-bit token approach a physical implementation scale on the order of eight active transistors dedicated to its immediate transport or state representation?

This should not be interpreted as a claim that eight transistors are sufficient to implement a robust general-purpose 8-bit register.

They are not.

A conventional static register requires substantially more circuitry.

The question is instead whether a **specialized transient transport mechanism** can approach a much lower physical device count than a conventional static storage structure.

Possible mechanisms to investigate include:

- dynamic storage;
- pass-transistor structures;
- transmission gates;
- dynamic shift registers;
- short-lived pipeline storage;
- charge-based transient representation;
- specialized low-storage-overhead CMOS structures.

The practical implementation must include all necessary supporting circuitry.

Therefore:

```text
8 data transistors
    ≠
complete 8-bit transport system
```

A complete implementation may require:

- drivers;
- receivers;
- clocking;
- regeneration;
- sensing;
- control;
- routing;
- buffering;
- refresh;
- error handling;
- power distribution.

The research objective is to determine the **total physical cost**, not merely the number of transistors in the smallest possible storage element.

---

# 9. The Correct Optimization Target

For this reason, transistor count alone is not sufficient.

The relevant optimization target should be something closer to:

```text
physical cost per correctly processed token
```

or:

```text
energy per correctly processed token
```

or:

```text
area × energy × latency per useful operation
```

A hypothetical token representation that uses fewer transistors but requires excessive signal regeneration may be worse than a larger conventional structure.

Similarly, a very small storage element that requires frequent refresh may lose its advantage.

The architecture therefore requires system-level evaluation.

---

# 10. Dynamic and Transient Logic

Dynamic logic is one possible implementation direction because it can exploit temporary charge storage rather than continuously maintaining a static state through complementary feedback structures.

Pass-transistor and transmission-gate techniques are other possible mechanisms for reducing the number of active devices in selected signal paths.

However, these techniques introduce important constraints.

The research must evaluate:

- leakage;
- charge sharing;
- noise;
- signal degradation;
- timing;
- clocking;
- refresh;
- process variation;
- temperature sensitivity;
- cascading limitations;
- signal restoration.

The fact that a structure can theoretically represent a temporary state with few transistors does not establish that it is practical.

The engineering objective is therefore:

> **minimum practical physical cost under a defined reliability and performance target.**

---

# 11. The Cost of Movement

The most important counterargument to this architecture is that movement is not free.

Every physical transition can involve capacitive charging and discharging.

A token travelling through a spatial fabric may activate:

- wires;
- drivers;
- receivers;
- buffers;
- multiplexers;
- clocking structures;
- routing switches.

Therefore the architecture must not assume:

```text
movement = free
```

Instead, it investigates:

```text
cost of spatial movement
        versus
cost of conventional data movement
```

This is the actual comparison.

A spatial architecture is attractive only if the organization of the computation reduces the overall system cost.

---

# 12. Locality as the Main Potential Advantage

The most plausible source of benefit is therefore not the token itself.

It is **locality**.

Suppose a workload repeatedly applies several transformations to a stream of data.

A conventional implementation might repeatedly move information between:

```text
registers
cache
ALU
load/store units
memory
control logic
```

A spatial implementation may instead establish a fixed processing path:

```text
Input
  │
  ▼
[Filter]
  │
  ▼
[Transform]
  │
  ▼
[Compare]
  │
  ▼
[Classify]
  │
  ▼
Output
```

The information moves through the computation rather than repeatedly returning to a centralized execution point.

This may reduce:

- long-distance communication;
- memory traffic;
- repeated instruction overhead;
- synchronization;
- data replication;
- cache pressure.

Whether it actually does so must be measured.

---

# 13. Application to Conventional Binary Chips

This architecture does not require ternary computing.

A conventional binary processor could contain a specialized spatial region:

```text
+--------------------------------------+
|          Conventional CPU           |
|                                      |
|  Fetch / Decode / ALU / Registers   |
|                                      |
|             │                        |
|             ▼                        |
|      +---------------+               |
|      | Spatial Unit  |               |
|      |               |               |
|      | Binary PE     |               |
|      | Binary PE     |               |
|      | Binary PE     |               |
|      | Binary PE     |               |
|      +---------------+               |
|                                      |
+--------------------------------------+
```

The CPU remains conventional.

The spatial unit becomes a specialized execution resource.

This makes the architecture potentially compatible with existing semiconductor manufacturing approaches.

The innovation would primarily be architectural rather than dependent on a new transistor type.

---

# 14. Hybrid Conventional / Spatial Architecture

A more general architecture could therefore contain both conventional and spatial execution.

```text
                    CPU
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
 Conventional              Spatial
 Execution                 Execution
          │                     │
          │              +------+------+
          │              |             |
          │           Binary PE     Specialized PE
          │              |             |
          └──────────────┴─────────────┘
                         │
                       Result
```

The conventional processor would handle workloads that benefit from:

- complex control flow;
- irregular memory access;
- sequential execution;
- operating-system tasks;
- general-purpose computation.

The spatial unit could handle workloads that benefit from:

- regular data flow;
- high locality;
- streaming;
- repeated transformations;
- parallel token processing;
- fixed or semi-fixed rules.

This hybrid approach avoids the assumption that one architecture must handle every workload.

---

# 15. Binary, Ternary, and Multi-Valued Processing

The same spatial architecture can support different logical representations.

### Binary

```text
0 / 1
```

### Ternary

For example:

```text
00 → T0
01 → T1
10 → T2
11 → X
```

### Other representations

The architecture could also investigate:

- packed multi-bit values;
- symbolic tokens;
- encoded states;
- application-specific representations.

This makes spatial organization an independent research variable.

The experiments can therefore separate:

```text
Representation
      │
      ├── Binary
      └── Ternary

Organization
      │
      ├── Conventional
      └── Spatial
```

This creates a useful experimental matrix.

---

# 16. Experimental Matrix

A rigorous evaluation could compare:

```text
System A
Conventional Binary Processor

System B
Binary Spatial Processor

System C
Conventional Ternary Datapath

System D
Ternary Spatial Processor

System E
Conventional CPU + Spatial Accelerator

System F
Conventional CPU + Binary/Ternary Spatial Accelerator
```

The purpose is not to assume that System F is the winner.

The purpose is to identify which architectural mechanisms produce measurable benefits.

---

# 17. Potential Workloads

The architecture is particularly interesting for workloads with strong spatial or streaming structure.

Potential examples include:

- pattern matching;
- rule engines;
- stream processing;
- packet processing;
- signal filtering;
- image processing;
- finite-state transformations;
- regular graph operations;
- cellular automata;
- pipeline-oriented transformations;
- selected inference workloads;
- repeated symbolic transformations.

The architecture may be less attractive for:

- highly irregular pointer chasing;
- unpredictable global memory access;
- strongly sequential algorithms;
- workloads with large shared mutable state;
- workloads dominated by unpredictable branching.

The important principle is:

> **Architectural advantage is expected to be workload-dependent.**

---

# 18. Relationship to Memory

The proposed architecture does not eliminate memory.

Persistent data still requires appropriate storage.

A possible system can therefore be divided into:

```text
+----------------------+
| Persistent Memory     |
| SRAM / DRAM / Cache  |
+----------+-----------+
           |
           | data
           ▼
+----------------------+
| Spatial Fabric       |
|                      |
| transient tokens     |
| localized rules      |
+----------+-----------+
           |
           ▼
       output / memory
```

The spatial fabric becomes an intermediate execution structure.

It can transform information while minimizing unnecessary persistent storage at intermediate stages.

This is conceptually similar to a pipeline, but the spatial architecture can make the processing topology itself part of the computation.

---

# 19. Persistent Versus Transient State

This distinction may be one of the most important principles emerging from the architecture.

A system contains at least two fundamentally different classes of state:

```text
PERSISTENT STATE

    Configuration
    Memory
    Rules
    Program state
    Long-lived data


TRANSIENT STATE

    Pipeline data
    Intermediate values
    Tokens
    Temporary results
    In-flight information
```

These states do not necessarily need the same physical implementation.

Persistent state should be optimized for:

- retention;
- density;
- reliable access.

Transient state can potentially be optimized for:

- movement;
- latency;
- local transformation;
- low switching cost;
- minimal temporary storage.

This separation could be useful even in entirely conventional binary systems.

---

# 20. Rule Reuse and Amortization

A complex processing tile becomes more attractive when it can be reused many times.

Suppose a tile implements a relatively expensive transformation.

If only one token uses it, the physical cost may not be justified.

If thousands or millions of tokens pass through it:

```text
              ┌──────────────┐
Token 1 ─────►│              │
Token 2 ─────►│   RULE TILE  │
Token 3 ─────►│              │
Token 4 ─────►│              │
Token 5 ─────►│              │
              └──────────────┘
```

the cost of the fixed rule can be amortized over many operations.

This is a central reason why spatial processing may be attractive for streaming workloads.

---

# 21. Data Movement as Computation

A more radical interpretation is that movement itself can become part of the computation.

For example:

```text
position = state
direction = operation
tile = rule
arrival = event
```

Under this interpretation, the physical path taken by a token is not merely communication.

It can encode part of the algorithm.

A token arriving at one tile can represent a different computational state from the same token arriving at another tile.

This produces a spatial computational model in which:

> **where the information is can be part of what the information means.**

This does not replace conventional instruction execution in general.

It provides another computational representation for selected problems.

---

# 22. Compatibility With Existing Fabrication Technologies

One of the most important research questions is whether such an architecture can be implemented without requiring a fundamentally different semiconductor manufacturing process.

The hypothesis is that the underlying devices can remain conventional CMOS devices.

The architectural changes would instead involve:

- placement;
- routing;
- processing-element organization;
- local memory;
- interconnect;
- control;
- pipeline structure;
- data representation.

This could make the concept compatible with mature semiconductor nodes as well as modern nodes, subject to actual physical design constraints.

However, compatibility with a fabrication process does not imply economic or technical superiority.

The architecture must still satisfy:

- design-rule constraints;
- timing;
- routing density;
- power limits;
- thermal constraints;
- manufacturing variability;
- yield;
- testability.

---

# 23. A Possible Conventional CPU Integration

A practical implementation could look like:

```text
                 GENERAL-PURPOSE CPU
                         │
        ┌────────────────┼────────────────┐
        │                │                │
      Cache            ALU             Control
        │                │                │
        └────────────────┼────────────────┘
                         │
                  Accelerator API
                         │
                         ▼
              +---------------------+
              | Spatial Accelerator |
              |                     |
              | PE ─ PE ─ PE ─ PE  |
              | │    │    │    │    |
              | PE ─ PE ─ PE ─ PE  |
              | │    │    │    │    |
              | PE ─ PE ─ PE ─ PE  |
              +---------------------+
                         │
                         ▼
                       Result
```

The CPU remains responsible for general-purpose execution.

The spatial accelerator becomes a specialized computational substrate.

The same concept could potentially be implemented as:

- an on-chip accelerator;
- a coprocessor;
- a tightly coupled execution unit;
- a reconfigurable fabric;
- an FPGA-like region;
- a dedicated ASIC block.

---

# 24. Theoretical Minimum Versus Practical Minimum

The concept of a minimum transistor count must be treated carefully.

There may be a theoretical minimum associated with representing information.

There is a different practical minimum associated with reliably operating a circuit.

And there is another system-level minimum associated with achieving useful computation.

These should not be confused.

```text
Information-theoretic minimum
            ↓
Device-level implementation
            ↓
Reliable circuit
            ↓
Processing element
            ↓
Complete spatial fabric
            ↓
Useful system
```

A design that minimizes the first quantity may perform poorly at the last level.

Therefore the project should measure the complete system.

---

# 25. Energy Model

A useful energy model should include at least:

```text
E_total =
    E_token_storage
  + E_transport
  + E_routing
  + E_processing
  + E_control
  + E_clock
  + E_memory
  + E_conversion
```

The exact model will depend on the implementation.

For a spatial architecture, the most important quantity may become:

```text
E_transport / useful operation
```

rather than simply:

```text
E_storage / bit
```

This is an important shift in perspective.

---

# 26. Performance Model

Performance should similarly be evaluated at system level.

Relevant metrics include:

- token throughput;
- latency;
- operations per cycle;
- operations per watt;
- tokens per second;
- area;
- energy per token;
- energy per useful operation;
- routing overhead;
- memory traffic;
- parallelism;
- scalability.

A smaller token is not automatically a faster token.

A smaller circuit is not automatically a more efficient circuit.

The architecture must be evaluated using complete workloads.

---

# 27. Research Hypotheses

The broader research program can formulate the following hypotheses.

### H1 — Transient State Efficiency

Transient information can be represented with substantially lower physical storage overhead than equivalent persistent state under suitable timing and reliability constraints.

### H2 — Spatial Locality

For selected workloads, localized processing can reduce the cost of data movement compared with centralized execution.

### H3 — Rule Reuse

Fixed or semi-static processing structures can amortize their physical complexity across many transient tokens.

### H4 — Conventional CMOS Compatibility

A spatial transient-token architecture can be implemented using conventional semiconductor devices and fabrication technologies without requiring a fundamentally new transistor technology.

### H5 — Hybrid Advantage

A conventional processor combined with a specialized spatial fabric can outperform either architecture alone for selected workloads.

### H6 — Representation Independence

The potential benefits of transient spatial processing can exist independently of whether the logical data representation is binary or ternary.

### H7 — Ternary Interaction

Ternary representation may provide additional benefits when combined with spatial processing, but those benefits must be distinguished from benefits caused by spatial organization itself.

---

# 28. Falsification

The architecture should be considered unsuccessful for a given workload if:

- transient storage requires excessive supporting circuitry;
- leakage dominates;
- refresh overhead dominates;
- routing consumes excessive area;
- synchronization dominates latency;
- token movement consumes more energy than conventional movement;
- rule reuse is insufficient;
- the spatial fabric cannot scale;
- a conventional processor achieves better performance and energy under equivalent constraints.

These outcomes are not failures of the research methodology.

They are valid experimental results.

---

# 29. Required Experimental Comparisons

A fair evaluation should compare equivalent workloads under equivalent assumptions.

At minimum:

```text
Conventional Binary
        versus
Binary Spatial

Conventional Binary
        versus
Hybrid Binary + Spatial

Conventional Ternary
        versus
Ternary Spatial

Binary Spatial
        versus
Ternary Spatial
```

The experiments should measure:

```text
Performance
Area
Energy
Memory traffic
Routing activity
Control overhead
Parallelism
Scalability
```

The architecture should not be considered successful merely because one metric improves.

---

# 30. Relationship to TernaryBreath

This research direction can be integrated naturally into TernaryBreath, but it should not be considered dependent on ternary logic.

TernaryBreath can therefore investigate several independent dimensions:

```text
                  TERNARYBREATH RESEARCH SPACE

                         Representation
                         /            \
                    Binary          Ternary
                       │                │
                       └──────┬─────────┘
                              │
                       Organization
                       /             \
              Conventional         Spatial
                       │               │
                       └──────┬────────┘
                              │
                         Hybrid System
```

This allows the project to ask:

> Does the benefit come from representation?

> Does the benefit come from spatial organization?

> Does the benefit come from lightweight transient data movement?

> Does the benefit come from fixed rule reuse?

> Do these mechanisms reinforce one another?

The value of the research therefore does not depend on any single architectural assumption being correct.

---

# 31. Broader Architectural Principle

The broader principle emerging from this research can be stated as:

> **Optimize the physical representation of information according to its lifetime and role in the computation.**

Persistent information should be optimized for retention.

Transient information should be optimized for movement.

Processing structures should be optimized for computation and reuse.

This produces a three-part architectural model:

```text
              INFORMATION LIFETIME

        Persistent       Transient
             │               │
             ▼               ▼
        Memory Layer     Transport Layer
             │               │
             └───────┬───────┘
                     ▼
              Processing Layer
                     │
                     ▼
                  Result
```

This principle could potentially apply far beyond TernaryBreath.

---

# 32. Research Scope

The research should initially focus on small, measurable systems.

A possible progression is:

```text
Logical token model
        ↓
Transient transport model
        ↓
Single processing tile
        ↓
Linear spatial pipeline
        ↓
Small 2D mesh
        ↓
Binary implementation
        ↓
Ternary implementation
        ↓
Hybrid CPU + spatial accelerator
        ↓
Physical implementation
```

This progression allows each architectural assumption to be evaluated before introducing additional complexity.

---

# 33. Hardware Validation Roadmap

A suitable validation path is:

```text
Concept
   ↓
Logical model
   ↓
Cycle-accurate simulator
   ↓
RTL
   ↓
Logic synthesis
   ↓
Place and Route
   ↓
Timing analysis
   ↓
Power estimation
   ↓
FPGA prototype
   ↓
Physical characterization
   ↓
ASIC feasibility study
```

Physical claims should only be made after appropriate physical validation.

Simulation results should remain clearly identified as simulation results.

Theoretical transistor counts should remain clearly identified as estimates.

---

# 34. What This Research Does Not Claim

This architecture does not claim that:

- one transistor is sufficient to implement a universally usable bit;
- eight transistors are sufficient to implement a practical 8-bit register;
- dynamic logic is automatically superior to SRAM;
- transient storage is automatically lower power;
- spatial movement is free;
- spatial computation is universally faster;
- spatial computation is universally more energy efficient;
- a spatial accelerator will outperform a CPU for general-purpose workloads;
- ternary representation is automatically advantageous;
- conventional CMOS fabrication automatically makes the architecture economical.

The purpose is to establish experimentally whether the proposed separation of persistent state, transient transport, and localized computation produces measurable benefits.

---

# 35. Central Research Question

The entire architectural direction can be summarized by one question:

> **Can a computing system reduce the physical cost of data movement by representing transient information with lightweight transport structures while keeping computational complexity localized in reusable spatial processing elements?**

A second question follows:

> **Can this architecture be integrated with conventional binary processors without fundamentally changing semiconductor fabrication technology?**

A third question extends the investigation:

> **Can binary and ternary representations coexist within the same spatial substrate, and under what workloads does either representation provide an advantage?**

These questions define an open research program rather than a predetermined architecture.

---

# 36. Final Perspective

The most important idea is not the claim that a bit can be implemented with a particular number of transistors.

The deeper idea is that **a bit has different physical requirements depending on what the system asks that bit to do**.

A persistent bit may need reliable long-term storage.

A transient bit may only need to survive a controlled transfer.

A computational rule may need significantly more physical complexity but can remain fixed and serve many tokens.

This suggests a possible architectural separation:

```text
                DATA
                 │
                 │
          lightweight token
                 │
                 ▼
        ┌─────────────────┐
        │ Spatial Transport│
        └────────┬────────┘
                 │
                 ▼
        ┌─────────────────┐
        │ Fixed Rule Tile │
        └────────┬────────┘
                 │
                 ▼
        transformed token
                 │
                 ▼
        next spatial stage
```

The architecture therefore attempts to move **information rather than complete computational state**.

Whether this provides a real advantage is an empirical question.

The answer may be positive for some workloads, neutral for others, or negative in many cases.

That uncertainty is precisely what makes the concept suitable for research.

The same architectural principle can be studied in:

- conventional binary processors;
- spatial binary processors;
- ternary processors;
- ternary spatial processors;
- accelerators;
- reconfigurable hardware;
- hybrid CPU/spatial systems.

The objective is not to replace conventional computing with a single new paradigm.

The objective is to determine whether **matching the physical implementation of information to its lifetime, movement pattern, and computational role can produce measurable improvements in area, energy, latency, throughput, or scalability.**

For TernaryBreath, this provides an additional research dimension:

> **The potential value of the architecture may come from representation, from spatial organization, from lightweight transient data movement, from fixed rule reuse, from their combination, or from none of them.**

Only controlled experiments can determine which of these mechanisms, if any, provides a genuine advantage.
