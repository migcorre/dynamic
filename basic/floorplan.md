k# Floorplan
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

**We add the tracks.**
Tracks are like invisible guide lines that show where the metal wires (routing) can go. They're used during the place-and-route stage to help the design too ls know where it's safe to place wires without violating spacing or design rules

```tcl
#source the tcl script for track generations for sky130hd PDK
source $vars(design,path,pdk)/sky130hd.tracks
```

The rules comes from technology lef. example:

```tcl
make_tracks metX -x_offset $OFFSET -x_pitch $PITCH -y_offset $OFFESET -y_pitch $PITCH
```
seeing tlef:

![image](https://github.com/user-attachments/assets/24851758-c4b7-406b-b7d9-6e263e0c7671)

then:

```tcl
make_tracks met1 -x_offset 0.17 -x_pitch 0.34 -y_offset 0.17 -y_pitch 0.34
```

![image](https://github.com/user-attachments/assets/0edbef1c-5fde-46b1-b69b-94bba0e3fa3b)


### place ports
We add the ports. the position could be generated ramdonly as We do here or manually. \
for more info ==> https://openroad.readthedocs.io/en/latest/main/src/ppl/README.html

```tcl
place_pins -random -hor_layers met3 -ver_layers met2
```

### tap cells
Tap cells are special non-logic cells used to connect the substrate or well of the chip to power (VDD) or ground (VSS). They don't perform logic functions — their job is to make sure the transistors inside the chip behave correctly and don't cause electrical problems (latchups)
Tap cells are inserted regularly across the layout,each x-distance given by the Fab.
more info ==> https://teamvlsi.com/2020/08/well-tap-cell-in-asic-design.html

```tcl
tapcell -tapcell_master $vars(tech,tapcell) -distance $vars(tech,tapcell_distance)
```

![image](https://github.com/user-attachments/assets/48ffd429-def1-4fd7-8fee-59acb24944ef)

### global connections
We need connect power and ground instancces pins to a global power/ground net:

```tcl
add_global_connection -net {VDD} -inst_pattern {.*} -pin_pattern {VPWR} -power    
add_global_connection -net {VDD} -inst_pattern {.*} -pin_pattern {VPB}
add_global_connection -net {VSS} -inst_pattern {.*} -pin_pattern {VGND} -ground
add_global_connection -net {VSS} -inst_pattern {.*} -pin_pattern {VNB}
global_connect
```
add_global_connection maps instance pins (like VPWR, VGND, or VDD, VSS) to global nets (like VDD and VSS). It’s especially useful when the standard cell library uses different pin names for power/ground than your Verilog or DEF.

We define a voltage domain:
```tcl
set_voltage_domain -name {CORE} -power {VDD} -ground {VSS}
```

### power grid
We will use the power distribution network (PDN) generator module in OpenROAD (pdn) is based on the PDNGEN tool. This utility aims to simplify the process of adding a power grid into a floorplan. The aim is to specify a small set of power grid policies to be applied to the design, such as layers to use, stripe width and spacing, then have the utility generate the actual metal straps. Grid policies can be defined over the stdcell area, and over areas occupied by macros.

for more information ==> https://openroad.readthedocs.io/en/latest/main/src/pdn/README.html

```tcl
#create the pdn object calls grid
define_pdn_grid -name {grid} -voltage_domains {CORE}
#create a ring around the design over metal 4/5.
add_pdn_ring -grid {grid} -layers {met4 met5} -widths 1.6 -spacings 1.6 -core_offsets 0
#add met1 strips for standard cells connections
add_pdn_stripe -grid {grid} -layer {met1} -width {0.48} -pitch {5.44} -offset {0} -followpins -extend_to_core_ring
#create connection from met until met4
add_pdn_connect -grid {grid} -layers {met1 met4}
#craate connections between met4 and met5
add_pdn_connect -grid {grid} -layers {met4 met5}
#create pdn
pdngen
```
![image](https://github.com/user-attachments/assets/cb0d6e74-f483-4bd5-ab6e-11a9ac932864)

Finally we generate the floorplan def file and save the floorplan stage .

```tcl
write_def $vars(design,path,outputs)/floorplan.def
ord::write_db_cmd $vars(design,path,outputs)/floorplan.odb
```

### final script:

```tcl
read_lef $vars(tech,tlef)
read_lef $vars(tech,stdcell_lefs)
read_liberty $vars(tech,libs,synthesis)      
read_verilog $vars(design,path,outputs)/$vars(design,top_name).v
link_design $vars(design,top_name)
read_sdc $vars(design,path,inputs)/func.sdc
initialize_floorplan -site unithd -utilization 10 -aspect_ratio 1.0 -core_space 4.7
source $vars(design,path,pdk)/sky130hd.tracks
place_pins -random -hor_layers met3 -ver_layers met2

tapcell -tapcell_master $vars(tech,tapcell) -distance $vars(tech,tapcell_distance)

####################################
# global connections
####################################
add_global_connection -net {VDD} -inst_pattern {.*} -pin_pattern {VPWR} -power
add_global_connection -net {VDD} -inst_pattern {.*} -pin_pattern {VPB}
add_global_connection -net {VSS} -inst_pattern {.*} -pin_pattern {VGND} -ground
add_global_connection -net {VSS} -inst_pattern {.*} -pin_pattern {VNB}
global_connect
####################################
# voltage domains
####################################
set_voltage_domain -name {CORE} -power {VDD} -ground {VSS}
####################################
# standard cell grid
####################################
define_pdn_grid -name {grid} -voltage_domains {CORE}
add_pdn_ring -grid {grid} -layers {met4 met5} -widths 1.6 -spacings 1.6 -core_offsets 0                                                                                                                        
add_pdn_stripe -grid {grid} -layer {met1} -width {0.48} -pitch {5.44} -offset {0} -followpins -extend_to_core_ring
add_pdn_connect -grid {grid} -layers {met1 met4}
add_pdn_connect -grid {grid} -layers {met4 met5}

pdngen                                                                                                                                                                                                                                      
                                                                                                                                                                                                                                            
write_def $vars(design,path,outputs)/floorplan.def                                                                                                                                                                                          
ord::write_db_cmd $vars(design,path,outputs)/floorplan.o
```

