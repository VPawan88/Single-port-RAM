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

# 1. Top Testbench (`top_tb.sv`)

```systemverilog
`include "test.sv"
`include "interface.sv"

module top_tb;

  // Clock Generation
  always #5 inf.clk = ~inf.clk;

  // Dump File for GTKWave
  initial begin
    $dumpfile("dump.vcd");
    $dumpvars;
    inf.clk = 1'b0;
  end

  // Interface Instance
  RAM_intf inf();

  // Test Instance
  test t1(inf);

  // DUT Instance
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
# Line-by-Line Explanation of `top_tb.sv`

```systemverilog
`include "test.sv"
```

# Explanation
- Includes the `test.sv` file.

### Purpose
- Makes the `test` program block visible to compiler.

---

```systemverilog
`include "interface.sv"
```

# Explanation
- Includes the interface definition file.

### Purpose
- Makes `RAM_intf` visible to compiler.

---

# Top-Level Testbench Module

```systemverilog
module top_tb;
```

# Explanation
- Declares top-level testbench module.

### Purpose
- Connects:
  - DUT
  - interface
  - verification environment

This is the top-most simulation module.

---

# Clock Generation

```systemverilog
always #5 inf.clk = ~inf.clk;
```

# Explanation
- Generates clock signal continuously.

---

# How It Works

### `always`
- Infinite loop.

### `#5`
- Waits 5 time units.

### `~inf.clk`
- Inverts current clock value.

---

# Clock Timing

Clock toggles every:

\[
5ns
\]

Complete clock period:

\[
10ns
\]

---

# Clock Waveform

```text
Time(ns)   Clock
0          0
5          1
10         0
15         1
20         0
```

---

# Initial Block

```systemverilog
initial begin
```

# Explanation
- Executes once at simulation start.

Usually used for:
- initialization
- waveform dumping

---

# Dump File Creation

```systemverilog
$dumpfile("dump.vcd");
```

# Explanation
- Creates VCD waveform dump file.

### VCD
- Value Change Dump

Stores signal transitions for waveform viewing.

Generated file:

```text
dump.vcd
```

---

# Dump Variables

```systemverilog
$dumpvars;
```

# Explanation
- Dumps all simulation variables into VCD file.

### Purpose
- Enables waveform viewing in GTKWave.

---

# Clock Initialization

```systemverilog
inf.clk = 1'b0;
```

# Explanation
- Initializes clock to zero at time 0.

Without this:
- clock may start with unknown (`X`) value.

---

```systemverilog
end
```

# Explanation
- Ends initial block.

---

# Interface Instance

```systemverilog
RAM_intf inf();
```

# Explanation
- Creates interface instance named `inf`.

---

# Purpose of Interface

The interface groups DUT signals:

| Signal |
|---|
| clk |
| en |
| we |
| address |
| data_in |
| data_out |

---

# Why Interface is Useful

Without interface:
- many individual ports required
- difficult connectivity

With interface:
- cleaner design
- simpler verification

---

# Test Instance

```systemverilog
test t1(inf);
```

# Explanation
- Creates instance of `test` program.

### Inputs
- passes interface instance `inf`

---

# What Happens Here

The verification environment now gains access to:
- DUT signals
- clock
- inputs
- outputs

through interface.

---

# DUT Instance

```systemverilog
RAM_64B DUT(
```

# Explanation
- Instantiates DUT (Design Under Test).

DUT is:
- Single-Port RAM module.

---

# Clock Connection

```systemverilog
.clk(inf.clk),
```

# Explanation
- Connects DUT clock to interface clock.

---

# Enable Connection

```systemverilog
.en(inf.en),
```

# Explanation
- Connects enable signal.

---

# Write Enable Connection

```systemverilog
.we(inf.we),
```

# Explanation
- Connects write-enable signal.

---

# Data Input Connection

```systemverilog
.data_in(inf.data_in),
```

# Explanation
- Connects input data bus.

---

# Address Connection

```systemverilog
.address(inf.address),
```

# Explanation
- Connects address bus.

---

# Data Output Connection

```systemverilog
.data_out(inf.data_out)
```

# Explanation
- Connects output data bus.

---

```systemverilog
);
```

# Explanation
- Ends DUT port connections.

---

```systemverilog
endmodule
```

# Explanation
- Ends top-level testbench module.

---

# Overall Top-Level Flow

```text
top_tb
   ├── Interface
   ├── Clock Generator
   ├── DUT
   └── Test Program
            ↓
       Environment
            ↓
  Generator / Driver / Monitor / Scoreboard
```

---

# Role of `top_tb.sv`

| Component | Purpose |
|---|---|
| Clock Generator | Creates system clock |
| Interface Instance | Groups DUT signals |
| DUT Instance | Actual RAM design |
| Test Instance | Starts verification |
| Dumpfile | Generates waveforms |

---

# Important Concepts Used

| Concept | Purpose |
|---|---|
| Top Module | Simulation entry point |
| Interface | Signal grouping |
| Program Block | Verification execution |
| DUT Instantiation | Hardware testing |
| Clock Generation | Synchronization |
| VCD Dumping | Waveform viewing |

---

# Simulation Workflow

```text
top_tb starts
      ↓
Clock generated
      ↓
test program starts
      ↓
environment created
      ↓
generator creates transactions
      ↓
driver drives DUT
      ↓
monitor captures DUT outputs
      ↓
scoreboard verifies outputs
      ↓
coverage report generated
```

---

# Why `top_tb.sv` is Important

Without top_tb:
- DUT cannot connect to verification environment
- no clock generation
- no waveform dumping
- simulation cannot start

It acts as:

```text
Main Simulation Wrapper
```

