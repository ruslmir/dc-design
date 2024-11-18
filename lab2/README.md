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
 config
```

### Проверка

Spine1
```
Spine1#
Spine1#ping 10.0.1.1
PING 10.0.1.1 (10.0.1.1) 72(100) bytes of data.
80 bytes from 10.0.1.1: icmp_seq=1 ttl=64 time=11.8 ms
80 bytes from 10.0.1.1: icmp_seq=2 ttl=64 time=4.31 ms
80 bytes from 10.0.1.1: icmp_seq=3 ttl=64 time=37.0 ms
80 bytes from 10.0.1.1: icmp_seq=4 ttl=64 time=28.5 ms
80 bytes from 10.0.1.1: icmp_seq=5 ttl=64 time=19.6 ms

--- 10.0.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 51ms
rtt min/avg/max/mdev = 4.319/20.293/37.079/11.632 ms, pipe 3, ipg/ewma 12.781/16.442 ms
Spine1#
Spine1#ping 10.0.1.3
PING 10.0.1.3 (10.0.1.3) 72(100) bytes of data.
80 bytes from 10.0.1.3: icmp_seq=1 ttl=64 time=9.79 ms
80 bytes from 10.0.1.3: icmp_seq=2 ttl=64 time=4.87 ms
80 bytes from 10.0.1.3: icmp_seq=3 ttl=64 time=3.80 ms
80 bytes from 10.0.1.3: icmp_seq=4 ttl=64 time=4.41 ms
80 bytes from 10.0.1.3: icmp_seq=5 ttl=64 time=4.38 ms

--- 10.0.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 37ms
rtt min/avg/max/mdev = 3.801/5.455/9.794/2.196 ms, ipg/ewma 9.437/7.544 ms
Spine1#
Spine1#ping 10.0.1.5
PING 10.0.1.5 (10.0.1.5) 72(100) bytes of data.
80 bytes from 10.0.1.5: icmp_seq=1 ttl=64 time=33.3 ms
80 bytes from 10.0.1.5: icmp_seq=2 ttl=64 time=23.5 ms
80 bytes from 10.0.1.5: icmp_seq=3 ttl=64 time=17.0 ms
80 bytes from 10.0.1.5: icmp_seq=4 ttl=64 time=4.57 ms
80 bytes from 10.0.1.5: icmp_seq=5 ttl=64 time=5.25 ms

--- 10.0.1.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 80ms
rtt min/avg/max/mdev = 4.575/16.747/33.355/10.970 ms, pipe 3, ipg/ewma 20.239/24.309 ms
```
Spine2
```
Spine2#ping 10.0.2.1
PING 10.0.2.1 (10.0.2.1) 72(100) bytes of data.
80 bytes from 10.0.2.1: icmp_seq=1 ttl=64 time=14.9 ms
80 bytes from 10.0.2.1: icmp_seq=2 ttl=64 time=8.41 ms
80 bytes from 10.0.2.1: icmp_seq=3 ttl=64 time=4.60 ms
80 bytes from 10.0.2.1: icmp_seq=4 ttl=64 time=4.86 ms
80 bytes from 10.0.2.1: icmp_seq=5 ttl=64 time=4.55 ms

--- 10.0.2.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 80ms
rtt min/avg/max/mdev = 4.556/7.468/14.906/3.993 ms, ipg/ewma 20.027/10.984 ms
Spine2#ping 10.0.2.3
PING 10.0.2.3 (10.0.2.3) 72(100) bytes of data.
80 bytes from 10.0.2.3: icmp_seq=1 ttl=64 time=7.36 ms
80 bytes from 10.0.2.3: icmp_seq=2 ttl=64 time=8.23 ms
80 bytes from 10.0.2.3: icmp_seq=3 ttl=64 time=4.28 ms
80 bytes from 10.0.2.3: icmp_seq=4 ttl=64 time=6.97 ms
80 bytes from 10.0.2.3: icmp_seq=5 ttl=64 time=5.98 ms

--- 10.0.2.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 36ms
rtt min/avg/max/mdev = 4.282/6.568/8.233/1.355 ms, ipg/ewma 9.011/6.930 ms
Spine2#ping 10.0.2.5
PING 10.0.2.5 (10.0.2.5) 72(100) bytes of data.
80 bytes from 10.0.2.5: icmp_seq=1 ttl=64 time=43.7 ms
80 bytes from 10.0.2.5: icmp_seq=2 ttl=64 time=44.6 ms
80 bytes from 10.0.2.5: icmp_seq=3 ttl=64 time=33.6 ms
80 bytes from 10.0.2.5: icmp_seq=4 ttl=64 time=17.1 ms
80 bytes from 10.0.2.5: icmp_seq=5 ttl=64 time=5.07 ms

--- 10.0.2.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 92ms
rtt min/avg/max/mdev = 5.076/28.858/44.632/15.481 ms, pipe 4, ipg/ewma 23.054/35.132 ms
```
Client3
```
Clinet3> ping 10.4.0.4
84 bytes from 10.4.0.4 icmp_seq=1 ttl=64 time=11.313 ms
84 bytes from 10.4.0.4 icmp_seq=2 ttl=64 time=6.730 ms
84 bytes from 10.4.0.4 icmp_seq=3 ttl=64 time=6.406 ms
84 bytes from 10.4.0.4 icmp_seq=4 ttl=64 time=6.548 ms
84 bytes from 10.4.0.4 icmp_seq=5 ttl=64 time=7.517 ms
```
