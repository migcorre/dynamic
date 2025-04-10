# Placement

main steps:
  * config global routing
  * run global placement
  * set global capacitance and resitance in the net
  * generic parasitic estimation
  * repair transition/capacitance/fanout
  * tie cells insertion
  * detail placement

## Config global routing
openroad uses [Fastroute](https://openroad.readthedocs.io/en/latest/main/src/grt/README.html) module for global routing and [Triton Route](https://openroad.readthedocs.io/en/latest/main/src/drt/README.html) for detail routing. 

With the _set_routing_layers_ command We define the minimum and maximum layers that the router (FastRoute or TritonRoute) can use for connecting nets.

```tcl
set_routing_layers \
        -signal ${vars(tech,route,signal,bottom_layer)}-${vars(tech,route,signal,top_layer)} \
        -clock  ${vars(tech,route,clock,bottom_layer)}-${vars(tech,route,clock,top_layer)}
```

## Run global placement
We run a global placement. The global placement module in OpenROAD is based on the open-source [RePlAce tool](https://openroad.readthedocs.io/en/latest/main/src/gpl/README.html)
```tcl
global_placement -routability_driven -density ${vars(design,place,global,density)}
```
In this case we enable routability-driven mode (other could be Routability-Driven), with a target density of ${vars(design,place,global,density)}

## Set global capacitance and resitance in the net
set_wire_rc sets the resistance (R) and capacitance (C) values used by OpenROAD to estimate wire delays before detailed routing is complete. Because actual routing hasn’t happened yet, tools need a model to predict how wires will affect timing.

```tcl
source $vars(tech,rc)
set_wire_rc -signal -layer ${vars(tech,route,signal,wire_rc)}
set_wire_rc -clock  -layer ${vars(tech,route,clock,wire_rc)}
```

## Generic parasitic estimation
estimate_parasitics computes an early estimate of wire parasitics (resistance and capacitance) for your design’s nets, using a wireload or RC model.
This lets OpenROAD perform timing analysis and optimization (like buffer insertion) even before the real routes are available.

```tcl
estimate_parasitics -placement
```


