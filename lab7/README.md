# Лабораторная работа по теме "Построение multihoming"

### Цель:
- Настроить multihoming MCLAG и EVPN multihoming;

### Топология
![Требуемая топология](reference_topology.avif "Требуемая топология")

### Конфигурация
![Текущая топология](eve-ng_topology.png "Текущая топология")
Топология и адресация представлена выше на схеме. Добавим Leaf4 с аналогичными настройками Leaf1-Leaf4. В данной схеме Leaf1 и Leaf2 будут настроены с помощью mclag. lacp-neighbor-1 будет клиентом для этих двух коммутаторов. Leaf3 и Leaf4 будут настроены как EVPN multihoming. lacp-neighbor-2 будет клиентом для этих двух коммутаторов. 
### Настройка mlag
Для настройки mlag между Leaf1 и Leaf2 первое что надо сделать это интерфейс через который будет синхронизация и изучение маков и SVI. Можно выбирать любой vlan, но рекомендация Arista это 4094 влан, так и сделаем. в vlan 4094 укажем команду trunk group mlag-trunk, это означает, что каждый раз когда мы будет на транках включать команду switchport mode trunk у нас влан 4094 будет исключаться  для предотвращения петли. Этот влан должен быть только между Leaf1 и Leaf2. ip адреса Leaf настраиваем из сети 10.1.0.0, 3 октет указывает между какими Leaf строится (в нашем примере 1 и 2, т.е. 3 октет 12). 4 октет будет 1 для первого соседа, 2 для второго соседа. 
Leaf1
```
vlan 4094
   name mlag
   trunk group mlag-trunk
!
interface Vlan4094
   no autostate
   ip address 10.1.12.1/30
```
Отключаем stp на влане 4094 чтобы он никогда не блокировался и делаем Leaf1 и Leaf2 stp root коммутаторами, чтобы не было при падении линков перестроения stp и будут теряться плюсы mlag
```
no spanning-tree vlan-id 4094
spanning-tree mst 0 priority 8192
```
Далее настраиваем интерфейс между Leaf1 и Leaf2. Для резервирования и чтобы не было bottleneck рекомендуется делать агрегирование линков. Создаем транк и командой switchport trunk group mlag-trunk добавляем туда наш vlan 4094
```
interface Port-Channel4094
   description mlag
   switchport mode trunk
   switchport trunk group mlag-trunk
!   
interface Ethernet8
   channel-group 4094 mode active
!
interface Ethernet9
   channel-group 4094 mode active
```
Настройка самого mlag. Указываем наш SVI 4094 и соседа с которым будем обмениваться информацией. Ну и указываем сам peer-link через который это будет происходить. 
```
mlag configuration
   domain-id mlag-domain
   local-interface Vlan4094
   peer-address 10.1.12.2
   peer-link Port-Channel4094
```
Описания не нашел, но насколько понял tcp для синхронизации, а udp для изучения мак-адресов и пересылки трафика. 
![mlag-dump](mlag-dump.png "mlag-dump")
### Настройка Anycast GW
На всех Leaf настраиваем anycast gateway. Делаем виртуальный мак, который будет одинаковый для всех Leaf`ов. Создаем vrf Customer1 куда помещаем interface vlan10 и interface vlan 20 (шлюзы для вланов 10 и 20)
Leaf1
```

```
### Проверка работы anycast GW  
Clinet1 vlan 10
```

```
Client1 vlan 20
```

```
### Настройка VXLAN-EVPN для L3  
Подобно нашему mac-vrf мы также для L3 анонсируем наш созданный ip-vrf Customer1. Также делаем для всех всех Leaf где находится данный vrf. И добавляем в Vxlan туннель. 
Leaf1
```

```
###Проверка VXLAN-EVPN для L3
Проверим с клиента с 10 влана что он видит всех клиентов в 20 влане  
Client1 vlan 10
```

