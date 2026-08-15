### 1. Адресное пространство (IPAM)

#### Общие подсети

- **Loopback0:** `10.1.0.0/24` (для Router ID и Loopback-интерфейсов)
    
- **P2P Links (Underlay):** `10.1.16.0/20` (Point-to-Point линки между Spine и Leaf)
    
- **Infra / Services:** `10.1.32.0/19` (Инфраструктура и сервисы)

<br>

#### Loopback0

| **Устройство** | **IP-адрес / Маска** |
| -------------- | -------------------- |
| **spine1**     | `10.1.0.1/32`        |
| **spine2**     | `10.1.0.2/32`        |
| **leaf1**      | `10.1.0.3/32`        |
| **leaf2**      | `10.1.0.4/32`        |
| **leaf3**      | `10.1.0.5/32`        |

#### PtP-подключения

| **Подсеть**     | **Spine Устройство & Интерфейс** | **IP Spine** | **Leaf Устройство & Интерфейс** | **IP Leaf**  |
| --------------- | -------------------------------- | ------------ | ------------------------------- | ------------ |
| `10.1.16.0/31`  | **spine1** Ethernet1             | `10.1.16.0`  | **leaf1** Ethernet1             | `10.1.16.1`  |
| `10.1.16.2/31`  | **spine1** Ethernet2             | `10.1.16.2`  | **leaf2** Ethernet1             | `10.1.16.3`  |
| `10.1.16.4/31`  | **spine1** Ethernet3             | `10.1.16.4`  | **leaf3** Ethernet1             | `10.1.16.5`  |
| `10.1.16.6/31`  | **spine2** Ethernet1             | `10.1.16.6`  | **leaf1** Ethernet2             | `10.1.16.7`  |
| `10.1.16.8/31`  | **spine2** Ethernet2             | `10.1.16.8`  | **leaf2** Ethernet2             | `10.1.16.9`  |
| `10.1.16.10/31` | **spine2** Ethernet3             | `10.1.16.10` | **leaf3** Ethernet2             | `10.1.16.11` |

<br>

### 2. Конфигурации устройств (Arista vEOS)
### spine1

```
hostname spine1
!
spanning-tree mode mstp
!
interface Ethernet1
   description PtP to leaf1
   no switchport
   ip address 10.1.16.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description PtP to leaf2
   no switchport
   ip address 10.1.16.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description PtP to leaf3
   no switchport
   ip address 10.1.16.4/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.1.0.1/32
   ip ospf area 0.0.0.0
!
ip routing
!
router multicast
   ipv4
      software-forwarding kernel
   !
   ipv6
      software-forwarding kernel
!
router ospf 1
   router-id 10.1.0.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
```
<br>

### spine2

```
hostname spine2
!
spanning-tree mode mstp
!
interface Ethernet1
   description PtP to leaf1
   no switchport
   ip address 10.1.16.6/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description PtP to leaf2
   no switchport
   ip address 10.1.16.8/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description PtP to leaf3
   no switchport
   ip address 10.1.16.10/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.1.0.2/32
   ip ospf area 0.0.0.0
!
ip routing
!
router multicast
   ipv4
      software-forwarding kernel
   !
   ipv6
      software-forwarding kernel
!
router ospf 1
   router-id 10.1.0.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
```
<br>

### leaf1

```
hostname leaf1
!
spanning-tree mode mstp
!
interface Ethernet1
   description PtP to spine1
   no switchport
   ip address 10.1.16.1/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description PtP to spine2
   no switchport
   ip address 10.1.16.7/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.1.0.3/32
   ip ospf area 0.0.0.0
!
ip routing
!
router multicast
   ipv4
      software-forwarding kernel
   !
   ipv6
      software-forwarding kernel
!
router ospf 1
   router-id 10.1.0.3
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
```
<br>

### leaf2

```
hostname leaf2
!
spanning-tree mode mstp
!
interface Ethernet1
   description PtP to spine1
   no switchport
   ip address 10.1.16.3/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description PtP to spine2
   no switchport
   ip address 10.1.16.9/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.1.0.4/32
   ip ospf area 0.0.0.0
!
ip routing
!
router multicast
   ipv4
      software-forwarding kernel
   !
   ipv6
      software-forwarding kernel
!
router ospf 1
   router-id 10.1.0.4
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
```
<br>

### leaf3

```
hostname leaf3
!
spanning-tree mode mstp
!
interface Ethernet1
   description PtP to spine1
   no switchport
   ip address 10.1.16.5/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description PtP to leaf2
   no switchport
   ip address 10.1.16.11/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.1.0.5/32
   ip ospf area 0.0.0.0
!
ip routing
!
router multicast
   ipv4
      software-forwarding kernel
   !
   ipv6
      software-forwarding kernel
!
router ospf 1
   router-id 10.1.0.5
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
```
<br>

### 3. Проверка связности

