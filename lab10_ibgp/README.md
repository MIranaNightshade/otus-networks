# Лабораторная работа №10 iBGP

### Цели:

[1. Настроить iBGP в офисе Москва между маршрутизаторами R14 и R15.](#1)

[2. Настроить iBGP в провайдере Триада, с использованием RR.](#2)

[3. Настроить офис Москва так, чтобы приоритетным провайдером стал Ламас.](#3)

[4. Настроить офис С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.](#4)

[5. Все сети в лабораторной работе должны иметь IP связность.](#5)

### Выполнение:

Карта сети: 

![map](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/%D0%9A%D0%B0%D1%80%D1%82%D0%B0.png)

**Таблица адресов закрепленных за AS:**

|Локация | номер AS| сеть|
| -- | -- | --|
| Москва | 1001 | 100.30.0.0/24 |
| С. Петербург | 2042 | 100.30.6.0/24 |
| Ламас | 301 | 213.30.2.0/24 |
| Киторн | 101 | 176.30.1.0/24 |
| Триада | 520 | 94.30.1.0/24 |



**Таблица IP адресов интерфейсов маршрутизаторов:**

  | Расположение | маршрутизатор | интерфейс | IP адрес | сеть   |  направление |
  | ---- |----           | ----      | ----     | -------| -----        | 
  | Москва| R14 | e0/2 | 176.30.1.2/30|176.30.1.0/30| to R22 |
  | Москва| R14 | e1/0 | 100.30.0.1/30|100.30.0.0/30| to R15 |
 |  Москва | R14 |  lo1 | 100.30.0.14/32| 100.30.0.14/32 | - |
 | Москва | R15 |  e0/2 | 213.30.2.2/30| 213.30.2.0/30| to R21|
 | Москва | R15 |  lo1 | 100.30.0.15/32| 100.30.0.15/32 | - |
 | Москва| R15 | e1/0 | 100.30.0.2/30|100.30.0.0/30| to R14 |
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
  | С.-Петербург | R18 |  Lo1 | 100.30.6.18/32 | 100.30.6.18/32| - |
  | Лабытнанги   | R27 |  e0/0 |94.30.3.18/30 | 94.30.3.16/30 | to R25 | 
  | Чокурдах     | R28  | e0/0 | 94.30.3.30/30 | 94.30.3.28/30 | to R26 | 
  |Чокурдах      | R28 | e0/1 | 94.30.3.26/30 | 94.30.3.24/30 | to R25 |



#### <a id=1>Настроим iBGP в офисе Москва между маршрутизаторами R14 и R15.</a>

*Построим iBGP соседство между Loopback R14 (10.5.0.14) и R15 (10.5.0.15)*
*route map FILTER чтобы не анонсировать серые подсети по bgp*

Настройки на R14:

```
!
router bgp 1001
 bgp log-neighbor-changes
 network 100.30.0.0 mask 255.255.255.0
 redistribute connected route-map FILTER
 neighbor 10.5.0.15 remote-as 1001
 neighbor 10.5.0.15 update-source Loopback0
 neighbor 10.5.0.15 next-hop-self
 neighbor 176.30.1.1 remote-as 101
!
ip forward-protocol nd
!
ip route 100.30.0.0 255.255.255.0 Null0 210
!
ip prefix-list FILTER seq 5 deny 10.0.0.0/8 le 32
ip prefix-list FILTER seq 10 permit 0.0.0.0/0 le 32
!
route-map FILTER permit 10
 match ip address prefix-list FILTER
!
```

Настройки на R15:

```
!
router bgp 1001
 bgp log-neighbor-changes
 network 100.30.0.0 mask 255.255.255.0
 redistribute connected route-map FILTER
 neighbor 10.5.0.14 remote-as 1001
 neighbor 10.5.0.14 update-source Loopback0
 neighbor 10.5.0.14 next-hop-self
 neighbor 213.30.2.1 remote-as 301
!
ip route 100.30.0.0 255.255.255.0 Null0 210
!
ip prefix-list FILTER seq 5 deny 10.0.0.0/8 le 32
ip prefix-list FILTER seq 10 permit 0.0.0.0/0 le 32
!
route-map FILTER permit 10
 match ip address prefix-list FILTER
!
```


Проверим соседство:

![neighbours R14 R15](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/nei_R15_R14.png)


**[ВСЕ КОНФИГИ МОЖНО ПОСМОТРЕТЬ ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/tree/main/lab10_ibgp/all_config)**


#### <a id=2>Настроим iBGP в провайдере Триада, с использованием RR.</a>

В качестве RR-сервер выберем R23, R23 и R26 сформируют соседство по TCP опираясь на маршрут полученный из is-is.

таблица loopback интерфейсов в провайдере Триада:

| Маршрутизатор  | адрес loopback |
| ----- | ----- |
 |R23	|10.5.3.23/32 |
 |R24 |	10.5.3.24/32 |
| R25 |	10.5.3.25/32 |
| R26 |	10.5.3.26/32 |


*Настройки RR-сервера R23:*

```
R23#show  running-config | s r b
router bgp 520
 bgp log-neighbor-changes
 bgp listen range 10.5.3.0/24 peer-group TRIADA
 network 94.30.3.0 mask 255.255.255.0
 neighbor TRIADA peer-group
 neighbor TRIADA remote-as 520
 neighbor TRIADA update-source Loopback0
 neighbor TRIADA route-reflector-client
 neighbor TRIADA next-hop-self
 neighbor 176.30.1.5 remote-as 101
R23#
```
*Настройки RR-клента на примере R24*

```
R24#show  running-config | s r b
router bgp 520
 bgp log-neighbor-changes
 network 94.30.3.0 mask 255.255.255.0
 neighbor 10.5.3.23 remote-as 520
 neighbor 10.5.3.23 update-source Loopback0
 neighbor 94.30.3.14 remote-as 2042
 neighbor 213.30.2.9 remote-as 301
R24#
```

*Проверим соседство на R23:*

![neighbours R23](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/nei_R23.png)

**[ВСЕ КОНФИГИ МОЖНО ПОСМОТРЕТЬ ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/tree/main/lab10_ibgp/all_config)**

#### <a id=3> Настроим офис Москва так, чтобы приоритетным провайдером стал Ламас.</a>

*Используем атрибут Local preference чтобы управлять исходящим траффиком, используем route-map TO_LAMAS:*
*prefix-list AS1001_SUM чтобы не анонсировать сети меньше чем /24 ebgp соседу* 

Настройки на R15:

```
!
router bgp 1001
 bgp log-neighbor-changes
 network 100.30.0.0 mask 255.255.255.0
 redistribute connected route-map FILTER
 neighbor 10.5.0.14 remote-as 1001
 neighbor 10.5.0.14 update-source Loopback0
 neighbor 10.5.0.14 next-hop-self
 neighbor 213.30.2.1 remote-as 301
 neighbor 213.30.2.1 prefix-list AS1001_SUM out
 neighbor 213.30.2.1 route-map TO_LAMAS in
!
ip route 100.30.0.0 255.255.255.0 Null0 210
!
ip prefix-list AS1001_SUM seq 10 permit 0.0.0.0/0 le 24
!
ip prefix-list FILTER seq 5 deny 10.0.0.0/8 le 32
ip prefix-list FILTER seq 10 permit 0.0.0.0/0 le 32
!
route-map FILTER permit 10
 match ip address prefix-list FILTER
!
route-map TO_LAMAS permit 10
 set local-preference 200
!
``` 

Проверим таблицу маршрутизации bgp на R15 ожидаем что лучше маршруты в другие AS будут через R21(213.30.2.1):

![R15 routes](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/R15_routes_bgp.png)


Проверим таблицу маршрутизации bgp на R14 ожидаем что лучше маршруты в другие AS будут через R15(10.5.0.15):

![R14 routes](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/R14_routes_bgp.png)


*Используем as-path prepend на R14 для управления входящим трафиком, чтобы ухудшить маршрут до нас через R22 Киторн:"

Настройки на R14:

```
router bgp 1001
 bgp log-neighbor-changes
 network 100.30.0.0 mask 255.255.255.0
 redistribute connected route-map FILTER
 neighbor 10.5.0.15 remote-as 1001
 neighbor 10.5.0.15 update-source Loopback0
 neighbor 10.5.0.15 next-hop-self
 neighbor 176.30.1.1 remote-as 101
 neighbor 176.30.1.1 prefix-list AS1001_SUM out
 neighbor 176.30.1.1 route-map PREPEND out
!
ip route 100.30.0.0 255.255.255.0 Null0 210
!
!
ip prefix-list AS1001_SUM seq 10 permit 0.0.0.0/0 le 24
!
ip prefix-list FILTER seq 5 deny 10.0.0.0/8 le 32
ip prefix-list FILTER seq 10 permit 0.0.0.0/0 le 32
!
route-map FILTER permit 10
 match ip address prefix-list FILTER
!
route-map PREPEND permit 10
 set as-path prepend 1001 1001 1001 1001 1001 1001
!
```

Проверим маршруты до нас на Киторн R22:

![R22 routes](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/R22_routes_bgp.png)

**[ВСЕ КОНФИГИ МОЖНО ПОСМОТРЕТЬ ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/tree/main/lab10_ibgp/all_config)**


#### <a id=4> Настроим офис С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.</a>

Настройки на R18:

```
R18#show running-config | s r b
router bgp 2042
 bgp log-neighbor-changes
 network 100.30.6.0 mask 255.255.255.0
 neighbor 94.30.3.13 remote-as 520
 neighbor 94.30.3.33 remote-as 520
 maximum-paths 2
R18#
```

Проверим таблицу маршрутизации на R18 чтобы убедиться что теперь у нас два пути в любую внешнюю сеть:

![R18 routes](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/R18_routes_bgp.png)


#### <a id=5> Проверим IP связность между офисами. </a>

Таблица адресов интерфейсов в офисах для проверки:


 |  Локация        |  Маршрутизатьор   | Интерфейс       |  IP             |
 |-- | -- | -- | --|
 |  Москва | R14 |  lo1 | 100.30.0.14/32|
 | Москва | R15 |  lo1 | 100.30.0.15/32|
 | С.-Петербург | R18 |  Lo1 | 100.30.6.18/32 |
| Чокурдах     | R28  | e0/0 | 94.30.3.30/30 | 
  |Чокурдах      | R28 | e0/1 | 94.30.3.26/30 |
  | Лабытнанги   | R27 |  e0/0 |94.30.3.18/30 |


R14:

![R14 check](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/R14_ping_ibgp.png)

R15:

![R15 check](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/R15_ping_ibgp.png)

R18:

![R18 check](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/R18_ping_ibgp.png)

R27:

![R27 check](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/R27_ping_ibgp.png)

R28:

![R28 check](https://github.com/MIranaNightshade/otus-networks/blob/main/lab10_ibgp/jpeg/R28_ping_ibgp.png)

**[ВСЕ КОНФИГИ МОЖНО ПОСМОТРЕТЬ ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/tree/main/lab10_ibgp/all_config)**

  


