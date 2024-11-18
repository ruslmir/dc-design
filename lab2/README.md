# Лабораторная работа по теме "Построение Underlay сети(OSPF)"

### Цель:
- исследовать построение Underlay сети с использованием протокола OSPF;

### Топология
![Требуемая топология](reference_topology.avif "Требуемая топология")

### Конфигурация
![Текущая топология](eve-ng_topology.png "Текущая топология")
Топология и адресация представлена выше на схеме.   
Настройка OSPF на Spine коммутаторах, меняется только router-id в OSPF процессе. Для примера ниже конфигурация коммутатора Spine1
```
hostname Spine1
!
ip routing
!
router ospf 1
   router-id 10.255.254.1
!
interface Ethernet1
   description Leaf1
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Leaf2
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description Leaf3
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip ospf area 0.0.0.0 
```

Настройка OSPF на Leaf коммутаторах, меняется только router-id в OSPF процессе. Для примера ниже конфигурация коммутатора Leaf1
```
hostname Leaf1
!   
ip routing
!   
router ospf 1
   router-id 10.255.252.1
!
interface Ethernet1
   description Spine1
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Spine2
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip ospf area 0.0.0.0
```

### Проверка
OSPF соседство удобнее проверять со стороны Spine коммутаторов  
Spine1
```
Spine1#show ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.255.252.1    1        default  0   FULL                   00:00:37    10.0.1.1        Ethernet1
10.255.252.2    1        default  0   FULL                   00:00:29    10.0.1.3        Ethernet2
10.255.252.3    1        default  0   FULL                   00:00:34    10.0.1.5        Ethernet3
```
Spine2
```
Spine2#show ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.255.252.1    1        default  0   FULL                   00:00:29    10.0.2.1        Ethernet1
10.255.252.2    1        default  0   FULL                   00:00:36    10.0.2.3        Ethernet2
10.255.252.3    1        default  0   FULL                   00:00:32    10.0.2.5        Ethernet3

```
Проверку таблицу маршрутизации и IP доступности будем делать с Leaf1  


Leaf1
```
Leaf1#show ip route ospf

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

 O        10.0.1.2/31 [110/20] via 10.0.1.0, Ethernet1
 O        10.0.1.4/31 [110/20] via 10.0.1.0, Ethernet1
 O        10.0.2.2/31 [110/20] via 10.0.2.0, Ethernet2
 O        10.0.2.4/31 [110/20] via 10.0.2.0, Ethernet2
 O        10.255.252.2/32 [110/30] via 10.0.1.0, Ethernet1
                                   via 10.0.2.0, Ethernet2
 O        10.255.252.3/32 [110/30] via 10.0.1.0, Ethernet1
                                   via 10.0.2.0, Ethernet2
 O        10.255.253.2/32 [110/30] via 10.0.1.0, Ethernet1
                                   via 10.0.2.0, Ethernet2
 O        10.255.253.3/32 [110/30] via 10.0.1.0, Ethernet1
                                   via 10.0.2.0, Ethernet2
 O        10.255.254.1/32 [110/20] via 10.0.1.0, Ethernet1
 O        10.255.254.2/32 [110/20] via 10.0.2.0, Ethernet2
 O        10.255.255.1/32 [110/20] via 10.0.1.0, Ethernet1
 O        10.255.255.2/32 [110/20] via 10.0.2.0, Ethernet2


Leaf1#ping 10.255.252.2 source loopback 0
PING 10.255.252.2 (10.255.252.2) from 10.255.252.1 : 72(100) bytes of data.
80 bytes from 10.255.252.2: icmp_seq=1 ttl=63 time=34.2 ms
80 bytes from 10.255.252.2: icmp_seq=2 ttl=63 time=20.2 ms
80 bytes from 10.255.252.2: icmp_seq=3 ttl=63 time=21.1 ms
80 bytes from 10.255.252.2: icmp_seq=4 ttl=63 time=13.1 ms
80 bytes from 10.255.252.2: icmp_seq=5 ttl=63 time=12.4 ms

--- 10.255.252.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 89ms
rtt min/avg/max/mdev = 12.400/20.237/34.264/7.857 ms, pipe 3, ipg/ewma 22.452/26.791 ms
Leaf1#
Leaf1#
Leaf1#ping 10.255.252.3 source loopback 0
PING 10.255.252.3 (10.255.252.3) from 10.255.252.1 : 72(100) bytes of data.
80 bytes from 10.255.252.3: icmp_seq=1 ttl=63 time=30.2 ms
80 bytes from 10.255.252.3: icmp_seq=2 ttl=63 time=27.2 ms
80 bytes from 10.255.252.3: icmp_seq=3 ttl=63 time=34.3 ms
80 bytes from 10.255.252.3: icmp_seq=4 ttl=63 time=21.8 ms
80 bytes from 10.255.252.3: icmp_seq=5 ttl=63 time=15.1 ms

--- 10.255.252.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 87ms
rtt min/avg/max/mdev = 15.146/25.789/34.378/6.707 ms, pipe 3, ipg/ewma 21.868/27.604 ms
Leaf1#
Leaf1#
Leaf1#ping 10.255.254.1 source loopback 0
PING 10.255.254.1 (10.255.254.1) from 10.255.252.1 : 72(100) bytes of data.
80 bytes from 10.255.254.1: icmp_seq=1 ttl=64 time=7.36 ms
80 bytes from 10.255.254.1: icmp_seq=2 ttl=64 time=5.22 ms
80 bytes from 10.255.254.1: icmp_seq=3 ttl=64 time=5.12 ms
80 bytes from 10.255.254.1: icmp_seq=4 ttl=64 time=4.66 ms
80 bytes from 10.255.254.1: icmp_seq=5 ttl=64 time=5.41 ms

--- 10.255.254.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 30ms
rtt min/avg/max/mdev = 4.663/5.557/7.363/0.939 ms, ipg/ewma 7.548/6.430 ms
Leaf1#
Leaf1#
Leaf1#ping 10.255.254.2 source loopback 0
PING 10.255.254.2 (10.255.254.2) from 10.255.252.1 : 72(100) bytes of data.
80 bytes from 10.255.254.2: icmp_seq=1 ttl=64 time=42.5 ms
80 bytes from 10.255.254.2: icmp_seq=2 ttl=64 time=33.7 ms
80 bytes from 10.255.254.2: icmp_seq=3 ttl=64 time=26.5 ms
80 bytes from 10.255.254.2: icmp_seq=4 ttl=64 time=20.8 ms
80 bytes from 10.255.254.2: icmp_seq=5 ttl=64 time=3.98 ms

--- 10.255.254.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 73ms
rtt min/avg/max/mdev = 3.988/25.535/42.542/12.993 ms, pipe 4, ipg/ewma 18.455/33.083 ms
```

