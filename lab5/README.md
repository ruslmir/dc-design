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
Далее настраиваем bgp evpn. Т.к. соседство по лупбекам то делаем ebgp-multihop 3. В address-family ipv4 отключаем соседство лупбеко, т.к. иначе у меня строятся соседи (или это нормальное поведение или у меня OS такая, но пока не делал через bgp listen range этого делать не надо было). Обязательно включаем community exteneded
```
router bgp 65000
   router-id 10.255.254.1
   maximum-paths 4
   bgp listen range 10.255.252.0/24 peer-group evpn peer-filter AS-numbers
   bgp listen range 10.0.1.0/24 peer-group underlay peer-filter AS-numbers
   neighbor evpn peer group
   neighbor evpn next-hop-unchanged
   neighbor evpn update-source Loopback0
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor underlay peer group
   neighbor underlay bfd
   neighbor underlay password 7 DYTHXlpndyU=
   neighbor underlay send-community
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      network 10.255.254.1/32
      network 10.255.255.1/32

```

### Проверка
eBGP соседство удобнее проверять со стороны Spine коммутаторов. Заодно видно количество префиксов от каждого Leaf коммутатора (лупбек интерфейсы)  
Spine1
```

```
