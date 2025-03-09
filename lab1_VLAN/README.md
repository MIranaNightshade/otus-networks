# VLAN и маршрутизация между VLAN
### Задание: 
 1. Создать сеть и задать основные настройки устройствам.
 2. Настроить VLAN и назначить их портам коммутатора согласно топологии.
 3. Настроить маршрутизацию между VLAN на роутере.
 4. Проверить работу маршрутизаци между VLAN.
### Решение:
 1. [Создадим топологию "router on a stick" в eve-ng и выполним начальную настройку устройств.](#title1)
 2. [Создадим VLAN и назначим их портам коммутаторов/маршрутизатора согласно топологии.](#title2)
 3. [Настроим маршрутизацию между VLAN на роутере.](#title3)
 4. [Проверим работу маршрутизации между VLAN.](#title4)
#### <a id="title1">Создадим топологию "router on a stick" в eve-ng и выполним начальную настройку устройств.</a>

- Таблица адресов и связанных с ними интерфейсов:
----------------------------------------------------------------------------------   
| device | interface | ip address | subnet masc | default gateway | 
| ----   | --------  | --------   | --------    | -------------   |
 | R-1  | G0/1.3     |  192.168.3.1 | 255.255.255.0| N/A |
 |       | G0/1.4 | 192.168.4.1 | 255.255.255.0 | N/A |
  |       | G0/1.8|   N/A |  N/A|  N/A|  N/A|
  |S-1| VLAN 3| 192.168.3.11 | 255.255.255.0 | 192.168.3.1 |
  |S-2 | VLAN 3| 192.168.3.12 | 255.255.255.0 | 192.168.3.1 |
  |PC-A | NIC | 192.168.3.3 | 255.255.255.0 | 192.168.3.1 | 
  |PC-B | NIC |192.168.4.3 | 255.255.255.0 | 192.168.4.1 |
  
  - Топология сети выполненная в eve-ng:

![топология сети](https://github.com/MIranaNightshade/otus-networks/blob/main/lab1_VLAN/jpeg/%D1%82%D0%BE%D0%BF%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%8F%20%D1%81%D0%B5%D1%82%D0%B8.png)  

- Начальная настройка устройств:
  -  Назначить имя устройству.
  -  Отключить поиск DNS.
  -  Назначить cisco в качестве зашифрованного пароля привилегированного EXEC.
  -  Назначить cisco в качестве пароля консоли и включить вход.
  -  Назначьте cisco в качестве пароля VTY и включить вход.
  -  Зашифровать пароли в виде открытого текста.
  -  Создать баннер на вход.
  -  Установить часы на устройстве.
     - команда (clock set 20:04:00 09 march 2025)

###### Начальная конфигурация на примере коммутатора S-1:
```
!
hostname S-1
!
!
no ip domain-lookup
!
enable secret 5 $1$2Ql1$Z/WYRXu0AUqz/UiZvSGyD1
!
banner login ^C THIS IS S-1 ^C
!
line con 0
 password 7 05080F1C2243
 login
line aux 0
line vty 0 4
 password 7 1511021F0725
 login
line vty 5 15
 password 7 1511021F0725
 login
!
!
end
```
           
#### <a id="title2">Настроим VLAN и назначим их портам коммутаторов согласно топологии.</a>
- Таблица VLAN:
  
  |VLAN ID | НАЗНАЧЕНИЕ VLAN |
  | --- | ---|
  |VLAN 3 | MANAGEMANT|
  |VLAN 4 | OPERATIONS|
  |VLAN 7| PARKINGLOT |
  |VLAN 8 | NATIVE |
  
- Настроим VLAN согласно таблице выше 
  - Конфигурация на примере коммутатора S-1
    *(Конфигурация всех незадействованных портов аналогична конфигурации GigabitEthernet0/2)*:
 
 ```
interface GigabitEthernet0/0
 switchport access vlan 3
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/1
 switchport trunk allowed vlan 3,4
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/2
 switchport access vlan 7
 switchport mode access
 shutdown
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/3
 switchport trunk allowed vlan 3,4
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
 media-type rj45
 negotiation auto
!
interface Vlan3
 ip address 192.168.3.11 255.255.255.0
!
ip default-gateway 192.168.3.1
```
    
#### <a id="title3">Настроим маршрутизацию между VLAN на роутере.</a>
- Конфигурация на роутере:
```
!
interface GigabitEthernet0/1.3
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0
!
interface GigabitEthernet0/1.4
 encapsulation dot1Q 4
 ip address 192.168.4.1 255.255.255.0
```
  
#### <a id="title4">Проверим работу маршрутизации между VLAN.</a>
- Из командной строки PC-B выполним ping и tracroute на IP адрес PC-A.
  
