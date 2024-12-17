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
Описания не нашел, но насколько понял tcp для синхронизации, а udp для изучения мак-адресов, пересылки трафика и heartbeat. 
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
Поочередно отключим интерфейс Ethernet6 на Leaf2, а затем Leaf1 (shutdown)
```ruby
!оба линка работают, со стороны коммутатора lacp-neighbor-1 оба порта активные
lacp-neighbor-1#sh port-channel active-ports brief
Port Channel Port-Channel1:
  Active Ports: Ethernet1 Ethernet2
!
!Опускаем интерфейс на Leaf2, видно что Ethernet2 пропал из агрегированного канала
lacp-neighbor-1#sh port-channel active-ports brief
Port Channel Port-Channel1:
  Active Ports: Ethernet1
!
!Подняли оба интерфейса
lacp-neighbor-1#sh port-channel active-ports brief
Port Channel Port-Channel1:
  Active Ports: Ethernet1 Ethernet2
!
!Опустили интерефейс на Leaf1, видно что Ethernet2 пропал из агрегированного канала
lacp-neighbor-1#sh port-channel active-ports brief
Port Channel Port-Channel1:
  Active Ports: Ethernet2
```
Между между Client4_vl10 (подключен к lacp-neighbor-1) и Client3_vl20 (подключен к Leaf3) запущен постоянный ping. Тоже самое происходит если отключить один из Leaf.
```
Client4_vl10> ping 10.4.1.3 -c 200

10.4.1.3 icmp_seq=1 timeout
84 bytes from 10.4.1.3 icmp_seq=2 ttl=63 time=490.467 ms
84 bytes from 10.4.1.3 icmp_seq=3 ttl=63 time=386.395 ms
84 bytes from 10.4.1.3 icmp_seq=4 ttl=63 time=425.094 ms
84 bytes from 10.4.1.3 icmp_seq=5 ttl=63 time=410.323 ms
84 bytes from 10.4.1.3 icmp_seq=6 ttl=63 time=348.515 ms
84 bytes from 10.4.1.3 icmp_seq=7 ttl=63 time=318.691 ms
84 bytes from 10.4.1.3 icmp_seq=8 ttl=63 time=451.306 ms
84 bytes from 10.4.1.3 icmp_seq=9 ttl=63 time=331.981 ms
84 bytes from 10.4.1.3 icmp_seq=10 ttl=62 time=30.409 ms
84 bytes from 10.4.1.3 icmp_seq=11 ttl=62 time=25.408 ms
84 bytes from 10.4.1.3 icmp_seq=12 ttl=62 time=30.541 ms
84 bytes from 10.4.1.3 icmp_seq=13 ttl=62 time=39.027 ms
84 bytes from 10.4.1.3 icmp_seq=14 ttl=62 time=22.561 ms
84 bytes from 10.4.1.3 icmp_seq=15 ttl=62 time=269.243 ms
84 bytes from 10.4.1.3 icmp_seq=16 ttl=63 time=497.839 ms
84 bytes from 10.4.1.3 icmp_seq=17 ttl=62 time=67.659 ms
84 bytes from 10.4.1.3 icmp_seq=18 ttl=62 time=27.655 ms
84 bytes from 10.4.1.3 icmp_seq=19 ttl=62 time=32.388 ms
84 bytes from 10.4.1.3 icmp_seq=20 ttl=62 time=36.221 ms
84 bytes from 10.4.1.3 icmp_seq=21 ttl=62 time=28.239 ms
84 bytes from 10.4.1.3 icmp_seq=22 ttl=62 time=27.997 ms
84 bytes from 10.4.1.3 icmp_seq=23 ttl=62 time=39.350 ms
84 bytes from 10.4.1.3 icmp_seq=24 ttl=62 time=38.743 ms
84 bytes from 10.4.1.3 icmp_seq=25 ttl=62 time=30.136 ms
84 bytes from 10.4.1.3 icmp_seq=26 ttl=62 time=52.483 ms
84 bytes from 10.4.1.3 icmp_seq=27 ttl=62 time=30.293 ms
84 bytes from 10.4.1.3 icmp_seq=28 ttl=62 time=31.247 ms
84 bytes from 10.4.1.3 icmp_seq=29 ttl=62 time=34.097 ms
84 bytes from 10.4.1.3 icmp_seq=30 ttl=62 time=30.583 ms
84 bytes from 10.4.1.3 icmp_seq=31 ttl=62 time=28.314 ms
84 bytes from 10.4.1.3 icmp_seq=32 ttl=62 time=29.204 ms
84 bytes from 10.4.1.3 icmp_seq=33 ttl=62 time=81.321 ms
84 bytes from 10.4.1.3 icmp_seq=34 ttl=62 time=51.360 ms
84 bytes from 10.4.1.3 icmp_seq=35 ttl=62 time=29.838 ms
84 bytes from 10.4.1.3 icmp_seq=36 ttl=62 time=34.302 ms
84 bytes from 10.4.1.3 icmp_seq=37 ttl=62 time=76.772 ms
84 bytes from 10.4.1.3 icmp_seq=38 ttl=62 time=30.527 ms
84 bytes from 10.4.1.3 icmp_seq=39 ttl=62 time=41.214 ms
84 bytes from 10.4.1.3 icmp_seq=40 ttl=62 time=180.142 ms
10.4.1.3 icmp_seq=41 timeout
84 bytes from 10.4.1.3 icmp_seq=42 ttl=62 time=37.867 ms
84 bytes from 10.4.1.3 icmp_seq=43 ttl=62 time=24.507 ms
84 bytes from 10.4.1.3 icmp_seq=44 ttl=62 time=40.480 ms
84 bytes from 10.4.1.3 icmp_seq=45 ttl=62 time=32.993 ms
84 bytes from 10.4.1.3 icmp_seq=46 ttl=62 time=31.790 ms
84 bytes from 10.4.1.3 icmp_seq=47 ttl=62 time=25.891 ms
84 bytes from 10.4.1.3 icmp_seq=48 ttl=62 time=24.044 ms
^C
```
### Настройка EVPN multihoming
На всех Leaf настраиваем anycast gateway. Делаем виртуальный мак, который будет одинаковый для всех Leaf`ов. Создаем vrf Customer1 куда помещаем interface vlan10 и interface vlan 20 (шлюзы для вланов 10 и 20)
Leaf1
```

```
