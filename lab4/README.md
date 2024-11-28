# Лабораторная работа по теме "Построение Underlay сети с использованием протокола eBGP"

### Цель:
- Настроить eBGP для Underlay сети;

### Топология
![Требуемая топология](reference_topology.avif "Требуемая топология")

### Конфигурация
![Текущая топология](eve-ng_topology.png "Текущая топология")
Топология и адресация представлена выше на схеме.   
Все Spine коммутаторы будут находится в общей автономной системе 65000. Каждый Leaf коммутатор будет находится в своей автономной системе. Leaf1 - 65001, Leaf2 - 65002, Leaf3 - 65003 и т.д. при добавлении новых Leaf
Настройка Leaf коммутаторов одинакова, меняются только соседи. Т.к. у нас eBGP то route-reflector не нужен. Добавим bfd и аутентификацию bgp. Включаем ECMP, в IGP он по умолчанию, в BGP надо включать руками. Для небольшого уменьшения конфигурации добавим peer group

```
router bgp 65000
   router-id 10.255.254.1
   maximum-paths 4
   neighbor underlay peer group
   neighbor underlay bfd
   neighbor underlay password 0 test
   neighbor 10.0.1.1 peer group underlay
   neighbor 10.0.1.1 remote-as 65001
   neighbor 10.0.1.3 peer group underlay
   neighbor 10.0.1.3 remote-as 65002
   neighbor 10.0.1.5 peer group underlay
   neighbor 10.0.1.5 remote-as 65003
   network 10.255.254.1/32
   network 10.255.255.1/32
```

Настройка eBGP на Leaf коммутаторах одинакова, меняются только соседи и номер автономной системы. Для примера ниже конфигурация коммутатора Leaf1
```
router bgp 65001
   router-id 10.255.252.1
   maximum-paths 4
   neighbor underlay peer group
   neighbor underlay remote-as 65000
   neighbor underlay bfd
   neighbor underlay password 0 еуые
   neighbor 10.0.1.0 peer group underlay
   neighbor 10.0.2.0 peer group underlay
   network 10.255.252.1/32
   network 10.255.253.1/32
```

