# Лабораторная работа по теме "Построение Underlay сети(OSPF)"

### Цель:
- Настроить ISIS для Underlay сети;

### Топология
![Требуемая топология](reference_topology.avif "Требуемая топология")

### Конфигурация
![Текущая топология](eve-ng_topology.png "Текущая топология")
Топология и адресация представлена выше на схеме.   
Настройка ISIS на Spine коммутаторах, меняется только net адрес в ISIS процессе. Для примера ниже конфигурация коммутатора Spine1. net адрес задаем так: пишем 49 - это обозначение локального сегмента, аля RFC 1918 для ip адресов. Дальше указываем номер area 0001, т.к. для лабораторной работы у нас все в 1 area будет. Затем идет system-id, последние значения его запишем по аналогии с Loopback0. Spine1 имеет Loopback0 адрес 10.255.254.1, тогда system-id будет 0000.0000.0000.2541. В итоге net получается 49.0001.0000.0000.0000.2541.00 для Spine1   
По умолчанию в ISIS type L1/L2, зададим для простоты руками L1. В ISIS аутентификация настраивается отдельно для LSP, CSNP, PSNP и Hello пакетов.  Включим аутентификацию для   LSP, CSNP и PSNP пакетов текстом для простоты сбора через wireshark. 
```
router isis underlay
   net 49.0001.0000.0000.0000.2541.00
   is-type level-1
   authentication mode text
   authentication key global-pswd
```
![CSNP](CSNP.png "CSNP")   
Включаем ISIS на интерфейсах в сторону LEAF. Задаем аутентификацию для Hello пакетов для простоты сбора через wireshark.
```
interface Ethernet1
   description Leaf1
   no switchport
   ip address 10.0.1.0/31
   isis enable underlay
   isis bfd
   isis network point-to-point
   isis authentication mode text
   isis authentication key 0 interface-psw

interface Ethernet2
   description Leaf2
   no switchport
   ip address 10.0.1.2/31
   no ip ospf neighbor bfd
   isis enable underlay
   isis bfd
   isis network point-to-point
   isis authentication mode text
   isis authentication key 0 interface-psw

interface Ethernet3
   description Leaf3
   no switchport
   ip address 10.0.1.4/31
   no ip ospf neighbor bfd
   isis enable underlay
   isis bfd
   isis network point-to-point
   isis authentication mode text
   isis authentication key 0 interface-psw
```
![Hello](Hello.png "Hello")   
Настройка OSPF на Leaf коммутаторах, меняется только router-id в OSPF процессе. Для примера ниже конфигурация коммутатора Leaf1
```
ddd
```

### Проверка
OSPF соседство удобнее проверять со стороны Spine коммутаторов  
Spine1
```
ddd
```
Spine2
```
ddd
```
Проверку таблицу маршрутизации и IP доступности будем делать с Leaf1  


Leaf1
```
ddd
```


