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
interface Ethernet3
   no switchport
!
interface Ethernet3.999
   encapsulation dot1q vlan 999
   vrf Customer1
   ip address 172.16.1.5/30
!
interface Ethernet3.1001
   encapsulation dot1q vlan 1001
   vrf Customer2
   ip address 172.16.1.13/30
!
router bgp 65099
   vrf Customer1
      rd 10.255.252.99:1
      route-target import evpn 1:100666
      route-target export evpn 1:100666
      neighbor 172.16.1.6 remote-as 64999
      redistribute connected
   !
   vrf Customer2
      rd 10.255.252.99:2
      route-target import evpn 1:100667
      route-target export evpn 1:100667
      neighbor 172.16.1.14 remote-as 64999
      redistribute connected
```
На Branch добавим команду  bgp bestpath as-path multipath-relax чтобы  балансировать с разными  путями (у нас будет два пути через 2 BLeaf`а через AS 65098 и AS 65099). Делаем все это в vrf test, к сожалению без этого у меня команда as-override недоступна. 
```
ip vrf test
 rd 1:1
 route-target export 1:1
 route-target import 1:1
!
interface FastEthernet0/0
 no ip address
 duplex auto
 speed auto
!
interface FastEthernet0/0.998
 ip vrf forwarding test
 encapsulation dot1Q 998
 ip address 172.16.1.2 255.255.255.252
!
interface FastEthernet0/0.1000
 ip vrf forwarding test
 encapsulation dot1Q 1000
 ip address 172.16.1.10 255.255.255.252
!
interface FastEthernet0/1
 no ip address
 duplex auto
 speed auto
!
interface FastEthernet0/1.999
 ip vrf forwarding test
 encapsulation dot1Q 999
 ip address 172.16.1.6 255.255.255.252
!
interface FastEthernet0/1.1001
 ip vrf forwarding test
 encapsulation dot1Q 1001
 ip address 172.16.1.14 255.255.255.252
!
router bgp 64999
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 maximum-paths 4
 !
 address-family ipv4 vrf test
  neighbor 172.16.1.1 remote-as 65098
  neighbor 172.16.1.1 activate
  neighbor 172.16.1.5 remote-as 65099
  neighbor 172.16.1.5 activate
  neighbor 172.16.1.9 remote-as 65098
  neighbor 172.16.1.9 activate
  neighbor 172.16.1.13 remote-as 65099
  neighbor 172.16.1.13 activate
  maximum-paths 4
 exit-address-family

```
### Проверка сходимости на роутере Branch
Смотрим что сессии поднялись и что таблица маршрутов заполнилась из vrf Customer1 и vrf Customer2 от BorderLeaf`ов
```
Branch#sh bgp vpnv4 unicast vrf test
Branch#sh bgp vpnv4 unicast vrf test su
BGP router identifier 10.100.12.1, local AS number 64999
BGP table version is 43, main routing table version 43
6 network entries using 840 bytes of memory
8 path entries using 544 bytes of memory
14/3 BGP path/bestpath attribute entries using 1736 bytes of memory
8 BGP AS-PATH entries using 192 bytes of memory
1 BGP extended community entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 1 (at peak 2) using 32 bytes of memory
BGP using 3368 total bytes of memory
BGP activity 54/45 prefixes, 256/245 paths, scan interval 15 secs

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.1.1      4 65098      30      45       43    0    0 00:10:01        2
172.16.1.5      4 65099      32      44       43    0    0 00:09:50        2
172.16.1.9      4 65098      30      45       43    0    0 00:10:09        2
172.16.1.13     4 65099      29      45       43    0    0 00:10:08        2


Branch#sh bgp vpnv4 unicast vrf test
BGP table version is 22, local router ID is 10.100.12.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  10.4.0.0/24      172.16.1.5                             0 65099 65000 65001 i
*>                  172.16.1.1                             0 65098 65000 65002 i
*  10.4.0.1/32      172.16.1.1                             0 65098 65000 65002 i
*>                  172.16.1.5                             0 65099 65000 65002 i
*  10.4.0.2/32      172.16.1.1                             0 65098 65000 65002 i
*>                  172.16.1.5                             0 65099 65000 65002 i
*  10.4.1.0/24      172.16.1.13                            0 65099 65000 65001 i
*>                  172.16.1.9                             0 65098 65000 65001 i
*  10.4.1.1/32      172.16.1.13                            0 65099 65000 65001 i
*>                  172.16.1.9                             0 65098 65000 65002 i
*  10.4.1.2/32      172.16.1.13                            0 65099 65000 65002 i
*>                  172.16.1.9                             0 65098 65000 65002 i
*> 10.100.10.0/24   0.0.0.0                  0         32768 i
*> 10.100.11.0/24   0.0.0.0                  0         32768 i
*> 10.100.12.0/24   0.0.0.0                  0         32768 i
r> 172.16.1.0/30    172.16.1.1                             0 65098 i
r> 172.16.1.4/30    172.16.1.5                             0 65099 i
r> 172.16.1.8/30    172.16.1.9                             0 65098 i
r> 172.16.1.12/30   172.16.1.13                            0 65099 i

