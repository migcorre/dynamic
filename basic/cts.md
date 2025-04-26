# Clock tree synthesis

* What is Clock Tree Synthesis (CTS)? \
    Clock Tree Synthesis is the process of creating a balanced network that distributes the clock signal from the clock source (like a PLL or input port) to all the clocked elements (like flip-flops) in a chip. \
* Why is it important? \
    Ensures the clock arrives at the right time to all parts of the chip. \
    Minimizes clock skew (difference in arrival time between flip-flops). \
    Balances timing, power, and area. \
* How it works: \
    Finds all the clock sinks (flip-flops, latches, etc.) \
    Inserts buffers and inverters to build a tree-like structure. \
    Tries to balance delays across all paths. \
* Result: \
After CTS, your design has a real clock network ready for accurate timing analysis and routing. \


### we choose buffers and invertes what we are going to use to build the clock network.
We need to configure the variables $vars(tech,cts_buffers) in vars_tech.tcl. \
In general we want stables buffer with same fall and rise transition behavior. For this reason Fabs provides balanced buffers and inverters, We need look for them in the libraries. \
It is common choise buffer/inverters with strenght greater than D3. \

```tcl
set vars(tech,cts_buffers) "sky130_fd_sc_hd__clkbuf_4"
```
### configuring RC extration
We need to tell the tool how RC for extration is configure. the _clock_tree_synthesis_ command will take it into account. In iur case We use same configuration than placement stage.

```tcl
source $vars(tech,rc)
set_wire_rc -signal -layer ${vars(tech,route,signal,wire_rc)}
set_wire_rc -clock  -layer ${vars(tech,route,clock,wire_rc)}
```

### clock tree synthesis
The _clock_tree_synthesis_ command in OpenROAD builds the actual clock network, inserting buffers and routing paths, so the clock signal reaches all flip-flops and clocked elements with balanced delays.

```tcl
clock_tree_synthesis -root_buf $vars(tech,cts_buffers) -buf_list $vars(tech,cts_buffers) -clk_nets {clk}
```
Before clock_tree_synthesis...

![image](https://github.com/user-attachments/assets/1872638c-d073-4ee1-aa62-afb8ff76a445)


After clock_tree_synthesis...

![image](https://github.com/user-attachments/assets/465bf4de-a27d-48c4-8ef1-907db757f7cd)


As you can see new stdcells were added in clock network

### fix long wires

After clock tree synthesis (clock_tree_synthesis), a long wire may exist, which can cause excessive delays or signal integrity issues. This command mitigates that by adding buffers to break up the wire, reducing RC delay.

```tcl
estimate_parasitics -placement
repair_clock_nets
```

### legalization of new buffers/inverters added by clock tree synthesis
We see that the new standard cells added by clock tree synthesis  are overlapping or our of site, for that reason rerun a detailed placement
```tcl
detailed_placement
```

### Propagates the clocks.
In OpenROAD, the set_propagated_clock command is used to configure the timing analysis to use actual (propagated) clock delays rather than ideal clock delays for a specified clock. This means the tool calculates clock insertion delays and skews based on the physical clock tree network after clock tree synthesis (CTS), instead of assuming ideal, zero-delay clock signals.

```tcl
set_propagated_clock [all_clocks]
```

here the report timing before and after clock propagation:

![image](https://github.com/user-attachments/assets/498ebd7f-4af0-45c6-9274-6a94852904e6)


![image](https://github.com/user-attachments/assets/305295f5-3860-4725-9835-30f5c43af1b9)


### do a global route.
We do a global routing. We configure the prefered layer for signal and clock routing, do a global routing.
```tcl
set_routing_layers \
        -signal ${vars(tech,route,signal,bottom_layer)}-${vars(tech,route,signal,top_layer)} \
        -clock  ${vars(tech,route,clock,bottom_layer)}-${vars(tech,route,clock,top_layer)}
global_route -congestion_iterations 100
estimate_parasitics -global_routing
detailed_placement
```

### final script:
```tcl
# load utils
source ../scripts/common.tcl

#load db
read_db $vars(design,path,outputs)/place.odb
read_liberty $vars(tech,libs,synthesis)
source $vars(design,path,inputs)/func.sdc

# configuring RC extration
source $vars(tech,rc)
set_wire_rc -signal -layer ${vars(tech,route,signal,wire_rc)}
set_wire_rc -clock  -layer ${vars(tech,route,clock,wire_rc)}

# clock tree synthesis
clock_tree_synthesis -root_buf $vars(tech,cts_buffers) -buf_list $vars(tech,cts_buffers) -clk_nets {clk}

# parasitic stimation
estimate_parasitics -placement

#repait clock nets. Fix long wires
repair_clock_nets

# legalize cells
detailed_placement

#report_checks -fields input -digits 3 > $vars(design,path,reports)/cts_timing_before_propagation.rpt

# propagate clocks
set_propagated_clock [all_clocks]

#report_checks -fields input -digits 3 > $vars(design,path,reports)/cts_timing_after_propagation.rpt

#save data base                                                                                                                                                                                                                             
write_db $vars(design,path,outputs)/cts.odb                    
```
