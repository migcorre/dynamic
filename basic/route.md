## Route

### Main Steps:
  * load CTS data base
  * Global route
  * Detail route
  * Filler insertion
  * parasitic extration
  * Generate def dile
  * Save Route database

### load CTS data Base
```
read_db $vars(design,path,dbs)/cts.odb
read_liberty $vars(tech,libs,synthesis)                                                                                                                                                                             
read_sdc $vars(design,path,sdc)/func.sdc
```
### Global Route
We first configure the signal and route preferer layers and the We perform a global route.

```
set_routing_layers \
        -signal ${vars(tech,route,signal,bottom_layer)}-${vars(tech,route,signal,top_layer)} \
        -clock  ${vars(tech,route,clock,bottom_layer)}-${vars(tech,route,clock,top_layer)}
global_route -congestion_iterations 100
```
After that We do detail placement in case of ilegal cell placement.
```
estimate_parasitics -global_routing
detailed_placement
```
NOTE: now the stimation for parasitics are done with -global_routing.
The -placement command uses GCell-level placement coordinates to estimate Manhattan wiring, applying uniform wire RC. No layer assignment is made here, and it uses a bounding-box half-perimeter/wirelength. \
The -global_routing command, on the other hand, considers global route segments, estimating actual route lengths and accounting for layer-specific RC, as well as via contributions \
‑placement takes whatever you set with
```
set_wire_rc -layer metal3      ;# or explicit -resistance/-capacitance
```
so every segment, regardless of where it would really route, gets that per‑micron R and C. \
‑global_routing fetches each segment’s layer from the guide and uses the layer table that comes from LEF or that you overrode with set_layer_rc. Vias pick up their own R. \

### detail routing
detail route is done using (TritonRoute module)[https://openroad.readthedocs.io/en/latest/main/src/drt/README.html] \

```
detailed_route -output_drc [make_result_file $vars(design,path,reports)/route_drc.rpt] \
      -output_maze [make_result_file $vars(design,path,reports)/maze.log] \
      -no_pin_access \
      -save_guide_updates \
      -bottom_routing_layer met1 \
      -top_routing_layer met5 \
      -verbose 0
```
Note that we use custom procs for write the reports "make_result_file". This is located in the utils.tcl script.

### Filler insertion:
Filler cells are used to fill gaps between placed cells, ensuring diffusion continuity and meeting design rules like N/P doping, well tap spacing, and power rail continuity. They're crucial for maintaining power grid continuity and preventing design rule violations.
```
filler_placement $vars(tech,fillers)
check_placement -verbose
```
### Parasitic extraction

```
define_process_corner -ext_model_index 0 ss
extract_parasitics -ext_model_file  $vars(tech,rcx_rules)
```
