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
    <summary>Spine1</summary>
    Какой-нибудь длиинный дополнительный текст,  
    который по умолчанию должен быть скрыт. Его можно показать, нажав на спойлер.
</details>


<details>
    <summary>Spine2</summary>
    Какой-нибудь длиинный дополнительный текст
    , который  
    
</details>

<details>
    <summary>Client3</summary>
```
Clinet3> ping 10.4.0.4
84 bytes from 10.4.0.4 icmp_seq=1 ttl=64 time=11.313 ms
84 bytes from 10.4.0.4 icmp_seq=2 ttl=64 time=6.730 ms
84 bytes from 10.4.0.4 icmp_seq=3 ttl=64 time=6.406 ms
84 bytes from 10.4.0.4 icmp_seq=4 ttl=64 time=6.548 ms
84 bytes from 10.4.0.4 icmp_seq=5 ttl=64 time=7.517 ms
```

</details>

```
Clinet3> ping 10.4.0.4

84 bytes from 10.4.0.4 icmp_seq=1 ttl=64 time=11.313 ms
84 bytes from 10.4.0.4 icmp_seq=2 ttl=64 time=6.730 ms
84 bytes from 10.4.0.4 icmp_seq=3 ttl=64 time=6.406 ms
84 bytes from 10.4.0.4 icmp_seq=4 ttl=64 time=6.548 ms
84 bytes from 10.4.0.4 icmp_seq=5 ttl=64 time=7.517 ms

```

## Results {.tabset}

### Plots
