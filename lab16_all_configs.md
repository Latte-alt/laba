# Lab 16 Configs

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

interface GigabitEthernet0/3
 no shutdown
exit

interface GigabitEthernet0/3.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
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
access-list 1 permit 10.0.12.0 0.0.0.3
access-list 1 permit 10.0.13.0 0.0.0.3

ip nat inside source list 1 interface GigabitEthernet0/0 overload

ip route 192.168.20.0 255.255.255.0 10.0.12.2
ip route 0.0.0.0 0.0.0.0 203.0.113.1

ip access-list extended OUTSIDE_IN
 permit tcp any any established
 permit icmp any any echo-reply
 deny ip any any
exit

interface GigabitEthernet0/0
 ip access-group OUTSIDE_IN in
exit

ip access-list extended INSIDE_OUT
 permit tcp 192.168.10.0 0.0.0.255 any eq 80
 permit tcp 192.168.10.0 0.0.0.255 any eq 443
 permit tcp 192.168.10.0 0.0.0.255 any eq 22
 permit udp 192.168.10.0 0.0.0.255 any eq 53
 permit icmp 192.168.10.0 0.0.0.255 any
 permit tcp 192.168.20.0 0.0.0.255 any eq 80
 permit tcp 192.168.20.0 0.0.0.255 any eq 443
 permit tcp 192.168.20.0 0.0.0.255 any eq 22
 permit udp 192.168.20.0 0.0.0.255 any eq 53
 permit icmp 192.168.20.0 0.0.0.255 any
 deny ip any any
exit

interface GigabitEthernet0/1
 ip access-group INSIDE_OUT in
exit

interface GigabitEthernet0/2
 ip access-group INSIDE_OUT in
exit

interface GigabitEthernet0/3.10
 ip access-group INSIDE_OUT in
exit

snmp-server community public RO
snmp-server community private RW
snmp-server location PT-LAB
snmp-server contact admin@lab.local

end
write memory
```

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

interface GigabitEthernet0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 10.0.12.1
exit

ip route 192.168.10.0 255.255.255.0 10.0.12.1
ip route 0.0.0.0 0.0.0.0 10.0.12.1

ip access-list extended VLAN20_FILTER
 deny tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 22
 permit ip 192.168.20.0 0.0.0.255 any
exit

interface GigabitEthernet0/1.20
 ip access-group VLAN20_FILTER in
exit

snmp-server community public RO
snmp-server location PT-LAB
snmp-server contact admin@lab.local

end
write memory
```

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
 ip address 172.16.30.1 255.255.255.0
 no shutdown
exit

ip route 192.168.10.0 255.255.255.0 10.0.13.1
ip route 192.168.20.0 255.255.255.0 10.0.13.1
ip route 0.0.0.0 0.0.0.0 10.0.13.1

snmp-server community public RO
snmp-server location PT-LAB
snmp-server contact admin@lab.local

end
write memory
```

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
 name USERS_A
exit

vlan 20
 name USERS_B
exit

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit

interface FastEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit

interface FastEthernet0/3
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit

interface FastEthernet0/4
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit

interface range FastEthernet0/5 - 24
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
exit

snmp-server community public RO

end
write memory
```

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
exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
exit

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
exit

interface range FastEthernet0/4 - 24
 shutdown
exit

snmp-server community public RO

end
write memory
```

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
exit

interface FastEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 20
exit

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
exit

interface range FastEthernet0/4 - 24
 shutdown
exit

snmp-server community public RO

end
write memory
```

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
exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
exit

interface range FastEthernet0/3 - 24
 shutdown
exit

snmp-server community public RO

end
write memory
```

## PC1
```text
IP Configuration -> DHCP
```

## PC2
```text
IP Configuration -> DHCP
```

## PC3
```text
IP Configuration -> DHCP
```

## PC4
```text
IP Configuration -> DHCP
```

## Проверка
```cisco
show ip interface brief
show ip route
show arp
show ip nat translations
show vlan brief
show interfaces trunk
show vtp status
```
