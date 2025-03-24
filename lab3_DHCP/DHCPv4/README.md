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
   <td>G0/0/0</td>
   <td>10.0.0.1</td>
   <td>255.255.255.252</td>
   <td rowspan = 5>N/A</td>
   </tr>
   <td>G0/0/1</td>
   <td>N/A</td>
   <td>N/A</td>
   <tr>
  <td>G0/0/1.100</td>
  <td></td>
  <td></td>

   </tr>
   <tr>
    <td>G0/0/1.200</td>
	<td></td>
  <td></td>
   </tr>
   <td>G0/0/1.1000</td>
    <td></td>
	 <td></td>
   <tr>
   <td rowspan = 2>R2</td>
   <td >G0/0/0</td>
   <td >10.0.0.2</td>
   <td >255.255.255.252</td>
   <td rowspan = 2 >N/A</td>
   </tr>
   <tr>
   <td >G0/0/1</td>
	 <td ></td>
	  <td ></td>
   </tr>
   <tr>
   <td >S1</td>
   <td >VLAN 200</td>
   <td ></td>
   <td ></td>
   <td ></td>
   </tr>
   <tr>
   <td >S2</td>
   <td >VLAN 1</td>
    <td ></td>
   <td ></td>
   <td ></td>
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
  
   
   
