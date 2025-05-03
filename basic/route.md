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
NOTE: now the stimation for parasitics are done with -global_routing

