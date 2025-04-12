# Clock tree synthesis

What is Clock Tree Synthesis (CTS)? \
    Clock Tree Synthesis is the process of creating a balanced network that distributes the clock signal from the clock source (like a PLL or input port) to all the clocked elements (like flip-flops) in a chip. \
Why is it important? \
    Ensures the clock arrives at the right time to all parts of the chip. \
    Minimizes clock skew (difference in arrival time between flip-flops). \
    Balances timing, power, and area. \
How it works: \
    Finds all the clock sinks (flip-flops, latches, etc.) \
    Inserts buffers and inverters to build a tree-like structure. \
    Tries to balance delays across all paths. \
Result: \
After CTS, your design has a real clock network ready for accurate timing analysis and routing. \


### we choose buffers and invertes what we are going to use to build the clock network.
We need to configure the variables $vars(tech,cts_buffers) in vars_tech.tcl. \
In general we want stables buffer with same fall and rise transition behavior. For this reason Fabs provides balanced buffers and inverters, We need look for them in the libraries. \
It is common choise buffer/inverters with strenght greater than D3. \

  * clock tree synthesis
  * fix long wires until root buffer
  * legalization of new buffers/inverters added by clock tree synthesis
  * tool propagates the clocks.
  * do a global route.
  * extract RC
  * detail placement
