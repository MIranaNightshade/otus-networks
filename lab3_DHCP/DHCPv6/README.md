# Лабораторная работа №3 DHCP (DHCPv6)
- Топология сети выполненная в EVE-NG:
  ![топология](https://github.com/MIranaNightshade/otus-networks/blob/main/lab3_DHCP/DHCPv6/jpeg/topology.png)

- Таблица адресов IPv6:
  
  <table>
<thead>
<tr>
<th>Device</th>
<th>Interface</th>
<th>IPv6 address</th>
</tr>
</thead>
<tbody>
<tr>
<td rowspan=4>R1</td>
<td rowspan=2>G0/0</td>
<td>2001:db8:acad:2::1/64</td>
</tr>
<tr>
<td>fe80::1</td>
</tr>
<tr>
<td rowspan=2>G0/1</td>
<td>2001:db8:acad:1::1/64</td>
</tr>
<tr>
<td>fe80::1</td>
</tr>
<tr>
<td rowspan=4>R2</td>
<td rowspan=2>G0/0</td>
<td>2001:db8:acad:2::2/64</td>
</tr>
<tr>
<td>fe80::2</td>
</tr>
<tr>
<td rowspan=2>G0/1</td>
<td>2001:db8:acad:3::1/64</td>
</tr>
<tr>
<td>fe80::1</td>
</tr>
<tr>
<td>PC1</td>
<td>NIC</td>
<td>DHCP</td>
</tr>
<tr>
<td>PC2</td>
<td>NIC</td>
<td>DHCP</td>
</tr>
</tbody>
</table>

- Цели:
  
  1. Создать сетевую топологию и задать устройствам базовые настройки.
  2. Проверить получение адресов с помощью SLAAC от R1.
  3. Настроить и проверить работу stateless DHCPv6 server на R1.
  4. Настроить и проверить работу stateful DHCPv6 Server на R1.
  5. Настроить и проверить работу DHCPv6 Relay на R2.

## 1. Создадим сетевую тьопологию и введем базовые настройки на устройствах.

- **Базовые настройки для коммутаторов:**
  
  1. Назначить hostname устройству.
  2. Отключить dns lookup чтобы коммутатор не пытался преобразовать неправильно введенные команды в имена хостов.
  3. Hазначить зашифрованный пароль cisco для входа в привелегированный EXEC режим.
  4. Назначить пароль cisco для входа в консоль.
  5. Назначить пароль cisco для входа в VTY. 
  6. Включить шифрование паролей. 
  7. Создать баннер предупреждающий о том что несанкционированный доступ к устройству запреещен.  
  8. Отключить все неиспользуемые порты.  
  9. Сохранить конфигурацию.

- **Базовые настройки для маршрутизатора:**
  
  1. Назначить hostname устройству.
  3. Отключить dns lookup чтобы коммутатор не пытался преобразовать неправильно введенные команды в имена хостов.
  4. Hазначить зашифрованный пароль cisco для входа в привелегированный EXEC режим.
  5. Назначить пароль cisco для входа в консоль.
  6. Назначить пароль cisco для входа в VTY.
  7. Включить шифрование паролей.
  8. Создать баннер предупреждающий о том что несанкционированный доступ к устройству запреещен.
  9. Включить маршрутизацию IPv6.
  10. Сохранить конфигурацию.

 - **Базовая настройка на примере маршрутизатора R1:**     

    ```  
    service password-encryption
    !
    hostname R1
    !
    enable secret 5 $1$zdki$phCu/QuYvLjI7kmOGkWeF0
    !
    no ip domain lookup
    ip cef
    ipv6 unicast-routing
    ipv6 cef
    !
    banner motd ^C
    *******************************
    *******************************
    * HANDS OFF FROM MY LITTLE R1 *
    *  AND GET OUT OF HERE        *
    *******************************
    *******************************
    ^C
    !
    line con 0
     password 7 13061E01080344
     login
    line aux 0
    !
    line vty 5 15
     password 7 060506324F41
     login
     transport input none
    !
    ```
- **Настроим интерфейсы и маршрутизацию на обоих роутерах.**
   - Настроим IP адреса из таблицы выше на интерфейсах G0/0 и G0/1 на R1 и R2.
     
     *Пример настройки IP адресов на R1 G0/0:*
        ```
            Current configuration : 175 bytes
            !
            interface GigabitEthernet0/0
             no ip address
             duplex auto
             speed auto
             media-type rj45
             ipv6 address FE80::1 link-local
             ipv6 address 2001:DB8:ACAD:2::1/64
             ipv6 enable
            end
        ```
          
    - Настроим маршрут по умолчанию на каждом роутере с привязкой к IP адресу на интерфейсе G0/0 другого роутера.
      *Пример настройки маршрута по умолчанию на R1:*
       ```
         !
         ipv6 route ::/0 2001:DB8:ACAD:2::2
         !
       ```     
    - Проверим работу маршрутизации отправив ping с R1 до IP адреса R2 на интерфейсе G0/1.
      ![ping](https://github.com/MIranaNightshade/otus-networks/blob/main/lab3_DHCP/DHCPv6/jpeg/ping.png)
  

  ## 2. Проверим получение адресов с помощью SLAAC от R1.
  
  *Для этого включим на PC-A получение IPv6 адреса с помощью SLAAC*
  
  ![PC_A_SLAAC](https://github.com/MIranaNightshade/otus-networks/blob/main/lab3_DHCP/DHCPv6/jpeg/PC_A_SLAAC.png)

          

  
