# Lab 16 — New Symmetric Topology Configs

## Topology

```text
Cloud -> R1 G0/0
R1 G0/1 -> R2 G0/0
R1 G0/2 -> R3 G0/0

R2 G0/1 -> SW1 Fa0/1 -> PC1 Fa0
R2 G0/2 -> SW2 Fa0/1 -> PC2 Fa0

R3 G0/1 -> SW3 Fa0/1 -> PC3 Fa0
R3 G0/2 -> SW4 Fa0/1 -> PC4 Fa0
```

## Addressing

```text
R1 G0/0  = 203.0.113.2/24
Cloud    = 203.0.113.1/24

R1 G0/1  = 10.0.12.1/30
R2 G0/0  = 10.0.12.2/30

R1 G0/2  = 10.0.13.1/30
R3 G0/0  = 10.0.13.2/30

R2 G0/1  = 192.168.10.1/24
R2 G0/2  = 192.168.20.1/24

R3 G0/1  = 192.168.30.1/24
R3 G0/2  = 192.168.40.1/24
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

access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.30.0 0.0.0.255
access-list 1 permit 192.168.40.0 0.0.0.255

ip nat inside source list 1 interface GigabitEthernet0/0 overload

ip route 192.168.10.0 255.255.255.0 10.0.12.2
ip route 192.168.20.0 255.255.255.0 10.0.12.2
ip route 192.168.30.0 255.255.255.0 10.0.13.2
ip route 192.168.40.0 255.255.255.0 10.0.13.2
ip route 0.0.0.0 0.0.0.0 203.0.113.1

snmp-server community public RO
snmp-server community private RW
snmp-server location PT-LAB
snmp-server contact admin@lab.local

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
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/2
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit

ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.20.1 192.168.20.20

ip dhcp pool LAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
exit

ip dhcp pool LAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
exit

ip route 0.0.0.0 0.0.0.0 10.0.12.1
ip route 192.168.30.0 255.255.255.0 10.0.12.1
ip route 192.168.40.0 255.255.255.0 10.0.12.1

snmp-server community public RO
snmp-server location PT-LAB
snmp-server contact admin@lab.local

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
 ip address 192.168.30.1 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/2
 ip address 192.168.40.1 255.255.255.0
 no shutdown
exit

ip dhcp excluded-address 192.168.30.1 192.168.30.20
ip dhcp excluded-address 192.168.40.1 192.168.40.20

ip dhcp pool LAN30
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
exit

ip dhcp pool LAN40
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
 dns-server 8.8.8.8
exit

ip route 0.0.0.0 0.0.0.0 10.0.13.1
ip route 192.168.10.0 255.255.255.0 10.0.13.1
ip route 192.168.20.0 255.255.255.0 10.0.13.1

snmp-server community public RO
snmp-server location PT-LAB
snmp-server contact admin@lab.local

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

interface FastEthernet0/1
 switchport mode access
 no shutdown
exit

interface FastEthernet0/2
 switchport mode access
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

## SW2

```cisco
enable
configure terminal
hostname SW2
no ip domain-lookup

interface FastEthernet0/1
 switchport mode access
 no shutdown
exit

interface FastEthernet0/2
 switchport mode access
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

interface FastEthernet0/1
 switchport mode access
 no shutdown
exit

interface FastEthernet0/2
 switchport mode access
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

## SW4

```cisco
enable
configure terminal
hostname SW4
no ip domain-lookup

interface FastEthernet0/1
 switchport mode access
 no shutdown
exit

interface FastEthernet0/2
 switchport mode access
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

## Check Commands

```cisco
show ip interface brief
show ip route
show arp
show ip nat translations
show ip dhcp binding
```
