# Проектная работа "Связывание двух ЦОД с использованием технологии VXLAN Multisite"

### Цель:
- Реализовать построение двух ЦОД и связать их между собой с ипользованием технологии VXLAN Multisite;

### Топология
Дата-центр 1
![Дата центр 1](data-center1.png "Дата центр 1")  
Дата-центр 2
![Дата центр 2](data-center2.png "Дата центр 2")

### Конфигурация

### Настройка EVPN-VXLAN в ЦОД1 и ЦОД2
ЦОД 1 настроен в лабораторных работах 1-8. По аналогии настроим ЦОД 2, адресация будет задана по аналогии ![лабораторная работа 1](https://github.com/ruslmir/dc-design/tree/main/lab1 "лабораторная работа 1") 
underlay и overlay строится с использованием протокола маршрутизации BGP. На спайнах автономная система 65100, Лифы нумеруются с 65101 и далее. БордерЛифы с конца диапазона. Также настроим esi lag, в будущем это будет полезно показывая что маршруты типа 1 и 4 не распространяются между ЦОД`ами


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
