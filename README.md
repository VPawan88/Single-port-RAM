# Design and Verification of Single-Port RAM using Verilog/SystemVerilog

## Project Overview

This project focuses on the **design and verification of a parameterized Single-Port RAM** using **Verilog** and **SystemVerilog**.  
The RAM supports **synchronous write and synchronous read operations** and is verified using a **self-checking constrained-random verification environment**.

The objective of this project is to demonstrate:
- RTL design of memory architectures
- Functional verification methodology
- SystemVerilog OOP concepts
- Assertion-based verification
- Functional coverage collection
- Transaction-level communication using mailboxes

---

# Features

## RTL Design Features
- Parameterized RAM Design
- Configurable:
  - Data Width
  - Address Width
- Synchronous Write Operation
- Synchronous Read Operation
- Enable-Controlled Access
- Synthesizable RTL Code

---

## Verification Features
- Constrained Random Verification
- Self-Checking Testbench
- Functional Coverage
- Assertion-Based Verification (SVA)
- Mailbox-Based Communication
- Transaction-Level Modeling
- Reference Memory Model in Scoreboard
- Automatic PASS/FAIL Reporting
- Waveform Dumping using VCD

---

# RAM Specifications

| Parameter | Value |
|---|---|
| Memory Type | Single-Port RAM |
| Data Width | 8 bits |
| Address Width | 6 bits |
| Depth | 64 Locations |
| Read Operation | Synchronous |
| Write Operation | Synchronous |
| Clock Edge | Positive Edge Triggered |

---

# Design Description

The RAM is implemented using:
- Verilog RTL
- Parameterized architecture

## Functional Behavior

### Write Operation
When:

```text
en = 1
we = 1
```

- Input data is written into memory
- Output remains zero during write

---

### Read Operation
When:

```text
en = 1
we = 0
```

- Data is read from memory
- Output provides stored data from selected address

---

### Disabled State
When:

```text
en = 0
```

- RAM remains inactive
- Output is driven to zero

---

# Verification Architecture

The verification environment is built using **SystemVerilog class-based methodology**.

## Verification Components

### 1. Transaction
Represents RAM stimulus and response information.

Contains:
- Enable signal
- Write enable signal
- Address
- Input data
- Output data
- Functional coverage

---

### 2. Generator
Generates constrained-random transactions.

Features:
- Randomized stimulus generation
- Constraint application
- Coverage sampling
- Transaction synchronization

---

### 3. Driver
Drives transactions to DUT through virtual interface.

Responsibilities:
- Apply DUT inputs
- Synchronize with clock
- Handle read/write operations

---

### 4. Monitor
Observes DUT activity and captures outputs.

Responsibilities:
- Sample DUT signals
- Convert signals into transactions
- Send transactions to scoreboard

---

### 5. Scoreboard
Acts as reference model and checker.

Responsibilities:
- Maintain reference memory model
- Compare DUT output with expected output
- Display PASS/FAIL messages

---

### 6. Environment
Top-level verification container.

Responsibilities:
- Connect all components
- Create mailboxes
- Synchronize execution
- Generate coverage report

---

# Verification Flow

```text
Generator
    ↓
Driver
    ↓
DUT (RAM)
    ↓
Monitor
    ↓
Scoreboard
```

---

# Assertion-Based Verification

SystemVerilog Assertions (SVA) are implemented inside the interface.

Assertions verify:
- Output remains zero when disabled
- Output remains zero during write
- No invalid activity occurs when RAM is disabled

---

# Functional Coverage

Functional coverage is implemented using covergroups.

Coverage Points:
- Enable signal coverage
- Write enable coverage
- Address coverage
- Input data coverage

---

# Technologies Used

| Technology | Purpose |
|---|---|
| Verilog | RTL Design |
| SystemVerilog | Verification |
| Assertions (SVA) | Protocol Checking |
| Functional Coverage | Verification Metrics |
| Mailboxes | Component Communication |
| GTKWave | Waveform Viewing |
| Icarus Verilog / ModelSim | Simulation |

---

# Simulation Output

The environment generates:
- PASS/FAIL logs
- Functional coverage report
- VCD waveform dump

Waveforms can be viewed using:

```bash
gtkwave dump.vcd
```

---

# Project Structure

```text
├── RAM_64B.sv
├── interface.sv
├── transaction.sv
├── generator.sv
├── driver.sv
├── monitor.sv
├── scoreboard.sv
├── environment.sv
├── test.sv
├── top_tb.sv
└── README.md
```

---

# Key Learning Outcomes

- Verilog RTL Coding
- Parameterized Hardware Design
- SystemVerilog OOP Concepts
- Constrained Random Verification
- Functional Coverage
- Assertion-Based Verification
- Transaction-Level Communication
- Self-Checking Testbench Design
- Verification Component Integration

---

# Future Improvements

Possible future enhancements:
- UVM-based implementation
- Dual-Port RAM design
- Burst read/write support
- Error injection testing
- Memory initialization file support
- Coverage-driven constrained random testing
- Regression automation

---

# Conclusion

This project demonstrates a complete RTL-to-verification flow for a Single-Port RAM using Verilog and SystemVerilog.  
The verification environment successfully validates RAM functionality using constrained-random stimulus, assertions, functional coverage, and self-checking mechanisms, making the project a strong foundation for advanced digital design and verification methodologies.
