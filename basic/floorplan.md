# Floorplan
main steps:
  * read libraries (lef, tech lef, timing libraries)
  * read netlist 
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
### read netlist:

```tcl
#verlig netlist 
read_verilog $vars(design,path,outputs)/$vars(design,top_name).v
```

### link design:

```tcl
#link_design resolves all the module references and connects the top-level module with the definitions provided by the loaded Liberty (.lib) files and
#LEF (for physical info). It makes the design ready for placement, timing analysis, and optimization.
link_design $vars(design,top_name)
```

### initialize floorplan:
```tcl
# We initialize a floorplan with 10% utilization, square (aspect_ratio 1.0), and with space between the core and die of 4.7
# more info ==> https://openroad.readthedocs.io/en/latest/main/src/ifp/README.html
initialize_floorplan -site unithd -utilization 10 -aspect_ratio 1.0 -core_space 4.7
```
to see the gui (graphic interface user)
```tcl
openroad> gui::show
```
![image](https://github.com/user-attachments/assets/e3dabea9-99a2-4c00-b71d-624914c33c5c)


