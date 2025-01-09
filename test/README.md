```python
BLeaf1#sh bgp evpn route-type mac-ip 10.4.0.22 detail
BGP routing table information for VRF default
Router identifier 10.255.252.98, local AS number 65098
BGP routing table entry for mac-ip 0050.7966.681a 10.4.0.22, Route Distinguisher: 65098:100010
 Paths: 1 available
  65198 65100 65102
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:1:100010 TunnelEncap:tunnelTypeVxlan
      VNI: 100010 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.681a 10.4.0.22 remote, Route Distinguisher: 65198:100010
 Paths: 1 available
  65198 65100 65102
    10.254.253.98 from 10.254.252.98 (10.254.252.98)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:1:100010 TunnelEncap:tunnelTypeVxlan
      VNI: 100010 ESI: 0000:0000:0000:0000:0000

```