for the complete verification project.

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

  // ---------------------------------------------------------
  // Assertions Block
  // ---------------------------------------------------------

  // 1. If enable is low, output should become zero
  property p_data_out_zero_when_en_low;
    @(posedge clk)
    disable iff (en === 1'bx)
    (!en) |=> (data_out === '0);
  endproperty

  assert property (p_data_out_zero_when_en_low)
    else $error("ASSERTION FAILED : data_out is not zero when en = 0");


  // 2. During write operation, output should remain zero
  property p_output_zero_during_write;
    @(posedge clk)
    disable iff (!en)
    (en && we) |=> (data_out === '0);
  endproperty

  assert property (p_output_zero_during_write)
    else $error("ASSERTION FAILED : data_out is not zero during write");


  // 3. When disabled, no read/write activity should occur
  property p_output_zero_when_disabled;
    @(posedge clk)
    (!en) |=> (data_out === '0);
  endproperty

  assert property (p_output_zero_when_disabled)
    else $error("ASSERTION FAILED : Unexpected activity when disabled");

endinterface
```

# Line-by-Line Explanation of `interface.sv`

```systemverilog
interface RAM_intf #(parameter width = 8, addr_width = 6);
```

## Explanation
- Declares an interface named `RAM_intf`.
- Interface is used to group signals between DUT and verification components.

### Parameters

```systemverilog
parameter width = 8
```
- Defines data width as 8 bits.

```systemverilog
parameter addr_width = 6
```
- Defines address width as 6 bits.

Since address width is 6:

\[
2^6 = 64
\]

So the RAM contains 64 memory locations.

---

# Signal Declarations

```systemverilog
logic clk;
```

## Explanation
- Clock signal used for synchronous operations.

---

```systemverilog
logic en;
```

## Explanation
- Enable signal.
- RAM operates only when `en = 1`.

---

```systemverilog
logic we;
```

## Explanation
- Write Enable signal.
- Determines read or write operation.

| we | Operation |
|---|---|
| 1 | Write |
| 0 | Read |

---

```systemverilog
logic [width-1:0] data_in;
```

## Explanation
- Input data bus.
- Width is determined by parameter `width`.

Default width:

\[
8 \text{ bits}
\]

---

```systemverilog
logic [addr_width-1:0] address;
```

## Explanation
- Address bus used to select memory locations.

Default width:

\[
6 \text{ bits}
\]

Possible addresses:

\[
0 \text{ to } 63
\]

---

```systemverilog
logic [width-1:0] data_out;
```

## Explanation
- Output data bus used during read operation.

---

# Assertions Block

```systemverilog
// ---------------------------------------------------------
// Assertions Block
// ---------------------------------------------------------
```

## Explanation
- Section used for assertion-based verification.
- Assertions automatically check DUT behavior during simulation.

---

# Assertion 1

```systemverilog
// 1. If enable is low, output should become zero
```

## Purpose
- Verifies DUT behavior when RAM is disabled.

---

```systemverilog
property p_data_out_zero_when_en_low;
```

## Explanation
- Declares a property named:
  
```text
p_data_out_zero_when_en_low
```

- Property defines a verification condition.

---

```systemverilog
@(posedge clk)
```

## Explanation
- Assertion is checked at every positive clock edge.

---

```systemverilog
disable iff (en === 1'bx)
```

## Explanation
- Disables assertion if `en` becomes unknown (`X`).

### Why?
- Prevents false assertion failures caused by unknown values.

---

```systemverilog
(!en) |=> (data_out === '0);
```

## Explanation

### `!en`
- Condition:
  
```text
en = 0
```

### `|=>`
- Non-overlapping implication operator.

Meaning:

```text
If condition before |=> is true,
then condition after |=> must be true
on the next clock cycle.
```

### `(data_out === '0)`
- Checks whether output becomes zero.

### `===`
- Case equality operator.
- Checks exact value including:
  - X
  - Z

---

```systemverilog
endproperty
```

## Explanation
- Ends property definition.

---

```systemverilog
assert property (p_data_out_zero_when_en_low)
```

## Explanation
- Activates the assertion.

- During simulation:
  - If property passes → no issue
  - If property fails → assertion error generated

---

```systemverilog
else $error("ASSERTION FAILED : data_out is not zero when en = 0");
```

## Explanation
- Prints error message if assertion fails.

---

# Assertion 2

```systemverilog
// 2. During write operation, output should remain zero
```

## Purpose
- Ensures no valid output appears during write operation.

---

```systemverilog
property p_output_zero_during_write;
```

## Explanation
- Defines property for write operation checking.

---

```systemverilog
@(posedge clk)
```

## Explanation
- Assertion checked at positive clock edge.

---

```systemverilog
disable iff (!en)
```

## Explanation
- Assertion disabled when RAM is not enabled.

---

```systemverilog
(en && we) |=> (data_out === '0);
```

## Explanation

### `(en && we)`
Checks:

| Signal | Value |
|---|---|
| en | 1 |
| we | 1 |

Meaning:
- RAM enabled
- Write operation active

### `|=>`
- Next-cycle implication.

### `(data_out === '0)`
- Output should remain zero during write.

---

```systemverilog
endproperty
```

## Explanation
- Ends property definition.

---

```systemverilog
assert property (p_output_zero_during_write)
```

## Explanation
- Activates assertion.

---

```systemverilog
else $error("ASSERTION FAILED : data_out is not zero during write");
```

## Explanation
- Prints error if assertion fails.

---

# Assertion 3

```systemverilog
// 3. When disabled, no read/write activity should occur
```

## Purpose
- Ensures RAM remains inactive when disabled.

---

```systemverilog
property p_output_zero_when_disabled;
```

## Explanation
- Defines property for disabled-state behavior.

---

```systemverilog
@(posedge clk)
```

## Explanation
- Assertion checked at every positive edge.

---

```systemverilog
(!en) |=> (data_out === '0);
```

## Explanation

### `!en`
- RAM disabled.

### `|=>`
- Next-cycle implication.

### `(data_out === '0)`
- Output should remain zero.

---

```systemverilog
endproperty
```

## Explanation
- Ends property definition.

---

```systemverilog
assert property (p_output_zero_when_disabled)
```

## Explanation
- Activates assertion.

---

```systemverilog
else $error("ASSERTION FAILED : Unexpected activity when disabled");
```

## Explanation
- Displays error if assertion fails.

---

```systemverilog
endinterface
```

## Explanation
- Marks end of interface definition.

---

# Summary of Assertions

| Assertion | Purpose |
|---|---|
| `p_data_out_zero_when_en_low` | Output should be zero when disabled |
| `p_output_zero_during_write` | Output should remain zero during write |
| `p_output_zero_when_disabled` | No activity when RAM disabled |

---

# Key Concepts Used

| Concept | Purpose |
|---|---|
| Interface | Groups DUT signals |
| Assertion | Automatic checking |
| Property | Defines behavior rules |
| `|=>` | Next-cycle implication |
| `===` | Exact comparison |
| `disable iff` | Temporarily disables assertion |
| `@(posedge clk)` | Clock synchronization |

---



# 3. Transaction Class (`transaction.sv`)

```systemverilog
class Transaction #(parameter width = 8,
                    addr_width = 6);

  rand logic en;
  rand logic we;

  randc logic [width-1:0] data_in;
  randc logic [addr_width-1:0] address;

  logic [width-1:0] data_out;

  // ---------------------------------------------------------
  // Functional Coverage
  // ---------------------------------------------------------

  covergroup cg;

    inp1 : coverpoint en;

    inp2 : coverpoint we;

    inp3 : coverpoint data_in;

    inp4 : coverpoint address;

  endgroup


  // ---------------------------------------------------------
  // Constraints
  // ---------------------------------------------------------

  // Enable is active more frequently
  constraint c1 {
    en dist {0 := 1, 1 := 9};
  }

  // Equal probability for read and write
  constraint c2 {
    we dist {0 := 5, 1 := 5};
  }

  // Avoid zero data
  constraint c3 {
    data_in > 0;
  }


  // ---------------------------------------------------------
  // Constructor
  // ---------------------------------------------------------

  function new();

    cg = new();

  endfunction


  // ---------------------------------------------------------
  // Display Function
  // ---------------------------------------------------------

  function void display(string name);

    $display($time,
             " [%s] en = %0d we = %0d address = %0d data_in = %0d data_out = %0d",
             name, en, we, address, data_in, data_out);

  endfunction


  // ---------------------------------------------------------
  // Randomization Function
  // ---------------------------------------------------------

  function void RandomGen();

    this.randomize();

    cg.sample();

  endfunction


  // ---------------------------------------------------------
  // Copy Function
  // ---------------------------------------------------------

  function Transaction copy();

    copy = new();

    copy.en       = this.en;
    copy.we       = this.we;
    copy.data_in  = this.data_in;
    copy.address  = this.address;
    copy.data_out = this.data_out;

    return copy;

  endfunction


  // ---------------------------------------------------------
  // Coverage Functions
  // ---------------------------------------------------------

  function real get_inp1_cov();
    return cg.inp1.get_coverage();
  endfunction

  function real get_inp2_cov();
    return cg.inp2.get_coverage();
  endfunction

  function real get_inp3_cov();
    return cg.inp3.get_coverage();
  endfunction

  function real get_inp4_cov();
    return cg.inp4.get_coverage();
  endfunction

  function real get_total_cov();
    return cg.get_coverage();
  endfunction

endclass
```
# Line-by-Line Explanation of `transaction.sv`

```systemverilog
class Transaction #(parameter width = 8,
                    addr_width = 6);
```

# Explanation
- Declares a class named `Transaction`.
- Used to store stimulus and response data.

### Parameters

```systemverilog
parameter width = 8
```
- Defines data width as 8 bits.

```systemverilog
parameter addr_width = 6
```
- Defines address width as 6 bits.

---

# Random Variables

```systemverilog
rand logic en;
```

## Explanation
- `rand` keyword makes variable randomizable.
- `en` will get random values during simulation.

Possible values:

| Value | Meaning |
|---|---|
| 0 | RAM Disabled |
| 1 | RAM Enabled |

---

```systemverilog
rand logic we;
```

## Explanation
- Randomizable write-enable signal.

Possible values:

| we | Operation |
|---|---|
| 1 | Write |
| 0 | Read |

---

```systemverilog
randc logic [width-1:0] data_in;
```

## Explanation

### `randc`
- Cyclic randomization.

### Difference between `rand` and `randc`

| Type | Behavior |
|---|---|
| rand | Values may repeat randomly |
| randc | Values do not repeat until all possibilities are exhausted |

For 8-bit data:

\[
2^8 = 256
\]

So:
- all 256 values appear once before repetition.

---

```systemverilog
randc logic [addr_width-1:0] address;
```

## Explanation
- Random cyclic address generation.

For 6-bit address:

\[
2^6 = 64
\]

Possible addresses:

```text
0 to 63
```

All addresses are generated before repetition occurs.

---

```systemverilog
logic [width-1:0] data_out;
```

## Explanation
- Stores DUT output during read operation.

---

# Functional Coverage

```systemverilog
covergroup cg;
```

## Explanation
- Declares a covergroup named `cg`.
- Used to measure verification completeness.

---

# Coverpoint 1

```systemverilog
inp1 : coverpoint en;
```

## Explanation
- Tracks coverage of enable signal.

Checks whether:
- `en = 0`
- `en = 1`

have both occurred.

---

# Coverpoint 2

```systemverilog
inp2 : coverpoint we;
```

## Explanation
- Tracks coverage of write enable signal.

Checks whether:
- read operations occurred
- write operations occurred

---

# Coverpoint 3

```systemverilog
inp3 : coverpoint data_in;
```

## Explanation
- Tracks input data coverage.

Checks how many data values have been generated.

For 8-bit data:

\[
256 \text{ possible values}
\]

---

# Coverpoint 4

```systemverilog
inp4 : coverpoint address;
```

## Explanation
- Tracks address coverage.

For 6-bit address:

\[
64 \text{ possible addresses}
\]

Checks how many addresses were accessed.

---

```systemverilog
endgroup
```

## Explanation
- Ends covergroup definition.

---

# Constraints Block

```systemverilog
constraint c1 {
  en dist {0 := 1, 1 := 9};
}
```

# Explanation

### `dist`
- Distribution operator.

Controls probability of generated values.

---

## Probability Distribution

| Value | Weight |
|---|---|
| 0 | 1 |
| 1 | 9 |

Meaning:
- `en = 1` generated more frequently.
- RAM stays enabled most of the time.

---

# Constraint 2

```systemverilog
constraint c2 {
  we dist {0 := 5, 1 := 5};
}
```

# Explanation

Equal probability for:
- read operations
- write operations

| we | Probability |
|---|---|
| 0 | 50% |
| 1 | 50% |

---

# Constraint 3

```systemverilog
constraint c3 {
  data_in > 0;
}
```

# Explanation
- Prevents generation of zero data.
- Avoids unnecessary repeated zero values.

---

# Constructor

```systemverilog
function new();
```

## Explanation
- Constructor function.
- Automatically called when object is created.

---

```systemverilog
cg = new();
```

## Explanation
- Creates covergroup object.

Without this:
- coverage collection will not work.

---

```systemverilog
endfunction
```

## Explanation
- Ends constructor.

---

# Display Function

```systemverilog
function void display(string name);
```

## Explanation
- Prints transaction information.
- Used for debugging and monitoring.

---

```systemverilog
$display($time,
```

## Explanation
- Prints current simulation time.

---

```systemverilog
" [%s] en = %0d we = %0d address = %0d data_in = %0d data_out = %0d",
```

## Explanation
- Formatted output string.

### Format Specifiers

| Specifier | Meaning |
|---|---|
| `%s` | String |
| `%0d` | Decimal value |

---

```systemverilog
name, en, we, address, data_in, data_out);
```

## Explanation
- Variables passed into display statement.

---

# Randomization Function

```systemverilog
function void RandomGen();
```

## Explanation
- Generates randomized transaction.

---

```systemverilog
this.randomize();
```

## Explanation
- Randomizes all `rand` and `randc` variables.
- Constraints are automatically applied.

---

```systemverilog
cg.sample();
```

## Explanation
- Samples functional coverage.

Without this:
- coverage values will not update.

---

```systemverilog
endfunction
```

## Explanation
- Ends randomization function.

---

# Copy Function

```systemverilog
function Transaction copy();
```

## Explanation
- Creates deep copy of transaction object.

---

```systemverilog
copy = new();
```

## Explanation
- Creates new transaction object.

---

```systemverilog
copy.en = this.en;
```

## Explanation
- Copies enable signal.

---

```systemverilog
copy.we = this.we;
```

## Explanation
- Copies write enable.

---

```systemverilog
copy.data_in = this.data_in;
```

## Explanation
- Copies input data.

---

```systemverilog
copy.address = this.address;
```

## Explanation
- Copies address.

---

```systemverilog
copy.data_out = this.data_out;
```

## Explanation
- Copies DUT output.

---

```systemverilog
return copy;
```

## Explanation
- Returns copied transaction object.

---

# Why Copy is Needed

Without copy:
- mailbox stores same object handle repeatedly.
- values may get overwritten.

Copy creates independent transaction objects.

---

# Coverage Getter Functions

```systemverilog
function real get_inp1_cov();
```

## Explanation
- Returns enable signal coverage percentage.

---

```systemverilog
return cg.inp1.get_coverage();
```

## Explanation
- Gets coverage value from coverpoint.

---

# Similar Functions

| Function | Purpose |
|---|---|
| `get_inp2_cov()` | Write-enable coverage |
| `get_inp3_cov()` | Data coverage |
| `get_inp4_cov()` | Address coverage |
| `get_total_cov()` | Total coverage |

---

# Total Coverage

```systemverilog
return cg.get_coverage();
```

## Explanation
- Returns overall covergroup coverage.

---

```systemverilog
endclass
```

## Explanation
- Ends Transaction class definition.

---

# Summary of Transaction Class

| Feature | Purpose |
|---|---|
| Random Variables | Generate stimulus |
| Constraints | Control stimulus behavior |
| Covergroup | Measure verification completeness |
| Copy Function | Prevent object overwrite |
| Display Function | Debugging |
| RandomGen | Generate constrained-random transactions |
| Coverage Functions | Report coverage |

---

# Important Concepts Used

| Concept | Purpose |
|---|---|
| `rand` | Random values |
| `randc` | Cyclic random values |
| Constraint | Restrict randomization |
| Covergroup | Functional coverage |
| Coverpoint | Coverage metric |
| Mailbox Copy | Safe transaction transfer |
| Constructor | Object initialization |

---


# 4. Generator (`generator.sv`)

```systemverilog
class Generator;

  Transaction t;

  mailbox #(Transaction) gen2driv;
  mailbox #(Transaction) rerf_value;

  int tran_count;

  event done;
  event scbnext;

  integer i = 0;

  function new(mailbox #(Transaction) gen2driv,
               mailbox #(Transaction) rerf_value);

    this.gen2driv  = gen2driv;
    this.rerf_value = rerf_value;

    t = new();

  endfunction


  task run();

    for(i = 0; i < tran_count; i = i + 1) begin

      t.RandomGen();

      gen2driv.put(t.copy());

      rerf_value.put(t.copy());

      t.display("Generator");

      @(scbnext);

    end

    ->done;

  endtask

endclass
```
# Line-by-Line Explanation of `generator.sv`

```systemverilog
class Generator;
```

# Explanation
- Declares a class named `Generator`.
- Responsible for generating randomized transactions.

The generator is the first active component in the verification environment.

---

# Transaction Handle

```systemverilog
Transaction t;
```

## Explanation
- Creates a handle for `Transaction` class.
- Used to store randomized stimulus.

---

# Mailboxes

```systemverilog
mailbox #(Transaction) gen2driv;
```

## Explanation
- Mailbox used for communication between:
  
```text
Generator → Driver
```

- Sends randomized transactions to driver.

### `#(Transaction)`
- Mailbox stores objects of type `Transaction`.

---

```systemverilog
mailbox #(Transaction) rerf_value;
```

## Explanation
- Mailbox used for communication between:
  
```text
Generator → Scoreboard
```

- Sends reference transactions directly to scoreboard.

### Why Needed?
- Scoreboard requires original stimulus values for comparison.

---

# Transaction Count

```systemverilog
int tran_count;
```

## Explanation
- Stores total number of transactions to generate.

Example:

```systemverilog
env = new(inf, 300);
```

Then:

```text
tran_count = 300
```

---

# Events

```systemverilog
event done;
```

## Explanation
- Event used to indicate:
  
```text
Generation completed
```

- Triggered after all transactions are generated.

---

```systemverilog
event scbnext;
```

## Explanation
- Synchronization event between:
  
```text
Scoreboard ↔ Generator
```

### Purpose
- Prevents generator from generating transactions too quickly.
- Ensures scoreboard processes current transaction before next one arrives.

---

# Loop Variable

```systemverilog
integer i = 0;
```

## Explanation
- Loop counter used inside `run()` task.

---

# Constructor

```systemverilog
function new(mailbox #(Transaction) gen2driv,
             mailbox #(Transaction) rerf_value);
```

## Explanation
- Constructor function.
- Automatically called when Generator object is created.

### Inputs
- Driver mailbox
- Scoreboard reference mailbox

---

```systemverilog
this.gen2driv = gen2driv;
```

## Explanation
- Assigns input mailbox to local class mailbox.

### `this`
- Refers to current class object.

---

```systemverilog
this.rerf_value = rerf_value;
```

## Explanation
- Assigns scoreboard mailbox.

---

```systemverilog
t = new();
```

## Explanation
- Creates Transaction object.

Without this:
- transaction object would not exist.

---

```systemverilog
endfunction
```

## Explanation
- Ends constructor.

---

# Run Task

```systemverilog
task run();
```

## Explanation
- Main generator task.
- Generates randomized transactions continuously.

---

# Loop

```systemverilog
for(i = 0; i < tran_count; i = i + 1)
```

## Explanation
- Generates transactions repeatedly.

### Loop Operation

| Statement | Meaning |
|---|---|
| `i = 0` | Start from 0 |
| `i < tran_count` | Continue until count reached |
| `i = i + 1` | Increment counter |

---

# Random Transaction Generation

```systemverilog
t.RandomGen();
```

## Explanation
- Calls transaction randomization function.

Inside `RandomGen()`:
- variables are randomized
- constraints are applied
- coverage is sampled

---

# Send Transaction to Driver

```systemverilog
gen2driv.put(t.copy());
```

## Explanation
- Sends copied transaction to driver mailbox.

### Why `copy()`?

Without copy:
- same object handle reused
- values may get overwritten

Copy creates independent transaction object.

---

# Send Reference Transaction to Scoreboard

```systemverilog
rerf_value.put(t.copy());
```

## Explanation
- Sends same transaction to scoreboard.

### Purpose
- Scoreboard uses this as reference model input.

---

# Display Transaction

```systemverilog
t.display("Generator");
```

## Explanation
- Prints generated transaction details.

Example output:

```text
[Generator] en = 1 we = 1 address = 10 data_in = 55
```

Useful for:
- debugging
- transaction tracing

---

# Synchronization Wait

```systemverilog
@(scbnext);
```

## Explanation
- Generator waits for scoreboard event.

### Meaning

```text
Do not generate next transaction
until scoreboard finishes current checking.
```

### Why Important?
- Prevents race conditions.
- Maintains transaction synchronization.

---

# End Loop

```systemverilog
end
```

## Explanation
- Ends `for` loop.

---

# Trigger Completion Event

```systemverilog
->done;
```

## Explanation
- Triggers `done` event.

### Meaning

```text
All transactions completed
```

Used by environment:

```systemverilog
wait(gen.done.triggered);
```

to know when simulation should stop.

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
- Ends Generator class.

---

# Generator Workflow

```text
Generate Random Transaction
            ↓
Apply Constraints
            ↓
Sample Coverage
            ↓
Send to Driver
            ↓
Send to Scoreboard
            ↓
Wait for Scoreboard
            ↓
Generate Next Transaction
```

---

# Purpose of Generator

| Feature | Purpose |
|---|---|
| Randomization | Creates stimulus |
| Constraints | Controls stimulus behavior |
| Mailbox Communication | Transfers transactions |
| Synchronization | Prevents race conditions |
| Coverage Sampling | Tracks verification progress |

---

# Important Concepts Used

| Concept | Purpose |
|---|---|
| Class | Verification component |
| Mailbox | Inter-process communication |
| Event | Synchronization |
| Randomization | Stimulus generation |
| Copy Function | Safe object transfer |
| Task | Time-consuming process |

---

# 5. Driver (`driver.sv`)

```systemverilog
class Driver;

  Transaction t;

  mailbox #(Transaction) gen2driv;

  virtual RAM_intf vif;

  function new(mailbox #(Transaction) gen2driv,
               virtual RAM_intf vif);

    this.gen2driv = gen2driv;
    this.vif = vif;

  endfunction


  task run();

    forever begin

      gen2driv.get(t);

      vif.en      <= t.en;
      vif.we      <= t.we;
      vif.address <= t.address;

      // Drive data only during write
      if (t.en == 1'b1 && t.we == 1'b1)
        vif.data_in <= t.data_in;
      else
        vif.data_in <= '0;

      t.display("Driver");

      #10;

    end

  endtask

endclass
```
# Line-by-Line Explanation of `driver.sv`

```systemverilog
class Driver;
```

# Explanation
- Declares a class named `Driver`.
- Driver is responsible for applying stimulus to the DUT.

The driver acts as a bridge between:
- Generator
- DUT Interface

---

# Transaction Handle

```systemverilog
Transaction t;
```

## Explanation
- Declares transaction object handle.
- Stores transaction received from generator.

---

# Mailbox Declaration

```systemverilog
mailbox #(Transaction) gen2driv;
```

## Explanation
- Mailbox used for communication between:
  
```text
Generator → Driver
```

- Driver receives transactions through this mailbox.

### `#(Transaction)`
- Mailbox stores `Transaction` objects.

---

# Virtual Interface

```systemverilog
virtual RAM_intf vif;
```

# Explanation
- Declares virtual interface handle.

### Purpose
- Connects driver to DUT signals indirectly.

Without virtual interface:
- class-based components cannot access DUT signals.

---

# Constructor

```systemverilog
function new(mailbox #(Transaction) gen2driv,
             virtual RAM_intf vif);
```

## Explanation
- Constructor function.
- Called automatically when driver object is created.

### Inputs
- Generator mailbox
- Virtual interface

---

```systemverilog
this.gen2driv = gen2driv;
```

## Explanation
- Assigns input mailbox to local mailbox handle.

### `this`
- Refers to current class object.

---

```systemverilog
this.vif = vif;
```

## Explanation
- Assigns virtual interface handle.

Now driver can access DUT signals using:
```systemverilog
vif.signal_name
```

---

```systemverilog
endfunction
```

## Explanation
- Ends constructor.

---

# Run Task

```systemverilog
task run();
```

## Explanation
- Main driver task.
- Continuously drives DUT inputs.

---

# Forever Loop

```systemverilog
forever begin
```

## Explanation
- Driver runs continuously throughout simulation.

### Why forever?
- Verification environment must continuously process transactions.

---

# Receive Transaction

```systemverilog
gen2driv.get(t);
```

## Explanation
- Receives transaction from generator mailbox.

### `get()`
- Blocking operation.
- Driver waits until transaction becomes available.

---

# Drive Enable Signal

```systemverilog
vif.en <= t.en;
```

## Explanation
- Drives enable signal to DUT.

### `<=`
- Non-blocking assignment.
- Preferred for sequential/timed behavior.

---

# Drive Write Enable

```systemverilog
vif.we <= t.we;
```

## Explanation
- Drives write-enable signal.

### Operation

| we | Operation |
|---|---|
| 1 | Write |
| 0 | Read |

---

# Drive Address

```systemverilog
vif.address <= t.address;
```

## Explanation
- Drives address signal to DUT.

Possible addresses:

\[
0 \text{ to } 63
\]

for:
```systemverilog
addr_width = 6
```

---

# Conditional Data Driving

```systemverilog
if (t.en == 1'b1 && t.we == 1'b1)
```

## Explanation
- Checks whether:
  
| Signal | Value |
|---|---|
| en | 1 |
| we | 1 |

Meaning:
- RAM enabled
- Write operation active

---

# Write Data to DUT

```systemverilog
vif.data_in <= t.data_in;
```

## Explanation
- Drives randomized input data to DUT.

Only occurs during write operation.

---

# Else Condition

```systemverilog
else
```

## Explanation
- Executes when:
  - read operation
  - disabled condition

---

# Clear Input Data

```systemverilog
vif.data_in <= '0;
```

## Explanation
- Drives zero when not writing.

### Why?
- During read operation:
  
```text
data_in is irrelevant
```

- Cleaner verification behavior.
- Avoids unknown/unwanted values.

---

# Display Transaction

```systemverilog
t.display("Driver");
```

## Explanation
- Prints driver transaction information.

Example output:

```text
[Driver] en = 1 we = 1 address = 5 data_in = 25
```

Useful for:
- debugging
- transaction tracing

---

# Delay

```systemverilog
#10;
```

## Explanation
- Waits for 10 time units before processing next transaction.

### Purpose
- Gives DUT time to respond.
- Synchronizes driver activity with clock.

Since clock period is:

```systemverilog
always #5 clk = ~clk;
```

Clock period:

\[
10ns
\]

So:
```systemverilog
#10
```

waits for one full clock cycle.

---

```systemverilog
end
```

## Explanation
- Ends forever loop body.

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
- Ends Driver class.

---

# Driver Workflow

```text
Receive Transaction
          ↓
Drive DUT Signals
          ↓
Drive Input Data
          ↓
Wait One Clock Cycle
          ↓
Receive Next Transaction
```

---

# Role of Driver in Verification

| Operation | Purpose |
|---|---|
| Receives Transactions | Gets randomized stimulus |
| Drives Interface Signals | Applies inputs to DUT |
| Controls Timing | Synchronizes operations |
| Supports Read/Write | Drives proper DUT behavior |

---

# Important Concepts Used

| Concept | Purpose |
|---|---|
| Mailbox | Generator-driver communication |
| Virtual Interface | Access DUT signals |
| Forever Loop | Continuous execution |
| Non-blocking Assignment | Sequential behavior |
| Conditional Driving | Write-only data transfer |
| Delay | Timing synchronization |

---


# 6. Monitor (`monitor.sv`)

```systemverilog
class Monitor;

  Transaction t;

  mailbox #(Transaction) mon2scb;

  virtual RAM_intf vif;


  function new(mailbox #(Transaction) mon2scb,
               virtual RAM_intf vif);

    this.mon2scb = mon2scb;
    this.vif     = vif;

    t = new();

  endfunction


  task run();

    forever begin

      @(posedge vif.clk);

      t.en       = vif.en;
      t.we       = vif.we;
      t.data_in  = vif.data_in;
      t.address  = vif.address;
      t.data_out = vif.data_out;

      mon2scb.put(t.copy());

      t.display("Monitor");

    end

  endtask

endclass
```
# Line-by-Line Explanation of `monitor.sv`

```systemverilog
class Monitor;
```

# Explanation
- Declares a class named `Monitor`.
- Monitor observes DUT activity.

The monitor is a passive component because:
- it does not drive signals
- it only samples DUT behavior

---

# Transaction Handle

```systemverilog
Transaction t;
```

## Explanation
- Declares transaction object handle.
- Used to store captured DUT signals.

---

# Mailbox Declaration

```systemverilog
mailbox #(Transaction) mon2scb;
```

## Explanation
- Mailbox used for communication between:
  
```text
Monitor → Scoreboard
```

- Monitor sends observed DUT transactions to scoreboard.

---

# Virtual Interface

```systemverilog
virtual RAM_intf vif;
```

## Explanation
- Declares virtual interface handle.

### Purpose
- Allows monitor to access DUT interface signals.

Without virtual interface:
- class cannot access DUT signals.

---

# Constructor

```systemverilog
function new(mailbox #(Transaction) mon2scb,
             virtual RAM_intf vif);
```

## Explanation
- Constructor function.
- Automatically called when monitor object is created.

### Inputs
- Monitor-to-scoreboard mailbox
- Virtual interface

---

```systemverilog
this.mon2scb = mon2scb;
```

## Explanation
- Assigns mailbox handle to local class variable.

### `this`
- Refers to current object.

---

```systemverilog
this.vif = vif;
```

## Explanation
- Assigns virtual interface handle.

Now monitor can access DUT signals using:

```systemverilog
vif.signal_name
```

---

# Transaction Object Creation

```systemverilog
t = new();
```

## Explanation
- Creates transaction object.

Without this:
- monitor cannot store sampled data.

---

```systemverilog
endfunction
```

## Explanation
- Ends constructor.

---

# Run Task

```systemverilog
task run();
```

## Explanation
- Main monitor task.
- Continuously observes DUT signals.

---

# Forever Loop

```systemverilog
forever begin
```

## Explanation
- Monitor runs continuously throughout simulation.

---

# Clock Synchronization

```systemverilog
@(posedge vif.clk);
```

## Explanation
- Monitor samples signals at positive clock edge.

### Why Important?
- DUT is synchronous.
- Sampling must occur at clock edge for accurate behavior observation.

---

# Capture Enable Signal

```systemverilog
t.en = vif.en;
```

## Explanation
- Copies DUT enable signal into transaction object.

---

# Capture Write Enable

```systemverilog
t.we = vif.we;
```

## Explanation
- Copies DUT write-enable signal.

---

# Capture Input Data

```systemverilog
t.data_in = vif.data_in;
```

## Explanation
- Captures DUT input data.

---

# Capture Address

```systemverilog
t.address = vif.address;
```

## Explanation
- Captures DUT address signal.

---

# Capture Output Data

```systemverilog
t.data_out = vif.data_out;
```

## Explanation
- Captures DUT output data.

This is important for:
- read verification
- scoreboard checking

---

# Send Transaction to Scoreboard

```systemverilog
mon2scb.put(t.copy());
```

## Explanation
- Sends copied transaction to scoreboard mailbox.

### Why `copy()`?
Without copy:
- same object reused repeatedly
- previous transaction values overwritten

Copy creates independent transaction object.

---

# Display Transaction

```systemverilog
t.display("Monitor");
```

## Explanation
- Prints monitored transaction details.

Example:

```text
[Monitor] en = 1 we = 0 address = 10 data_out = 55
```

Useful for:
- debugging
- tracing DUT activity

---

```systemverilog
end
```

## Explanation
- Ends forever loop body.

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
- Ends Monitor class.

---

# Monitor Workflow

```text
Wait for Clock Edge
          ↓
Sample DUT Signals
          ↓
Store in Transaction
          ↓
Send to Scoreboard
          ↓
Print Transaction
          ↓
Repeat
```

---

# Role of Monitor in Verification

| Function | Purpose |
|---|---|
| Observes DUT Signals | Captures actual DUT behavior |
| Creates Transactions | Converts signal activity into objects |
| Sends to Scoreboard | Enables checking |
| Passive Component | Does not drive DUT |

---

# Important Concepts Used

| Concept | Purpose |
|---|---|
| Virtual Interface | Access DUT signals |
| Mailbox | Communication with scoreboard |
| Passive Monitoring | Observe without driving |
| Clock Synchronization | Accurate sampling |
| Copy Function | Safe transaction transfer |
| Forever Loop | Continuous monitoring |

---

# Why Monitor is Important

Without monitor:
- scoreboard cannot know DUT output
- DUT behavior cannot be verified
- transaction-level checking becomes impossible

Monitor converts:
```text
Signal-Level Activity
          ↓
Transaction-Level Data
```

which simplifies verification.

---


# 7. Scoreboard (`scoreboard.sv`)

```systemverilog
class Scoreboard #(parameter width = 8,
                   addr_width = 6);

  Transaction t, tref;

  mailbox #(Transaction) mon2scb;
  mailbox #(Transaction) rerf_value;

  event scbnext;

  logic [width-1:0] local_memory[*];

  bit [addr_width-1:0] address_ref;


  function new(mailbox #(Transaction) mon2scb,
               mailbox #(Transaction) rerf_value);

    this.mon2scb    = mon2scb;
    this.rerf_value = rerf_value;

  endfunction


  task run();

    forever begin

      mon2scb.get(t);

      rerf_value.get(tref);

      t.display("Scoreboard");

      address_ref = unsigned'(t.address);

      // ---------------- WRITE CHECK ----------------

      if(t.en == 1'b1 && t.we == 1'b1) begin

        local_memory[address_ref] = t.data_in;

        $display("PASS - WRITE : Address = %0d Data = %0d",
                  t.address, t.data_in);

      end

      // ---------------- READ CHECK ----------------

      else if(t.en == 1'b1 &&
              t.we == 1'b0 &&
              t.data_out === local_memory[address_ref]) begin

        $display("PASS - READ : Address = %0d Expected = %0d Actual = %0d",
                  address_ref,
                  local_memory[address_ref],
                  t.data_out);

      end

      else if(t.en == 1'b1 &&
              t.we == 1'b0 &&
              t.data_out !== local_memory[address_ref]) begin

        $display("FAIL - READ : Address = %0d Expected = %0d Actual = %0d",
                  address_ref,
                  local_memory[address_ref],
                  t.data_out);

      end

      // ---------------- DISABLED CONDITION ----------------

      else if(t.en == 1'b0 &&
              t.data_out === '0) begin

        $display("PASS - DISABLED : data_out = %0d",
                  t.data_out);

      end

      else if(t.en == 1'b0 &&
              t.data_out !== '0) begin

        $display("FAIL - DISABLED : data_out = %0d",
                  t.data_out);

      end

      else begin

        $display("UNKNOWN CONDITION DETECTED");

      end

      ->scbnext;

    end

  endtask

endclass
```
# Line-by-Line Explanation of `scoreboard.sv`

```systemverilog
class Scoreboard #(parameter width = 8,
                   addr_width = 6);
```

# Explanation
- Declares a class named `Scoreboard`.
- Scoreboard is responsible for checking DUT correctness.

### Parameters

```systemverilog
parameter width = 8
```
- Data width = 8 bits.

```systemverilog
parameter addr_width = 6
```
- Address width = 6 bits.

---

# Transaction Handles

```systemverilog
Transaction t, tref;
```

## Explanation

### `t`
- Stores actual transaction received from monitor.

### `tref`
- Stores reference transaction received from generator.

---

# Mailboxes

```systemverilog
mailbox #(Transaction) mon2scb;
```

## Explanation
- Mailbox for communication:

```text
Monitor → Scoreboard
```

Contains actual DUT outputs.

---

```systemverilog
mailbox #(Transaction) rerf_value;
```

## Explanation
- Mailbox for communication:

```text
Generator → Scoreboard
```

Contains expected/reference transactions.

---

# Event Declaration

```systemverilog
event scbnext;
```

## Explanation
- Synchronization event.

Used to notify generator:

```text
Scoreboard finished checking current transaction
```

---

# Reference Memory Model

```systemverilog
logic [width-1:0] local_memory[*];
```

# Explanation

### Associative Array

```systemverilog
[*]
```

means associative array.

Used as:
- reference RAM model inside scoreboard.

### Purpose
- Mimics DUT memory behavior.

---

# Address Variable

```systemverilog
bit [addr_width-1:0] address_ref;
```

## Explanation
- Stores converted address value.
- Used for indexing associative array.

---

# Constructor

```systemverilog
function new(mailbox #(Transaction) mon2scb,
             mailbox #(Transaction) rerf_value);
```

## Explanation
- Constructor function.
- Initializes scoreboard mailboxes.

---

```systemverilog
this.mon2scb = mon2scb;
```

## Explanation
- Assigns monitor mailbox.

---

```systemverilog
this.rerf_value = rerf_value;
```

## Explanation
- Assigns reference mailbox.

---

```systemverilog
endfunction
```

## Explanation
- Ends constructor.

---

# Run Task

```systemverilog
task run();
```

## Explanation
- Main scoreboard task.
- Continuously checks DUT behavior.

---

# Forever Loop

```systemverilog
forever begin
```

## Explanation
- Scoreboard runs continuously throughout simulation.

---

# Receive Actual Transaction

```systemverilog
mon2scb.get(t);
```

## Explanation
- Receives actual DUT transaction from monitor.

Contains:
- DUT inputs
- DUT outputs

---

# Receive Reference Transaction

```systemverilog
rerf_value.get(tref);
```

## Explanation
- Receives reference transaction from generator.

Used as expected behavior reference.

---

# Display Transaction

```systemverilog
t.display("Scoreboard");
```

## Explanation
- Prints scoreboard transaction details.

Useful for:
- debugging
- tracking verification flow

---

# Address Conversion

```systemverilog
address_ref = unsigned'(t.address);
```

## Explanation

### Type Casting

```systemverilog
unsigned'
```

converts address into unsigned format.

### Why?
- Associative arrays require proper indexing.

---

# WRITE CHECK SECTION

```systemverilog
if(t.en == 1'b1 && t.we == 1'b1)
```

# Explanation

Checks:

| Signal | Value |
|---|---|
| en | 1 |
| we | 1 |

Meaning:
- RAM enabled
- Write operation active

---

# Update Reference Memory

```systemverilog
local_memory[address_ref] = t.data_in;
```

## Explanation
- Stores input data into reference memory.

### Purpose
- Mimics DUT write operation.

---

# PASS Message

```systemverilog
$display("PASS - WRITE : Address = %0d Data = %0d",
          t.address, t.data_in);
```

## Explanation
- Displays successful write operation.

Example:

```text
PASS - WRITE : Address = 10 Data = 55
```

---

# READ CHECK SECTION

```systemverilog
else if(t.en == 1'b1 &&
        t.we == 1'b0 &&
        t.data_out === local_memory[address_ref])
```

# Explanation

Checks:

| Condition | Meaning |
|---|---|
| en = 1 | RAM enabled |
| we = 0 | Read operation |
| output matches expected | Correct DUT behavior |

---

# PASS READ Message

```systemverilog
$display("PASS - READ : Address = %0d Expected = %0d Actual = %0d",
```

## Explanation
- Displays successful read verification.

---

```systemverilog
address_ref,
local_memory[address_ref],
t.data_out);
```

## Explanation

### Values Printed

| Value | Meaning |
|---|---|
| address_ref | Memory address |
| local_memory[address_ref] | Expected data |
| t.data_out | Actual DUT output |

---

# FAIL READ CHECK

```systemverilog
else if(t.en == 1'b1 &&
        t.we == 1'b0 &&
        t.data_out !== local_memory[address_ref])
```

# Explanation

Checks:
- read operation occurred
- DUT output mismatched expected value

### `!==`
- Case inequality operator.
- Detects:
  - X
  - Z
  - incorrect values

---

# FAIL Message

```systemverilog
$display("FAIL - READ : Address = %0d Expected = %0d Actual = %0d",
```

## Explanation
- Displays read mismatch.

Useful for debugging DUT issues.

---

# DISABLED CONDITION CHECK

```systemverilog
else if(t.en == 1'b0 &&
        t.data_out === '0)
```

# Explanation

Checks:
- RAM disabled
- output is zero

This is expected DUT behavior.

---

# PASS DISABLED Message

```systemverilog
$display("PASS - DISABLED : data_out = %0d",
          t.data_out);
```

## Explanation
- Indicates proper disabled-state behavior.

---

# FAIL DISABLED CHECK

```systemverilog
else if(t.en == 1'b0 &&
        t.data_out !== '0)
```

# Explanation
- RAM disabled but output not zero.
- Indicates DUT error.

---

# FAIL Message

```systemverilog
$display("FAIL - DISABLED : data_out = %0d",
          t.data_out);
```

## Explanation
- Prints failure information.

---

# UNKNOWN CONDITION

```systemverilog
else begin
```

## Explanation
- Executes if none of previous conditions match.

---

```systemverilog
$display("UNKNOWN CONDITION DETECTED");
```

## Explanation
- Generic debug message.
- Helps identify unexpected DUT behavior.

---

# Trigger Synchronization Event

```systemverilog
->scbnext;
```

# Explanation

Triggers scoreboard event.

### Purpose

Notifies generator:

```text
Scoreboard completed checking current transaction
```

Generator then produces next transaction.

---

```systemverilog
end
```

## Explanation
- Ends forever loop body.

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
- Ends Scoreboard class.

---

# Scoreboard Workflow

```text
Receive DUT Transaction
            ↓
Receive Reference Transaction
            ↓
Update Reference Memory
            ↓
Compare Expected vs Actual
            ↓
Display PASS/FAIL
            ↓
Notify Generator
```

---

# Role of Scoreboard

| Function | Purpose |
|---|---|
| Reference Modeling | Mimics DUT behavior |
| Comparison | Checks DUT correctness |
| Error Detection | Finds mismatches |
| Synchronization | Controls transaction flow |

---

# Important Concepts Used

| Concept | Purpose |
|---|---|
| Associative Array | Reference memory model |
| Mailbox | Inter-component communication |
| Case Equality (`===`) | Accurate comparisons |
| Event Synchronization | Coordinated execution |
| Reference Model | Expected DUT behavior |
| Scoreboarding | Functional verification |

---

# Why Scoreboard is Important

Without scoreboard:
- no automatic checking
- manual waveform analysis required
- verification becomes inefficient

Scoreboard enables:

```text
Self-Checking Testbench
```

which is a major verification feature.

---

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



# Test Execution

```systemverilog
`include "environment.sv"

program test(RAM_intf inf);

  Environment env;

  initial begin

    // Create Environment
    env = new(inf, 20);

    // Run Verification Environment
    env.run();

  end

endprogram
```
# Line-by-Line Explanation of `test.sv`

```systemverilog
`include "environment.sv"
```

# Explanation
- Includes the `environment.sv` file.

### Purpose
- Makes `Environment` class definition visible to compiler.

Without this:
- compiler will not recognize `Environment`.

---

# Program Block

```systemverilog
program test(RAM_intf inf);
```

# Explanation
- Declares a program block named `test`.

### Input Argument

```systemverilog
RAM_intf inf
```

- Interface instance passed from top testbench.

---

# Why Program Block is Used

In SystemVerilog:

| Block | Purpose |
|---|---|
| module | Hardware modeling |
| program | Verification/testbench modeling |

### Advantage of Program
- Executes in reactive region.
- Avoids race conditions between:
  - DUT
  - testbench

---

# Environment Handle

```systemverilog
Environment env;
```

# Explanation
- Declares handle for Environment object.

The environment controls:
- generator
- driver
- monitor
- scoreboard

---

# Initial Block

```systemverilog
initial begin
```

# Explanation
- Starts simulation activity at time 0.

Executed only once during simulation.

---

# Environment Creation

```systemverilog
env = new(inf, 20);
```

# Explanation
- Creates Environment object.

### Inputs

| Input | Purpose |
|---|---|
| `inf` | Virtual interface connection |
| `20` | Number of transactions |

---

# What Happens Internally

When this line executes:

```systemverilog
env = new(inf, 20);
```

the environment:
- creates mailboxes
- creates generator
- creates driver
- creates monitor
- creates scoreboard
- connects all components

---

# Transaction Count

```systemverilog
20
```

means:

```text
Generate 20 randomized transactions
```

---

# Better Coverage

For higher functional coverage:

Replace:

```systemverilog
env = new(inf, 20);
```

with:

```systemverilog
env = new(inf, 300);
```

or

```systemverilog
env = new(inf, 500);
```

because:
- address space = 64 values
- data space = 256 values

More transactions improve:
- address coverage
- data coverage
- total coverage

---

# Run Environment

```systemverilog
env.run();
```

# Explanation
- Starts complete verification environment.

Internally it:
- runs generator
- runs driver
- runs monitor
- runs scoreboard
- prints coverage report

---

# Overall Verification Flow

```text
env.run()
    ↓
test()
    ↓
fork
 ├── generator
 ├── driver
 ├── monitor
 └── scoreboard
```

---

```systemverilog
end
```

# Explanation
- Ends initial block.

---

```systemverilog
endprogram
```

# Explanation
- Ends program block.

---

# Purpose of `test.sv`

| Function | Purpose |
|---|---|
| Creates Environment | Builds verification system |
| Passes Interface | Connects DUT and TB |
| Starts Simulation | Executes verification flow |
| Controls Transaction Count | Determines test length |

---

# Overall Role of `test.sv`

`test.sv` acts as:

```text
Verification Test Controller
```

It is responsible for:
- creating environment
- starting verification
- controlling simulation execution

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
