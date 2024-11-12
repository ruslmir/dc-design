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

<details>
    <summary>Название</summary>
Spine1#ping 10.0.1.1
PING 10.0.1.1 (10.0.1.1) 72(100) bytes of data.
80 bytes from 10.0.1.1: icmp_seq=1 ttl=64 time=15.6 ms
80 bytes from 10.0.1.1: icmp_seq=2 ttl=64 time=4.48 ms
80 bytes from 10.0.1.1: icmp_seq=3 ttl=64 time=4.50 ms
80 bytes from 10.0.1.1: icmp_seq=4 ttl=64 time=4.63 ms
80 bytes from 10.0.1.1: icmp_seq=5 ttl=64 time=5.72 ms</details>
