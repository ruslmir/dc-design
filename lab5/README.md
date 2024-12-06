# Лабораторная работа по теме "Построение Overlay сети VXLAN-EVPN"

### Цель:
- Настроить VXLAN-EVPN для Overlay сети;

### Топология
![Требуемая топология](reference_topology.avif "Требуемая топология")

### Конфигурация
![Текущая топология](eve-ng_topology.png "Текущая топология")
Топология и адресация представлена выше на схеме.   
Underlay для топологии представлен eBGP (см. ![лабораторная работа 4](https://github.com/ruslmir/dc-design/tree/main/lab4 "лабораторная работа 4") ) Только в конфиге добавим настройку динамических BGP соседей.  
Spine1
```
router bgp 65000
   router-id 10.255.254.1
   maximum-paths 4
   bgp listen range 10.0.1.0/24 peer-group underlay peer-filter AS-numbers
   neighbor underlay peer group
   neighbor underlay bfd
   neighbor underlay password 7 DYTHXlpndyU=
   neighbor underlay send-community
   network 10.255.254.1/32
   network 10.255.255.1/32

peer-filter AS-numbers
   10 match as-range 65001-65099 result accept
```

На каждом коммутаторе включаем поддержку evpn вводя команду - service routing protocols model multi-agent
```
router bgp 65001
   router-id 10.255.252.1
   maximum-paths 4
   neighbor underlay peer group
   neighbor underlay remote-as 65000
   neighbor underlay bfd
   neighbor underlay password 0 test
   neighbor 10.0.1.0 peer group underlay
   neighbor 10.0.2.0 peer group underlay
   network 10.255.252.1/32
   network 10.255.253.1/32
```

### Проверка
eBGP соседство удобнее проверять со стороны Spine коммутаторов. Заодно видно количество префиксов от каждого Leaf коммутатора (лупбек интерфейсы)  
Spine1
```

```
