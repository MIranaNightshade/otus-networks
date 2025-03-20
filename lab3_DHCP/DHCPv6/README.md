# Лабораторная работа №3 DHCP (DHCPv6)
- Топология сети выполненная в EVE-NG:
  ![топология](https://github.com/MIranaNightshade/otus-networks/blob/main/lab3_DHCP/DHCPv6/jpeg/%D1%82%D0%BE%D0%BF%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%8F.png)

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
<td>2001:db8:acad:2::1 /64</td>
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
<td>2001:db8:acad:3::1 /64</td>
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
