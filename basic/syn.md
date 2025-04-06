## Basic synthesis
We will generate a verilog netlist starting from a RTL using [yosys tool](https://yosyshq.net/yosys/). 

We will use a project_name folder from where we will deploy the following structure directories:

```
├── workspace/
    └── run_syn.sh
├── sourcecode/
    └── rtl/
        └── counter.v
    └── tb/
        └── tb.v
├── inputs/
    └── vars_tech.tcl
    └── vars_design.tcl
├── outputs/
    └── dbs/
    └── exports
├── reports/
├── scripts/
    └── synthesis.tcl
├── pdk

```

workspace = directory where we can find the main executable of each stage (synthesys, floorplan,place,cts...) \
inputs = all the necesary files for the tools \
outputs = all the outputs dropped by the tools \
reports = all reports coming from tool \
scripts = main scripts that will execute by stage \
sourcecode = directory with designs files definitios. \
pdk = files necesaries to run this example. Files inside this folder are getting from a real PDK and here you can find the necessary ones for run this example.


## RTL example
We will use the following example RTL, a basic ounter 8bit
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

We put this file under project folder into /sourcecode/rtl with the name counter.v

## basic synthesis script

And We start to build our sythesis script under /scripts folder in the main folder.

```tcl
yosys -import 
#design variables
source $::env(PATH_INPUTS)/vars_design.tcl
#tech variables
source $vars(design,path,libraries)/vars_tech.tcl

# read verilog
read_verilog $vars(design,path,rtl)/counter.v

# elaborate design hierarchy
hierarchy -check -top $vars(design,top_name)

synth -top $vars(design,top_name) 
```

We see global variables that will be load previus synthesis script. They are specific to the design. This variables are setted in ../inputs/vars_design.tcl

```tcl
###########################################################
# MAIN PATHS FOR THIS DESIGN
###########################################################
set vars(design,path,pdk) $::env(PATH_PDK)
set vars(design,path,libraries) $::env(PATH_LIBRARIES)
set vars(design,path,inputs) $::env(PATH_INPUTS)
set vars(design,path,outputs) $::env(PATH_OUTPUTS)
set vars(design,path,rtl) $::env(PATH_RTL)
set vars(design,path,scripts) $::env(PATH_SCRIPTS)
set vars(design,path,sourcecode) $::env(PATH_SOURCECODE)
set vars(design,path,workspace) $::env(PATH_WORKSPACE)

############################################################
# CONFIG DESIGN
############################################################
set vars(design,top_name) "counter"
set vars(design,sdc,functional) "$vars(design,path,inputs)/func.sdc"
```

We see enviroment variables, these are coming from a global variables in env.sh (in worskape dir) this file is called before exec the main script, setting in this way all the enviroment necessary for run.

```bash
#!/usr/bin/env bash
#
export PATH_ABS_MAIN_PATH_WORK="$(pwd)"
export PATH_OUTPUTS="../outputs"
export PATH_LIBRARIES="../libraries"
export PATH_WORKSPACE="../workspace"
export PATH_PDK="../pdk"
export PATH_SOURCECODE="../sourcecode"
export PATH_RTL="../sourcecode/rtl"
export PATH_SCRIPTS="../scripts"
export PATH_INPUTS="../inputs"
```

we will lauch all from a workspace directory under main folder.

We create the bash script that will run our synthesis (run_syn.sh).

```bash
#!/usr/bin/env bash
. env.sh
echo $PATH_INPUTS
yosys -c ${PATH_SCRIPTS}/synthesis.tcl

```

## Mapping libraries
We need to cinfigure our PDK and choose the libraries that synthesis tool will use for mapping from generic to standard cells.
basic PDK config: inside ../libraries folder we add vars_tech.tcl who cotaints the paths and tech's config releated to the PDK that will use on the whole flow.

```tcl
#PDK FOLDER
set vars(tech,pdk_dir) "../pdk"
#TECHNOLOGY LEF FILE
set vars(tech,tlef) "$vars(tech,pdk_dir)/sky130_fd_sc_hd.tlef"
#LIBRARIES USED
set vars(tech,libs,max) "$vars(tech,pdk_dir)/sky130_fd_sc_hd__ff_n40C_1v95.lib"
set vars(tech,libs,tt) "$vars(tech,pdk_dir)/sky130_fd_sc_hd__tt_025C_1v80.lib"
set vars(tech,libs,min) "$vars(tech,pdk_dir)/sky130_fd_sc_hd__ss_100C_1v60.lib"
set vars(tech,libs,synthesis) $vars(tech,libs,max)
```
where pdk is the folder where the PDK was installed. In our case contains all the PDK files that are necessary to this example.

We need to add 2 lines to the synthesis script for mapping:
```tcl
# mapping to $vars(tech,libs,synthesis)
dfflibmap -liberty $vars(tech,libs,synthesis)
abc -liberty $vars(tech,libs,synthesis)
clean
```
then our final synthesis script:

```tcl
yosys -import 
#design variables
source $::env(PATH_INPUTS)/vars_design.tcl
#tech variables
source $vars(design,path,libraries)/vars_tech.tcl

# read verilog
read_verilog $vars(design,path,rtl)/counter.v

# elaborate design hierarchy
hierarchy -check -top $vars(design,top_name)

synth -top $vars(design,top_name) 

dfflibmap -liberty $vars(tech,libs,synthesis)
abc -liberty $vars(tech,libs,synthesis)
clean

# write synthesized design
write_verilog $vars(design,path,outputs)/$vars(design,top_name).v
```
