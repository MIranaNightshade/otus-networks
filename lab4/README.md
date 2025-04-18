# Лабораторная работа №4 "Архитектура сети"

### **Задание:**
  1. Разработать и задокументировать адресное пространство для лабораторного стенда.
  2. Настроить ip адреса на каждом активном порту
  3. Настроить каждый VPC в каждом офисе в своем VLAN.
  4. Настроить VLAN/Loopbackup interface управления для сетевых устройств.
  5. Настроить сети офисов так, чтобы не возникало broadcast штормов, а использование линков было максимально оптимизировано.

### **Решение:**
  1. [Разработаем и задокументируем адресное пространство для лабораторного стенда.](#title1)
  2. [Настроим ip адреса на каждом активном порту.](#title2)
  3. [Настроим каждый VPC в каждом офисе в своем VLAN.](#title3)
  4. [Настроим VLAN/Loopbackup interface управления для сетевых устройств.](#title4)
  5. [Настроим сети офисов так, чтобы не возникало broadcast штормов, а использование линков было максимально оптимизировано.](#title5)

#### <a id=title1> 1. Разработаем и задокументируем адресное пространство для лабораторного стенда.</a>
  Схема сети:
  
  ![Схема сети](https://github.com/MIranaNightshade/otus-networks/blob/main/lab4/jpeg/%D1%81%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8.png) 
  
  
  Будем использовать следующие подсети: 
  Суммарные подсети для loopback итерфейсов ротуеров 10.5.0.0/21, для интерфейсов управления коммутаторами 10.4.0.0./21,  для линков между роутерами (кроме провайдеров Киторн, Ламас, Триада и линков к провайдерам) 10.7.0.0/21, 
  разделим эти подсети на /24 таким образом, чтобы третий октет соотвествовал разным локациям по следующему принципу: 0 - Москва, 1 - Киторн, 2 - Ламас, 3 - Триада, 4 - Лабытнанги, 5 - Чокурдах, 6 - С.Петербург. Для хостов используем подсеть 
  172.16.0.0/12, разобем ее на подсети /24 для хостов в Москве 172.16.0.0/24, 172.17.0.0/24; для хостов в С.Петербурге 172.16.6.0/24, 172.17.6.0/24; для хостов в Чокурдахе 172.16.5.0/24, 172.17.5.0/24. Последний октет для всех
  будет соответствовать порядковому номеру устройства на схеме сети.     
  
  Общая таблица сетей:
  | №| LOCATION | NETWORK MANAGEMENT/LOOPBACK  | ROUTER'S LINKS | HOSTS NETWORKS |
  | --|----------| -------------------------| ---------------| -------------|
  | **0** | Москва  | 10.5.**0**.0/24  | 10.7.**0**.0/24    |172.16.**0**.0/24, 172.17.**0**.0/24 |
  | **1** | Киторн | 10.5.**1**.0/24 |  176.30.**1**.0/24 |  NOT USED |
  | **2** | Ламас | 10.5.**2**.0/24 | 213.30.**2**.0/244 |  NOT USED |
   |**3** | Триада | 10.5.**3**.0/24 | 94.30.**3**.0/24 | NOT USED |
  | **4**  | Лабытнинги | 10.5.**4**.0/24 | 10.7.**4**.0/24 |  NOT USED |
   | **5** | Чокурдах | 10.5.**5**.0/24 | 10.7.**5**.0/24 | 172.16.**5**.0/24, 172.17.**5**.0/24 |  
  | **6** | С.Петербург | 10.5.**6**.0/24 | 10.7.**6**.0/24 |   172.16.**6**.0/24, 172.17.**6**.0/24 |

  Таблица выделенных адресов маршрутизаторов:
  
  | Device | Loopback address |  Port | Address | Network | Direction | 
  | ----- | -----------| ------| ----    |  ---    | --------- | 
  | R12 | 10.5.0.12/32 | e0/0 | 10.7.0.1/30| 10.7.0.0/30| to SW4 |
  | R12 | 10.5.0.12/32 | e0/1 | 10.7.0.5/30|10.7.0.4/30 | to SW5 |
  | R12 | 10.5.0.12/32 | e0/2 | 10.7.0.9/30| 10.7.0.8/30| to R14 |
  | R12 | 10.5.0.12/32 | e0/3 | 10.7.0.13/30| 10.7.0.12/30| to R15 |
  | R13 | 10.5.0.13/32 | e0/0 |10.7.0.17/30 | 10.7.0.16/30| to R15 |
  | R13 | 10.5.0.13/32 | e0/1 |10.7.0.21/30 |10.7.0.20/30 | to SW4 |
  | R13 | 10.5.0.13/32 | e0/2 | 10.7.0.25/30| 10.7.0.24/30 | to R15|
  | R13 | 10.5.0.13/32 | e0/3 | 10.7.0.29/30| 10.7.0.28/30 | to R14|
  | R14 | 10.5.0.14/32 |e0/0 | 10.7.0.10/30| 10.7.0.8/30| to R12 |
  | R14 | 10.5.0.14/32 | e0/1 | 10.7.0.30/30 | 10.7.0.28/30| to R13 | 
  | R14 | 10.5.0.14/32 | e0/2 | 176.30.1.2/30|176.30.1.0/30| to R22 |
  | R14 | 10.5.0.14/32 | e0/3 | 10.7.0.33/30 | 10.7.0.32/30 | to R19 |
  | R15 | 10.5.0.15/32 | e0/0 | 10.7.0.26/30 | 10.7.0.24/30| to R13 |  
  | R15 | 10.5.0.15/32 | e0/1| 10.7.0.14/30| 10.7.0.12/30| to R12 | 
  | R15 | 10.5.0.15/32 | e0/2 | 213.30.2.2/30| 213.30.2.0/30| to R21|
  | R15 | 10.5.0.15/32 | e0/3 | 10.7.0.37/30 | 10.7.0.36/30| to R20 |
  | R16 | 10.5.6.16/32 | e0/1 | 10.7.0.41/30 | 10.7.0.40/30 | to R18 | 
  | R16 | 10.5.6.16/32 | e0/2 | 10.7.0.45/30 | 10.7.0.44/30 | to R17 | 
  | R16 | 10.5.6.16/32 | e0/3 | 10.7.0.49/30 | 10.7.0.48/30 | to R32 |
  | R17 | 10.5.6.17/32 | e0/1 | 10.7.0.53/30 | 10.7.0.52/30 | to R18 |  
  | R17 | 10.5.6.17/32 | e0/2 | 10.7.0.46/30 | 10.7.0.44/30 | to R16 | 
  | R18 | 10.5.6.18/32 | e0/0 | 10.7.0.42/30 | 10.7.0.40/30 | to R16 |
  | R18 | 10.5.6.18/32 | e0/1 | 10.7.0.54/30 | 10.7.0.52/30 | to R17 |
  | R18 | 10.5.6.18/32 | e0/2 | 94.30.3.14/30| 94.30.3.12/30| to R24|
  | R18 | 10.5.6.18/32 | e0/3 | 94.30.3.34/30| 94.30.3.32/30| to R26 |
  | R19 | 10.5.0.19/32 | e0/0 | 10.7.0.34/30 | 10.7.0.32/30 | to R14 |
  | R20 | 10.5.0.20/32 | e0/0 | 10.7.0.38/30 |10.7.0.36/30  | to R15 | 
  | R21 | 10.5.2.21/32 | e0/0 | 213.30.2.1/30 | 213.30.2.0/30 | to R15 |
  | R21 | 10.5.2.21/32 | e0/1 | 213.30.2.5/30 | 213.30.2.4/30 | to R22  |
  | R21 | 10.5.2.21/32 | e0/2 | 213.30.2.9/30 | 213.30.2.8/30 | to R24 | 
  | R22 | 10.5.1.22/32 | e0/0 | 176.30.1.1/30 | 176.30.1.0/30 | to R14| 
  | R22 | 10.5.1.22/32 | e0/1 | 213.30.2.6/30 | 213.30.2.4/30 | to R21 |
  | R22 | 10.5.1.22/32 | e0/2 | 176.30.1.5/30 |176.30.1.4/30| to R23 |
  | R23 | 10.5.3.23/32 | e0/0 | 176.30.1.6/30 | 176.30.1.4/30| to R22  |
  | R23 | 10.5.3.23/32 | e0/1 | 94.30.3.1/30 | 94.30.3.0/30 | to R25 | 
  | R23 | 10.5.3.23/32 | e0/2 | 94.30.3.5/30 | 94.30.3.4/30 | to R24  |
  | R24 | 10.5.3.24/32 | e0/0 | 213.30.2.10/30 | 213.30.2.8/30 | to R21 | 
  | R24 | 10.5.3.24/32 | e0/1 | 94.30.3.9/30 | 94.30.3.8/30 | to R26 | 
  | R24 | 10.5.3.24/32 | e0/2 | 94.30.3.6/30 | 94.30.3.4/30 | to R23 |
  | R24 | 10.5.3.24/32 | e0/3 | 94.30.3.13/30 | 94.30.3.12/30 | to R18 |
  | R25 | 10.5.3.25/32 | e0/0 | 94.30.3.2/30 | 94.30.3.0/30 | to R23 | 
  | R25 | 10.5.3.25/32 | e0/1 | 94.30.3.17/30 | 94.30.3.16/30 | to R27 | 
  | R25 | 10.5.3.25/32 | e0/2 | 94.30.3.21/30 | 94.30.3.20/30 | to R26  |
  | R25 | 10.5.3.25/32 | e0/3 | 94.30.3.25/30 | 94.30.3.24/30 | to R28 | 
  | R26 | 10.5.3.26/32 | e0/0 | 94.30.3.10/30 | 94.30.3.8/30 | to R24 |  
  | R26 | 10.5.3.26/32 | e0/1 | 94.30.3.29/30 | 94.30.3.28/30 | to R28 | 
  | R26 | 10.5.3.26/32 | e0/2 | 94.30.3.22/30 | 94.30.3.20/30 | to R25  |
  | R26 | 10.5.3.26/32 | e0/3 | 94.30.3.33/30 | 94.30.3.32/30 | to R18 |
  | R27 | 10.5.4.27/32 | e0/0 |94.30.3.18/30 | 94.30.3.16/30 | to R25 | 
  | R28 | 10.5.5.28/32 | e0/0 | 94.30.3.30/30 | 94.30.3.28/30 | to R26 | 
  | R28 | 10.5.5.28/32 | e0/1 | 94.30.3.26/30 | 94.30.3.24/30 | to R25 |
  | R32 | 10.5.6.32/32 | e0/0 | 10.7.0.50/30 | 10.7.0.48/30 | to R16 | 
  | SW4 | -            | e1/0 | 10.7.0.2/30| 10.7.0.0/30| to R12|
  | SW4 | -            | e1/1 | 10.7.0.22/30 | 10.7.0.20/30| to R13 |
  | SW5 | -            | e1/0 | 10.7.0.18/30 | 10.7.0.16/30 | to R13 |
  | SW5 | -            | e1/1 | 10.7.0.6/30 | 10.7.0.4/30 | to R12 |

  Таблица адресов для управления коммутаторами:
  
 | device | management IP | management network | management vlan |
 |--------| ----------| ------------|-----|
 |SW2 | 10.4.0.2/24 | 10.4.0.0/24 | 100 |
 |SW3 | 10.4.0.3/24 | 10.4.0.0/24 | 100 | 
 |SW4 | 10.4.0.4/24 | 10.4.0.0/24 | 100 |
 |SW5 | 10.4.0.5/24 | 10.4.0.0/24 | 100 | 
 |SW9 | 10.4.6.9/24 | 10.4.6.0/24 | 100 |
 |SW10| 10.4.6.10/24 | 10.4.6.0/24 | 100 | 
 |SW29| 10.4.5.29/24 |10.4.5.0/24 | 100|
  
 #### <a id=title1>2. Настроим ip адреса на каждом активном порту.</a>

  Пример настройки ip на порту маршрутизатора (R15): 
  
  ```
  interface Ethernet0/0
   ip address 10.7.0.26 255.255.255.252
  end
  ```

  Пример настройки ip на порту L3 коммутатора (SW5): 

  ```
  interface Ethernet1/0
   no switchport
   ip address 10.7.0.18 255.255.255.252
   duplex auto
  end
  ```
#### <a id=title3>3. Настроим каждый VPC в каждом офисе в своем VLAN.</a> 

  Таблица IP адресов для VPC:
  
  | VPC | Location | IP address | network | gateway | vlan | 
  |------| ----- | ----| -- | ---- | --- |
  | VPC1 | Москва | 172.16.0.1/24 | 172.16.0.0/24 | 172.16.0.254 | 10 |
  | VPC7 | Москва |  172.17.0.7/24 |  172.17.0.0/24 | 172.17.0.254 | 20 |
  | VPC8 | C.-Петербург | 172.16.6.8/24 | 172.16.6.0/24 | 172.17.6.254| 10 |  
  | VPC33| C.-Петербург | 172.17.6.33/24 | 172.17.6.0/24 | 172.17.6.254| 20 |
  | VPC30| Чокурдах | 172.16.5.30/24 | 172.16.5.0/24 | 172.16.5.254| 10 |
  |VPC31 | Чокурдах | 172.17.5.31/24 | 172.17.5.0/24| 172.17.5.254| 20 |

#### <a id=title4>4. Настроим VLAN/Loopbackup interface управления для сетевых устройств</a>

  Таблица loopback интерфейсов роутеров:

  | Device |	Loopback address|
  |------ | ------|
  |R12	 |10.5.0.12/32 |
   |R13 |	10.5.0.13/32 |
   |R14	 |10.5.0.14/32 |
   |R15	 |10.5.0.15/32 |
   |R16	 |10.5.6.16/32 |
   |R17	 |10.5.6.17/32 |
   |R18  | 10.5.6.18/32 |
   |R19	 |10.5.0.19/32 |
   |R20 |	10.5.0.20/32 |
   |R21	 |10.5.2.21/32 |
   |R22	 |10.5.1.22/32 |
   |R23	 |10.5.3.23/32 |
   |R24	 |10.5.3.24/32 |
   |R25	 |10.5.3.25/32 |
   |R26 |10.5.3.26/32 |
   |R27	 |10.5.4.27/32 |
   |R28	 |10.5.5.28/32 |
   |R32	 |10.5.6.32/32 |
  
   
Пример настройки Loopback (R15):
   
 ```   
  interface Loopback0
    ip address 10.5.0.15 255.255.255.0
  end
```        
      
Таблица vlan/IP адресов управления для коммутаторов: 
    
| device | IP address | VLAN | gateway | 
| ------|---------| ------ | ----- |
| SW2 | 10.4.0.2/24 | 100 | 10.4.0.254 | 
|SW3 | 10.4.0.3/24 | 100 | 10.4.0.254 | 
|SW4 | 10.4.0.4/24 | 100 | N/A |
|SW5 |  10.4.0.5/24 | 100 | N/A |  
| SW9 | 10.4.6.9/24 | 100 | 10.4.6.254 | 
|SW10 | 10.4.6.10/24 | 100 | 10.4.6.254 | 
|SW29 |  10.4.5.29/24 | 100 | 10.4.5.254 | 

  
Пример настройки management vlan на коммутаторах (SW5):

  ```        
  interface Vlan100
    ip address 10.4.0.5 255.255.255.0
  end
  ```
#### <a id=title5> 5. Настроим сети офисов так, чтобы не возникало broadcast штормов, а использование линков было максимально оптимизировано.</a> 

**Москва:**

Таблица vlan:

|vlan id | name | 
|----| ---|
| 10 | HOST1 |
| 20 | HOST2 |
| 100 | MANAGEMENT |

|device | IP | default gateway| vlan| 
| ----- | --- | ----| --- |
| VPC1 | 172.16.0.1/24 | 172.16.0.254(SW4,SW5) | 10 |
| VPC7 |172.17.0.7/24 | 172.17.0.254 (SW4,SW5)| 20 |
| SW2 | 10.4.0.2/24 | 10.4.0.254 (SW4,SW5) | 100 |
| SW3 | 10.4.6.3/24 | 10.4.0.254 (SW4,SW5) | 100 |
| SW4 | 10.4.0.4/24 | N/A | 100 |
| SW5 | 10.4.0.5/24 | N/A| 100 |

Для коммутатором SW2 SW3 SW4 SW5 настроим MSTP Следующим образом:

Instance0 (vlan 10) SW4 root, красным показаны активные линки: 

![instance0](https://github.com/MIranaNightshade/otus-networks/blob/main/lab4/jpeg/instance0.png)


Instance1 (vlan 20, 100) SW5 root, красным показаны активные линки:

![instance1](https://github.com/MIranaNightshade/otus-networks/blob/main/lab4/jpeg/instance1.png)


[Конфиги в Москве можно посмотреть здесь](https://github.com/MIranaNightshade/otus-networks/tree/main/lab4/configs).

Между SW4 и SW5 настроим агрегацию (e0/3,e0/2 = e0/3,e0/2), с помощью протокола LACP:

пример настройки на SW5, на SW4 зеркальные настройки, выберем mode active с двух сторон:

```
!
interface Ethernet0/2
 switchport trunk allowed vlan 10,20,100
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
end

SW5#show running-config interface e0/3
Building configuration...

Current configuration : 160 bytes
!
interface Ethernet0/3
 switchport trunk allowed vlan 10,20,100
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
end

SW5#show running-config interface po1
Building configuration...

Current configuration : 155 bytes
!
interface Port-channel1
 switchport trunk allowed vlan 10,20,100
 switchport trunk encapsulation dot1q
 switchport mode trunk
 lacp fast-switchover
end
```

Проверим работу LACP: 

```
SW5#show etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator

        M - not in use, minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port


Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         LACP      Et0/2(P)    Et0/3(P)

SW5#
```

Настроим VRRP между SW4 и SW5, таким образом чтобы во vlan 10 (VPC1) master был SW4, для vlan 20 (VPC7), vlan 100 (MANAGEMENT) master SW5:

```
SW5#show vrrp
Vlan10 - Group 2
  State is Backup
  Virtual IP address is 172.16.0.254
  Virtual MAC address is 0000.5e00.0102
  Advertisement interval is 1.000 sec
  Preemption enabled
  Priority is 100
  Master Router is 172.16.0.254, priority is 255
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.609 sec (expires in 3.074 sec)

Vlan20 - Group 3
  State is Master
  Virtual IP address is 172.17.0.254
  Virtual MAC address is 0000.5e00.0103
  Advertisement interval is 1.000 sec
  Preemption enabled
  Priority is 255
  Master Router is 172.17.0.254 (local), priority is 255
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.003 sec

Vlan100 - Group 1
  State is Master
  Virtual IP address is 10.4.0.254
  Virtual MAC address is 0000.5e00.0101
  Advertisement interval is 1.000 sec
  Preemption enabled
  Priority is 254
  Master Router is 10.4.0.5 (local), priority is 254
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.007 sec

SW5#
```
[Конфиги в Москве можно посмотреть здесь](https://github.com/MIranaNightshade/otus-networks/tree/main/lab4/configs).

**C.-Петербург:**

Таблица vlan:

|vlan id | name | 
|----| ---|
| 10 | HOST1 |
| 20 | HOST2 |
| 100 | MANAGEMENT |

|device | IP | default gateway| vlan| 
| ----- | --- | ----| --- |
| VPC8 | 172.16.6.8/24 | 172.16.6.254(R16,R17) | 10 |
| VPC33 |172.17.6.33/24 | 172.17.6.254 (R16,R17)| 20 |
| SW9 | 10.4.6.9/24 | 10.4.6.254 (R16,R17) | 100 |
| SW10 | 10.4.6.10/24 | 10.4.6.254 (R16,R17) | 100 |

Настроим VRRP между R17 и R16 так чтобы для vlan 10 (HOST1) master был R17, а для vlan 20, 100 master R16:
Проверим статус VRRP на R16:

```
R16#         show vrrp
Ethernet0/0.10 - Group 1
  State is Backup
  Virtual IP address is 172.16.6.254
  Virtual MAC address is 0000.5e00.0101
  Advertisement interval is 1.000 sec
  Preemption enabled
  Priority is 100
  Master Router is 172.16.6.254, priority is 255
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.609 sec (expires in 3.325 sec)

Ethernet0/0.20 - Group 2
  State is Master
  Virtual IP address is 172.17.6.254
  Virtual MAC address is 0000.5e00.0102
  Advertisement interval is 1.000 sec
  Preemption enabled
  Priority is 255
  Master Router is 172.17.6.254 (local), priority is 255
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.003 sec

Ethernet0/0.100 - Group 3
  State is Master
  Virtual IP address is 10.4.6.254
  Virtual MAC address is 0000.5e00.0103
  Advertisement interval is 1.000 sec
  Preemption enabled
  Priority is 255
  Master Router is 10.4.6.254 (local), priority is 255
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.003 sec

R16#
```

Настроим LACP между SW9 SW10, потому же принципу как в Москве:

```
!
interface Port-channel1
 switchport trunk allowed vlan 10,20,100
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/0
 switchport trunk allowed vlan 10,20,100
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/1
 switchport trunk allowed vlan 10,20,100
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
!

```

[Конфиги в С.-Петербурге можно посмотреть здесь](https://github.com/MIranaNightshade/otus-networks/tree/main/lab4/configs/S.Peterburg)


**Чокурдах**

|vlan id | name | 
|----| ---|
| 10 | HOST1 |
| 20 | HOST2 |
| 100 | MANAGEMENT |

|device | IP | default gateway| vlan| 
| ----- | --- | ----| --- |
| VPC30 | 172.16.5.8/24 | 172.16.5.254(R28) | 10 |
| VPC31 |172.17.5.33/24 | 172.17.5.254 (R28)| 20 |
| SW29 | 10.4.5.9/24 | 10.4.5.254 (R28) | 100 |

Настроим вланы 10, 20 на коммутаторе SW29 access в строну VPC, trunk в сторону R28.
Настроим Router on a stick на R28:

```
!
interface Ethernet0/2.10
 encapsulation dot1Q 10
 ip address 172.16.5.254 255.255.255.0
!
interface Ethernet0/2.20
 encapsulation dot1Q 20
 ip address 172.17.5.254 255.255.255.0
!
interface Ethernet0/2.100
 encapsulation dot1Q 100
 ip address 10.4.5.254 255.255.255.0
!
```
[Все конфиги в Чокурдахе здесь](https://github.com/MIranaNightshade/otus-networks/tree/main/lab4/configs/Chokurdah)



  

  

  


  
