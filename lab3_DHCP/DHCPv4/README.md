# Лабораторная работа №3 DHCPv4
  - **Топология сети, выполненная в EVE-NG:**

 ![topology](https://github.com/MIranaNightshade/otus-networks/blob/main/lab3_DHCP/DHCPv4/jpeg/Topologyv4.png)
 
   
  - **Таблица адресов:**
    
<table>
   <tr>
   <th>Device</th>
        <th>Interface</th>
        <th>IP Address</th>
		<th>Subnet Mask</th>
		<th>Default Gateway</th>
   </tr>
   <tr>
   <td rowspan = 5>R1</td>
   <td>G0/0</td>
   <td>10.0.0.1</td>
   <td>255.255.255.252</td>
   <td rowspan = 5>N/A</td>
   </tr>
   <td>G0/1</td>
   <td>N/A</td>
   <td>N/A</td>
   <tr>
  <td>G0/1.100</td>
  <td>192.168.0.1</td>
  <td>255.255.255.128</td>

   </tr>
   <tr>
    <td>G0/1.200</td>
	<td>192.168.0.129</td>
  <td>255.255.255.192</td>
   </tr>
   <td>G0/1.1000</td>
    <td>N/A</td>
	 <td>N/A</td>
   <tr>
   <td rowspan = 2>R2</td>
   <td >G0/0</td>
   <td >10.0.0.2</td>
   <td >255.255.255.252</td>
   <td rowspan = 2 >N/A</td>
   </tr>
   <tr>
   <td >G0/1</td>
	 <td >192.168.0.193</td>
	  <td >255.255.255.192</td>
   </tr>
   <tr>
   <td >S1</td>
   <td >VLAN 200</td>
   <td >192.168.0.130</td>
   <td >255.255.255.192</td>
   <td >192.168.0.129</td>
   </tr>
   <tr>
   <td >S2</td>
   <td >VLAN 1</td>
    <td >192.168.0.194</td>
   <td >255.255.255.192</td>
   <td >192.168.0.193</td>
   </tr>
   <tr>
    <td >PC-A</td>
	<td >NIC</td>
	<td >DHCP</td>
	<td >DHCP</td>
	<td >DHCP</td>
   </tr>
	<tr>
	<td >PC-B</td>
	<td >NIC</td>
	<td >DHCP</td>
	<td >DHCP</td>
	<td >DHCP</td>
	</tr>
</table>


  - **Таблица VLAN:**

   | VLAN | NAME  | INTERFACE ASSIGNED |
  | -----| ----| -----|
  | 1 | N/A | S2: e0/3 |
  | 100 | clients | S1: e0/3 |
  | 200 | management | S1: VLAN 200   |
  | 999 | parking lot | S1: e0/0, e0/2|
  |1000 | native | N/A |

  ## Цели:
  __________________________________________________________________________________________
   1. [Создать сеть и настроить базовую конфигурацию.](#title1)
   2. [Настроить два DHCPv4 сервера на R1 и проверить их работу.](#title2)
   3. [Настроить DHCP relay на R2 и проверить его работу.](#title3)
   __________________________________________________________________________________________
   
  ### <a id="title1"> 1. Создать сеть и настроить базовую конфигурацию.</a>
  
   1. **Разделим сеть 192.168.0.0/24 на подсети.**
      - Подсеть А 192.168.0.0/25 (client vlan R1, 192.168.0.1/25 для **R1 G0/0/1.100**)
      - Подсеть В 192.168.0.128/26 (management VLAN R1, 192.168.0.129/26 для **R1 G0/0/1.200**, 192.168.0.130/26 **S1 VLAN 200**)
      - Подсеть С 192.168.0.192/26 (сеть для клиентов на R2, 192.168.0.193/26 **R2 G0/0/1**)
   2. **Зададим базовые настройки на обоих роутерах и настроим межвланную маршрутизацию на R1, настроим статическую маршрутизацию между R1 и R2.**
      - Конфигурация R1.
	       ```  
		hostname R1
		!
		enable secret 5 $1$zdpZ$8brH5mFX7sIogZEFYEvCU/
		!
		interface GigabitEthernet0/0
		 ip address 10.0.0.1 255.255.255.252
		 duplex auto
		 speed auto
		 media-type rj45
		!
		interface GigabitEthernet0/1
		 no ip address
		 duplex auto
		 speed auto
		 media-type rj45
		!
		interface GigabitEthernet0/1.100
		 encapsulation dot1Q 100
		 ip address 192.168.0.1 255.255.255.128
		!
		interface GigabitEthernet0/1.200
		 encapsulation dot1Q 200
		 ip address 192.168.0.129 255.255.255.192
		!
		no ip http server
		no ip http secure-server
		ip route 0.0.0.0 0.0.0.0 10.0.0.2
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
		 password 7 045802150C2E
		 login
		line aux 0
		line vty 0 4
		 password 7 110A1016141D
		 login
		 transport input none
		line vty 5 15
		 password 7 110A1016141D
		 login
		 transport input none
		!
		no scheduler allocate
		!
		end
		
		R1#
	        		
      - Конфигурация R2:
      
	        
		```
		hostname R2
		!
		!
		enable secret 5 $1$GhJj$M.us3si26yx9bxSneA58U/
		!
		no aaa new-model
		ethernet lmi ce
		!
		no ip domain lookup
		!
		interface GigabitEthernet0/0
		 ip address 10.0.0.2 255.255.255.252
		 duplex auto
		 speed auto
		 media-type rj45
		!
		interface GigabitEthernet0/1
		 ip address 192.168.0.193 255.255.255.192
		 duplex auto
		 speed auto
		 media-type rj45
		!
		no ip http server
		no ip http secure-server
		ip route 0.0.0.0 0.0.0.0 10.0.0.1
		!
		banner motd ^C
		*******************************
		*******************************
		* HANDS OFF FROM MY LITTLE R2 *
		*  AND GET OUT OF HERE        *
		*******************************
		*******************************
		^C
		!
		line con 0
		 password 7 14141B180F0B
		 login
		line aux 0
		line vty 0 4
		 password 7 030752180500
		 login
		 transport input none
		line vty 5 15
		 password 7 030752180500
		 login
		 transport input none
		!
		no scheduler allocate
		!
		end  
        
      - **Проверим работу маршрутизации между роутерами, отправим ping с R1 на IP 192.168.0.193 (R2 G0/0/1):**
        
        ![ping](https://github.com/MIranaNightshade/otus-networks/blob/main/lab3_DHCP/DHCPv4/jpeg/ping_check.png)

 3. **Настроим коммутаторы S1 и S2, зададим базовые настройки, настроим VLAN на S1 согласно таблице VLAN, зададим коммутаторам IP адреса согласно таблице и зададим default-gateway**
    
      - **Конфигурация S1**
        
	       ```
	        hostname S1
		!
		!
		enable secret 5 $1$Dfee$zXHUxlvHVZwXBLcH4X8b4/
		!
		no ip domain-lookup
		!
		interface Ethernet0/0
		 switchport access vlan 999
		 switchport mode access
		 shutdown
		!
		interface Ethernet0/1
		 switchport trunk allowed vlan 100,200,1000
		 switchport trunk encapsulation dot1q
		 switchport trunk native vlan 1000
		 switchport mode trunk
		!
		interface Ethernet0/2
		 switchport access vlan 999
		 switchport mode access
		 shutdown
		!
		interface Ethernet0/3
		 switchport access vlan 100
		 switchport mode access
		!
		interface Vlan200
		 ip address 192.168.0.130 255.255.255.192
		!
		ip default-gateway 192.168.0.129
		!
		banner motd ^C
		*******************************
		*******************************
		* HANDS OFF FROM MY LITTLE S1 *
		*  AND GET OUT OF HERE        *
		*******************************
		*******************************
		^C
		!
		line con 0
		 password 7 110A1016141D
		 logging synchronous
		 login
		line aux 0
		line vty 0 4
		 password 7 030752180500
		 login
		!
		!
		end
                

        
      - **Конфигурация S2**
        
	       ```
		hostname S2
		!
		enable secret 5 $1$4o.V$WnkNberihLmZHsmGJ.6Zt.
		!
		no ip domain-lookup
		ip cef
		no ipv6 cef
		!
		interface Ethernet0/0
		 shutdown
		!
		interface Ethernet0/1
		!
		interface Ethernet0/2
		 shutdown
		!
		interface Ethernet0/3
		!
		interface Vlan1
		 ip address 192.168.0.194 255.255.255.192
		 shutdown
		!
		ip default-gateway 192.168.0.193
		ip forward-protocol nd
		!
		no ip http server
		no ip http secure-server
		!
		banner motd ^C
		*******************************
		*******************************
		* HANDS OFF FROM MY LITTLE S2 *
		*  AND GET OUT OF HERE        *
		*******************************
		*******************************
		^C
		!
		line con 0
		 password 7 121A0C041104
		 logging synchronous
		 login
		line aux 0
		line vty 0 4
		 password 7 110A1016141D
		 login
		!
		!
		end

  ### <a id="title2"> 1. Настроим два DHCPv4 сервера на R1.</a>

  
         
