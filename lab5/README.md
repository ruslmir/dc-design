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
Далее настраиваем bgp evpn. Т.к. соседство по лупбекам то делаем ebgp-multihop 3. В address-family ipv4 отключаем соседство лупбеков, т.к. иначе у меня строятся соседи в глобальной таблице (или это нормальное поведение или у меня OS такая, но пока не делал через bgp listen range этого делать не надо было). Обязательно включаем community exteneded  
Spine 1
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
Leaf1
```
router bgp 65001
   router-id 10.255.252.1
   maximum-paths 4
   neighbor evpn peer group
   neighbor evpn remote-as 65000
   neighbor evpn update-source Loopback0
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor underlay peer group
   neighbor underlay remote-as 65000
   neighbor underlay bfd
   neighbor underlay password 7 DYTHXlpndyU=
   neighbor underlay send-community
   neighbor 10.0.1.0 peer group underlay
   neighbor 10.0.2.0 peer group underlay
   neighbor 10.255.254.1 peer group evpn
   neighbor 10.255.254.2 peer group evpn
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      network 10.255.252.1/32
      network 10.255.253.1/32
```

### Проверка bgp evpn 
Пока префиксы по нулям, т.к. vxlan туннели еще не созданы и не подняты
Spine1
```
Spine1#sh bgp evpn su
BGP summary information for VRF default
Router identifier 10.255.254.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.255.252.1 4 65001             71        72    0    0 00:52:59 Estab   0      0
  10.255.252.2 4 65002             68        67    0    0 00:50:46 Estab   0      0
  10.255.252.3 4 65003             69        67    0    0 00:50:50 Estab   0      0
```
Leaf1
```
Leaf1#sh bgp ev su
BGP summary information for VRF default
Router identifier 10.255.252.1, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.255.254.1 4 65000             24        25    0    0 00:00:30 Estab   0      0
  10.255.254.2 4 65000             26        27    0    0 00:00:30 Estab   0      0

```
