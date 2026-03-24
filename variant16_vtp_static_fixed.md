# Вариант 16 — VTP + статическая маршрутизация + проверка ARP, Packet Tracer

Для **варианта 16** по заданию нужно:
- использовать **статическую маршрутизацию**;
- настроить **VTP для управления VLAN**;
- выполнить **проверку ARP-таблицы**.

Ниже — рабочий вариант под Packet Tracer с 3 роутерами, 4 коммутаторами, 4 ПК, NAT, DHCP, trunk, Port Security, ACL, SNMP и командами проверки.

## Топология

```text
Cloud -> R1 G0/0
R1 G0/1 -> R2 G0/0
R1 G0/2 -> R3 G0/0

R2 G0/1 -> SW1 Fa0/1
SW1 Fa0/2 -> SW2 Fa0/1
SW1 Fa0/3 -> SW3 Fa0/1
SW3 Fa0/2 -> SW4 Fa0/1

SW1 Fa0/4 -> PC1
SW2 Fa0/2 -> PC2
SW3 Fa0/3 -> PC3
SW4 Fa0/2 -> PC4
```

## Логика VLAN

```text
VLAN 10 -> PC1, PC2
VLAN 20 -> PC3, PC4
```

## Роли устройств

```text
R1 -> центральный роутер, NAT, DHCP, ACL, статические маршруты
R2 -> шлюз VLAN 10
R3 -> шлюз VLAN 20

SW1 -> VTP Server
SW2 -> VTP Client
SW3 -> VTP Client
SW4 -> VTP Client
```

## Адресация

```text
R1 G0/0      = 203.0.113.2/24
Cloud        = 203.0.113.1/24

R1 G0/1      = 10.0.12.1/30
R2 G0/0      = 10.0.12.2/30

R1 G0/2      = 10.0.13.1/30
R3 G0/0      = 10.0.13.2/30

R2 G0/1.10   = 192.168.10.1/24
R3 G0/1.20   = 192.168.20.1/24
```

---

## R1

```cisco
enable
configure terminal
hostname R1
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 203.0.113.2 255.255.255.0
 ip nat outside
 no shutdown
exit

interface GigabitEthernet0/1
 ip address 10.0.12.1 255.255.255.252
 ip nat inside
 no shutdown
exit

interface GigabitEthernet0/2
 ip address 10.0.13.1 255.255.255.252
 ip nat inside
 no shutdown
exit

ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.20.1 192.168.20.20

ip dhcp pool VLAN10_POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
exit

ip dhcp pool VLAN20_POOL
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
exit

access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255

ip nat inside source list 1 interface GigabitEthernet0/0 overload

ip route 192.168.10.0 255.255.255.0 10.0.12.2
ip route 192.168.20.0 255.255.255.0 10.0.13.2
ip route 0.0.0.0 0.0.0.0 203.0.113.1

ip access-list extended OUTSIDE_IN
 permit tcp any any established
 permit icmp any any echo-reply
 deny ip any any
exit

interface GigabitEthernet0/0
 ip access-group OUTSIDE_IN in
exit

snmp-server community public RO
snmp-server community private RW

end
write memory
```

---

## R2

```cisco
enable
configure terminal
hostname R2
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 10.0.12.2 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/1
 no shutdown
exit

interface GigabitEthernet0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 10.0.12.1
 no shutdown
exit

ip route 0.0.0.0 0.0.0.0 10.0.12.1
ip route 192.168.20.0 255.255.255.0 10.0.12.1

snmp-server community public RO

end
write memory
```

---

## R3

```cisco
enable
configure terminal
hostname R3
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 10.0.13.2 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/1
 no shutdown
exit

interface GigabitEthernet0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 10.0.13.1
 no shutdown
exit

ip route 0.0.0.0 0.0.0.0 10.0.13.1
ip route 192.168.10.0 255.255.255.0 10.0.13.1

snmp-server community public RO

end
write memory
```

---

## SW1

```cisco
enable
configure terminal
hostname SW1
no ip domain-lookup

vtp domain LAB16
vtp mode server
vtp password cisco
vtp version 2

vlan 10
 name VLAN10
exit

vlan 20
 name VLAN20
exit

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10
 no shutdown
exit

interface FastEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit

interface FastEthernet0/3
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit

interface FastEthernet0/4
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 no shutdown
exit

interface range FastEthernet0/5 - 24
 shutdown
exit

snmp-server community public RO

end
write memory
```

---

## SW2

```cisco
enable
configure terminal
hostname SW2
no ip domain-lookup

vtp domain LAB16
vtp mode client
vtp password cisco
vtp version 2

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 no shutdown
exit

interface range FastEthernet0/3 - 24
 shutdown
exit

snmp-server community public RO

end
write memory
```

---

## SW3

```cisco
enable
configure terminal
hostname SW3
no ip domain-lookup

vtp domain LAB16
vtp mode client
vtp password cisco
vtp version 2

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit

interface FastEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 no shutdown
exit

interface range FastEthernet0/4 - 24
 shutdown
exit

snmp-server community public RO

end
write memory
```

---

## SW4

```cisco
enable
configure terminal
hostname SW4
no ip domain-lookup

vtp domain LAB16
vtp mode client
vtp password cisco
vtp version 2

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 no shutdown
exit

interface range FastEthernet0/3 - 24
 shutdown
exit

snmp-server community public RO

end
write memory
```

---

## PC1

```text
Desktop -> IP Configuration -> DHCP
```

## PC2

```text
Desktop -> IP Configuration -> DHCP
```

## PC3

```text
Desktop -> IP Configuration -> DHCP
```

## PC4

```text
Desktop -> IP Configuration -> DHCP
```

---

## Проверка VTP

```cisco
show vtp status
show vlan brief
show interfaces trunk
```

## Проверка статической маршрутизации

```cisco
show ip route
show ip interface brief
```

## Проверка DHCP/NAT

```cisco
show ip dhcp binding
show ip nat translations
```

## Проверка ARP

### На роутерах
```cisco
show arp
```

### На ПК
```text
arp -a
```
