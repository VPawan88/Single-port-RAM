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
