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

Проверим посмотрив в таблицу evpn для двух хостов 10.4.0.2 (vrf Customer1) и 10.4.1.2 (vrf Customer2). Видим что VNI: 100666 и VNI: 100667 соответственно. 

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
Каждая сеть видит  себя в пределах фабрики, но не видит друг друга, т.к. в разных vrf, т.е. отдельные таблицы маршрутизации
```
Client1_vl10> sh ip

NAME        : Client1_vl10[1]
IP/MASK     : 10.4.0.1/24
GATEWAY     : 10.4.0.254
DNS         :
MAC         : 00:50:79:66:68:06
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

Client1_vl10> ping 10.4.0.2
84 bytes from 10.4.0.2 icmp_seq=1 ttl=64 time=121.194 ms
84 bytes from 10.4.0.2 icmp_seq=2 ttl=64 time=13.113 ms
84 bytes from 10.4.0.2 icmp_seq=3 ttl=64 time=11.029 ms
84 bytes from 10.4.0.2 icmp_seq=4 ttl=64 time=10.563 ms
84 bytes from 10.4.0.2 icmp_seq=5 ttl=64 time=16.888 ms

Client1_vl10> ping 10.4.0.3
84 bytes from 10.4.0.3 icmp_seq=1 ttl=64 time=55.888 ms
84 bytes from 10.4.0.3 icmp_seq=2 ttl=64 time=17.925 ms
84 bytes from 10.4.0.3 icmp_seq=3 ttl=64 time=25.817 ms
84 bytes from 10.4.0.3 icmp_seq=4 ttl=64 time=44.057 ms
84 bytes from 10.4.0.3 icmp_seq=5 ttl=64 time=18.067 ms

Client1_vl10> ping 10.4.1.3
*10.4.0.254 icmp_seq=1 ttl=64 time=14.997 ms (ICMP type:3, code:0, Destination network unreachable)
*10.4.0.254 icmp_seq=2 ttl=64 time=21.239 ms (ICMP type:3, code:0, Destination network unreachable)
*10.4.0.254 icmp_seq=3 ttl=64 time=4.950 ms (ICMP type:3, code:0, Destination network unreachable)
*10.4.0.254 icmp_seq=4 ttl=64 time=4.477 ms (ICMP type:3, code:0, Destination network unreachable)
*10.4.0.254 icmp_seq=5 ttl=64 time=7.579 ms (ICMP type:3, code:0, Destination network unreachable)

Client1_vl10> trace 10.4.1.3
trace to 10.4.1.3, 8 hops max, press Ctrl+C to stop
 1   *10.4.0.254   13.148 ms (ICMP type:3, code:0, Destination network unreachable)  *

```
Сделаем стык BorderLeaf1 и BorderLeaf2 с роутером Branch (попытка сэмулировать удаленный филиал, поэтому там не l3 интерфейсы делаем, а подинтерфейсы, а interface Vlan ограниченное количество, поэтому по возможности лучше их не занимать). 
BLeaf1
```
interface Ethernet3
   no switchport
!
interface Ethernet3.998
   encapsulation dot1q vlan 998
   vrf Customer1
   ip address 172.16.1.1/30
!
interface Ethernet3.1000
   encapsulation dot1q vlan 1000
   vrf Customer2
   ip address 172.16.1.9/30
!
router bgp 65098
   vrf Customer1
      neighbor 172.16.1.2 remote-as 64999
      redistribute connected
      !
     
   !
   vrf Customer2
      neighbor 172.16.1.10 remote-as 64999
      redistribute connected
```

BLeaf2
```
```
