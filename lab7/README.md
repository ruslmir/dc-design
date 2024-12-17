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
Далее настраиваем агрегирование на коммутатор lacp-neighbor-1, включаем LACP и ассоциируем Portchannel с mlag id 1.
```
interface Port-Channel1
   switchport mode trunk
   mlag 1
!
interface Ethernet6
   channel-group 1 mode active
```
Со стороны коммутатора lacp-neighbor-1 это обычная настройка агрегирования, коммутатор ничего "не знает" про mlag.
```
interface Port-Channel1
   switchport mode trunk
!
interface Ethernet1
   channel-group 1 mode active
!
interface Ethernet2
   channel-group 1 mode active
```
###Провека mlag
Проверяем на Leaf, что mlag поднялся и Port-channel работает
```ruby
Leaf1#sh mlag
MLAG Configuration:
domain-id                          :         mlag-domain
local-interface                    :            Vlan4094
peer-address                       :           10.1.12.2
peer-link                          :    Port-Channel4094
peer-config                        :          consistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:03:37:66
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   0
Active-full                        :                   1
!
Leaf1#sh mlag interfaces detail
                                        local/remote
 mlag         state   local   remote    oper    config    last change   changes
------ ------------- ------- -------- ------- ---------- -------------- -------
    1   active-full     Po1      Po1   up/up   ena/ena    2:05:13 ago        10
!
Leaf1#
Leaf1#sh mlag interfaces members
Mlag1 is Port-Channel1
  Active Ports: Ethernet6 PeerEthernet6
```
Со стороны lacp-neighbor-1 оба порта в транке и активны
```
lacp-neighbor-1#sh port-channel active-ports brief
Port Channel Port-Channel1:
  Active Ports: Ethernet1 Ethernet2
```
Проверяем connectivity между Client4_vl10 (подключен к lacp-neighbor-1) и Client3_vl20 (подключен к Leaf3)
```
Client4_vl10> ping 10.4.1.3
84 bytes from 10.4.1.3 icmp_seq=1 ttl=63 time=302.309 ms
84 bytes from 10.4.1.3 icmp_seq=2 ttl=63 time=297.517 ms
84 bytes from 10.4.1.3 icmp_seq=3 ttl=63 time=294.466 ms
84 bytes from 10.4.1.3 icmp_seq=4 ttl=63 time=347.514 ms
84 bytes from 10.4.1.3 icmp_seq=5 ttl=63 time=321.830 ms
```
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

Таакже теперь при просмотре route-type 2 маршрутов видны не только мак-адреса но и ip адреса конечных хостов
```

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
