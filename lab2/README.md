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



Leaf1
```
config
```

