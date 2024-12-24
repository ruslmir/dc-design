# Лабораторная работа по теме "Оптимизация таблиц маршрутизации"

### Цель:
- Реализовать обмен между клиентами в разных vrf через route-type 5;

### Топология
![Требуемая топология](reference_topology.avif "Требуемая топология")

### Конфигурация
![Текущая топология](eve-ng_topology.png "Текущая топология")
Топология и адресация представлена выше на схеме. Добавим Border Leaf1 и Border Leaf2 в нашу топологию, их настроим аналогично обычным Leaf за исключением, что нет клиентов и не надо interface Vlan делать. Сеть 10.4.0.0/24 будет в vrf Customer1, сеть 10.4.1.0/24 в vrf Customer2. Для этого добавляем в нашу фабрику новый vrf и раскидываем по Leaf его. На бордерах не создаем interface Vlan
```
vrf instance Customer2
ip routing vrf Customer2
!
interface Vlan20
   vrf Customer2
   ip address virtual 10.4.1.254/24
! 
int vxl1   
vxlan vrf Customer2 vni 100667   
!
router bgp 650xx
 vrf Customer2
      rd 10.255.252.xx:2
      route-target import evpn 1:100667
      route-target export evpn 1:100667
      redistribute connected
```

Проверим посмотрив в таблицу evpn для двух хостов 10.4.0.2 (vrf Customer1) и 10.4.1.2 (vrf Customer2)

```python
Leaf3#sh bgp evpn route-type mac-ip 10.4.0.2 detail
BGP routing table information for VRF default
Router identifier 10.255.252.3, local AS number 65003
BGP routing table entry for mac-ip 0050.7966.6807 10.4.0.2, Route Distinguisher: 65001:100010
 Paths: 1 available
  65000 65001
    10.255.253.1 from 10.255.254.1 (10.255.254.1)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:1:100010 Route-Target-AS:1:100666 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:d5:5d:c0
      VNI: 100010 L3 VNI: 100666 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.6807 10.4.0.2, Route Distinguisher: 65002:100010
 Paths: 1 available
  65000 65002
    10.255.253.1 from 10.255.254.1 (10.255.254.1)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:1:100010 Route-Target-AS:1:100666 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:03:37:66
      VNI: 100010 L3 VNI: 100666 ESI: 0000:0000:0000:0000:0000


Leaf3#sh bgp evpn route-type mac-ip 10.4.1.2 detail
BGP routing table information for VRF default
Router identifier 10.255.252.3, local AS number 65003
BGP routing table entry for mac-ip 0050.7966.6811 10.4.1.2, Route Distinguisher: 65001:100020
 Paths: 1 available
  65000 65001
    10.255.253.1 from 10.255.254.1 (10.255.254.1)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:1:100020 Route-Target-AS:1:100667 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:d5:5d:c0
      VNI: 100020 L3 VNI: 100667 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.6811 10.4.1.2, Route Distinguisher: 65002:100020
 Paths: 1 available
  65000 65002
    10.255.253.1 from 10.255.254.1 (10.255.254.1)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:1:100020 Route-Target-AS:1:100667 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:03:37:66
      VNI: 100020 L3 VNI: 100667 ESI: 0000:0000:0000:0000:0000

```
