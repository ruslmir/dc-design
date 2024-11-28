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
