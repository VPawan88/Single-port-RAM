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
