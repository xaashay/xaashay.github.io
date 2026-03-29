---
layout: post
title: "FPGA Clock Divider Tutorial"
date: 2026-03-30
categories: fpga verilog
---

Clock dividers are one of the most common circuits you'll build when working with FPGAs. Whether you need to drive a UART baud rate generator, a low-speed display controller, or any peripheral slower than your system clock, understanding clock division is essential.

## Why Divide Clocks?

FPGAs typically run at a high system clock — often 100 MHz or faster. Many peripherals operate at much lower speeds. Rather than running everything at the system clock rate, we count down to the needed frequency.

## Even Division

For dividing by an even number, a simple toggle counter works perfectly:

```verilog
module clk_div_even #(
  parameter DIV = 4  // Divide by this amount (must be even)
)(
  input  wire clk,
  input  wire rst,
  output reg  clk_out
);
  reg [$clog2(DIV)-1:0] count;

  always @(posedge clk or posedge rst) begin
    if (rst) begin
      count   <= 0;
      clk_out <= 0;
    end else begin
      if (count == (DIV/2) - 1) begin
        count   <= 0;
        clk_out <= ~clk_out;
      end else begin
        count <= count + 1;
      end
    end
  end
endmodule
```

With `DIV=4`, this produces a clock at `clk/4`. The counter runs from 0 to `DIV/2 - 1`, toggling the output each wrap — guaranteeing a 50% duty cycle.

## A Word of Warning

Be careful about routing generated clocks. Most FPGA synthesis tools prefer you to use dedicated PLLs or CMTs for actual clock signals. A counter-generated clock is fine for low-speed control logic, but for proper timing analysis always consider your vendor's clock management resources.

## Testing in Simulation

```verilog
module tb_clk_div;
  reg clk = 0;
  reg rst = 1;
  wire clk_out;

  clk_div_even #(.DIV(4)) uut (
    .clk(clk), .rst(rst), .clk_out(clk_out)
  );

  always #5 clk = ~clk;  // 100 MHz clock

  initial begin
    #20 rst = 0;
    #200 $finish;
  end

  initial $dumpvars(0, tb_clk_div);
endmodule
```

Simulate with Icarus Verilog and view in GTKWave — `clk_out` will toggle at exactly 1/4 the rate of `clk`.
