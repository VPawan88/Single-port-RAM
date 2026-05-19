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
