## Basic synthesis
We will generate a verilog netlist starting from a RTL using [yosys tool](https://yosyshq.net/yosys/). 

We will use a project_name folder from where we will deploy the following structure directories:

```
├── work
    └── run_syn.sh
├── inputs
    └── rtl
        └── counter.v
    └── sdc
├── outputs
├── reports
├── scripts
    └── synthesis.tcl
├── vars
    └── globals.sh
```

work = directory where we can find the main executable of each stage (synthesys, floorplan,place,cts...)
inputs = all the necesary files for the tools
outputs = all the outputs dropped by the tools
reports = all reports coming from tool
scripts = main scripts that will execute by stage
vars = all the variables that will be use

## RTL example
Wi will use the following example RTL, a basic ounter 8bit
```verilog
module counter_8bit (
    input wire clk,        // Clock input
    input wire reset,      // Reset input (active high)
    output reg [7:0] count // 8-bit counter output
);

    // Always block triggered on positive edge of clock
    always @(posedge clk) begin
        if (reset) begin
            count <= 8'b0;    // Reset counter to 0 when reset is high
        end
        else begin
            count <= count + 1; // Increment counter
        end
    end

endmodule
```

We put this file under project folder into /src/rtl with the name counter.v

And We start to build our sythesis script under /scripts folder in the main folder.

```tcl
yosys -import
# read design
read_verilog $::env(DESIGN__RTL_DIR)/counter.v

# elaborate design hierarchy
hierarchy -check -top $::env(DESIGN__TOP_NAME)

synth -top counter

# write synthesized design
write_verilog $::env(DESIGN__OUTPUTS_DIR)/$::env(DESIGN__TOP_NAME).v
```

We see global variables that will be load previus synthesis script. We will save them in a global.sh

```bash
#!/usr/bin/env bash
#
export DESIGN__TOP_NAME="counter"
export DESIGN__OUTPUTS_DIR="../outputs"
export DESIGN__RTL_DIR="../inputs/rtl"
export DESIGN__SDC_DIR="../inputs/rtl"
export DESIGN__SCRIPTS_DIR="../scripts"
```
we will lauch all from a work directory under main folder.

We create the bash script that will run our synthesis.

```bash
#!/bin/sh
. ../vars/global.sh
#echo ${DESIGN__SCRIPTS_DIR}
yosys -c ${DESIGN__SCRIPTS_DIR}/synthesis.tcl
```

