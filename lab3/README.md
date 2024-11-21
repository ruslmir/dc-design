# Лабораторная работа по теме "Построение Underlay сети(ISIS)"

### Цель:
- Настроить ISIS для Underlay сети;

### Топология
![Требуемая топология](reference_topology.avif "Требуемая топология")

### Конфигурация
![Текущая топология](eve-ng_topology.png "Текущая топология")
Топология и адресация представлена выше на схеме.   
Настройка ISIS на Spine коммутаторах, меняется только net адрес в ISIS процессе. Для примера ниже конфигурация коммутатора Spine1. net адрес задаем так: пишем 49 - это обозначение локального сегмента, аля RFC 1918 для ip адресов. Дальше указываем номер area 0001, т.к. для лабораторной работы у нас все в 1 area будет. Затем идет system-id, последние значения его запишем по аналогии с Loopback0. Spine1 имеет Loopback0 адрес 10.255.254.1, тогда system-id будет 0000.0000.0000.2541. В итоге net получается 49.0001.0000.0000.0000.2541.00 для Spine1   
По умолчанию в ISIS type L1/L2, зададим для простоты руками L1. В ISIS аутентификация настраивается отдельно для LSP, CSNP, PSNP и Hello пакетов.  Включим аутентификацию для   LSP, CSNP и PSNP пакетов текстом для простоты сбора через wireshark. 
```
router isis underlay
   net 49.0001.0000.0000.0000.2541.00
   is-type level-1
   authentication mode text
   authentication key global-pswd
   address-family ipv4 unicast
```
![CSNP](CSNP.png "CSNP")   
Включаем ISIS на интерфейсах в сторону LEAF. Задаем аутентификацию для Hello пакетов для простоты сбора через wireshark.
```
interface Ethernet1
   description Leaf1
   no switchport
   ip address 10.0.1.0/31
   isis enable underlay
   isis bfd
   isis network point-to-point
   isis authentication mode text
   isis authentication key 0 interface-psw

interface Ethernet2
   description Leaf2
   no switchport
   ip address 10.0.1.2/31
   isis enable underlay
   isis bfd
   isis network point-to-point
   isis authentication mode text
   isis authentication key 0 interface-psw

interface Ethernet3
   description Leaf3
   no switchport
   ip address 10.0.1.4/31
   isis enable underlay
   isis bfd
   isis network point-to-point
   isis authentication mode text
   isis authentication key 0 interface-psw
```
![Hello](Hello.png "Hello")   
Настройка ISIS на Leaf коммутаторах одинакова, меняется только net адрес. Для примера ниже конфигурация коммутатора Leaf1
```
router isis underlay
   net 49.0001.0000.0000.0000.2521.00
   is-type level-1
   authentication mode text
   authentication key global-pswd
   address-family ipv4 unicast
```
Также настраиваем интерфейсы смотрящие в стороны SPINE коммтутаторов.
```
interface Ethernet1
   description Leaf1
   no switchport
   ip address 10.0.1.0/31
   isis enable underlay
   isis bfd
   isis network point-to-point
   isis authentication mode text
   isis authentication key 0 interface-pswd

interface Ethernet2
   description Leaf2
   no switchport
   ip address 10.0.1.2/31
   isis enable underlay
   isis bfd
   isis network point-to-point
   isis authentication mode text
   isis authentication key 0 interface-pswd
```

### Проверка
ISIS соседство удобнее проверять со стороны Spine коммутаторов  
Spine1
```
Spine1#sh isis neighbors
Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
underlay  default  Leaf1            L1   Ethernet1          P2P               UP    24          0F
underlay  default  Leaf2            L1   Ethernet2          P2P               UP    22          0F
underlay  default  Leaf3            L1   Ethernet3          P2P               UP    26          0F

Spine1#show bfd peers protocol isis
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type          LastUp  LastDown       LastDiag  State
--------- ----------- ----------- -------------------- ------- --------------- --------- -------------- -----
10.0.1.1  2269639566  2267063155        Ethernet1(15)  normal  11/21/24 04:30        NA  No Diagnostic     Up
10.0.1.3  1288286244  2344210618        Ethernet2(16)  normal  11/21/24 04:30        NA  No Diagnostic     Up
10.0.1.5  2085804511  2676713411        Ethernet3(17)  normal  11/21/24 04:31        NA  No Diagnostic     Up

```
Spine2
```
Spine2#sh  isis neighbors
Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
underlay  default  Leaf1            L1   Ethernet1          P2P               UP    25          10
underlay  default  Leaf2            L1   Ethernet2          P2P               UP    21          10
underlay  default  Leaf3            L1   Ethernet3          P2P               UP    28          10

Spine2#sh bfd peers protocol isis
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type          LastUp  LastDown       LastDiag  State
--------- ----------- ----------- -------------------- ------- --------------- --------- -------------- -----
10.0.2.1  4249411532   891249246        Ethernet1(15)  normal  11/21/24 04:30        NA  No Diagnostic     Up
10.0.2.3  1804296625  3554137351        Ethernet2(16)  normal  11/21/24 04:30        NA  No Diagnostic     Up
10.0.2.5  3791290555  2299320696        Ethernet3(17)  normal  11/21/24 04:31        NA  No Diagnostic     Up

```
Проверку таблицу маршрутизации и IP доступности будем делать с Leaf1  