### spine1 to spine2
```
spine1#ping 10.1.0.2 source 10.1.0.1
PING 10.1.0.2 (10.1.0.2) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.2: icmp_seq=1 ttl=63 time=13.6 ms
80 bytes from 10.1.0.2: icmp_seq=2 ttl=63 time=5.49 ms
80 bytes from 10.1.0.2: icmp_seq=3 ttl=63 time=5.39 ms
80 bytes from 10.1.0.2: icmp_seq=4 ttl=63 time=4.17 ms
80 bytes from 10.1.0.2: icmp_seq=5 ttl=63 time=4.88 ms

--- 10.1.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 51ms
rtt min/avg/max/mdev = 4.174/6.696/13.553/3.459 ms, ipg/ewma 12.643/9.986 ms
```

<br>

### spine1 to leaf1
```
spine1#ping 10.1.0.3 source 10.1.0.1
PING 10.1.0.3 (10.1.0.3) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.3: icmp_seq=1 ttl=64 time=6.80 ms
80 bytes from 10.1.0.3: icmp_seq=2 ttl=64 time=2.48 ms
80 bytes from 10.1.0.3: icmp_seq=3 ttl=64 time=3.13 ms
80 bytes from 10.1.0.3: icmp_seq=4 ttl=64 time=1.79 ms
80 bytes from 10.1.0.3: icmp_seq=5 ttl=64 time=1.95 ms

--- 10.1.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 34ms
rtt min/avg/max/mdev = 1.787/3.228/6.795/1.843 ms, ipg/ewma 8.413/4.930 ms
```

<br>

### spine1 to leaf2
```
spine1#ping 10.1.0.4 source 10.1.0.1
PING 10.1.0.4 (10.1.0.4) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.4: icmp_seq=1 ttl=64 time=5.28 ms
80 bytes from 10.1.0.4: icmp_seq=2 ttl=64 time=2.03 ms
80 bytes from 10.1.0.4: icmp_seq=3 ttl=64 time=1.84 ms
80 bytes from 10.1.0.4: icmp_seq=4 ttl=64 time=1.71 ms
80 bytes from 10.1.0.4: icmp_seq=5 ttl=64 time=1.77 ms

--- 10.1.0.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 20ms
rtt min/avg/max/mdev = 1.705/2.525/5.279/1.381 ms, ipg/ewma 5.065/3.848 ms
```

<br>

### spine1 to leaf3
```
spine1#ping 10.1.0.5 source 10.1.0.1
PING 10.1.0.5 (10.1.0.5) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.5: icmp_seq=1 ttl=64 time=4.98 ms
80 bytes from 10.1.0.5: icmp_seq=2 ttl=64 time=2.12 ms
80 bytes from 10.1.0.5: icmp_seq=3 ttl=64 time=2.42 ms
80 bytes from 10.1.0.5: icmp_seq=4 ttl=64 time=2.16 ms
80 bytes from 10.1.0.5: icmp_seq=5 ttl=64 time=2.17 ms

--- 10.1.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 20ms
rtt min/avg/max/mdev = 2.116/2.770/4.981/1.110 ms, ipg/ewma 5.025/3.836 ms
```

<br>

### spine2 to leaf1
```
spine2#ping 10.1.0.3 source 10.1.0.2
PING 10.1.0.3 (10.1.0.3) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.3: icmp_seq=1 ttl=64 time=7.06 ms
80 bytes from 10.1.0.3: icmp_seq=2 ttl=64 time=6.97 ms
80 bytes from 10.1.0.3: icmp_seq=3 ttl=64 time=3.61 ms
80 bytes from 10.1.0.3: icmp_seq=4 ttl=64 time=1.83 ms
80 bytes from 10.1.0.3: icmp_seq=5 ttl=64 time=4.02 ms

--- 10.1.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 31ms
rtt min/avg/max/mdev = 1.830/4.697/7.057/2.029 ms, ipg/ewma 7.688/5.768 ms

```

<br>

### spine2 to leaf2
```
spine2#ping 10.1.0.4 source 10.1.0.2
PING 10.1.0.4 (10.1.0.4) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.4: icmp_seq=1 ttl=64 time=6.94 ms
80 bytes from 10.1.0.4: icmp_seq=2 ttl=64 time=2.07 ms
80 bytes from 10.1.0.4: icmp_seq=3 ttl=64 time=2.45 ms
80 bytes from 10.1.0.4: icmp_seq=4 ttl=64 time=1.75 ms
80 bytes from 10.1.0.4: icmp_seq=5 ttl=64 time=1.64 ms

--- 10.1.0.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 26ms
rtt min/avg/max/mdev = 1.639/2.967/6.937/2.004 ms, ipg/ewma 6.451/4.869 ms
```

<br>

### spine2 to leaf3
```
spine2#ping 10.1.0.5 source 10.1.0.2
PING 10.1.0.5 (10.1.0.5) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.5: icmp_seq=1 ttl=64 time=6.76 ms
80 bytes from 10.1.0.5: icmp_seq=2 ttl=64 time=2.00 ms
80 bytes from 10.1.0.5: icmp_seq=3 ttl=64 time=2.29 ms
80 bytes from 10.1.0.5: icmp_seq=4 ttl=64 time=2.03 ms
80 bytes from 10.1.0.5: icmp_seq=5 ttl=64 time=1.98 ms

--- 10.1.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 26ms
rtt min/avg/max/mdev = 1.980/3.012/6.758/1.876 ms, ipg/ewma 6.419/4.817 ms
```
