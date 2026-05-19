# Verification of Single-Port RAM using SystemVerilog

This project verifies a parameterized Single-Port RAM using SystemVerilog-based verification methodology.  
The verification environment includes:

- Generator
- Driver
- Monitor
- Scoreboard
- Interface
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
