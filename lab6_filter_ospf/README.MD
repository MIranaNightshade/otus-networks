# Лабораторная работа №5 filter ospf. 

## Цели:
1. Настроить OSPF офисе Москва
2. Разделить сеть на зоны
3. Настроить фильтрацию между зонами

## Задачи: 
1. [Определить маршрутизаторы R14-R15 в зону 0 - backbone.](#title1)
2. [Определить маршрутизаторы R12-R13, L3 коммутаторы SW4-SW5 в зону 10. Дополнительно к маршрутам R12-R13 и SW4-SW5 должны получать маршрут по умолчанию.](#title2)
3. [Определить маршрутизатор R19 в зону 101, R19 должен получать только маршрут по умолчанию.](#title3)
4. [Определить маршрутизатор R20 в зону 102, R20 должен получать все маршруты, кроме маршрутов до сетей зоны 101.](#title4)

### Выполнение: 
**Графическое изображение зон в офисе Москва:**

![Москва ospf](https://github.com/MIranaNightshade/otus-networks/blob/main/lab6_filter_ospf/jpeg/area%20OSPF.png)

Таблица ospf router-id:

| device | router-id | 
|-----| ----| 
|R14 | 14.14.14.14 |
|R15 | 15.15.15.15 |
|R12 | 12.12.12.12 |
|R13 | 13.13.13.13 |
|R19 | 19.19.19.19 |
|R20 | 20.20.20.20 |
|SW4 | 4.4.4.4 |
|SW5 | 5.5.5.5 |

#### <a id=title1> 1. Определим маршрутизаторы R14-R15 в зону 0 - backbone.</a> 
*R14-R15 подключены друг к другу следующим образом: R14 e1/0 -R15 e1/0 следовательно эти интерфейсы будут в area 0*
настройки зеркальные, для примера часть конфига R14, объявим в area 0 интерфейс e1/0 и loopback:

```
!
interface Loopback0
 ip address 10.5.0.14 255.255.255.255
!

!
interface Ethernet0/1
 ip address 10.7.0.30 255.255.255.252
 ip ospf 1 area 10
!
!
router ospf 1
 router-id 14.14.14.14
 area 101 stub no-summary
 network 10.5.0.14 0.0.0.0 area 0
!
```
Проверка состояния соседства: 

```
R14#show  ip ospf  neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
15.15.15.15       1   FULL/BDR        00:00:39    10.7.0.58       Ethernet1/0
```
#### <a id=title2> 2. Определим маршрутизаторы R12-R13, L3 коммутаторы SW4-SW5 в зону 10. Настроим на R14 R15 маршруты по умолчанию и анонсируем их в OSPF. </a>

*R12 R13 SW4 SW5 являются inter area маршрутизаторами, поэтому все их интерфейсы определим в area 10. У R14 R15 в area 10 определим интерфейсы e0/0 и e0/1, они являются ABR.*

**Пример настройки на R12:**

```
interface Loopback0
 ip address 10.5.0.12 255.255.255.255
!
interface Ethernet0/0
 ip address 10.7.0.1 255.255.255.252
 ip ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.7.0.5 255.255.255.252
 ip ospf 1 area 10
!
interface Ethernet0/2
 ip address 10.7.0.9 255.255.255.252
 ip ospf 1 area 10
!
interface Ethernet0/3
 ip address 10.7.0.13 255.255.255.252
 ip ospf 1 area 10
!
interface Ethernet1/0
 ip address 10.7.0.61 255.255.255.252
 ip ospf 1 area 10
!
!
router ospf 1
 router-id 12.12.12.12
 network 10.5.0.12 0.0.0.0 area 10
!
```

*Для SW4 SW5 включим в OSPF management vlan 100 и подсети для VPC, обозначим интерфейсф на которых не будем искать соседей как passive.*

**Пример настройки на SW4:**

```
!
interface Ethernet1/0
 no switchport
 ip address 10.7.0.2 255.255.255.252
 ip ospf 1 area 10
 duplex auto
!
interface Ethernet1/1
 no switchport
 ip address 10.7.0.22 255.255.255.252
 ip ospf 1 area 10
 duplex auto
!
!
router ospf 1
 router-id 4.4.4.4
 passive-interface default
 no passive-interface Ethernet1/0
 no passive-interface Ethernet1/1
 no passive-interface Port-channel1
 no passive-interface Vlan100
 network 172.16.0.0 0.0.0.255 area 10
 network 172.17.0.0 0.0.0.255 area 10
!
```

**Проверим соседство на R12:** 

```
R12#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
15.15.15.15       1   FULL/BDR        00:00:33    10.7.0.14       Ethernet0/3
14.14.14.14       1   FULL/DR         00:00:37    10.7.0.10       Ethernet0/2
5.5.5.5           1   FULL/BDR        00:00:36    10.7.0.6        Ethernet0/1
4.4.4.4           1   FULL/BDR        00:00:35    10.7.0.2        Ethernet0/0
```

**Пропишем на R14 R15 маршрут по умолчанию (в сторону провайдеров) и анонсируем его в ospf.**
Пример конфига на R14:

```
!
router ospf 1
 router-id 14.14.14.14
 network 10.5.0.14 0.0.0.0 area 0
 default-information originate
!
ip route 0.0.0.0 0.0.0.0 176.30.1.1
!
```

**Проверка наличия маршрута на R12 (ожидаем увидеть маршруты O*E2  0.0.0.0/0):**

```
O*E2  0.0.0.0/0 [110/1] via 10.7.0.14, 00:53:31, Ethernet0/3
                [110/1] via 10.7.0.10, 01:01:06, Ethernet0/2
      10.0.0.0/8 is variably subnetted, 23 subnets, 3 masks
O        10.4.0.0/24 [110/11] via 10.7.0.6, 00:11:39, Ethernet0/1
                     [110/11] via 10.7.0.2, 00:11:39, Ethernet0/0
O        10.5.0.13/32 [110/21] via 10.7.0.14, 01:01:16, Ethernet0/3
                      [110/21] via 10.7.0.10, 01:01:06, Ethernet0/2
                      [110/21] via 10.7.0.6, 01:01:06, Ethernet0/1
                      [110/21] via 10.7.0.2, 01:01:06, Ethernet0/0
O IA     10.5.0.14/32 [110/11] via 10.7.0.10, 01:01:06, Ethernet0/2
O IA     10.5.0.15/32 [110/11] via 10.7.0.14, 00:53:26, Ethernet0/3
O IA     10.5.0.19/32 [110/21] via 10.7.0.10, 01:01:06, Ethernet0/2
O        10.7.0.16/30 [110/20] via 10.7.0.6, 01:01:06, Ethernet0/1
O        10.7.0.20/30 [110/20] via 10.7.0.2, 01:01:06, Ethernet0/0
O        10.7.0.24/30 [110/20] via 10.7.0.14, 00:53:32, Ethernet0/3
O        10.7.0.28/30 [110/20] via 10.7.0.10, 01:01:06, Ethernet0/2
O IA     10.7.0.32/30 [110/20] via 10.7.0.10, 01:01:06, Ethernet0/2
O IA     10.7.0.36/30 [110/20] via 10.7.0.14, 00:53:27, Ethernet0/3
O IA     10.7.0.56/30 [110/20] via 10.7.0.14, 00:53:26, Ethernet0/3
                      [110/20] via 10.7.0.10, 01:01:06, Ethernet0/2
      172.16.0.0/24 is subnetted, 1 subnets
O        172.16.0.0 [110/11] via 10.7.0.6, 00:11:59, Ethernet0/1
                    [110/11] via 10.7.0.2, 00:12:47, Ethernet0/0
      172.17.0.0/24 is subnetted, 1 subnets
O        172.17.0.0 [110/11] via 10.7.0.6, 00:11:49, Ethernet0/1
                    [110/11] via 10.7.0.2, 00:12:37, Ethernet0/0
R12#
```

#### <a id=title3> 3. Определить маршрутизатор R19 в зону 101, R19 должен получать только маршрут по умолчанию.</a>

*Раз R19 должен получать только маршрут по умолчанию определим тип area 101 как totally stub.* 

**R14 e0/3 и R19 e0/0 опрделим в зону 101:** 

**R14:**

```
!
interface Ethernet0/3
 ip address 10.7.0.33 255.255.255.252
 ip ospf 1 area 101
!
!
router ospf 1
 router-id 14.14.14.14
 area 101 stub no-summary
 network 10.5.0.14 0.0.0.0 area 0
 default-information originate
!
```

**R19:**

```
interface Ethernet0/0
 ip address 10.7.0.34 255.255.255.252
 ip ospf 1 area 101
!
!
router ospf 1
 router-id 19.19.19.19
 area 101 stub no-summary
 network 10.5.0.19 0.0.0.0 area 101
!
```
**Проверим соседство на R19:**

```
R19#show  ip ospf  neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
14.14.14.14       1   FULL/DR         00:00:32    10.7.0.33       Ethernet0/0
R19#
```

**Проверим маршруты на R19:**

```
R19#show  ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.7.0.33 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 10.7.0.33, 01:06:23, Ethernet0/0
R19#
```

#### 4. <a id=title4> Определим маршрутизатор R20 в зону 102, R20 должен получать все маршруты, кроме маршрутов до сетей зоны 101. </a>

*на R20 настроим стандартную зону 102.* 

**Проверим соседство на R20:**

```
R20#show  ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
15.15.15.15       1   FULL/BDR        00:00:34    10.7.0.37       Ethernet0/0
R20#
```
**Отфильтруем маршруты из зоны 101, фильтрацию будем производить на R15 т.к он для зоны 102 является ABR:**

```
!
router ospf 1
 router-id 15.15.15.15
 area 102 filter-list prefix WITHOUT_R19 in
 network 10.5.0.15 0.0.0.0 area 0
 default-information originate
!
!
ip prefix-list WITHOUT_R19 seq 5 deny 10.5.0.19/32
ip prefix-list WITHOUT_R19 seq 10 deny 10.7.0.32/30 le 32
ip prefix-list WITHOUT_R19 seq 15 permit 0.0.0.0/0 le 32
!
```

**Проверим маршруты на R20:**

```
R20#show  ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.7.0.37 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.7.0.37, 01:06:50, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 18 subnets, 3 masks
O IA     10.4.0.0/24 [110/31] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.5.0.12/32 [110/21] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.5.0.13/32 [110/21] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.5.0.14/32 [110/21] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.5.0.15/32 [110/11] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.7.0.0/30 [110/30] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.7.0.4/30 [110/30] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.7.0.8/30 [110/30] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.7.0.12/30 [110/20] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.7.0.16/30 [110/30] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.7.0.20/30 [110/30] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.7.0.24/30 [110/20] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.7.0.28/30 [110/30] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.7.0.56/30 [110/20] via 10.7.0.37, 01:06:46, Ethernet0/0
O IA     10.7.0.60/30 [110/30] via 10.7.0.37, 01:06:46, Ethernet0/0
      172.16.0.0/24 is subnetted, 1 subnets
O IA     172.16.0.0 [110/31] via 10.7.0.37, 00:26:07, Ethernet0/0
      172.17.0.0/24 is subnetted, 1 subnets
O IA     172.17.0.0 [110/31] via 10.7.0.37, 00:25:57, Ethernet0/0
R20#
```

[ВСЕ КОНФИГИ ПОЛНОСТЬЮ МОЖНО ПОСМОТРЕТЬ ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/blob/main/lab6_filter_ospf/configs/)








