# Лабораторная работа №9 eBGP.
### Цель:

**Настроить BGP между автономными системами
Организовать доступность между офисами Москва и С.-Петербург**

### Задание:
[1. Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.](#1)

[2. Настроить eBGP между провайдерами Киторн и Ламас.](#2)

[3. Настроить eBGP между Ламас и Триада.](#3)

[4. Настроить eBGP между офисом С.-Петербург и провайдером Триада.](#4)

[5. Организовать IP доступность между пограничным роутерами офисами Москва и С.-Петербург.](#5)

**Схема сети:**

![схема](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/ebgp.png)

*Таблица IP адресов закрепленных за AS:*

| расположение |  номер AS   |  network |
|-------------| ------------| ---------|
| Москва |         1001   |  -         |
| С.-Петербург | 2042 | - |
| Киторн |  101 | 176.30.1.0/24 |
| Ламас | 301 | 213.30.2.0/24 | 
| Триада | 520 | 94.30.3.0/24

*Таблица IP адресов на линках:*

  | Расположение | маршрутизатор | интерфейс | IP адрес | сеть   |  направление |
  | ---- |----           | ----      | ----     | -------| -----        | 
  | Москва| R14 | e0/2 | 176.30.1.2/30|176.30.1.0/30| to R22 |
 | Москва | R15 |  e0/2 | 213.30.2.2/30| 213.30.2.0/30| to R21|
  | Ламас| R21 |  e0/0 | 213.30.2.1/30 | 213.30.2.0/30 | to R15 |
 | Ламас | R21 | e0/1 | 213.30.2.5/30 | 213.30.2.4/30 | to R22  |
 | Ламас | R21 | e0/2 | 213.30.2.9/30 | 213.30.2.8/30 | to R24 | 
 | Киторн | R22 | e0/0 | 176.30.1.1/30 | 176.30.1.0/30 | to R14| 
  | Киторн| R22 |  e0/1 | 213.30.2.6/30 | 213.30.2.4/30 | to R21 |
  | Киторн| R22 | e0/2 | 176.30.1.5/30 |176.30.1.4/30| to R23 |
 | Триада | R23 |  e0/0 | 176.30.1.6/30 | 176.30.1.4/30| to R22  |
 | Триада | R23 | e0/1 | 94.30.3.1/30 | 94.30.3.0/30 | to R25 | 
 | Триада | R23 |  e0/2 | 94.30.3.5/30 | 94.30.3.4/30 | to R24  |
 | Триада | R24 |  e0/0 | 213.30.2.10/30 | 213.30.2.8/30 | to R21 | 
 | Триада | R24 | e0/1 | 94.30.3.9/30 | 94.30.3.8/30 | to R26 | 
  | Триада| R24 |  e0/2 | 94.30.3.6/30 | 94.30.3.4/30 | to R23 |
 | Триада | R24 |  e0/3 | 94.30.3.13/30 | 94.30.3.12/30 | to R18 |
 | Триада | R25 |  e0/0 | 94.30.3.2/30 | 94.30.3.0/30 | to R23 | 
 | Триада | R25 | e0/1 | 94.30.3.17/30 | 94.30.3.16/30 | to R27 | 
 | Триада | R25 | e0/2 | 94.30.3.21/30 | 94.30.3.20/30 | to R26  |
 | Триада | R25 |  e0/3 | 94.30.3.25/30 | 94.30.3.24/30 | to R28 | 
 | Триада | R26 |  e0/0 | 94.30.3.10/30 | 94.30.3.8/30 | to R24 |  
 | Триада | R26 |  e0/1 | 94.30.3.29/30 | 94.30.3.28/30 | to R28 | 
 | Триада | R26 | e0/2 | 94.30.3.22/30 | 94.30.3.20/30 | to R25  |
 | Триада | R26 | e0/3 | 94.30.3.33/30 | 94.30.3.32/30 | to R18 |
  | С.-Петербург | R18 |  e0/2 | 94.30.3.14/30| 94.30.3.12/30| to R24|
  | С.-Петербург | R18 |  e0/3 | 94.30.3.34/30| 94.30.3.32/30| to R26 |


#### <a id=1> 1. Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас. </a>

**Пример настройки eBGP между Москва и Киторн (R14 - R22)**

**Настройки на R14 (Москва AS1001):**

```
R14#show  running-config | s bgp
router bgp 1001
 bgp log-neighbor-changes
 neighbor 176.30.1.1 remote-as 101
R14#

```
**Настройки на R22 (Киторн AS101):**

*добавим статический маршрут 176.30.1.0 255.255.255.0 > Null0:, чтобы подсеть появилась в таблице маршрутизации и мы могли всю ее анонсировать соседу*
*т.о хоть в таблице маршрутизации и появится запись отправляющая подсеть 176.30.1.0/24 в null, но так же там будут directly connected сети, которые имеют более узкую маску и значит более высокий приоритет для маршрутизации*

```
R22# show running-config | s bgp
router bgp 101
 bgp log-neighbor-changes
 network 176.30.1.0 mask 255.255.255.0
 neighbor 176.30.1.2 remote-as 1001
 neighbor 176.30.1.6 remote-as 520
 neighbor 213.30.2.5 remote-as 301
R22#
!
ip route 176.30.1.0 255.255.255.0 Null0
!
```
**Проверим соседство R14 - R22 (Москва AS1001 c Киторн AS101 ):**

![R14nei](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R14_nei.png)

**Аналогично настроим eBGP между Москва и Ламас.**

[ВСЕ КОНФИГИ МОЖНО ПОСМОТРЕТЬ ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/tree/main/lab9_eBGP/all_config)


#### <a id=2> 2. Настроить eBGP между провайдерами Киторн и Ламас.</a>

**Настройки на R22 (Киторн AS101):**

```
!
router bgp 101
 bgp log-neighbor-changes
 network 176.30.1.0 mask 255.255.255.0
 neighbor 176.30.1.2 remote-as 1001
 neighbor 176.30.1.6 remote-as 520
 neighbor 213.30.2.5 remote-as 301
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 176.30.1.0 255.255.255.0 Null0
!
```

**Настройки на R21 (Ламас AS301):**

```
!
router bgp 301
 bgp log-neighbor-changes
 network 213.30.2.0
 neighbor 213.30.2.2 remote-as 1001
 neighbor 213.30.2.6 remote-as 101
 neighbor 213.30.2.10 remote-as 520
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 213.30.2.0 255.255.255.0 Null0
!
```

**Проверим bgp соседство на R21 (Ламас AS301 c Киторн AS101):**

![R21nei](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R21_nei.png)


[ВСЕ КОНФИГИ МОЖНО ПОСМОТРЕТЬ ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/tree/main/lab9_eBGP/all_config)


#### <a id=3> 3. Настроить eBGP между Ламас и Триада.</a>

**Настройки на R21 (Ламас AS301):**

```
!
router bgp 301
 bgp log-neighbor-changes
 network 213.30.2.0
 neighbor 213.30.2.2 remote-as 1001
 neighbor 213.30.2.6 remote-as 101
 neighbor 213.30.2.10 remote-as 520
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 213.30.2.0 255.255.255.0 Null0
!
```

**Настройки на R24 (Триада AS520):**

```
!
router bgp 520
 bgp log-neighbor-changes
 network 94.30.3.0 mask 255.255.255.0
 neighbor 94.30.3.14 remote-as 2042
 neighbor 213.30.2.9 remote-as 301
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 94.30.3.0 255.255.255.0 Null0
!
```

**Проверим bgp соседство на R21 (Ламас AS301 c Триада AS520):**

![R21_nei](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R21_nei_1.png)


[ВСЕ КОНФИГИ МОЖНО ПОСМОТРЕТЬ ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/tree/main/lab9_eBGP/all_config)


#### <a id=4> 4. Настроить eBGP между офисом С.-Петербург и провайдером Триада. </a>

**Настройки на R18 (С.-Петербург AS2042):**

```
!
router bgp 2042
 bgp log-neighbor-changes
 neighbor 94.30.3.13 remote-as 520
 neighbor 94.30.3.33 remote-as 520
 maximum-paths 2
!
```

**Настройки на R24 (Триада AS520):**

```
!
router bgp 520
 bgp log-neighbor-changes
 network 94.30.3.0 mask 255.255.255.0
 neighbor 94.30.3.14 remote-as 2042
 neighbor 213.30.2.9 remote-as 301
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 94.30.3.0 255.255.255.0 Null0
!
```

**Настройки на R26 (Триада AS520):**

```
!
router bgp 520
 bgp log-neighbor-changes
 network 94.30.3.0 mask 255.255.255.0
 neighbor 94.30.3.34 remote-as 2042
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 94.30.3.0 255.255.255.0 Null0
!
```

**Проверим bgp соседей на R18 (С.-Петербург AS2042 c Триада AS520):**

![R18_nei](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R18_nei.png)


[ВСЕ КОНФИГИ МОЖНО ПОСМОТРЕТЬ ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/tree/main/lab9_eBGP/all_config)


**ПРОВЕРИМ BGP МАРШРУТЫ НА ВСЕХ РОУТЕРАХ:**

R14 (Москва AS1001):

![R14_routes](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R14_routes.png)


R15 (Москва AS1001):

![R15_routes](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R15_routes.png)

R21 (Ламас AS301):

![R21_routes](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R21_routes.png)


R22 (Киторн AS101):

![R22_routes](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R22_routes.png)

R24 (Триада AS520):

![R24_routes](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R24_routes.png)


R26 (Триада AS520):

![R26_routes](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R26_routes.png)

R18 (С.-Петербург  AS2042):

![R18_routes](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R18_routes.png)


#### <a id=5> 5. Организовать IP доступность между пограничным роутерами офисами Москва и С.-Петербург. </a>

Проверим связность между R18 С.-Петербург. и R14 R15 Москва:

 | Расположение | маршрутизатор | интерфейс | IP адрес | сеть   |  направление |
 | ----  |----           | ----      | ----     | -------| -----        | 
 | Москва| R14 | e0/2 | 176.30.1.2/30|176.30.1.0/30| to R22 |
 | Москва | R15 |  e0/2 | 213.30.2.2/30| 213.30.2.0/30| to R21|
 | С.-Петербург | R18 |  e0/2 | 94.30.3.14/30| 94.30.3.12/30| to R24|
 | С.-Петербург | R18 |  e0/3 | 94.30.3.34/30| 94.30.3.32/30| to R26 |


ping с R18 до R14 R15: 

![R18_ping](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R18_ping.png)

ping с R14 до R18: 
 
![R14_ping](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R14_ping.png)
  
 ping с R15 до R18: 

![R15_ping](https://github.com/MIranaNightshade/otus-networks/blob/main/lab9_eBGP/jpeg/R15_ping.png)


[ВСЕ КОНФИГИ МОЖНО ПОСМОТРЕТЬ ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/tree/main/lab9_eBGP/all_config)
