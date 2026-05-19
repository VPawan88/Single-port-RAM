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
