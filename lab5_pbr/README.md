# Лабораторная работа №5 PBR (policy based routing).
## Цели:
1. Настроить политику маршрутизации в офисе Чокурдах
2. Распределить трафик между 2 линками

## Задачи: 
1. Настроить политику маршрутизации для сетей офиса.
2. Распределить трафик между двумя линками с провайдером.
3. Настроить отслеживание линка через технологию IP SLA.(только для IPv4)
4. Настройть для офиса Лабытнанги маршрут по-умолчанию.

### Выполнение:

**СХЕМА СЕТИ:**

![схема сети](https://github.com/MIranaNightshade/otus-networks/blob/main/lab5_pbr/jpeg/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0_%D1%81%D0%B5%D1%82%D0%B8.png)

**Реализация:**

***Настроим сеть таким образом чтобы траффик от VPC30 шел через R28 e0/0, а от VPC31 через R28 e0/1, при недоступности одного из интерфейсов весь траффик будет направляться на доступный.*** 

Таблица IP адресов для офиса Чокурдах и провайдера (R25, R26): 

| device |  port/interface   | ip address | summary network | vlan | 
|--- | ----| ---| ----| ----|
|VPC30 | - |   172.16.5.30|  172.16.5.0/24  |   10   | 
|VPC31 | - |   172.17.5.31|  172.17.5.0/24  |   20   |
| R28 | e0/2.10 | 172.16.5.254 | 172.16.5.0/24  | 10 |
| R28  | e0/2.20 | 172.17.5.254 | 172.17.5.0/24 | 20 |
| R28 | e0/0 | 94.30.3.30 | 94.30.3.28/30 | - |
| R28 | e0/1  | 94.30.3.26 | 94.30.3.24/30 | - |
| R25  | e0/3 | 94.30.3.25 | 94.30.3.24/30 | - | 
| R26 | e0/1| 94.30.3.29 | 94.30.3.28/30 | - |

**Созадим ip sla для отслеживания доступности next-hop 94.30.3.29 (R26 e0/1) и 94.30.3.25 (R25 e0/3):**

```
!
ip sla 1
 icmp-echo 94.30.3.29
 threshold 300
 timeout 2000
 frequency 3
ip sla schedule 1 life forever start-time now
ip sla 2
 icmp-echo 94.30.3.25
 threshold 300
 timeout 2000
 frequency 3
ip sla schedule 2 life forever start-time now
!
```
**Создадим track для отслеживания состояния ip sla и линков на интерфейсах R28 e0/0 и e0/1:**
*установим так-же delay для предупреждения частого срабатывания 10 для перехода в состояние down (мы отправляем ping с частотой 1 раз в 3 сек, итого где-то 3 потерянных пинга)*

```
!
track 1 ip sla 1 reachability
 delay down 10 up 30
!
track 2 ip sla 2 reachability
 delay down 10 up 30
!
track 10 interface Ethernet0/0 line-protocol
 delay down 10 up 30
!
track 20 interface Ethernet0/1 line-protocol
 delay down 10 up 30
!
```

**Создадим track list с логикой and, который будет переходить в состояние down если хотя-бы один из track'ов в списке перейдет в состояние down:**

```
!
track 100 list boolean and
 object 1
 object 10
!
track 200 list boolean and
 object 2
 object 20
!
```
**Создадим access-list для VPC30 и VPC31:**
*Разрешим в них только те подсети, к которым принадлежат VPC30 (access-list 1) и VPC31 (access-list 2) соответственно*

```
access-list 1 permit 172.16.5.0 0.0.0.255
access-list 2 permit 172.17.5.0 0.0.0.255
```
**Создадим route-map для VPC30 и VPC31 c отслеживанием состояния интерфейсов**:
*для VPC30 приоритетный маршрут на R26 94.30.3.29 при его доступности и для VPC31 R25 94.30.3.25*

```
!
route-map FOR_VPC30 permit 10
 match ip address 1
 set ip next-hop verify-availability 94.30.3.29 20 track 100
 set ip next-hop verify-availability 94.30.3.25 30 track 200
!
route-map FOR_VPC31 permit 10
 match ip address 2
 set ip next-hop verify-availability 94.30.3.25 20 track 200
 set ip next-hop verify-availability 94.30.3.29 30 track 100
!
!
access-list 1 permit 172.16.5.0 0.0.0.255
access-list 2 permit 172.17.5.0 0.0.0.255
!
```

**Применим route-map на интерфейсы e0/2.10 и e0/2.20 R28:**

```
!
interface Ethernet0/2.10
 encapsulation dot1Q 10
 ip address 172.16.5.254 255.255.255.0
 ip policy route-map FOR_VPC30
!
interface Ethernet0/2.20
 encapsulation dot1Q 20
 ip address 172.17.5.254 255.255.255.0
 ip policy route-map FOR_VPC31
!
```

**Проверим работу pbr выполнив trace с VPC30 до Lo0 R26 (10.5.3.26) и с VPC31 до lo0 R25 (10.5.3.25):**
*для этого временно настрим default route для R26 и R25 в сторону R28*
*Ожидаемый 2-й хоп для VPC30 -  94.30.3.29, для VPC31 94.30.3.25*  

Таблица адресов R26 и R25 для наглядности:

| device | interface | address |
|---| ---| --- |
|R25 |  e0/3 | 94.30.3.25 |
|R25 | lo0 | 10.5.3.25 | 
|R26 |  e0/1| 94.30.3.29 | 94.30.3.28/30 |
|R26 | lo0 | 10.5.3.26 | 


![trace 1](https://github.com/MIranaNightshade/otus-networks/blob/main/lab5_pbr/jpeg/trace_VPC30.png)


![trace 2](https://github.com/MIranaNightshade/otus-networks/blob/main/lab5_pbr/jpeg/trace_VPC31.png)


[ВЕСЬ КОНФИГ R28 ЗДЕСЬ](https://github.com/MIranaNightshade/otus-networks/blob/main/lab5_pbr/config/R28)


**Настроим маршрут по умолчанию для офиса Лабытнанги:**

*т.к R27 подключен к R25 то маршрут по умолчанию будет в сторону R25*

| device | port | ip address|
|----| ----| ---- |
| R25 | e0/1 | 94.30.3.17/30 |
|R 27| e0/0 | 94.30.3.18/30 |

Настройки default-gateway на R27:

```
ip route 0.0.0.0 0.0.0.0 94.30.3.17
!
```








