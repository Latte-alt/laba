# Конфиг для схемы Cisco Packet Tracer

Ниже — итоговый конфиг для твоей схемы, где для ПК используются только **VLAN 101** и **VLAN 102**.

- **VLAN 101**: PC0, PC1, PC5
- **VLAN 102**: PC2, PC3, PC4
- **VLAN 6**: Server0
- **VLAN 7**: Server1
- **VLAN 5**: связь MLS ↔ Router1

## 1. Схема подключений по портам

### Multilayer Switch0 (MLS)
- `Gi0/1` ↔ `Gi0/1` Switch1
- `Gi0/2` ↔ `Gi0/2` Switch1
- `Fa0/1` ↔ `Fa0/0` Router1
- `Fa0/2` ↔ `Fa0/2` Switch2
- `Fa0/3` ↔ `Fa0/5` Switch4
- `Fa0/4` ↔ `Fa0/4` Switch3

### Switch1
- `Gi0/1` ↔ `Gi0/1` MLS
- `Gi0/2` ↔ `Gi0/2` MLS
- `Fa0/24` ↔ `Fa0/24` Switch0

### Switch0
- `Fa0/1` ↔ `Fa0` Server0
- `Fa0/2` ↔ `Fa0` Server1
- `Fa0/24` ↔ `Fa0/24` Switch1

### Switch2
- `Fa0/1` ↔ `Fa0` PC0
- `Fa0/2` ↔ `Fa0/2` MLS
- `Fa0/3` ↔ `Fa0` PC1

### Switch3
- `Fa0/1` ↔ `Fa0` PC2
- `Fa0/2` ↔ `Fa0` PC3
- `Fa0/4` ↔ `Fa0/4` MLS

### Switch4
- `Fa0/1` ↔ `Fa0` PC4
- `Fa0/2` ↔ `Fa0` PC5
- `Fa0/5` ↔ `Fa0/3` MLS

### Router1
- `Fa0/0` ↔ `Fa0/1` MLS
- `Fa0/1` ↔ `Fa0` Server2

## 2. VLAN и адресация

### VLAN
- VLAN 5 → `192.168.10.0/24`
- VLAN 6 → `192.168.6.0/24`
- VLAN 7 → `192.168.7.0/24`
- VLAN 101 → `192.168.1.0/24`
- VLAN 102 → `192.168.2.0/24`

### IP шлюзов на MLS
- VLAN 5 → `192.168.10.1`
- VLAN 6 → `192.168.6.1`
- VLAN 7 → `192.168.7.1`
- VLAN 101 → `192.168.1.1`
- VLAN 102 → `192.168.2.1`

### Серверы
- Server0 → `192.168.6.2/24`, GW `192.168.6.1`
- Server1 → `192.168.7.2/24`, GW `192.168.7.1`
- Server2 → `213.80.15.2/16`, GW `213.80.15.1`

### Router1
- к MLS → `192.168.10.2/24`
- к внешнему серверу → `213.80.15.1/16`

## 3. Распределение ПК по VLAN

### VLAN 101
- PC0
- PC1
- PC5

### VLAN 102
- PC2
- PC3
- PC4

---

## 4. Настройка MLS

```bash
enable
configure terminal
hostname MLS
no ip domain-lookup
```

### VLAN

```bash
vlan 5
name TRANSIT
exit

vlan 6
name SERVER_6
exit

vlan 7
name SERVER_7
exit

vlan 101
name USERS_101
exit

vlan 102
name USERS_102
exit
```

### EtherChannel к Switch1

```bash
interface range gi0/1 - 2
channel-group 1 mode active
no shutdown
exit
```

### Trunk на Port-channel

```bash
interface port-channel 1
switchport mode dynamic desirable
switchport mode trunk
no shutdown
exit
```

### Trunk к Switch2

```bash
interface fa0/2
switchport mode dynamic desirable
switchport mode trunk
no shutdown
exit
```

### Trunk к Switch4

```bash
interface fa0/3
switchport mode dynamic desirable
switchport mode trunk
no shutdown
exit
```

### Trunk к Switch3

```bash
interface fa0/4
switchport mode dynamic desirable
switchport mode trunk
no shutdown
exit
```

### Порт к Router1 через VLAN 5

```bash
interface fa0/1
switchport mode access
switchport access vlan 5
no shutdown
exit
```

### SVI

```bash
interface vlan 5
ip address 192.168.10.1 255.255.255.0
no shutdown
exit

interface vlan 6
ip address 192.168.6.1 255.255.255.0
no shutdown
exit

interface vlan 7
ip address 192.168.7.1 255.255.255.0
no shutdown
exit

interface vlan 101
ip address 192.168.1.1 255.255.255.0
no shutdown
exit

interface vlan 102
ip address 192.168.2.1 255.255.255.0
no shutdown
exit
```

### Маршрутизация

```bash
ip routing
ip route 0.0.0.0 0.0.0.0 192.168.10.2
```

### DHCP

```bash
ip dhcp excluded-address 192.168.1.1 192.168.1.20
ip dhcp excluded-address 192.168.2.1 192.168.2.20
```

