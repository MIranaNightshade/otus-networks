# Лабораторная работа №13 DMVPN

## Схема сети:

![map](https://github.com/MIranaNightshade/otus-networks/blob/main/lab13_DMVPN/jpeg/DMVPN_map.png)

### Задачи:

[1. Настроить GRE между офисами Москва и С.-Петербург.](#1)
 
[2. Настроить DMVMN между Москва и Чокурдах, Лабытнанги.](#2)
   
[3. Все узлы в офисах в лабораторной работе должны иметь IP связность.](#3)

#### <a id=1>1.Настроим GRE между офисами Москва и С.-Петербург.</a>
***Построим до R18 (С.-Петербург), два GRE туннеля от R14 и R15 (Москва) для обеспечения отказоустойчивости***

![GRE](https://github.com/MIranaNightshade/otus-networks/blob/main/lab13_DMVPN/jpeg/GRE_map.png)

Настройки на R14:

```
interface Tunnel0
 ip address 10.10.1.14 255.255.255.0
 tunnel source 100.30.0.214
 tunnel destination 100.30.6.18
```
Настройки на R15:

```
interface Tunnel0
 ip address 10.10.2.15 255.255.255.0
 tunnel source 100.30.0.215
 tunnel destination 100.30.6.18
```

Настройки на R18:

```
!
interface Tunnel0
 ip address 10.10.1.18 255.255.255.0
 tunnel source 100.30.6.18
 tunnel destination 100.30.0.214
end
!
interface Tunnel1
 ip address 10.10.2.18 255.255.255.0
 tunnel source 100.30.6.18
 tunnel destination 100.30.0.215
end

```

Проверим работу туннелей:

![GRE](https://github.com/MIranaNightshade/otus-networks/blob/main/lab13_DMVPN/jpeg/GRE.png)

#### <a id=2>2. Настроим DMVMN между Москва и Чокурдах, Лабытнанги.</a>

Настроим два облака DMVPN для обеспечения отказоустойчивости.
R14, R15 настраиваем как hub.
R27, R28 как spoke.

![DMVPN](https://github.com/MIranaNightshade/otus-networks/blob/main/lab13_DMVPN/jpeg/DMVPN.png)

Настройки на R14:

```
!
interface Tunnel1
 ip address 10.10.3.14 255.255.255.0
 no ip redirects
 ip nhrp map multicast dynamic
 ip nhrp network-id 1414
 tunnel source 100.30.0.114
 tunnel mode gre multipoint
 tunnel key 1414
```

Настройки на R15:

```
!
interface Tunnel1
 ip address 10.10.4.15 255.255.255.0
 no ip redirects
 ip nhrp map multicast dynamic
 ip nhrp network-id 1515
 tunnel source 100.30.0.115
 tunnel mode gre multipoint
 tunnel key 1515
end

```
Настройки на R27:

```
!
interface Tunnel0
 ip address 10.10.3.27 255.255.255.0
 no ip redirects
 ip nhrp map 10.10.3.14 100.30.0.114
 ip nhrp map multicast 100.30.0.114
 ip nhrp network-id 1414
 ip nhrp nhs 10.10.3.14
 ip nhrp registration no-unique
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 1414
end
!
interface Tunnel1
 ip address 10.10.4.27 255.255.255.0
 no ip redirects
 ip nhrp map 10.10.4.15 100.30.0.115
 ip nhrp map multicast 100.30.0.115
 ip nhrp network-id 1515
 ip nhrp nhs 10.10.4.15
 ip nhrp registration no-unique
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 1515
end
```

Настройки на R28:

```
!
interface Tunnel0
 ip address 10.10.3.28 255.255.255.0
 no ip redirects
 ip nhrp map 10.10.3.14 100.30.0.114
 ip nhrp map multicast 100.30.0.114
 ip nhrp network-id 1414
 ip nhrp nhs 10.10.3.14
 ip nhrp registration no-unique
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 1414
end
!
interface Tunnel1
 ip address 10.10.4.28 255.255.255.0
 no ip redirects
 ip nhrp map 10.10.4.15 100.30.0.115
 ip nhrp map multicast 100.30.0.115
 ip nhrp network-id 1515
 ip nhrp nhs 10.10.4.15
 ip nhrp registration no-unique
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
 tunnel key 1515
```

Проверим работу туннелей:

![DMVPN_R14](https://github.com/MIranaNightshade/otus-networks/blob/main/lab13_DMVPN/jpeg/DMVPN_R14.png)

![DMVPN_R15](https://github.com/MIranaNightshade/otus-networks/blob/main/lab13_DMVPN/jpeg/DMVPN_R15.png)

#### <a id=2>3. Настроим IP связность между офисами через туннели с помощью EIGRP.</a>

***EIGRP через GRE будет в AS100, через DMVPN в AS200, при редистрибьюции OSPF отфильтруем линковочные подсети 10.7.0.0/24 с помощью route-map***

![EIGRP](https://github.com/MIranaNightshade/otus-networks/blob/main/lab13_DMVPN/jpeg/EIGRP_map.png)

**Настройки EIGRP на R14:**

```
router eigrp ALL
 !
 address-family ipv4 unicast autonomous-system 100
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Tunnel0
   summary-address 10.4.0.0 255.255.255.0
   summary-address 10.5.0.0 255.255.255.0
   summary-address 172.16.0.0 255.255.255.0
   summary-address 172.17.0.0 255.255.255.0
   no passive-interface
  exit-af-interface
  !
  topology base
   redistribute ospf 1 metric 100000 10 255 1 1500 route-map FOR_EIGRP_ALL
   redistribute eigrp 200
  exit-af-topology
  network 10.4.0.0 0.0.0.255
  network 10.5.0.0 0.0.0.255
  network 10.10.1.0 0.0.0.255
  network 172.16.0.0 0.0.0.255
  network 172.17.0.0 0.0.0.255
 exit-address-family
router eigrp ALL1
 !
 address-family ipv4 unicast autonomous-system 200
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Tunnel1
   summary-address 10.4.0.0 255.255.255.0
   summary-address 10.5.0.0 255.255.255.0
   summary-address 172.16.0.0 255.255.255.0
   summary-address 172.17.0.0 255.255.255.0
   no next-hop-self
   no passive-interface
   no split-horizon
  exit-af-interface
  !
  topology base
   redistribute eigrp 100 route-map NEXTHOP_SELF
   redistribute ospf 1 metric 100000 10 255 1 1500 route-map FOR_EIGRP_ALL
  exit-af-topology
  network 10.4.0.0 0.0.0.255
  network 10.5.0.0 0.0.0.255
  network 10.10.3.0 0.0.0.255
  network 172.16.0.0 0.0.0.255
  network 172.17.0.0 0.0.0.255
 exit-address-family
!
ip prefix-list FOR_EIGRP_ALL seq 5 deny 10.7.0.0/24 le 32
ip prefix-list FOR_EIGRP_ALL seq 10 permit 0.0.0.0/0 le 32
!
route-map NEXTHOP_SELF permit 10
 set ip next-hop self
!
route-map FOR_EIGRP_ALL permit 10
 match ip address prefix-list FOR_EIGRP_ALL
!
```

**Настройки EIGRP на R15:**

```
router eigrp ALL
 !
 address-family ipv4 unicast autonomous-system 100
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Tunnel0
   summary-address 10.4.0.0 255.255.255.0
   summary-address 10.5.0.0 255.255.255.0
   summary-address 172.16.0.0 255.255.255.0
   summary-address 172.17.0.0 255.255.255.0
   no passive-interface
  exit-af-interface
  !
  topology base
   redistribute ospf 1 metric 100000 10 255 1 1500 route-map FOR_EIGRP_ALL
   redistribute eigrp 200
  exit-af-topology
  network 10.4.0.0 0.0.0.255
  network 10.5.0.0 0.0.0.255
  network 10.10.2.0 0.0.0.255
  network 172.16.0.0 0.0.0.255
  network 172.17.0.0 0.0.0.255
 exit-address-family
router eigrp ALL1
 !
 address-family ipv4 unicast autonomous-system 200
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Tunnel1
   summary-address 10.4.0.0 255.255.255.0
   summary-address 10.5.0.0 255.255.255.0
   summary-address 172.16.0.0 255.255.255.0
   summary-address 172.17.0.0 255.255.255.0
   no next-hop-self
   no passive-interface
   no split-horizon
  exit-af-interface
  !
  topology base
   redistribute eigrp 100 route-map NEXTHOP_SELF
   redistribute ospf 1 metric 100000 10 255 1 1500 route-map FOR_EIGRP_ALL
  exit-af-topology
  network 10.4.0.0 0.0.0.255
  network 10.5.0.0 0.0.0.255
  network 10.10.4.0 0.0.0.255
  network 172.16.0.0 0.0.0.255
  network 172.17.0.0 0.0.0.255
 exit-address-family
!
ip prefix-list FOR_EIGRP_ALL seq 5 deny 10.7.0.0/24 le 32
ip prefix-list FOR_EIGRP_ALL seq 10 permit 0.0.0.0/0 le 32
!
route-map NEXTHOP_SELF permit 10
 set ip next-hop self
!
route-map FOR_EIGRP_ALL permit 10
 match ip address prefix-list FOR_EIGRP_ALL
!
```

**Настройки EIGRP на R18:**

'''
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
  af-interface Tunnel0
   no passive-interface
  exit-af-interface
  !
  af-interface Tunnel1
   no passive-interface
  exit-af-interface
  !
  topology base
   distribute-list prefix TO_R14_R15 out Tunnel0
   distribute-list prefix TO_R14_R15 out Tunnel1
   redistribute static
  exit-af-topology
  network 10.4.6.0 0.0.0.255
  network 10.5.6.18 0.0.0.0
  network 10.7.0.0 0.0.0.255
  network 10.10.1.0 0.0.0.255
  network 10.10.2.0 0.0.0.255
  network 172.16.6.0 0.0.0.255
  network 172.17.6.0 0.0.0.255
  eigrp router-id 18.18.18.18
 exit-address-family
R18#
``

**Настройки EIGRP на R27:**

```
R27#show  running-config | s r e
router eigrp ALL1
 !
 address-family ipv4 unicast autonomous-system 200
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Tunnel0
   no passive-interface
  exit-af-interface
  !
  af-interface Tunnel1
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 10.5.4.0 0.0.0.255
  network 10.10.3.0 0.0.0.255
  network 10.10.4.0 0.0.0.255
 exit-address-family
R27#
```

**Настройки EIGRP на R28:**

```
router eigrp ALL1
 !
 address-family ipv4 unicast autonomous-system 200
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Tunnel0
   summary-address 10.4.5.0 255.255.255.0
   summary-address 10.5.5.0 255.255.255.0
   summary-address 172.16.5.0 255.255.255.0
   summary-address 172.17.5.0 255.255.255.0
   no passive-interface
  exit-af-interface
  !
  af-interface Tunnel1
   summary-address 10.4.5.0 255.255.255.0
   summary-address 10.5.5.0 255.255.255.0
   summary-address 172.16.5.0 255.255.255.0
   summary-address 172.17.5.0 255.255.255.0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 10.4.5.0 0.0.0.255
  network 10.5.5.0 0.0.0.255
  network 10.10.3.0 0.0.0.255
  network 10.10.4.0 0.0.0.255
  network 172.16.5.0 0.0.0.255
  network 172.17.5.0 0.0.0.255
 exit-address-family
```



