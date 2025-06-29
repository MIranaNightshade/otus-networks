# Лабораторная работа №11 BGP управление анонсами.

## Задачи:
[1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).](#1)

[2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).](#2)

[3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.](#3)

[4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.](#4)

[5. Все сети в лабораторной работе должны иметь IP связность.](#5)

Карта сети: 

![map](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/%D0%9A%D0%B0%D1%80%D1%82%D0%B0.png)


**Таблица адресов закрепленных за AS:**

|Локация | номер AS| сеть|
| -- | -- | --|
| Москва | 1001 | 100.30.0.0/24 |
| С. Петербург | 2042 | 100.30.6.0/24 |
| Ламас | 301 | 213.30.2.0/24 |
| Киторн | 101 | 176.30.1.0/24 |
| Триада | 520 | 94.30.3.0/24 |

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


#### <a id=1>Настроим фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).</a>

*Чтобы не появлялось транзитного траффика нам нужно анонсировать соседям только сети принадлежащие нашей AS*
*Настроим для этого ip as-path access list и отфильтруем с его помощью анонсы в сторону R22 (176.30.1.1)*
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
 neighbor 176.30.1.1 prefix-list AS1001_SUM out
 neighbor 176.30.1.1 route-map PREPEND out
 neighbor 176.30.1.1 filter-list 1 out
!
ip forward-protocol nd
!
ip as-path access-list 1 permit ^$
!
```
Аналогично на R15:

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
 neighbor 213.30.2.1 filter-list 1 out
!
ip forward-protocol nd
!
ip as-path access-list 1 permit ^$
!
```

Проверим анонсы соседям с R14 и R15.

R14:

![R14 anonce](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/R14_anonce.png)

R15:

![R15 anonce](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/R15_anonce.png)

#### <a id=2>Настроим фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list)</a>

*Чтобы не появлялось транзитного траффика нам нужно анонсировать соседям только сети принадлежащие нашей AS*
*Настроим для этого prefix list и отфильтруем с его помощью анонсы в сторону R24 и R26*

```
!
router bgp 2042
 bgp log-neighbor-changes
 network 100.30.6.0 mask 255.255.255.0
 neighbor 94.30.3.13 remote-as 520
 neighbor 94.30.3.13 prefix-list NO_TRANSITE out
 neighbor 94.30.3.33 remote-as 520
 neighbor 94.30.3.33 prefix-list NO_TRANSITE out
 maximum-paths 2
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 94.30.3.33
ip route 0.0.0.0 0.0.0.0 94.30.3.13
ip route 100.30.6.0 255.255.255.0 Null0
!
!
ip prefix-list NO_TRANSITE seq 5 permit 100.30.6.0/24
```
Проверим анонсы соседям на R18: 

![R18 anonce](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/R18_anonce.png)


#### <a id=3>Настроим провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию</a>

Настройки на R22:

```
!
router bgp 101
 bgp log-neighbor-changes
 network 176.30.1.0 mask 255.255.255.0
 neighbor 176.30.1.2 remote-as 1001
 neighbor 176.30.1.2 default-originate
 neighbor 176.30.1.2 prefix-list DEFAULT_ROUTE out
 neighbor 176.30.1.6 remote-as 520
 neighbor 213.30.2.5 remote-as 301
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 Null0
ip route 176.30.1.0 255.255.255.0 Null0
!
!
ip prefix-list DEFAULT_ROUTE seq 5 permit 0.0.0.0/0
```

Проверим с R22 анонсы в сторону R14:

![R22 anonce](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/R18_anonce.png)

Проверим что мы получаем на R14:

От R22 пришел маршрут на Default route но лучшим маршрутом был выбран R15 из-за local preference. 

![R14 receive](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/R14_receive.png)

#### <a id=4> Настроим провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург</a>

Настройки на R21:

*ip as-path access-list 1 permit 2042$ фильтрует маршруты зародившиеся в AS2042 C.Петербург*
```
!
router bgp 301
 bgp log-neighbor-changes
 network 213.30.2.0
 neighbor 213.30.2.2 remote-as 1001
 neighbor 213.30.2.2 default-originate
 neighbor 213.30.2.2 filter-list 1 out
 neighbor 213.30.2.6 remote-as 101
 neighbor 213.30.2.10 remote-as 520
!
ip forward-protocol nd
!
ip as-path access-list 1 permit 2042$
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 Null0
```
Приверим на R15 маршруты пришедшие от R21 (213.30.2.1):

![R15 receive](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/R15_receive.png)


####  <a id=5>проверим IP связность.</a>

 |  Локация        |  Маршрутизатьор   | Интерфейс       |  IP   |
 |-- | -- | -- | --|
 |  Москва | R14 |  lo1 | 100.30.0.14/32|
 | Москва | R15 |  lo1 | 100.30.0.15/32|
 | С.-Петербург | R18 |  Lo1 | 100.30.6.18/32 |
| Чокурдах     | R28  | e0/0 | 94.30.3.30/30 | 
  |Чокурдах      | R28 | e0/1 | 94.30.3.26/30 |
  | Лабытнанги   | R27 |  e0/0 |94.30.3.18/30 |


R14:

![R14 check](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/R14_check.png)

R15:

![R15 check](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/R15_check.png)

R18: 

![R18 check](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/R18_check.png)

R27:

![R27 check](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/R27_check.png)

R28:

![R28 check](https://github.com/MIranaNightshade/otus-networks/blob/main/lab11_bgp_filter/jpeg/R28_check.png)





