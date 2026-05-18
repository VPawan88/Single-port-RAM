# Verification of Single-Port RAM using SystemVerilog

This project verifies a parameterized Single-Port RAM using SystemVerilog-based verification methodology.  
The verification environment includes:

- Generator
- Driver
- Monitor
- Scoreboard
- Interface
- Assertions
- Functional Coverage
- Mailbox-based communication

The DUT supports:
- Synchronous Read
- Synchronous Write
- Configurable data width
- Configurable address width

---

# 1. Top Testbench (`top_tb.sv`)

```systemverilog
`include "test.sv"
`include "interface.sv"

module top_tb;

  always #5 inf.clk = ~inf.clk;

  initial begin
    $dumpfile("dump.vcd");
    $dumpvars;
    inf.clk = 1'b0;
  end

  RAM_intf inf();

  test t1(inf);

  RAM_64B DUT(
      .clk(inf.clk),
      .en(inf.en),
      .we(inf.we),
      .data_in(inf.data_in),
      .address(inf.address),
      .data_out(inf.data_out)
  );

endmodule
```

## Explanation

### `include`
- Includes test and interface files.

### `always #5 inf.clk = ~inf.clk;`
- Generates clock with 10ns period.

### `$dumpfile("dump.vcd");`
- Creates waveform dump file.

### `$dumpvars;`
- Dumps simulation variables for waveform viewing.

### `RAM_intf inf();`
- Instantiates interface.

### `test t1(inf);`
- Creates test instance.

### `RAM_64B DUT(...)`
- Instantiates DUT.

---

# 2. Interface (`interface.sv`)

```systemverilog
interface RAM_intf #(parameter width = 8, addr_width = 6);

  logic clk;
  logic en;
  logic we;
  logic [width-1:0] data_in;
  logic [addr_width-1:0] address;
  logic [width-1:0] data_out;
```

## Explanation

- Interface groups DUT signals together.
- Simplifies DUT and verification component connections.

---

# Assertions

## Assertion 1

```systemverilog
property p_data_out_is_z_when_en_low;
  @(posedge clk) disable iff (en === 1'bx)
  (!en) |=> (data_out === {width{1'bz}});
endproperty
```

### Purpose
- Checks whether output becomes high impedance when enable is low.

---

## Assertion 2

```systemverilog
property p_no_data_out_on_write;
  @(posedge clk) disable iff (!en)
  (en && we) |=> (data_out === {width{1'b0}});
endproperty
```

### Purpose
- Ensures no valid data appears during write operation.

---

## Assertion 3

```systemverilog
property p_no_activity_when_disabled;
  @(posedge clk)
  (!en) |=> (data_out === {width{1'b0}});
endproperty
```

### Purpose
- Ensures no activity occurs when RAM is disabled.

---

# 3. Transaction Class (`transaction.sv`)

```systemverilog
class Transaction #(parameter width = 8,addr_width = 6);
```

## Purpose
- Stores stimulus and response data.

---

# Random Variables

```systemverilog
rand logic en;
rand logic we;
randc logic [width-1:0] data_in;
randc logic [addr_width-1:0] address;
```

## Explanation

### `rand`
- Generates random values.

### `randc`
- Generates cyclic random values without repetition.

---

# Functional Coverage

```systemverilog
covergroup cg;
  inp1 : coverpoint en;
  inp2 : coverpoint we;
  inp3 : coverpoint data_in;
  inp4 : coverpoint address;
endgroup
```

## Purpose
- Measures verification completeness.

Coverage collected for:
- Enable signal
- Write enable
- Input data
- Address values

---

# Constraints

```systemverilog
constraint c1{en dist {0:=1,1:=9};}
constraint c3{we dist {0:=5,1:=5};}
constraint c2{data_in>0;}
```

## Explanation

### `en dist`
- Enable is active most of the time.

### `we dist`
- Equal probability for read and write.

### `data_in > 0`
- Avoids zero data generation.

---

# Randomization

```systemverilog
function void RandomGen();
  this.randomize();
  cg.sample();
endfunction
```

## Purpose
- Generates randomized transactions.
- Samples functional coverage.

---

# 4. Generator (`generator.sv`)

```systemverilog
class Generator;
```

## Purpose
- Generates randomized transactions.

---

# Mailbox Communication

```systemverilog
mailbox #(Transaction) gen2driv;
mailbox #(Transaction) rerf_value;
```

## Explanation

### `gen2driv`
- Sends transactions to driver.

### `rerf_value`
- Sends reference transactions to scoreboard.

---

# Generator Run Task

```systemverilog
task run();
```

## Operations
- Randomizes transactions.
- Sends transactions to driver and scoreboard.
- Waits for scoreboard synchronization.

---

# 5. Driver (`driver.sv`)

```systemverilog
class Driver;
```

## Purpose
- Drives DUT signals.

---

# Virtual Interface

```systemverilog
virtual RAM_intf vif;
```

## Purpose
- Connects driver to DUT interface.

---

# Driver Operation

```systemverilog
gen2driv.get(t);
```

- Receives transaction from generator.

```systemverilog
vif.en <= t.en;
vif.we <= t.we;
vif.address <= t.address;
```

- Drives DUT signals.

---

# 6. Monitor (`monitor.sv`)

```systemverilog
class Monitor;
```

## Purpose
- Observes DUT activity.

---

# Monitor Operation

```systemverilog
t.en = vif.en;
t.we = vif.we;
t.data_in = vif.data_in;
t.address = vif.address;
t.data_out = vif.data_out;
```

- Captures DUT inputs and outputs.

```systemverilog
mon2scb.put(t.copy());
```

- Sends observed transaction to scoreboard.

---

# 7. Scoreboard (`scoreboard.sv`)

```systemverilog
class Scoreboard;
```

## Purpose
- Checks DUT correctness.

---

# Reference Memory

```systemverilog
logic [width-1:0] local_memory[*];
```

## Purpose
- Stores expected values.

---

# Write Checking

```systemverilog
local_memory[address_ref] = t.data_in;
```

- Updates reference memory during write.

---

# Read Checking

```systemverilog
t.data_out === local_memory[address_ref]
```

- Compares DUT output with expected value.

---

# Pass/Fail Messages

### PASS Example

```text
PASS - Data Read : Location - 5, Retrieved 100, Actual = 100
```

### FAIL Example

```text
FAIL - Data Read : Location - 5, Retrieved 20, Actual = 100
```

---

# 8. Environment (`environment.sv`)

```systemverilog
class Environment;
```

## Purpose
- Connects all verification components.

---

# Components Created

```systemverilog
Generator gen;
Driver driv;
Monitor mon;
Scoreboard scb;
```

---

# Mailboxes

```systemverilog
mailbox #(Transaction) gen2driv;
mailbox #(Transaction) mon2scb;
mailbox #(Transaction) rerf_value;
```

## Purpose
- Enables communication between components.

---

# Test Execution

```systemverilog
fork
  gen.run();
  driv.run();
  mon.run();
  scb.run();
join_any
```

## Purpose
- Runs all components concurrently.

---

# Coverage Reporting

```systemverilog
$display("TOTAL : %0.2f%%", t.get_total_cov());
```

## Purpose
- Displays functional coverage results.

---

# 9. Test (`test.sv`)

```systemverilog
program test(RAM_intf inf);
```

## Purpose
- Top-level verification control block.

---

# Environment Creation

```systemverilog
env = new(inf,20);
```

## Explanation
- Creates environment.
- Generates 20 randomized transactions.

---

# Test Execution

```systemverilog
env.run();
```

- Starts complete verification flow.

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
