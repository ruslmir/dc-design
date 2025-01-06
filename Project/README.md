# Проектная работа "Связывание двух ЦОД с использованием технологии VXLAN Multisite"

### Цель:
- Реализовать построение двух ЦОД и связать их между собой с ипользованием технологии VXLAN Multisite на базе оборудования Arista;

### Топология
Дата-центр 1
![Дата центр 1](data-center1.png "Дата центр 1")  
Дата-центр 2
![Дата центр 2](data-center2.png "Дата центр 2")

### Конфигурация

### Настройка EVPN-VXLAN в ЦОД1 и ЦОД2
ЦОД 1 настроен в лабораторных работах 1-8. По аналогии настроим ЦОД 2, адресация будет задана по аналогии ![лабораторная работа 1](https://github.com/ruslmir/dc-design/tree/main/lab1 "лабораторная работа 1") 
underlay и overlay строится с использованием протокола маршрутизации BGP. На спайнах автономная система 65100, Лифы нумеруются с 65101 и далее. БордерЛифы с конца диапазона. Также настроим esi lag, в будущем это будет полезно показывая что маршруты типа 1 и 4 не распространяются между ЦОД`ами

### Настройка Multisite
Для отказоустойчивости будет два линка между 1) BLeaf1 и DC2-BLeaf1 2) BLeaf2 и DC2-BLeaf2. Провайдер предоставляет темное волокно, поэтому смело можем включать mtu 9214 на интерфейсах. Рассмотрим на примере BLeaf1, на других настройки аналогичные. Стыковочные сети берем /31   
BLeaf1
```
interface Ethernet7
   description DC2-BLeaf1
   mtu 9214
   no switchport
   ip address 172.16.0.0/31

Leaf1#ping 172.16.0.1 source 172.16.0.0
PING 172.16.0.1 (172.16.0.1) from 172.16.0.0 : 72(100) bytes of data.
80 bytes from 172.16.0.1: icmp_seq=1 ttl=64 time=5.27 ms
80 bytes from 172.16.0.1: icmp_seq=2 ttl=64 time=3.12 ms
80 bytes from 172.16.0.1: icmp_seq=3 ttl=64 time=4.46 ms
80 bytes from 172.16.0.1: icmp_seq=4 ttl=64 time=2.69 ms
80 bytes from 172.16.0.1: icmp_seq=5 ttl=64 time=5.61 ms
```
   
Строим underlay bgp между BLeaf1 и DC2-BLeaf1 и анонсируем Loopback для построения overlay. Вешаем prefix-list чтобы между фабриками двух ЦОД не получать лупбеки всех лифов и спайнов, т.к. все-равно нужны только лупбеки БордерЛифов
```
ip prefix-list Border-loopback
   seq 10 permit 10.255.252.98/32
   seq 20 permit 10.255.253.98/32

router bgp 65098
   router-id 10.255.252.98
   neighbor underlay-dci peer group
   neighbor underlay-dci remote-as 65198
   neighbor underlay-dci bfd
   neighbor underlay-dci password 7 qBOkfTkALkc=
   neighbor 172.16.0.1 peer group underlay-dci
   address-family ipv4
      no neighbor evpn activate
      no neighbor evpn-dci activate
      neighbor underlay-dci prefix-list Border-loopback out
      network 10.255.252.98/32
      network 10.255.253.98/32

BLeaf1#sh ip bgp | i 65198
 * >      10.254.252.98/32       172.16.0.1            0       -          100     0       65198 i
 * >      10.254.253.98/32       172.16.0.1            0       -          100     0       65198 i
```


```
vrf instance Customer2
ip routing vrf Customer2
!
interface Vlan20
   vrf Customer2
   ip address virtual 10.4.1.254/24
! 
int vxl1   
vxlan vrf Customer2 vni 100667   
!
router bgp 650xx
 vrf Customer2
      rd 10.255.252.xx:2
      route-target import evpn 1:100667
      route-target export evpn 1:100667
      redistribute connected
```
