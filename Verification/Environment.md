# 8. Environment (`environment.sv`)

```systemverilog
`include "transaction.sv"
`include "generator.sv"
`include "driver.sv"
`include "monitor.sv"
`include "scoreboard.sv"

class Environment;

  Transaction t;

  Generator gen;
  Driver driv;
  Monitor mon;
  Scoreboard scb;

  event next;

  mailbox #(Transaction) gen2driv;
  mailbox #(Transaction) mon2scb;
  mailbox #(Transaction) rerf_value;

  virtual RAM_intf vif;


  function new(virtual RAM_intf vif,
               int count);

    this.vif = vif;

    t = new();

    // Mailboxes
    gen2driv  = new();
    mon2scb   = new();
    rerf_value = new();

    // Component Creation
    gen  = new(gen2driv, rerf_value);

    gen.tran_count = count;

    driv = new(gen2driv, vif);

    mon  = new(mon2scb, vif);

    scb  = new(mon2scb, rerf_value);

    // Synchronization Events
    scb.scbnext = next;

    gen.scbnext = next;

  endfunction


  // ---------------------------------------------------------
  // Test Execution
  // ---------------------------------------------------------

  task test();

    fork

      gen.run();

      driv.run();

      mon.run();

      scb.run();

    join_any

  endtask


  // ---------------------------------------------------------
  // Coverage Report
  // ---------------------------------------------------------

  task post_test();

    wait(gen.done.triggered);

    $display("\n----------------------------------");
    $display("Coverage Results");
    $display("----------------------------------");

    $display("Enable Coverage   : %0.2f%%",
              t.get_inp1_cov());

    $display("Write Coverage    : %0.2f%%",
              t.get_inp2_cov());

    $display("Data Coverage     : %0.2f%%",
              t.get_inp3_cov());

    $display("Address Coverage  : %0.2f%%",
              t.get_inp4_cov());

    $display("Total Coverage    : %0.2f%%",
              t.get_total_cov());

    $display("----------------------------------\n");

    $finish();

  endtask


  // ---------------------------------------------------------
  // Run Task
  // ---------------------------------------------------------

  task run();

    test();

    post_test();

  endtask

endclass
```

# Line-by-Line Explanation of `environment.sv`

```systemverilog
`include "transaction.sv"
`include "generator.sv"
`include "driver.sv"
`include "monitor.sv"
`include "scoreboard.sv"
```

# Explanation
- Includes all verification component files.

### Purpose
- Makes class definitions visible to compiler.

| File | Purpose |
|---|---|
| `transaction.sv` | Transaction object |
| `generator.sv` | Stimulus generation |
| `driver.sv` | DUT driving |
| `monitor.sv` | DUT observation |
| `scoreboard.sv` | Result checking |

---

# Environment Class

```systemverilog
class Environment;
```

# Explanation
- Declares `Environment` class.
- Top-level verification container.

### Purpose
- Connects all verification components together.

---

# Transaction Handle

```systemverilog
Transaction t;
```

## Explanation
- Transaction object handle.
- Used for coverage reporting.

---

# Component Handles

```systemverilog
Generator gen;
```

## Explanation
- Handle for Generator component.

---

```systemverilog
Driver driv;
```

## Explanation
- Handle for Driver component.

---

```systemverilog
Monitor mon;
```

## Explanation
- Handle for Monitor component.

---

```systemverilog
Scoreboard scb;
```

## Explanation
- Handle for Scoreboard component.

---

# Event Declaration

```systemverilog
event next;
```

## Explanation
- Synchronization event shared between:
  
```text
Generator ↔ Scoreboard
```

Used to control transaction flow.

---

# Mailboxes

```systemverilog
mailbox #(Transaction) gen2driv;
```

## Explanation
- Mailbox between:
  
```text
Generator → Driver
```

Transfers randomized transactions.

---

```systemverilog
mailbox #(Transaction) mon2scb;
```

## Explanation
- Mailbox between:
  
```text
Monitor → Scoreboard
```

Transfers DUT-observed transactions.

---

```systemverilog
mailbox #(Transaction) rerf_value;
```

## Explanation
- Mailbox between:
  
```text
Generator → Scoreboard
```

Transfers reference transactions.

---

# Virtual Interface

```systemverilog
virtual RAM_intf vif;
```

## Explanation
- Virtual interface handle.

Used to connect class-based components with DUT interface signals.

---

# Constructor

```systemverilog
function new(virtual RAM_intf vif,
             int count);
```

# Explanation
- Constructor function.
- Initializes environment.

### Inputs

| Input | Purpose |
|---|---|
| `vif` | DUT interface |
| `count` | Number of transactions |

---

```systemverilog
this.vif = vif;
```

## Explanation
- Assigns interface handle.

---

# Transaction Object Creation

```systemverilog
t = new();
```

## Explanation
- Creates transaction object.

Used later for coverage reporting.

---

# Mailbox Creation

```systemverilog
gen2driv = new();
```

## Explanation
- Creates generator-to-driver mailbox.

---

```systemverilog
mon2scb = new();
```

## Explanation
- Creates monitor-to-scoreboard mailbox.

---

```systemverilog
rerf_value = new();
```