```
Теперь при создании interface Vlan у нас появились еще route-type 5 маршруты 10.4.0.0/24 и 10.4.1.0/24, которые видны со всех трех Leaf где они заведены
```
Leaf1#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.255.252.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.255.252.1:1 ip-prefix 10.4.0.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.255.252.2:1 ip-prefix 10.4.0.0/24
                                 10.255.253.2          -       100     0       65000 65002 i
 *  ec    RD: 10.255.252.2:1 ip-prefix 10.4.0.0/24
                                 10.255.253.2          -       100     0       65000 65002 i
 * >Ec    RD: 10.255.252.3:1 ip-prefix 10.4.0.0/24
                                 10.255.253.3          -       100     0       65000 65003 i
 *  ec    RD: 10.255.252.3:1 ip-prefix 10.4.0.0/24
                                 10.255.253.3          -       100     0       65000 65003 i
 * >      RD: 10.255.252.1:1 ip-prefix 10.4.1.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.255.252.2:1 ip-prefix 10.4.1.0/24
                                 10.255.253.2          -       100     0       65000 65002 i
 *  ec    RD: 10.255.252.2:1 ip-prefix 10.4.1.0/24
                                 10.255.253.2          -       100     0       65000 65002 i
 * >Ec    RD: 10.255.252.3:1 ip-prefix 10.4.1.0/24
                                 10.255.253.3          -       100     0       65000 65003 i
 *  ec    RD: 10.255.252.3:1 ip-prefix 10.4.1.0/24
                                 10.255.253.3          -       100     0       65000 65003 i
```
Таакже теперь при просмотре route-type 2 маршрутов видны не только мак-адреса но и ip адреса конечных хостов
```
Leaf1#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.255.252.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:100010 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >      RD: 65001:100010 mac-ip 0050.7966.6806 10.4.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 65002:100010 mac-ip 0050.7966.6807
                                 10.255.253.2          -       100     0       65000 65002 i
 *  ec    RD: 65002:100010 mac-ip 0050.7966.6807
                                 10.255.253.2          -       100     0       65000 65002 i
 * >Ec    RD: 65002:100010 mac-ip 0050.7966.6807 10.4.0.2
                                 10.255.253.2          -       100     0       65000 65002 i
 *  ec    RD: 65002:100010 mac-ip 0050.7966.6807 10.4.0.2
                                 10.255.253.2          -       100     0       65000 65002 i
 * >      RD: 65001:100020 mac-ip 0050.7966.6810
                                 -                     -       -       0       i
 * >      RD: 65001:100020 mac-ip 0050.7966.6810 10.4.1.1
                                 -                     -       -       0       i
 * >Ec    RD: 65002:100020 mac-ip 0050.7966.6811
                                 10.255.253.2          -       100     0       65000 65002 i
 *  ec    RD: 65002:100020 mac-ip 0050.7966.6811
                                 10.255.253.2          -       100     0       65000 65002 i
 * >Ec    RD: 65002:100020 mac-ip 0050.7966.6811 10.4.1.2
                                 10.255.253.2          -       100     0       65000 65002 i
 *  ec    RD: 65002:100020 mac-ip 0050.7966.6811 10.4.1.2
                                 10.255.253.2          -       100     0       65000 65002 i
