# Лабораторная работа №8 EGRP
#### Задание:

1. В офисе С.-Петербург настроить EIGRP.
2. R32 получает только маршрут по умолчанию.
3. R16-17 анонсируют только суммарные префиксы.
4. Использовать EIGRP named-mode для настройки сети.

**Схема сети:**

![Piter](https://github.com/MIranaNightshade/otus-networks/blob/main/lab8_EIGRP/jpeg/%D0%9F%D0%B5%D1%82%D0%B5%D1%80%D0%B1%D1%83%D1%80%D0%B3.png)

**Таблица IP адресов интерфейсов роутеров С.-Петербург:**
| device | interface | ip address | description |
| -----| ----- | ----- | ----- |
| R16 |  e0/1 | 10.7.0.41/30 |  to R18 | 
  | R16 |  e0/2 | 10.7.0.45/30 | to R17 | 
  | R16 |  e0/3 | 10.7.0.49/30 |  to R32 |
  | R17 |  e0/1 | 10.7.0.53/30 | to R18 |  
  | R17 |  e0/2 | 10.7.0.46/30 | to R16 | 
  | R18 |  e0/0 | 10.7.0.42/30 |  to R16 |
  | R18 |  e0/1 | 10.7.0.54/30 |  to R17 |
  | R18 |  e0/2 | 94.30.3.14/30| to R24|
  | R18 | e0/3 | 94.30.3.34/30|  to R26 |
  | R32 | e0/0 | 10.7.0.50/30 | to R16 |

##### <a id=title1>**Таблица loopback адресов/адресов управления и адресов хостов + суммарные префиксы:</a>**

| device | ip address | description | summary |
| ---- | ----- | --------- | ----- |
|R16	 |10.5.6.16/32 | LOOPBACK | 10.4.0.0/15 |
   |R17	 |10.5.6.17/32 |  LOOPBACK | 10.4.0.0/15 |
   |R18  | 10.5.6.18/32 |  LOOPBACK | 10.4.0.0/15 |
|R32	 |10.5.6.32/32 |  LOOPBACK | 10.4.0.0/15 |
| SW9 | 10.4.6.9/24 | MANAGEMENT | 10.4.0.0/15 |
|SW10 | 10.4.6.10/24 | MANAGEMENT | 10.4.0.0/15 |
|VPC8 | 172.16.6.8 |  - | 172.16.0.0/15 |
|VPC33 | 172.17.6.33 | - | 172.16.0.0/15 |

#### Выполнение:

##### *R16-17 анонсируют только суммарные префиксы*
*Суммируем адреса LOOPBACK роутеров и MANAGEMENT интерфейсов коммутаторов, а так же адреса конечных хостов согласно [таблице выше](#title1):*

**Настройка EIGRP на R16:**


  ```
  R16#show  running-config | s r e
  router eigrp PETERBURG
   !
   address-family ipv4 unicast autonomous-system 100
    !
    af-interface default
     passive-interface
    exit-af-interface
    !
    af-interface Ethernet0/1
     summary-address 10.4.0.0 255.254.0.0
     summary-address 172.16.0.0 255.254.0.0
     no passive-interface
    exit-af-interface
    !
    af-interface Ethernet0/2
     no passive-interface
    exit-af-interface
    !
    af-interface Ethernet0/3
     no passive-interface
    exit-af-interface
    !
    topology base
     distribute-list prefix DEFAULT_ONLY out Ethernet0/3
    exit-af-topology
    network 10.4.6.0 0.0.0.255
    network 10.5.6.16 0.0.0.0
    network 10.7.0.0 0.0.0.255
    network 172.16.6.0 0.0.0.255
    network 172.17.7.0 0.0.0.255
    eigrp router-id 16.16.16.16
   exit-address-family
  R16#
  ```


**Настройка EIGRP на R17:**


  ```
  R17#show  running-config | s r e
  router eigrp PETERBURG
   !
   address-family ipv4 unicast autonomous-system 100
    !
    af-interface default
     passive-interface
    exit-af-interface
    !
    af-interface Ethernet0/1
     summary-address 10.4.0.0 255.254.0.0
     summary-address 172.16.0.0 255.254.0.0
     no passive-interface
    exit-af-interface
    !
    af-interface Ethernet0/2
     no passive-interface
    exit-af-interface
    !
    topology base
    exit-af-topology
    network 10.4.6.0 0.0.0.255
    network 10.5.6.17 0.0.0.0
    network 10.7.0.0 0.0.0.255
    network 172.16.6.0 0.0.0.255
    network 172.17.7.0 0.0.0.255
    eigrp router-id 17.17.17.17
   exit-address-family
  R17#
  ```

**Настройка EIGRP на R18:**


  ```
  R18#show  running-config | s r e
  router eigrp PETERBURG
   !
   address-family ipv4 unicast autonomous-system 100
    !
    af-interface default
     passive-interface
    exit-af-interface
    !
    af-interface Ethernet0/0
     no passive-interface
    exit-af-interface
    !
    af-interface Ethernet0/1
     no passive-interface
    exit-af-interface
    !
    topology base
     redistribute static
    exit-af-topology
    network 10.4.6.0 0.0.0.255
    network 10.5.6.18 0.0.0.0
    network 10.7.0.0 0.0.0.255
    network 172.16.6.0 0.0.0.255
    network 172.17.7.0 0.0.0.255
    eigrp router-id 18.18.18.18
   exit-address-family
  R18#
  ```


**Проверим соседство и таблицу маршрутизации на R18:**


*соседство:*

![Соседство](https://github.com/MIranaNightshade/otus-networks/blob/main/lab8_EIGRP/jpeg/R18_NEI.png)


*Таблица маршрутизации:*


![Таблица маршрутизации](https://github.com/MIranaNightshade/otus-networks/blob/main/lab8_EIGRP/jpeg/R18_IP_ROUTE.png)


**Проверим связность отправив ping до конечных хостов, роутер loopback и management интерфейсов коммутаторов:**


![PING](https://github.com/MIranaNightshade/otus-networks/blob/main/lab8_EIGRP/jpeg/R18_PING_.png)



##### *R32 получает только маршрут по умолчанию.*

*Зададим статитикой на R18 маршруты по умолчанию в сторону R24 и R26, так же используем redistribute static чтобы анонсировать статические маршруты соседям EIGRP:*


```
!
router eigrp PETERBURG
 !
 address-family ipv4 unicast autonomous-system 100
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  !
  topology base
   redistribute static
  exit-af-topology
  network 10.4.6.0 0.0.0.255
  network 10.5.6.18 0.0.0.0
  network 10.7.0.0 0.0.0.255
  network 172.16.6.0 0.0.0.255
  network 172.17.7.0 0.0.0.255
  eigrp router-id 18.18.18.18
 exit-address-family
!
!
ip route 0.0.0.0 0.0.0.0 94.30.3.33
ip route 0.0.0.0 0.0.0.0 94.30.3.13
!
```

Проверим наличие маршрута по умолчанию, например, на R16:

![R16_default_gw](https://github.com/MIranaNightshade/otus-networks/blob/main/lab8_EIGRP/jpeg/R16_DEFAULT.png)

*На R16 настроим prefix-list чтобы анонсировать только default gw в сторону R32:*

```
!
router eigrp PETERBURG
 !
 address-family ipv4 unicast autonomous-system 100
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   summary-address 10.4.0.0 255.254.0.0
   summary-address 172.16.0.0 255.254.0.0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/2
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   no passive-interface
  exit-af-interface
  !
  topology base
   distribute-list prefix DEFAULT_ONLY out Ethernet0/3
  exit-af-topology
  network 10.4.6.0 0.0.0.255
  network 10.5.6.16 0.0.0.0
  network 10.7.0.0 0.0.0.255
  network 172.16.6.0 0.0.0.255
  network 172.17.7.0 0.0.0.255
  eigrp router-id 16.16.16.16
 exit-address-family
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
!
ip prefix-list DEFAULT_ONLY seq 5 permit 0.0.0.0/0
!
```

*Проверим маршруты полученные по EIGRP на R32:*

![Route_R32](https://github.com/MIranaNightshade/otus-networks/blob/main/lab8_EIGRP/jpeg/R32_ROUTE.png)

*Пропингуем с R32 лупбеки/интерфейсы управления/конечные хосты:*

![PING_R32](https://github.com/MIranaNightshade/otus-networks/blob/main/lab8_EIGRP/jpeg/R32_PING.png)

[ВСЕ КОНФИГИ МОЖНО ПОСМОТРЕТЬ ТУТ](https://github.com/MIranaNightshade/otus-networks/tree/main/lab8_EIGRP/ALL_CONFIG)





  
