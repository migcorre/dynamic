# Floorplan
main steps:
  * read libraries (lef, tech lef, timing libraries)
  * link design
  * initialize floorplan
  * place ports design
  * tapcells insertion
  * global connections
  * power grid


### read libraries:

```tcl
#read technology LEF.
#A LEF (Library Exchange Format) technology is a file that contains the manufacturing rules and the physical and electrical characteristics of a VLSI design.
read_lef $vars(tech,tlef)
# Read standard cells LEF
# A standard cell LEF (Library Exchange Format) file is an ASCII representation of the physical layout of standard cells in a VLSI design.
# It's used to describe the physical aspects of cells, such as their dimensions, pin locations, and routing blockages    
read_lef $vars(tech,stdcell_lefs)                                                                                                                                                                               # Read library cells. 
#The timing library (. lib) is an ASCII representation of the Timing, Power and Area associated with the standard cells. Characterization of cells
#under different PVT conditions results in the timing library (. lib). The delay calculation happens based on input transition (Slew) and the output capacitance (Load) 
read_liberty $vars(tech,libs,synthesis)
```
