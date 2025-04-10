### Placement

main steps:
  * config global routing
  * run global placement
  * set global capacitance and resitance in the net
  * generic parasitic estimation
  * repair transition/capacitance/fanout
  * tie cells insertion
  * detail placement

## config global routing
openroad uses [Fastroute](https://openroad.readthedocs.io/en/latest/main/src/grt/README.html) module for global routing and [Triton Route](https://openroad.readthedocs.io/en/latest/main/src/drt/README.html) for detail routing. 

With the _set_routing_layers_ command We define the minimum and maximum layers that the router (FastRoute or TritonRoute) can use for connecting nets.

```tcl
set_routing_layers \
        -signal ${vars(tech,route,signal,bottom_layer)}-${vars(tech,route,signal,top_layer)} \
        -clock  ${vars(tech,route,clock,bottom_layer)}-${vars(tech,route,clock,top_layer)}
```
We run a global placement. The global placement module in OpenROAD is based on the open-source [RePlAce tool](https://openroad.readthedocs.io/en/latest/main/src/gpl/README.html)
set_wire_rc sets the resistance (R) and capacitance (C) values used by OpenROAD to estimate wire delays before detailed routing is complete.
