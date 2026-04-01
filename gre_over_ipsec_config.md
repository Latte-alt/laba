# GRE over IPsec — готовый конфиг для Packet Tracer

Ниже приведён **исправленный конфиг** под задание, где нужно:

- создать **туннель**
- **защитить туннель через IPsec**
- настроить связность между сетями:
  - слева `192.168.3.0/24`
  - справа `192.168.4.0/24`

Этот вариант сделан как **GRE over IPsec**.

---

## 1. Схема адресации

### Левая локальная сеть
- **Router1 Fa0/0** — `192.168.3.1/24`
- **PC0** — `192.168.3.2/24`, шлюз `192.168.3.1`
- **PC1** — `192.168.3.3/24`, шлюз `192.168.3.1`

### Правая локальная сеть
- **Router2 Fa0/0** — `192.168.4.1/24`
- **PC2** — `192.168.4.2/24`, шлюз `192.168.4.1`
- **PC3** — `192.168.4.3/24`, шлюз `192.168.4.1`

### Между роутерами
- **Router1 Fa0/1** — `10.1.0.1/30`
- **Router0 Fa0/0** — `10.1.0.2/30`

- **Router2 Fa0/1** — `10.2.0.1/30`
- **Router0 Fa0/1** — `10.2.0.2/30`

### Адреса туннеля
- **Router1 Tunnel0** — `172.16.0.1/30`
- **Router2 Tunnel0** — `172.16.0.2/30`

---

## 2. Подключение портов

### Switch0
- `Fa0/1` -> `Router1 Fa0/0`
- `Fa0/2` -> `PC0`
- `Fa0/3` -> `PC1`

### Switch1
- `Fa0/1` -> `Router2 Fa0/0`
- `Fa0/2` -> `PC2`
- `Fa0/3` -> `PC3`

### Между роутерами
- `Router1 Fa0/1` -> `Router0 Fa0/0`
- `Router0 Fa0/1` -> `Router2 Fa0/1`

---

## 3. Настройка Switch0

```cisco
enable
configure terminal
hostname Switch0
interface range fa0/1 - 3
 switchport mode access
 no shutdown
end
write memory
```

---

## 4. Настройка Switch1

```cisco
enable
configure terminal
hostname Switch1
interface range fa0/1 - 3
 switchport mode access
 no shutdown
end
write memory
```

---

## 5. Настройка Router1

Скопируй и вставь целиком:

```cisco
enable
configure terminal
hostname Router1

interface fa0/0
 ip address 192.168.3.1 255.255.255.0
 no shutdown

interface fa0/1
 ip address 10.1.0.1 255.255.255.252
 no shutdown

interface tunnel0
 ip address 172.16.0.1 255.255.255.252
 tunnel source fa0/1
 tunnel destination 10.2.0.1

ip route 0.0.0.0 0.0.0.0 10.1.0.2
ip route 192.168.4.0 255.255.255.0 172.16.0.2

access-list 110 permit gre host 10.1.0.1 host 10.2.0.1

crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400

crypto isakmp key cisco123 address 10.2.0.1

crypto ipsec transform-set MYSET esp-aes esp-sha-hmac

crypto map VPNMAP 10 ipsec-isakmp
 set peer 10.2.0.1
 set transform-set MYSET
 match address 110

interface fa0/1
 crypto map VPNMAP

end
write memory
```

---

## 6. Настройка Router0

Скопируй и вставь целиком:

```cisco
enable
configure terminal
hostname Router0

interface fa0/0
 ip address 10.1.0.2 255.255.255.252
 no shutdown

interface fa0/1
 ip address 10.2.0.2 255.255.255.252
 no shutdown

ip route 192.168.3.0 255.255.255.0 10.1.0.1
ip route 192.168.4.0 255.255.255.0 10.2.0.1

end
write memory
```

---

## 7. Настройка Router2

Скопируй и вставь целиком:

```cisco
enable
configure terminal
hostname Router2

interface fa0/0
 ip address 192.168.4.1 255.255.255.0
 no shutdown

interface fa0/1
 ip address 10.2.0.1 255.255.255.252
 no shutdown

interface tunnel0
 ip address 172.16.0.2 255.255.255.252
 tunnel source fa0/1
 tunnel destination 10.1.0.1

ip route 0.0.0.0 0.0.0.0 10.2.0.2
ip route 192.168.3.0 255.255.255.0 172.16.0.1

access-list 110 permit gre host 10.2.0.1 host 10.1.0.1

crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400

crypto isakmp key cisco123 address 10.1.0.1

crypto ipsec transform-set MYSET esp-aes esp-sha-hmac

crypto map VPNMAP 10 ipsec-isakmp
 set peer 10.1.0.1
 set transform-set MYSET
 match address 110

interface fa0/1
 crypto map VPNMAP

end
write memory
```

---

## 8. Настройка ПК

### PC0
- IP: `192.168.3.2`
- Mask: `255.255.255.0`
- Gateway: `192.168.3.1`

### PC1
- IP: `192.168.3.3`
- Mask: `255.255.255.0`
- Gateway: `192.168.3.1`

### PC2
- IP: `192.168.4.2`
- Mask: `255.255.255.0`
- Gateway: `192.168.4.1`

### PC3
- IP: `192.168.4.3`
- Mask: `255.255.255.0`
- Gateway: `192.168.4.1`

---

## 9. Проверка

### Проверка интерфейсов
На Router1 и Router2:

```cisco
show ip interface brief
```

Должно быть:
- `Fa0/0` — up
- `Fa0/1` — up
- `Tunnel0` — up

### Проверка туннеля
На Router1:

```cisco
ping 172.16.0.2
```

На Router2:

```cisco
ping 172.16.0.1
```

### Проверка связи между сетями
С **PC0**:

```bash
ping 192.168.4.2
```

С **PC2**:

```bash
ping 192.168.3.2
```

### Проверка IPsec
На Router1 и Router2:

```cisco
show crypto isakmp sa
show crypto ipsec sa
```

Нормальный признак работы:
- в `show crypto isakmp sa` есть `QM_IDLE`
- в `show crypto ipsec sa` растут `pkts encaps` и `pkts decaps`

---

## 10. Что сделано

В этой работе был создан **GRE-туннель** между `Router1` и `Router2` через промежуточный `Router0`.

После этого на крайних роутерах был настроен **IPsec**, который защищает не сразу локальные сети напрямую, а именно **GRE-трафик** между внешними адресами роутеров:

- с `10.1.0.1`
- на `10.2.0.1`

То есть:
- **GRE создаёт туннель**
- **IPsec шифрует этот туннель**

Маршруты в удалённые локальные сети были направлены через `Tunnel0`, поэтому трафик между `192.168.3.0/24` и `192.168.4.0/24` идёт внутри GRE, а снаружи защищён IPsec.

---

## 11. Если Packet Tracer не принимает AES/SHA

Замени на Router1 и Router2:

```cisco
crypto isakmp policy 10
 encr 3des
 hash md5
 authentication pre-share
 group 2
 lifetime 86400
```

И:

```cisco
crypto ipsec transform-set MYSET esp-3des esp-md5-hmac
```

---

## 12. Кратко для преподавателя

Мы настроили две локальные сети, соединили крайние роутеры через транзитный Router0, после чего создали GRE-туннель между Router1 и Router2. Далее настроили IPsec для защиты GRE-трафика. Маршруты до удалённых локальных сетей были направлены через Tunnel0. В результате получился защищённый туннель GRE over IPsec между двумя локальными сетями.
