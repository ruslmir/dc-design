# Лабораторная работа по теме "Проектирование адресного пространства"

### Цель:
- Собрать схему CLOS;
- Распределить адресное пространство;

### Топология
![Требуемая топология](reference_topology.avif "Требуемая топология")

### Конфигурация
![Текущая топология](eve-ng_topology.png "Текущая топология")
Адреса Loopback интерфейсов

Loopback0: для Spine - 10.255.254.0/24, для Leaf - 10.255.252.0/24  
Loopback1: для Spine - 10.255.255.0/24, для Leaf - 10.255.253.0/24

Адреса p2p интерефейсов

10.0.x.y/31, где  
x - номер spine коммутатора  
y - порядковый номер (первый номер в сети для spine, второй номер в сети для leaf)

Адреса для сервисов
10.4.x.0/24, где  
х - сервис 

### Проверка

Spine1

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
