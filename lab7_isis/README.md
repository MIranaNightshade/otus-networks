# Лабораторная работа №7 IS-IS

### Задание: 
1. Настроить IS-IS в ISP Триада.
2. R23 и R25 находятся в зоне 2222.
3. R24 находится в зоне 24.
4. R26 находится в зоне 26.

#### Схема сети: 
![Схема сети](https://github.com/MIranaNightshade/otus-networks/blob/main/lab7_isis/jpeg/ISIS.png)

Таблица адресов маршрутизаторов ISP Триада:

| Маршрутизатор| Интерфейс   | IPv4 адрес|
| --------     | -------     | --------  |
| R23          | e0/0 | 176.30.1.6/30 | 
| R23          |  e0/1 | 94.30.3.1/30 | 
| R23          | e0/2   | 94.30.3.5/30 |
| R23          | Lo0    | 10.5.3.23/32 |
| R24          | e0/0   | 213.30.2.10/30 |
| R24          |  e0/1  | 94.30.3.9/30 | 
| R24 | e0/2   | 94.30.3.6/30 | 
| R24 |  e0/3  | 94.30.3.13/30 |
| R24 | Lo0    | 10.5.3.24/32 |
| R25 |  e0/0  | 94.30.3.2/30 | 
| R25 |  e0/1  | 94.30.3.17/30 | 
| R25 |  e0/2  | 94.30.3.21/30 | 
| R25 |  e0/3  | 94.30.3.25/30 | 
| R25 | Lo0    | 10.5.3.25/32 |
| R26 |  e0/0  | 94.30.3.10/30 | 
| R26 |  e0/1  | 94.30.3.29/30 |
| R26 |  e0/2  | 94.30.3.22/30 | 
| R26 | e0/3   | 94.30.3.33/30 |
| R26 | Lo0    | 10.5.3.26/32 |

#### Решение:
**Построим между всеми роутерами соседство L2, т.к в нашей топологии всего 4 роутера и создание дополнительно L1 соседства между R23 - R25 будет избыточным.**

**Настроим IS-IS на R23:**

*поместим его в area 2222, обозначим интерфейсы на которых не будем искать соседей isis как  passive-interface*

```
!
hostname R23
!
interface Loopback0
 ip address 10.5.3.23 255.255.255.255
!
interface Ethernet0/0
 ip address 176.30.1.6 255.255.255.252
!
interface Ethernet0/1
 ip address 94.30.3.1 255.255.255.252
 ip router isis
 isis network point-to-point
!
interface Ethernet0/2
 ip address 94.30.3.5 255.255.255.252
 ip router isis
 isis network point-to-point
!
router isis
 net 49.2222.0000.0000.0023.00
 is-type level-2-only
 passive-interface Ethernet0/0
 passive-interface Loopback0
!
```
**Настроим ISIS на R25, аналогично R23:**

```
!
hostname R25
!
interface Loopback0
 ip address 10.5.3.25 255.255.255.255
!
interface Ethernet0/0
 ip address 94.30.3.2 255.255.255.252
 ip router isis
 isis network point-to-point
!
interface Ethernet0/1
 ip address 94.30.3.17 255.255.255.252
!
interface Ethernet0/2
 ip address 94.30.3.21 255.255.255.252
 ip router isis
 isis network point-to-point
!
interface Ethernet0/3
 ip address 94.30.3.25 255.255.255.252
!
!
router isis
 net 49.2222.0000.0000.0025.00
 is-type level-2-only
 passive-interface Ethernet0/1
 passive-interface Ethernet0/3
 passive-interface Loopback0
!
```
**Настроим R24:**

*поместим его в area 24, обозначим интерфейсы на которых не будем искать соседей isis как  passive-interface*

```
!
hostname R24
!
interface Loopback0
 ip address 10.5.3.24 255.255.255.255
!
interface Ethernet0/0
 ip address 213.30.2.10 255.255.255.252
!
interface Ethernet0/1
 ip address 94.30.3.9 255.255.255.252
 ip router isis
 isis network point-to-point
!
interface Ethernet0/2
 ip address 94.30.3.6 255.255.255.252
 ip router isis
 isis network point-to-point
!
interface Ethernet0/3
 ip address 94.30.3.13 255.255.255.252
!
router isis
 net 49.0024.0000.0000.0024.00
 is-type level-2-only
 passive-interface Ethernet0/0
 passive-interface Ethernet0/3
 passive-interface Loopback0
!
```
**Настроим R26:**

*поместим его в area 26, обозначим интерфейсы на которых не будем искать соседей isis как  passive-interface*

```
!
hostname R26
!
interface Loopback0
 ip address 10.5.3.26 255.255.255.255
!
interface Ethernet0/0
 ip address 94.30.3.10 255.255.255.252
 ip router isis
 isis network point-to-point
!
interface Ethernet0/1
 ip address 94.30.3.29 255.255.255.252
!
interface Ethernet0/2
 ip address 94.30.3.22 255.255.255.252
 ip router isis
 isis network point-to-point
!
interface Ethernet0/3
 ip address 94.30.3.33 255.255.255.252
!
router isis
 net 49.0026.0000.0000.0026.00
 is-type level-2-only
 passive-interface Ethernet0/1
 passive-interface Ethernet0/3
 passive-interface Loopback0
!

```

**Проверим ISIS соседей на роутерах:**

R23:

![R23](https://github.com/MIranaNightshade/otus-networks/blob/main/lab7_isis/jpeg/R23_nei.png)

R25: 

![R25](https://github.com/MIranaNightshade/otus-networks/blob/main/lab7_isis/jpeg/R25_nei.png)

R24:

![R24](https://github.com/MIranaNightshade/otus-networks/blob/main/lab7_isis/jpeg/R24_nei.png)

R26:

![R26](https://github.com/MIranaNightshade/otus-networks/blob/main/lab7_isis/jpeg/R26_nei.png)

**Проверим связность, пропинговав с R23 IP адреса Loopback интерфейсов R24, R25, R26:**

![R23_ping](https://github.com/MIranaNightshade/otus-networks/blob/main/lab7_isis/jpeg/R23_ping.png)


[ВСЕ КОНФИГИ МОЖНО ПОСМОТРЕТЬ ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/tree/main/lab7_isis/all_config)