Leaf1
```
Leaf1#sh ip route isis

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

 I L1     10.0.1.2/31 [115/20] via 10.0.1.0, Ethernet1
 I L1     10.0.1.4/31 [115/20] via 10.0.1.0, Ethernet1
 I L1     10.0.2.2/31 [115/20] via 10.0.2.0, Ethernet2
 I L1     10.0.2.4/31 [115/20] via 10.0.2.0, Ethernet2
 I L1     10.255.252.2/32 [115/30] via 10.0.1.0, Ethernet1
                                   via 10.0.2.0, Ethernet2
 I L1     10.255.252.3/32 [115/30] via 10.0.1.0, Ethernet1
                                   via 10.0.2.0, Ethernet2
 I L1     10.255.253.2/32 [115/30] via 10.0.1.0, Ethernet1
                                   via 10.0.2.0, Ethernet2
 I L1     10.255.253.3/32 [115/30] via 10.0.1.0, Ethernet1
                                   via 10.0.2.0, Ethernet2
 I L1     10.255.254.1/32 [115/20] via 10.0.1.0, Ethernet1
 I L1     10.255.254.2/32 [115/20] via 10.0.2.0, Ethernet2
 I L1     10.255.255.1/32 [115/20] via 10.0.1.0, Ethernet1
 I L1     10.255.255.2/32 [115/20] via 10.0.2.0, Ethernet2

Leaf1#sh bfd peer protocol isis
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type          LastUp  LastDown       LastDiag  State
--------- ----------- ----------- -------------------- ------- --------------- --------- -------------- -----
10.0.1.0  2267063155  2269639566        Ethernet1(15)  normal  11/21/24 04:30        NA  No Diagnostic     Up
10.0.2.0   891249246  4249411532        Ethernet2(16)  normal  11/21/24 04:30        NA  No Diagnostic     Up

Leaf1#ping 10.255.252.2 source loopback 0
PING 10.255.252.2 (10.255.252.2) from 10.255.252.1 : 72(100) bytes of data.
80 bytes from 10.255.252.2: icmp_seq=1 ttl=63 time=23.7 ms
80 bytes from 10.255.252.2: icmp_seq=2 ttl=63 time=16.0 ms
80 bytes from 10.255.252.2: icmp_seq=3 ttl=63 time=14.0 ms
80 bytes from 10.255.252.2: icmp_seq=4 ttl=63 time=12.3 ms
80 bytes from 10.255.252.2: icmp_seq=5 ttl=63 time=10.7 ms

--- 10.255.252.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 78ms
rtt min/avg/max/mdev = 10.765/15.385/23.751/4.543 ms, pipe 2, ipg/ewma 19.636/19.303 ms
Leaf1#
Leaf1#ping 10.255.252.3 source loopback 0
PING 10.255.252.3 (10.255.252.3) from 10.255.252.1 : 72(100) bytes of data.
80 bytes from 10.255.252.3: icmp_seq=1 ttl=63 time=16.0 ms
80 bytes from 10.255.252.3: icmp_seq=2 ttl=63 time=10.8 ms
80 bytes from 10.255.252.3: icmp_seq=3 ttl=63 time=8.92 ms
80 bytes from 10.255.252.3: icmp_seq=4 ttl=63 time=9.50 ms
80 bytes from 10.255.252.3: icmp_seq=5 ttl=63 time=9.18 ms

--- 10.255.252.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 57ms
rtt min/avg/max/mdev = 8.926/10.896/16.046/2.658 ms, pipe 2, ipg/ewma 14.335/13.354 ms
Leaf1#
Leaf1#ping 10.255.254.1 source loopback 0
PING 10.255.254.1 (10.255.254.1) from 10.255.252.1 : 72(100) bytes of data.
80 bytes from 10.255.254.1: icmp_seq=1 ttl=64 time=8.28 ms
80 bytes from 10.255.254.1: icmp_seq=2 ttl=64 time=4.38 ms
80 bytes from 10.255.254.1: icmp_seq=3 ttl=64 time=4.36 ms
80 bytes from 10.255.254.1: icmp_seq=4 ttl=64 time=4.70 ms
80 bytes from 10.255.254.1: icmp_seq=5 ttl=64 time=5.83 ms

--- 10.255.254.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 30ms
rtt min/avg/max/mdev = 4.367/5.517/8.289/1.487 ms, ipg/ewma 7.692/6.888 ms
Leaf1#ping 10.255.254.2 source loopback 0
PING 10.255.254.2 (10.255.254.2) from 10.255.252.1 : 72(100) bytes of data.
80 bytes from 10.255.254.2: icmp_seq=1 ttl=64 time=7.28 ms
80 bytes from 10.255.254.2: icmp_seq=2 ttl=64 time=3.67 ms
80 bytes from 10.255.254.2: icmp_seq=3 ttl=64 time=5.08 ms
80 bytes from 10.255.254.2: icmp_seq=4 ttl=64 time=3.76 ms
80 bytes from 10.255.254.2: icmp_seq=5 ttl=64 time=4.53 ms

--- 10.255.254.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 29ms
rtt min/avg/max/mdev = 3.679/4.870/7.289/1.316 ms, ipg/ewma 7.321/6.046 ms
```
Посмотрим базу LSDB. ISIS переносит hostname устройств, поэтому выглядит она довольно удобно
```
Leaf1#show isis hostname
IS-IS Instance: underlay VRF: default
Level  System ID           Hostname
L1     0000.0000.2521      Leaf1
L1     0000.0000.2522      Leaf2
L1     0000.0000.2523      Leaf3
L1     0000.0000.2541      Spine1
L1     0000.0000.2542      Spine2

Leaf1#sh isis database
IS-IS Instance: underlay VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    Leaf1.00-00                   7  22431   754    150 L1 <>
    Leaf2.00-00                   7   2271   666    150 L1 <>
    Leaf3.00-00                   7  44841   521    150 L1 <>
    Spine1.00-00                  8  55799   692    175 L1 <>
    Spine2.00-00                  8  36151   548    175 L1 <>

Leaf1# show isis database detail

IS-IS Instance: underlay VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    Leaf1.00-00                  12  22681   715    150 L1 <>
      LSP generation remaining wait time: 0 ms
      Time remaining until refresh: 415 s
      NLPID: 0xCC(IPv4)
      Hostname: Leaf1
      Authentication mode: Text Length: 12
      Area addresses: 49.0001.0000
      Interface address: 10.0.1.1
      Interface address: 10.0.2.1
      Interface address: 10.255.253.1
      Interface address: 10.255.252.1
      IS Neighbor          : Spine1.00           Metric: 10
      IS Neighbor          : Spine2.00           Metric: 10
      Reachability         : 10.0.1.0/31 Metric: 10 Type: 1 Up
      Reachability         : 10.0.2.0/31 Metric: 10 Type: 1 Up
      Reachability         : 10.255.253.1/32 Metric: 10 Type: 1 Up
      Reachability         : 10.255.252.1/32 Metric: 10 Type: 1 Up
      Router Capabilities: Router Id: 10.255.253.1 Flags: []
        Area leader priority: 250 algorithm: 0
    Leaf2.00-00                   9   1249   794    150 L1 <>
      Remaining lifetime received: 1199 s Modified to: 1200 s
      NLPID: 0xCC(IPv4)
      Hostname: Leaf2
      Authentication mode: Text Length: 12
      Area addresses: 49.0001.0000
      Interface address: 10.0.2.3
      Interface address: 10.0.1.3
      Interface address: 10.255.253.2
      Interface address: 10.255.252.2
      IS Neighbor          : Spine1.00           Metric: 10
      IS Neighbor          : Spine2.00           Metric: 10
      Reachability         : 10.0.2.2/31 Metric: 10 Type: 1 Up
      Reachability         : 10.0.1.2/31 Metric: 10 Type: 1 Up
      Reachability         : 10.255.253.2/32 Metric: 10 Type: 1 Up
      Reachability         : 10.255.252.2/32 Metric: 10 Type: 1 Up
      Router Capabilities: Router Id: 10.255.253.2 Flags: []
        Area leader priority: 250 algorithm: 0
    Leaf3.00-00                   9  43819   534    150 L1 <>
      Remaining lifetime received: 1199 s Modified to: 1200 s
      NLPID: 0xCC(IPv4)
      Hostname: Leaf3
      Authentication mode: Text Length: 12
      Area addresses: 49.0001.0000
      Interface address: 10.0.2.5
      Interface address: 10.0.1.5
      Interface address: 10.255.253.3
      Interface address: 10.255.252.3
      IS Neighbor          : Spine2.00           Metric: 10
      IS Neighbor          : Spine1.00           Metric: 10
      Reachability         : 10.0.2.4/31 Metric: 10 Type: 1 Up
      Reachability         : 10.0.1.4/31 Metric: 10 Type: 1 Up
      Reachability         : 10.255.253.3/32 Metric: 10 Type: 1 Up
      Reachability         : 10.255.252.3/32 Metric: 10 Type: 1 Up
      Router Capabilities: Router Id: 10.255.253.3 Flags: []
        Area leader priority: 250 algorithm: 0
    Spine1.00-00                 13   6834   818    175 L1 <>
      Remaining lifetime received: 1199 s Modified to: 1200 s
      NLPID: 0xCC(IPv4)
      Hostname: Spine1
      Authentication mode: Text Length: 12
      Area addresses: 49.0001.0000
      Interface address: 10.0.1.0
      Interface address: 10.0.1.4
      Interface address: 10.0.1.2
      Interface address: 10.255.254.1
      Interface address: 10.255.255.1
      IS Neighbor          : Leaf1.00            Metric: 10
      IS Neighbor          : Leaf3.00            Metric: 10
      IS Neighbor          : Leaf2.00            Metric: 10
      Reachability         : 10.0.1.0/31 Metric: 10 Type: 1 Up
      Reachability         : 10.0.1.4/31 Metric: 10 Type: 1 Up
      Reachability         : 10.0.1.2/31 Metric: 10 Type: 1 Up
      Reachability         : 10.255.254.1/32 Metric: 10 Type: 1 Up
      Reachability         : 10.255.255.1/32 Metric: 10 Type: 1 Up
      Router Capabilities: Router Id: 10.255.255.1 Flags: []
        Area leader priority: 250 algorithm: 0
    Spine2.00-00                 10  35129   505    175 L1 <>
      Remaining lifetime received: 1199 s Modified to: 1200 s
      NLPID: 0xCC(IPv4)
      Hostname: Spine2
      Authentication mode: Text Length: 12
      Area addresses: 49.0001.0000
      Interface address: 10.0.2.2
      Interface address: 10.0.2.4
      Interface address: 10.0.2.0
      Interface address: 10.255.255.2
      Interface address: 10.255.254.2
      IS Neighbor          : Leaf3.00            Metric: 10
      IS Neighbor          : Leaf1.00            Metric: 10
      IS Neighbor          : Leaf2.00            Metric: 10
      Reachability         : 10.0.2.2/31 Metric: 10 Type: 1 Up
      Reachability         : 10.0.2.4/31 Metric: 10 Type: 1 Up
      Reachability         : 10.0.2.0/31 Metric: 10 Type: 1 Up
      Reachability         : 10.255.255.2/32 Metric: 10 Type: 1 Up
      Reachability         : 10.255.254.2/32 Metric: 10 Type: 1 Up
      Router Capabilities: Router Id: 10.255.255.2 Flags: []
        Area leader priority: 250 algorithm: 0

```
Не нашел как включить passive interface default для ISIS. Насколько понимаю надо каждый интерфейс либо включать, либо выключать. Loopback интерфейсы автоматом в passive
```
Leaf1#sh isis interface brief
IS-IS Instance: underlay VRF: default
Interface Level IPv4 Metric IPv6 Metric Type           Adjacency
--------- ----- ----------- ----------- -------------- ---------
Loopback0 L1             10          10 loopback       (passive)
Loopback1 L1             10          10 loopback       (passive)
Ethernet1 L1             10          10 point-to-point         1
Ethernet2 L1             10          10 point-to-point         1
```

