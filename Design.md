# Single-Port RAM Design using SystemVerilog

```systemverilog
module RAM_64B #(parameter width = 8,parameter addr_width = 6)
(
    input  logic clk, en, we,
    input  logic [width-1:0]  data_in,
    input  logic [addr_width-1:0] address,
    output logic [width-1:0] data_out
);

    // Memory declaration : 64 locations for addr_width = 6
    logic [width-1:0] mem[(2**addr_width)-1:0];

    // Synchronous Read and Write
    always_ff @(posedge clk) begin
        if (!en) begin
            data_out <= '0;
        end
        else begin
            if (we) begin
                mem[address] <= data_in;
                data_out    <= '0;
            end
            else begin
                data_out <= mem[address];
            end
        end
    end

endmodule
```

# Line-by-Line Explanation

## Module Declaration

```systemverilog
module RAM_64B #(parameter width = 8,parameter addr_width = 6)(
```

- `module RAM_64B`
  - Declares the RAM module named `RAM_64B`.

- `parameter width = 8`
  - Defines the data width as 8 bits.

- `parameter addr_width = 6`
  - Defines the address width as 6 bits.

- Since address width is 6:

\[
2^6 = 64
\]

- Therefore, the RAM contains 64 memory locations.

---

## Input and Output Ports

```systemverilog
input  logic clk, en, we,
input  logic [width-1:0]       data_in,
input  logic [addr_width-1:0]  address,
output logic [width-1:0]       data_out
```

### `clk`
- Clock signal.
- All operations occur on the positive edge of the clock.

### `en`
- Enable signal.
- RAM operates only when `en = 1`.

### `we`
- Write Enable signal.
- `we = 1` → Write operation.
- `we = 0` → Read operation.

### `data_in`
- Input data bus.
- Width is determined by parameter `width`.

### `address`
- Address bus used to select memory locations.

### `data_out`
- Output data bus used during read operation.

---

## Memory Declaration

```systemverilog
logic [width-1:0] mem[(2**addr_width)-1:0];
```

- Declares the memory array.

### Breakdown

### `logic [width-1:0]`
- Each memory location stores `width` bits.
- Default width = 8 bits.

### `mem[(2**addr_width)-1:0]`
- Creates total memory locations.
- Default address width = 6.

\[
2^6 = 64
\]

- Memory locations become:

```text
mem[0] to mem[63]
```

---

## Sequential Always Block

```systemverilog
always_ff @(posedge clk)
```

- `always_ff` is used for sequential logic in SystemVerilog.
- Executes only at the positive edge of the clock.
- Represents flip-flop based hardware.

---

## Enable Condition

```systemverilog
if (!en) begin
    data_out <= '0;
end
```

- If RAM is disabled (`en = 0`):
  - Output is forced to zero.
  - No read or write operation occurs.

### `'0`
- Automatically fills all bits with zero.
- Better scalable coding style.

---

## RAM Enabled Condition

```systemverilog
else begin
```

- Executes when RAM is enabled (`en = 1`).

---

## Write Operation

```systemverilog
if (we) begin
    mem[address] <= data_in;
    data_out    <= '0;
end
```

### `if (we)`
- Checks whether write operation is requested.

### `mem[address] <= data_in;`
- Stores input data into the selected memory location.

Example:

```text
address = 5
data_in = 8'hAA
```

Then:

```text
mem[5] = 8'hAA
```

### `data_out <= '0;`
- Output is cleared during write operation.

---

## Read Operation

```systemverilog
else begin
    data_out <= mem[address];
end
```

- Executes when:

```text
en = 1
we = 0
```

- Reads data from the selected memory location.

Example:

```text
address = 5
```

Then:

```text
data_out = mem[5]
```

---

## Endmodule

```systemverilog
endmodule
```

- Marks the end of the RAM module.

---

# Features of this RAM Design

| Feature | Description |
|---|---|
| RAM Type | Single-Port RAM |
| Read Type | Synchronous Read |
| Write Type | Synchronous Write |
| Configurable Width | Yes |
| Configurable Depth | Yes |
| SystemVerilog Construct | `always_ff` |
| Synthesizable | Yes |
| Clocked Design | Yes |

---

# Operation Summary

| en | we | Operation |
|---|---|---|
| 0 | X | Disabled |
| 1 | 1 | Write |
| 1 | 0 | Read |

---

### Memory Capacity

\[
2^6 = 64 \text{ locations}
\]

Each location stores:

\[
8 \text{ bits}
\]

Total memory size:

\[
64 \times 8 = 512 \text{ bits}
\]

---

