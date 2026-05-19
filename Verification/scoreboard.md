
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
