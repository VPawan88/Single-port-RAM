# Verification of Single-Port RAM using SystemVerilog

This project verifies a parameterized Single-Port RAM using SystemVerilog-based verification methodology.  
The verification environment includes:

- Interface
- Generator
- Driver
- Scoreboard
- Monitor
- Assertions
- Functional Coverage
- Mailbox-based communication

The DUT supports:
- Synchronous Read
- Synchronous Write
- Configurable data width
- Configurable address width

---

# Important Concepts Used

| Concept | Purpose |
|---|---|
| Program Block | Testbench execution |
| Initial Block | Start simulation |
| Object Creation | Instantiate environment |
| Virtual Interface Passing | DUT-TB connection |
| Environment Control | Start verification flow |

---

# Complete Verification Hierarchy

```text
top_tb
   ↓
program test
   ↓
Environment
   ├── Generator
   ├── Driver
   ├── Monitor
   └── Scoreboard
```

---


# Verification Flow Summary

```text
Generator
   ↓
Driver
   ↓
DUT
   ↓
Monitor
   ↓
Scoreboard
```

---

# Features Implemented

| Feature | Status |
|---|---|
| Randomized Stimulus | Yes |
| Constrained Randomization | Yes |
| Functional Coverage | Yes |
| Assertions | Yes |
| Mailbox Communication | Yes |
| Scoreboard Checking | Yes |
| Parameterized Design | Yes |
| Synchronous RAM Verification | Yes |

---

# Tools Used

| Tool | Purpose |
|---|---|
| SystemVerilog | Design & Verification |
| GTKWave | Waveform Viewing |
| Icarus Verilog / QuestaSim | Simulation |

---

# Key Learning Outcomes

- SystemVerilog OOP Concepts
- Functional Coverage
- Assertions
- Mailbox-based Communication
- Verification Environment Architecture
- Randomized Verification
- Scoreboard-based Checking
- DUT Monitoring

---