```
Для примера устройство подключенное к Leaf2 10.4.1.2 (mac-address 0050.7966.6811).  
Два маршрута для мак-адреса 0050.7966.6811 (L2) через 10.255.254.1 (Spine1) и 10.255.254.2 (Spine2). VNI: 100020, Route-target для mac-vrf 1:100020  
Два маршрута для мак-адреса 0050.7966.6811 (L3) через 10.255.254.1 (Spine1) и 10.255.254.2 (Spine2). L3 VNI: 100666, который мы делали для vrf Customer. Route-target для ip-vrf 1:100666
```ruby
Leaf1#sh bgp evpn route-type mac-ip 0050.7966.6811 det
BGP routing table information for VRF default
Router identifier 10.255.252.1, local AS number 65001
BGP routing table entry for mac-ip 0050.7966.6811, Route Distinguisher: 65002:100020
 Paths: 2 available
  65000 65002
    10.255.253.2 from 10.255.254.2 (10.255.254.2)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, ECMP head, ECMP, best, ECMP contributor
      Extended Community: Route-Target-AS:1:100020 TunnelEncap:tunnelTypeVxlan
      VNI: 100020 ESI: 0000:0000:0000:0000:0000
  65000 65002
    10.255.253.2 from 10.255.254.1 (10.255.254.1)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, ECMP, ECMP contributor
      Extended Community: Route-Target-AS:1:100020 TunnelEncap:tunnelTypeVxlan
      VNI: 100020 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.6811 10.4.1.2, Route Distinguisher: 65002:100020
 Paths: 2 available
  65000 65002
    10.255.253.2 from 10.255.254.2 (10.255.254.2)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, ECMP head, ECMP, best, ECMP contributor
      Extended Community: Route-Target-AS:1:100020 Route-Target-AS:1:100666 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:03:37:66
      VNI: 100020 L3 VNI: 100666 ESI: 0000:0000:0000:0000:0000
  65000 65002
    10.255.253.2 from 10.255.254.1 (10.255.254.1)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, ECMP, ECMP contributor
      Extended Community: Route-Target-AS:1:100020 Route-Target-AS:1:100666 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:03:37:66
      VNI: 100020 L3 VNI: 100666 ESI: 0000:0000:0000:0000:0000

```
Смотрим таблицу мак-адресов Vxlan и таблицу маршрутизации vrf Customer1
```
Leaf1#sh vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6807  EVPN      Vx1  10.255.253.2     1       0:00:38 ago
  10  0050.7966.6808  EVPN      Vx1  10.255.253.3     1       0:00:44 ago
  20  0050.7966.6809  EVPN      Vx1  10.255.253.3     1       0:00:52 ago
  20  0050.7966.6811  EVPN      Vx1  10.255.253.2     1       0:00:59 ago
4094  5000.0003.3766  EVPN      Vx1  10.255.253.2     1       1:06:40 ago
4094  5000.0015.f4e8  EVPN      Vx1  10.255.253.3     1       1:06:40 ago
Total Remote Mac Addresses for this criterion: 6
Leaf1#sh vlan id 4094
VLAN  Name                             Status    Ports
----- -------------------------------- --------- -------------------------------
4094* VLAN4094                         active    Cpu, Vx1

* indicates a Dynamic VLAN

Leaf1#sh ip route vrf Customer1

VRF: Customer1
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

Gateway of last resort is not set

 B E      10.4.0.2/32 [200/0] via VTEP 10.255.253.2 VNI 100666 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 C        10.4.0.0/24 is directly connected, Vlan10
 B E      10.4.1.2/32 [200/0] via VTEP 10.255.253.2 VNI 100666 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B E      10.4.1.3/32 [200/0] via VTEP 10.255.253.3 VNI 100666 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        10.4.1.0/24 is directly connected, Vlan20
```

У нас получилась симметричная модель Integrated Routing and Bridging. Это видно даже из wireshark dump пакетов. Добавил для удобства столбец VNI. Запустим пинг с клиента в vlan10 (за Leaf1) в vlan20 (за Leaf2)
![symmetric IRB](symmetric-irb.png "symmetric IRB")
Насколько я понял в отличие от Cisco Nexus у Arista есть поддержка и symmetric и assymetric IRB. Для проверки assymetric IRB поправим немного конфигурацию на Leaf1 и Leaf2
```
interface VXlan1
 no vxlan vrf Customer1 vni 100666
```
Теперь запустим повторно пинг с клиента в vlan10 (за Leaf1) в vlan20 (за Leaf2). Как видно по разным VNI происходит локальный роутинг а потом бриджинг на Leaf2 со стороны Leaf1. И также в обратную сторорну. 
![asymmetric IRB](asymmetric-irb.png "asymmetric IRB")