```bash
ip dhcp pool VLAN101
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
exit

ip dhcp pool VLAN102
network 192.168.2.0 255.255.255.0
default-router 192.168.2.1
dns-server 8.8.8.8
exit
```

```bash
end
write memory
```

---

## 5. Настройка Switch1

```bash
enable
configure terminal
hostname SW1
no ip domain-lookup
```

```bash
vlan 5
vlan 6
vlan 7
vlan 101
vlan 102
exit
```

### EtherChannel к MLS

```bash
interface range gi0/1 - 2
channel-group 1 mode active
no shutdown
exit
```

### Trunk на Port-channel

```bash
interface port-channel 1
switchport mode trunk
no shutdown
exit
```

### Trunk к Switch0

```bash
interface fa0/24
switchport mode trunk
no shutdown
exit
```

```bash
end
write memory
```

---

## 6. Настройка Switch0

Серверный свитч.

```bash
enable
configure terminal
hostname SW0
no ip domain-lookup
```

```bash
vlan 6
name SERVER_6
exit

vlan 7
name SERVER_7
exit
```

```bash
interface fa0/1
switchport mode access
switchport access vlan 6
no shutdown
exit

interface fa0/2
switchport mode access
switchport access vlan 7
no shutdown
exit

interface fa0/24
switchport mode trunk
no shutdown
exit
```

```bash
end
write memory
```

---

## 7. Настройка Switch2

Тут **PC0 и PC1**, оба в **VLAN 101**.

```bash
enable
configure terminal
hostname SW2
no ip domain-lookup
```

```bash
vlan 101
name USERS_101
exit
```

```bash
interface fa0/1
switchport mode access
switchport access vlan 101
no shutdown
exit

interface fa0/2
switchport mode trunk
no shutdown
exit

interface fa0/3
switchport mode access
switchport access vlan 101
no shutdown
exit
```

```bash
end
write memory
```

---

## 8. Настройка Switch3

Тут **PC2 и PC3**, оба в **VLAN 102**.

```bash
enable
configure terminal
hostname SW3
no ip domain-lookup
```

```bash
vlan 102
name USERS_102
exit
```

```bash
interface fa0/1
switchport mode access
switchport access vlan 102
no shutdown
exit

interface fa0/2
switchport mode access
switchport access vlan 102
no shutdown
exit

interface fa0/4
switchport mode trunk
no shutdown
exit
```

```bash
end
write memory
```

---

## 9. Настройка Switch4

- **PC5 → VLAN 101**
- **PC4 → VLAN 102**

```bash
enable
configure terminal
hostname SW4
no ip domain-lookup
```

```bash
vlan 101
name USERS_101
exit

vlan 102
name USERS_102
exit
```

### PC4
```bash
interface fa0/1
switchport mode access
switchport access vlan 102
no shutdown
exit
```

### PC5
```bash
interface fa0/2
switchport mode access
switchport access vlan 101
no shutdown
exit
```

### Uplink к MLS
```bash
interface fa0/5
switchport mode trunk
no shutdown
exit
```

```bash
end
write memory
```

---

## 10. Настройка Router1

```bash
enable
configure terminal
hostname R1
no ip domain-lookup
```

```bash
interface fa0/0
ip address 192.168.10.2 255.255.255.0
no shutdown
exit

interface fa0/1
ip address 213.80.15.1 255.255.0.0
no shutdown
exit
```

### Маршруты назад
```bash
ip route 192.168.1.0 255.255.255.0 192.168.10.1
ip route 192.168.2.0 255.255.255.0 192.168.10.1
ip route 192.168.6.0 255.255.255.0 192.168.10.1
ip route 192.168.7.0 255.255.255.0 192.168.10.1
```

```bash
end
write memory
```

---

## 11. Настройка серверов

### Server0
```text
IP: 192.168.6.2
Mask: 255.255.255.0
Gateway: 192.168.6.1
```

### Server1
```text
IP: 192.168.7.2
Mask: 255.255.255.0
Gateway: 192.168.7.1
```

### Server2
```text
IP: 213.80.15.2
Mask: 255.255.0.0
Gateway: 213.80.15.1
```

---

## 12. Настройка ПК

На всех ПК ставишь **DHCP**:

- PC0
- PC1
- PC2
- PC3
- PC4
- PC5

Ожидаемо:
- PC0, PC1, PC5 получат `192.168.1.x`
- PC2, PC3, PC4 получат `192.168.2.x`

---

## 13. Проверка

### На MLS
```bash
show vlan brief
show interfaces trunk
show etherchannel summary
show ip interface brief
show ip route
show ip dhcp binding
```

### На Router1
```bash
show ip interface brief
show ip route
```

---

## 14. Ping для проверки

### С PC0
```bash
ping 192.168.1.1
ping 192.168.6.2
ping 192.168.7.2
ping 213.80.15.2
```

### С PC2
```bash
ping 192.168.2.1
ping 192.168.6.2
ping 192.168.7.2
ping 213.80.15.2
```

### С PC5
```bash
ping 192.168.1.1
ping 213.80.15.2
```

### С PC4
```bash
ping 192.168.2.1
ping 213.80.15.2
```