Branch#sh ip route vrf test
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     172.16.0.0/30 is subnetted, 4 subnets
C       172.16.1.12 is directly connected, FastEthernet0/1.1001
C       172.16.1.8 is directly connected, FastEthernet0/0.1000
C       172.16.1.4 is directly connected, FastEthernet0/1.999
C       172.16.1.0 is directly connected, FastEthernet0/0.998
     10.0.0.0/8 is variably subnetted, 9 subnets, 2 masks
B       10.4.0.2/32 [20/0] via 172.16.1.5, 00:01:12
                    [20/0] via 172.16.1.1, 00:01:12
B       10.4.1.2/32 [20/0] via 172.16.1.13, 00:01:04
                    [20/0] via 172.16.1.9, 00:01:05
B       10.4.1.1/32 [20/0] via 172.16.1.13, 00:01:17
                    [20/0] via 172.16.1.9, 00:01:17
B       10.4.0.0/24 [20/0] via 172.16.1.5, 00:13:51
                    [20/0] via 172.16.1.1, 00:13:51
B       10.4.1.0/24 [20/0] via 172.16.1.13, 00:13:52
                    [20/0] via 172.16.1.9, 00:13:52
B       10.4.0.1/32 [20/0] via 172.16.1.5, 00:00:54
                    [20/0] via 172.16.1.1, 00:00:54
```
Как видим у нас слишком много /32 маршрутов (evpn-hosts), которые нам тут не нужны. Отфильтруем их со стороны BorderLeaf1 и BorderLeaf2. Примеер как это на BLeaf1  
BLeaf1
```
ip prefix-list deny-evpn-hosts seq 10 deny 0.0.0.0/0 eq 32
ip prefix-list deny-evpn-hosts seq 20 permit 0.0.0.0/0 le 32
!
router bgp 65098
 vrf Customer1
      address-family ipv4
         neighbor 172.16.1.2 prefix-list deny-evpn-hosts out
!
   vrf Customer2
      address-family ipv4
         neighbor 172.16.1.10 prefix-list deny-evpn-hosts out

```
В итоге теперь таблица bgp, а соответственно и таблица маршрутизации выглядат в разы меньше
```
Branch#sh ip bgp
BGP table version is 34, local router ID is 10.100.12.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  10.4.0.0/24      172.16.1.5                             0 65099 65000 65001 i
*>                  172.16.1.1                             0 65098 65000 65002 i
*  10.4.1.0/24      172.16.1.13                            0 65099 65000 65001 i
*>                  172.16.1.9                             0 65098 65000 65001 i
*> 10.100.10.0/24   0.0.0.0                  0         32768 i
*> 10.100.11.0/24   0.0.0.0                  0         32768 i
*> 10.100.12.0/24   0.0.0.0                  0         32768 i
r> 172.16.1.0/30    172.16.1.1                             0 65098 i
r> 172.16.1.4/30    172.16.1.5                             0 65099 i
r> 172.16.1.8/30    172.16.1.9                             0 65098 i
r> 172.16.1.12/30   172.16.1.13                            0 65099 i
Branch#
Branch#
Branch#sh ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     172.16.0.0/30 is subnetted, 4 subnets
C       172.16.1.12 is directly connected, FastEthernet0/1.1001
C       172.16.1.8 is directly connected, FastEthernet0/0.1000
C       172.16.1.4 is directly connected, FastEthernet0/1.999
C       172.16.1.0 is directly connected, FastEthernet0/0.998
     10.0.0.0/24 is subnetted, 5 subnets
B       10.4.0.0 [20/0] via 172.16.1.5, 00:54:21
                 [20/0] via 172.16.1.1, 00:54:21
B       10.4.1.0 [20/0] via 172.16.1.13, 00:54:21
                 [20/0] via 172.16.1.9, 00:54:23
```