### Проверка
eBGP соседство удобнее проверять со стороны Spine коммутаторов. Заодно видно количество префиксов от каждого Leaf коммутатора (лупбек интерфейсы)  
Spine1
```
Spine1#sh ip bgp su
BGP summary information for VRF default
Router identifier 10.255.254.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.1.1         4  65001             37        37    0    0 00:17:36 Estab   2      2
  10.0.1.3         4  65002             37        37    0    0 00:17:36 Estab   2      2
  10.0.1.5         4  65003             36        37    0    0 00:17:36 Estab   2      2

Spine1#sh bfd peers detail | i Peer|Registered
Peer Addr 10.0.1.1, Intf Ethernet1, Type normal, Role active, State Up
Registered protocols: bgp
Peer Addr 10.0.1.3, Intf Ethernet2, Type normal, Role active, State Up
Registered protocols: bgp
Peer Addr 10.0.1.5, Intf Ethernet3, Type normal, Role active, State Up
Registered protocols: bgp
```
Spine2
```
Spine2#sh ip bgp su
BGP summary information for VRF default
Router identifier 10.255.254.2, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.2.1         4  65001             33        32    0    0 00:23:51 Estab   2      2
  10.0.2.3         4  65002             36        38    0    0 00:18:01 Estab   2      2
  10.0.2.5         4  65003             34        32    0    0 00:23:38 Estab   2      2

Spine2#sh bfd peers detail | i Peer|Registered
Peer Addr 10.0.2.1, Intf Ethernet1, Type normal, Role active, State Up
Registered protocols: bgp
Peer Addr 10.0.2.3, Intf Ethernet2, Type normal, Role active, State Up
Registered protocols: bgp
Peer Addr 10.0.2.5, Intf Ethernet3, Type normal, Role active, State Up
Registered protocols: bgp
```
Проверку таблицу маршрутизации и IP доступности будем делать с Leaf1  
Leaf1
```
Leaf1#sh ip route bgp

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 B E      10.255.252.2/32 [200/0] via 10.0.1.0, Ethernet1
                                  via 10.0.2.0, Ethernet2
 B E      10.255.252.3/32 [200/0] via 10.0.1.0, Ethernet1
                                  via 10.0.2.0, Ethernet2
 B E      10.255.253.2/32 [200/0] via 10.0.1.0, Ethernet1
                                  via 10.0.2.0, Ethernet2
 B E      10.255.253.3/32 [200/0] via 10.0.1.0, Ethernet1
                                  via 10.0.2.0, Ethernet2
 B E      10.255.254.1/32 [200/0] via 10.0.1.0, Ethernet1
 B E      10.255.254.2/32 [200/0] via 10.0.2.0, Ethernet2
 B E      10.255.255.1/32 [200/0] via 10.0.1.0, Ethernet1
 B E      10.255.255.2/32 [200/0] via 10.0.2.0, Ethernet2

Leaf1#ping 10.255.252.2 source 10.255.252.1
PING 10.255.252.2 (10.255.252.2) from 10.255.252.1 : 72(100) bytes of data.
80 bytes from 10.255.252.2: icmp_seq=1 ttl=63 time=29.4 ms
80 bytes from 10.255.252.2: icmp_seq=2 ttl=63 time=21.5 ms
80 bytes from 10.255.252.2: icmp_seq=3 ttl=63 time=14.1 ms
80 bytes from 10.255.252.2: icmp_seq=4 ttl=63 time=14.0 ms
80 bytes from 10.255.252.2: icmp_seq=5 ttl=63 time=14.7 ms

--- 10.255.252.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 94ms
rtt min/avg/max/mdev = 14.017/18.767/29.407/6.016 ms, pipe 2, ipg/ewma 23.505/23.769 ms
Leaf1#ping 10.255.252.3 source 10.255.253.1
PING 10.255.252.3 (10.255.252.3) from 10.255.253.1 : 72(100) bytes of data.
80 bytes from 10.255.252.3: icmp_seq=1 ttl=63 time=10.5 ms
80 bytes from 10.255.252.3: icmp_seq=2 ttl=63 time=10.5 ms
80 bytes from 10.255.252.3: icmp_seq=3 ttl=63 time=12.8 ms
80 bytes from 10.255.252.3: icmp_seq=4 ttl=63 time=23.4 ms
80 bytes from 10.255.252.3: icmp_seq=5 ttl=63 time=13.1 ms

--- 10.255.252.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 49ms
rtt min/avg/max/mdev = 10.521/14.110/23.461/4.805 ms, pipe 2, ipg/ewma 12.294/12.510 ms


Leaf1#ping 10.255.253.2 source 10.255.253.1
PING 10.255.253.2 (10.255.253.2) from 10.255.253.1 : 72(100) bytes of data.
80 bytes from 10.255.253.2: icmp_seq=1 ttl=63 time=14.1 ms
80 bytes from 10.255.253.2: icmp_seq=2 ttl=63 time=19.1 ms
80 bytes from 10.255.253.2: icmp_seq=3 ttl=63 time=15.2 ms
80 bytes from 10.255.253.2: icmp_seq=4 ttl=63 time=11.2 ms
80 bytes from 10.255.253.2: icmp_seq=5 ttl=63 time=12.2 ms

--- 10.255.253.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 59ms
rtt min/avg/max/mdev = 11.211/14.407/19.163/2.771 ms, pipe 2, ipg/ewma 14.863/14.117 ms
Leaf1#
Leaf1#
Leaf1#
Leaf1#ping 10.255.253.3 source 10.255.253.1
PING 10.255.253.3 (10.255.253.3) from 10.255.253.1 : 72(100) bytes of data.
80 bytes from 10.255.253.3: icmp_seq=1 ttl=63 time=13.2 ms
80 bytes from 10.255.253.3: icmp_seq=2 ttl=63 time=10.3 ms
80 bytes from 10.255.253.3: icmp_seq=3 ttl=63 time=39.2 ms
80 bytes from 10.255.253.3: icmp_seq=4 ttl=63 time=35.3 ms
80 bytes from 10.255.253.3: icmp_seq=5 ttl=63 time=22.9 ms

--- 10.255.253.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 90ms
rtt min/avg/max/mdev = 10.397/24.240/39.263/11.520 ms, pipe 2, ipg/ewma 22.702/19.117 ms
```
Посмотрим таблицу BGP.
```
Leaf1#sh ip bgp
BGP routing table information for VRF default
Router identifier 10.255.252.1, local AS number 65001
Route status codes: * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network                Next Hop            Metric  LocPref Weight  Path
 * >     10.255.252.1/32        -                     0       0       -       i
 * >Ec   10.255.252.2/32        10.0.2.0              0       100     0       65000 65002 i
 *  ec   10.255.252.2/32        10.0.1.0              0       100     0       65000 65002 i
 * >Ec   10.255.252.3/32        10.0.2.0              0       100     0       65000 65003 i
 *  ec   10.255.252.3/32        10.0.1.0              0       100     0       65000 65003 i
 * >     10.255.253.1/32        -                     0       0       -       i
 * >Ec   10.255.253.2/32        10.0.2.0              0       100     0       65000 65002 i
 *  ec   10.255.253.2/32        10.0.1.0              0       100     0       65000 65002 i
 * >Ec   10.255.253.3/32        10.0.2.0              0       100     0       65000 65003 i
 *  ec   10.255.253.3/32        10.0.1.0              0       100     0       65000 65003 i
 * >     10.255.254.1/32        10.0.1.0              0       100     0       65000 i
 * >     10.255.254.2/32        10.0.2.0              0       100     0       65000 i
 * >     10.255.255.1/32        10.0.1.0              0       100     0       65000 i
 * >     10.255.255.2/32        10.0.2.0              0       100     0       65000 i

```
