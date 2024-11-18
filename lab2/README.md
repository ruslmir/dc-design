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

Spine1
```
config
```
Leaf1
```
config
```

