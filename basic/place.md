# Placement

Main steps:
  * config global routing
  * run global placement
  * set global capacitance and resitance in the net
  * generic parasitic estimation
  * repair transition/capacitance/fanout
  * tie cells insertion
  * detail placement

### Config global routing
openroad uses [Fastroute](https://openroad.readthedocs.io/en/latest/main/src/grt/README.html) module for global routing and [Triton Route](https://openroad.readthedocs.io/en/latest/main/src/drt/README.html) for detail routing. 

With the _set_routing_layers_ command We define the minimum and maximum layers that the router (FastRoute or TritonRoute) can use for connecting nets.

```tcl
set_routing_layers \
        -signal ${vars(tech,route,signal,bottom_layer)}-${vars(tech,route,signal,top_layer)} \
        -clock  ${vars(tech,route,clock,bottom_layer)}-${vars(tech,route,clock,top_layer)}
```

### Run global placement
We run a global placement. The global placement module in OpenROAD is based on the open-source [RePlAce tool](https://openroad.readthedocs.io/en/latest/main/src/gpl/README.html)
```tcl
global_placement -routability_driven -density ${vars(design,place,global,density)}
```
In this case we enable routability-driven mode (other could be Routability-Driven), with a target density of ${vars(design,place,global,density)}

![image](https://github.com/user-attachments/assets/66d8de12-cb5d-4b91-adea-38bf38ba16dd)


As you can see is a generic placement some cells are overlapped:

![image](https://github.com/user-attachments/assets/5264f5f6-c697-4fde-81db-de0360b43fea)

the command detail_placement fix this. But first we need to configure the engine to do some maths over Resistance and capacitance and to have in this way the real delay (in this case, at placement stage an stimate)

### Set global capacitance and resitance in the net
set_wire_rc sets the resistance (R) and capacitance (C) values used by OpenROAD to estimate wire delays before detailed routing is complete. Because actual routing hasn’t happened yet, tools need a model to predict how wires will affect timing.

```tcl
source $vars(tech,rc)
set_wire_rc -signal -layer ${vars(tech,route,signal,wire_rc)}
set_wire_rc -clock  -layer ${vars(tech,route,clock,wire_rc)}
```

### Generic parasitic estimation
estimate_parasitics computes an early estimate of wire parasitics (resistance and capacitance) for your design’s nets, using a wireload or RC model.
This lets OpenROAD perform timing analysis and optimization (like buffer insertion) even before the real routes are available.

```tcl
estimate_parasitics -placement
```
### repair transition/capacitance/fanout
```tcl
repair_design
```

The _repair_design command_ inserts buffers on nets to repair max slew, max capacitance and max fanout violations, and on long wires to reduce RC delay in the wire. It also resizes gates to normalize slews. We used _estimate_parasitics -placement_ before repair_design to estimate parasitics considered during repair. Placement-based parasitics cannot accurately predict routed parasitics, so a margin can be used to "over-repair" the design to compensate.

```tcl
repair_design 
    [-max_wire_length max_length]
    [-slew_margin slew_margin]
    [-cap_margin cap_margin]
    [-max_utilization util]
    [-buffer_gain gain_ratio]
    [-match_cell_footprint]
    [-verbose]
```

![image](https://github.com/user-attachments/assets/04efe7d5-e901-433b-8c4a-8e447e35129c)


This command is hand by the module [Risizer module](https://github.com/The-OpenROAD-Project/OpenROAD/blob/master/src/rsz/README.md)
This module manages the _set_wire_rc_ and _estimate_parasitics_ 

### tie cells insertion

```tcl
repair_tie_fanout -separation $vars(tech,tie_distance) $vars(tech,tielo_port)
repair_tie_fanout -separation $vars(tech,tie_distance) $vars(tech,tiehi_port)
```
Tie cells (tiehi, tielo) are special cells that provide constant logic values (1 or 0) to inputs like unconnected gates, reset, or enable pins. But they’re not designed to drive many loads. If too many gates are connected to a single tie cell, it causes electrical violations — like excessive fanout, weak drive strength, or signal integrity issues.

### detail placement
```tcl
detail placement
```
The -detailed_placement- command performs detailed placement of instances to legal locations after global placement.

The detailed placement module in OpenROAD (dpl) is based on [OpenDP](https://github.com/sanggido/OpenDP), or Open-Source Detailed Placement Engine

![image](https://github.com/user-attachments/assets/8e183d91-dbe2-415f-8807-fe43c0da899d)


You can see that the cells are in site.

![image](https://github.com/user-attachments/assets/3cb08caa-7821-416e-a25c-42fa32d5b254)


the full script:

```tcl
# read common utils
source ../scripts/common.tcl

#read floorplan data base (we need provide the libarties and sdc)
read_db $vars(design,path,outputs)/floorplan.odb
read_liberty $vars(tech,libs,synthesis)
read_sdc $vars(design,path,inputs)/func.sdc

# config global routing. which metal will be used for signals and clocks
set_routing_layers \
        -signal ${vars(tech,route,signal,bottom_layer)}-${vars(tech,route,signal,top_layer)} \
        -clock  ${vars(tech,route,clock,bottom_layer)}-${vars(tech,route,clock,top_layer)}

#global placement command
global_placement -routability_driven -density ${vars(design,place,global,density)}

#Set global capacitance and resitance in the net
source $vars(tech,rc)
set_wire_rc -signal -layer ${vars(tech,route,signal,wire_rc)}
set_wire_rc -clock  -layer ${vars(tech,route,clock,wire_rc)}

# Generic parasitic estimation
estimate_parasitics -placement

# repair transition/capacitance/fanout
repair_design

# tie cells insertion
repair_tie_fanout -separation $vars(tech,tie_distance) $vars(tech,tielo_port)
repair_tie_fanout -separation $vars(tech,tie_distance) $vars(tech,tiehi_port)

# detail placement
detailed_placement                                                                                                                                                                                                                          
                                                                                                                                                                                                                                            
# save db                                                                                                                                                                                                                                   
write_db $vars(design,path,outputs)/place.odb                      
```

where ../script/common.tcl is:
```tcl
source $::env(PATH_LIBRARIES)/vars_tech.tcl
source $::env(PATH_INPUTS)/vars_design.tcl      
source $::env(PATH_SCRIPTS)/utils.tcl              
```

We will see this in more detail later.
