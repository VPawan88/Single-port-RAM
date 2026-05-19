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
