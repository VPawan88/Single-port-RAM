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
