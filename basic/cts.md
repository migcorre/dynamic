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
clock_tree_synthesis -root_buf $vars(tech,cts_buffers) -buf_list $vars(tech,cts_buffers)
```

  * fix long wires until root buffer
  * legalization of new buffers/inverters added by clock tree synthesis
  * tool propagates the clocks.
  * do a global route.
  * extract RC
  * detail placement
