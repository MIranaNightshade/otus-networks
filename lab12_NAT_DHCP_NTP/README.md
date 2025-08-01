# Лабораторная работа № 12 "Основные протоколы сети интернет"
## Задачи: 
[1. Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.](#1)

[2. Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.](#2)

[3. Настроить статический NAT для R20.](#3)

[4. Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления.](#4)

[5. Настроить статический NAT(PAT) для офиса Чокурдах.](#5)

[6. Настроить IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.](#6)

[7. Настроить NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.](#7)

[8. Все офисы в лабораторной работе должны иметь IP связность.](#8)

Карта сети: 

![карта](https://github.com/MIranaNightshade/otus-networks/blob/main/lab12_NAT_DHCP_NTP/jpeg/map.png)

Публичные IP адреса принадлежащие AS офисов:

| Номер AS | подсеть | расположение |
| ---- | ---- | ----|
|1001 | 100.30.0.0/24| Москва |
|2042 | 100.30.6.0/24 | С.Петербург |

#### <a id=1> 1. Настроим NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.</a>

Конфиг на R14:

```
!
interface Ethernet0/0
 ip address 10.7.0.10 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.7.0.30 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 10
!
interface Ethernet0/2
 ip address 176.30.1.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/3
 ip address 10.7.0.33 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 101
!
interface Ethernet1/0
 ip address 100.30.0.1 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ip ospf 1 area 0
!
ip nat pool IP_NAT 100.30.0.14 100.30.0.14 netmask 255.255.255.0
ip nat inside source list USER_FOR_NAT pool IP_NAT overload
!
ip access-list extended USER_FOR_NAT
 permit ip 172.16.0.0 0.0.0.255 any
 permit ip 172.17.0.0 0.0.0.255 any
!
```
Конфиг на R15:

```
!
interface Ethernet0/0
 ip address 10.7.0.26 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.7.0.14 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 10
!
interface Ethernet0/2
 ip address 213.30.2.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/3
 ip address 10.7.0.37 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 102
!
interface Ethernet1/0
 ip address 100.30.0.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ip ospf 1 area 0
!
ip nat pool IP_NAT 100.30.0.15 100.30.0.15 netmask 255.255.255.0
ip nat inside source list USER_FOR_NAT pool IP_NAT overload
!
ip access-list extended USER_FOR_NAT
 permit ip 172.16.0.0 0.0.0.255 any
 permit ip 172.17.0.0 0.0.0.255 any
!
```

Проверим работу nat:

![r15 nat](https://github.com/MIranaNightshade/otus-networks/blob/main/lab12_NAT_DHCP_NTP/jpeg/R15_nat.png)


#### <a id=2> 2. Настроим NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042. </a>

Конфиг на R18:

```
!
interface Ethernet0/0
 ip address 10.7.0.42 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/1
 ip address 10.7.0.54 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/2
 ip address 94.30.3.14 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/3
 ip address 94.30.3.34 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
ip nat pool NAT_PETERBURG 100.30.6.1 100.30.6.5 netmask 255.255.255.0
ip nat inside source list USER_FOR_NAT pool NAT_PETERBURG overload
!
ip access-list extended USER_FOR_NAT
 permit ip 172.16.6.0 0.0.0.255 any
 permit ip 172.17.6.0 0.0.0.255 any
```

Проверим работу nat:

![r18 nat](https://github.com/MIranaNightshade/otus-networks/blob/main/lab12_NAT_DHCP_NTP/jpeg/R18_nat.png)

#### <a id=3>Настроим статический NAT для R20 </a>
Конфиг на R15:

```
interface Ethernet0/0
 ip address 10.7.0.26 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.7.0.14 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 10
!
interface Ethernet0/2
 ip address 213.30.2.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/3
 ip address 10.7.0.37 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 102
!
interface Ethernet1/0
 ip address 100.30.0.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ip ospf 1 area 0
!
ip nat inside source static 10.5.0.20 100.30.0.15
R15#
```

Проверим работу nat на R20:

![r20 nat](https://github.com/MIranaNightshade/otus-networks/blob/main/lab12_NAT_DHCP_NTP/jpeg/R20_nat.png)


#### <a id=4> 4. Настроим NAT так, чтобы R19 был доступен с любого узла для удаленного управления.</a>

Для начала настроим ssh на R19:
*сгенерируем пару ключей командой crypto key generate rsa modulus 2048*

```
ip domain name otus.local
!
!
username admin privilege 15 password 0 admin
!
line vty 0 4
 exec-timeout 60 0
 logging synchronous
 login local
 transport input ssh
!
```

Настройки nat на R14 для управления R19:

```
!
interface Loopback1
 ip address 100.30.0.19 255.255.255.255
!
interface Ethernet0/0
 ip address 10.7.0.10 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.7.0.30 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 10
!
interface Ethernet0/2
 ip address 176.30.1.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/3
 ip address 10.7.0.33 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 101
!
interface Ethernet1/0
 ip address 100.30.0.1 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ip ospf 1 area 0
!
ip nat inside source static tcp 10.5.0.19 22 100.30.0.19 6262 extendable
```

Подключимся к R19 по ssh c R18:

![r19 ssh](https://github.com/MIranaNightshade/otus-networks/blob/main/lab12_NAT_DHCP_NTP/jpeg/ssh_R19.png)


#### <a id=5> 5. Настроим статический NAT(PAT) для офиса Чокурдах.</a>

*С помощью route-map будем определять с какого source IP идет трафик и через какой интерфейс он будет проходить после применения политик,
чтобы принять решение о том через какой outside интерфейс будем производить трансляцию*

```
!
interface Ethernet0/0
 ip address 94.30.3.30 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/1
 ip address 94.30.3.26 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/2.10
 encapsulation dot1Q 10
 ip address 172.16.5.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly in
 ip policy route-map FOR_VPC30
!
interface Ethernet0/2.20
 encapsulation dot1Q 20
 ip address 172.17.5.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly in
 ip policy route-map FOR_VPC31
!
ip nat inside source route-map E00 interface Ethernet0/0 overload
ip nat inside source route-map E01 interface Ethernet0/1 overload
!
!
ip access-list standard LAN
 permit 172.16.5.0 0.0.0.255
 permit 172.17.5.0 0.0.0.255
!
route-map E01 permit 10
 match ip address LAN
 match interface Ethernet0/1
!
route-map E00 permit 10
 match ip address LAN
 match interface Ethernet0/0
```

Проверим работу nat на R28:

![r28 nat](https://github.com/MIranaNightshade/otus-networks/blob/main/lab12_NAT_DHCP_NTP/jpeg/nat_R28.png)

#### 6. Настроим IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.

