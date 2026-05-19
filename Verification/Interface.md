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
