# Day 2 — Gate-Level Synthesis of a 2‑bit Counter with Yosys + SKY130

## Overview
Day 2 focuses on taking a small sequential RTL (a 2‑bit synchronous counter) through simulation, synthesis with Yosys, technology mapping to SKY130 HD cells, netlist inspection, and basic gate‑level simulation setup.

## What I built

**Design:** 2‑bit up‑counter with async active‑high reset.  
**Testbench:** Clock generation, periodic reset toggle, VCD dump for waves.  
**Synthesis:** Yosys read_liberty → read_verilog → synth → dfflibmap/abc → show/write_verilog.  
**Mapping:** sky130_fd_sc_hd liberty for typical PVT (tt_025C_1v80).  

## Repository Structure
```
verilog_files/
├── good_counter.v          # RTL
├── tb_good_counter.v       # Testbench
├── good_counter_netlist.v  # Post-synthesis gate-level netlist
├── yosys_run.ys            # Yosys synthesis script
└── tb_good_counter.vcd     # Wave dump
```

## RTL and Testbench

### `good_counter.v`
```verilog
module good_counter(input clk, input reset, output reg [1:0] cnt);
  always @(posedge clk or posedge reset) begin
    if (reset) cnt <= 2'b00;
    else       cnt <= cnt + 2'b01;
  end
endmodule
```

### `tb_good_counter.v`
```verilog
`timescale 1ns/1ps
module tb_good_counter;
  reg clk, reset;
  wire [1:0] cnt;

  good_counter uut (.clk(clk), .reset(reset), .cnt(cnt));

  initial begin
    $dumpfile("tb_good_counter.vcd");
    $dumpvars(0, tb_good_counter);
    clk = 0;
    reset = 1;
    #150 reset = 0;
    #1000 reset = 1;
    #40 reset = 0;
    #1000 $finish;
  end

  always #10 clk = ~clk;   // 50 MHz equiv
endmodule
```

## Simulation Results (GTKWave)
![Waveform](/screenshot1)

## Testbench Source View
![Testbench](/Screenshot2)

## Gate-Level Netlist Schematic
![Netlist](/Screenshot3)

## Yosys Script
```
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog good_counter.v
synth -top good_counter
dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog good_counter_netlist.v
show -format dot -prefix yosys_show
```

## Key Learnings
- Asynchronous reset instantly clears counter state.
- Mapped netlist uses sky130_fd_sc_hd cells for both flops and logic.
- Schematic matches expected 2‑bit counter structure (2 DFFs + minimal logic).

## Next Steps
- Try parameterized counters and synthesize.
- Explore gate-level simulation with sky130 functional models.