## Explanation
- Creates reference mailbox.

---

# Component Creation

```systemverilog
gen = new(gen2driv, rerf_value);
```

## Explanation
- Creates Generator object.

### Inputs
- driver mailbox
- reference mailbox

---

# Transaction Count Assignment

```systemverilog
gen.tran_count = count;
```

## Explanation
- Sets total transactions generator should create.

Example:

```systemverilog
env = new(inf, 300);
```

Then:

```text
tran_count = 300
```

---

# Driver Creation

```systemverilog
driv = new(gen2driv, vif);
```

## Explanation
- Creates Driver object.

### Inputs
- generator mailbox
- virtual interface

---

# Monitor Creation

```systemverilog
mon = new(mon2scb, vif);
```

## Explanation
- Creates Monitor object.

### Inputs
- monitor mailbox
- virtual interface

---

# Scoreboard Creation

```systemverilog
scb = new(mon2scb, rerf_value);
```

## Explanation
- Creates Scoreboard object.

### Inputs
- monitor mailbox
- reference mailbox

---

# Event Synchronization

```systemverilog
scb.scbnext = next;
```

## Explanation
- Connects environment event to scoreboard.

---

```systemverilog
gen.scbnext = next;
```

## Explanation
- Connects same event to generator.

### Purpose
- Synchronizes:
  
```text
Generator ↔ Scoreboard
```

Generator waits until scoreboard finishes checking.

---

```systemverilog
endfunction
```

## Explanation
- Ends constructor.

---

# Test Task

```systemverilog
task test();
```

## Explanation
- Main test execution task.

---

# Parallel Execution

```systemverilog
fork
```

## Explanation
- Starts parallel execution.

All verification components run simultaneously.

---

# Generator Execution

```systemverilog
gen.run();
```

## Explanation
- Starts generator.

---

# Driver Execution

```systemverilog
driv.run();
```

## Explanation
- Starts driver.

---

# Monitor Execution

```systemverilog
mon.run();
```

## Explanation
- Starts monitor.

---

# Scoreboard Execution

```systemverilog
scb.run();
```

## Explanation
- Starts scoreboard.

---

```systemverilog
join_any
```

## Explanation
- Simulation continues when any thread finishes.

Usually generator completes first.

---

```systemverilog
endtask
```

## Explanation
- Ends `test()` task.

---

# Post-Test Task

```systemverilog
task post_test();
```

## Explanation
- Displays coverage report after simulation.

---

# Wait for Generator Completion

```systemverilog
wait(gen.done.triggered);
```

## Explanation
- Waits until generator finishes all transactions.

---

# Coverage Report Header

```systemverilog
$display("\n----------------------------------");
```

## Explanation
- Prints formatting line.

---

```systemverilog
$display("Coverage Results");
```

## Explanation
- Prints report title.

---

# Enable Coverage

```systemverilog
t.get_inp1_cov()
```

## Explanation
- Returns enable signal coverage percentage.

---

# Write Coverage

```systemverilog
t.get_inp2_cov()
```

## Explanation
- Returns write-enable coverage.

---

# Data Coverage

```systemverilog
t.get_inp3_cov()
```

## Explanation
- Returns data coverage.

---

# Address Coverage

```systemverilog
t.get_inp4_cov()
```

## Explanation
- Returns address coverage.

---

# Total Coverage

```systemverilog
t.get_total_cov()
```

## Explanation
- Returns overall functional coverage.

---

# Finish Simulation

```systemverilog
$finish();
```

## Explanation
- Ends simulation.

---

```systemverilog
endtask
```

## Explanation
- Ends `post_test()` task.

---

# Run Task

```systemverilog
task run();
```

## Explanation
- Top-level environment execution task.

---

# Execute Test

```systemverilog
test();
```

## Explanation
- Starts all verification components.

---

# Display Coverage

```systemverilog
post_test();
```

## Explanation
- Prints final coverage report.

---

```systemverilog
endtask
```

## Explanation
- Ends `run()` task.

---

```systemverilog
endclass
```

## Explanation
- Ends Environment class.

---

# Overall Environment Flow

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

# Verification Component Connections

| Component | Connected To |
|---|---|
| Generator | Driver + Scoreboard |
| Driver | DUT |
| Monitor | DUT + Scoreboard |
| Scoreboard | Generator + Monitor |

---

# Purpose of Environment Class

| Feature | Purpose |
|---|---|
| Component Creation | Builds verification environment |
| Mailbox Connections | Enables communication |
| Synchronization | Controls transaction flow |
| Parallel Execution | Runs components simultaneously |
| Coverage Reporting | Displays verification completeness |

---

# Important Concepts Used

| Concept | Purpose |
|---|---|
| Environment | Top-level verification container |
| Mailbox | Inter-component communication |
| Virtual Interface | DUT signal access |
| Fork-Join | Parallel execution |
| Event Synchronization | Coordinated execution |
| Functional Coverage | Verification measurement |

---

# Why Environment is Important

Without environment:
- components remain disconnected
- no communication occurs
- verification flow cannot operate

Environment acts as:

```text
Verification Testbench Controller
```

that manages the complete verification system.

---
