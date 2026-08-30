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

<br>

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
   mtu 9194
   no switchport
   ip address 10.1.16.0/31
   isis enable CORE
   isis bfd
   isis network point-to-point

!
interface Ethernet2
   description PtP to leaf2
   mtu 9194
   no switchport
   ip address 10.1.16.2/31
   isis enable CORE
   isis bfd
   isis network point-to-point
!
interface Ethernet3
   description PtP to leaf3
   mtu 9194
   no switchport
   ip address 10.1.16.4/31
   isis enable CORE
   isis bfd
   isis network point-to-point
!
interface Loopback0
   ip address 10.1.0.1/32
   isis enable CORE
!
ip routing
!
router isis CORE
   net 49.0001.0100.0100.0001.00
   is-type level-1
   !
   address-family ipv4 unicast
!
router multicast
   ipv4
      software-forwarding kernel
   !
   ipv6
      software-forwarding kernel
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
   mtu 9194
   no switchport
   ip address 10.1.16.6/31
   isis enable CORE
   isis bfd
   isis network point-to-point
!
interface Ethernet2
   description PtP to leaf2
   mtu 9194
   no switchport
   ip address 10.1.16.8/31
   isis enable CORE
   isis bfd
   isis network point-to-point
!
interface Ethernet3
   description PtP to leaf3
   mtu 9194
   no switchport
   ip address 10.1.16.10/31
   isis enable CORE
   isis bfd
   isis network point-to-point
!
interface Loopback0
   ip address 10.1.0.2/32
   isis enable CORE
!
ip routing
!
router isis CORE
   net 49.0001.0100.0100.0002.00
   is-type level-1
   !
   address-family ipv4 unicast
!
router multicast
   ipv4
      software-forwarding kernel
   !
   ipv6
      software-forwarding kernel
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
   mtu 9194
   no switchport
   ip address 10.1.16.1/31
   isis enable CORE
   isis bfd
   isis network point-to-point
!
interface Ethernet2
   description PtP to spine2
   mtu 9194
   no switchport
   ip address 10.1.16.7/31
   isis enable CORE
   isis bfd
   isis network point-to-point
!
interface Loopback0
   ip address 10.1.0.3/32
   isis enable CORE
!
ip routing
!
router isis CORE
   net 49.0001.0100.0100.0003.00
   is-type level-1
   !
   address-family ipv4 unicast
!
router multicast
   ipv4
      software-forwarding kernel
   !
   ipv6
      software-forwarding kernel
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
   mtu 9194
   no switchport
   ip address 10.1.16.3/31
   isis enable CORE
   isis bfd
   isis network point-to-point
!
interface Ethernet2
   description PtP to spine2
   mtu 9194
   no switchport
   ip address 10.1.16.9/31
   isis enable CORE
   isis bfd
   isis network point-to-point
!
interface Loopback0
   ip address 10.1.0.4/32
   isis enable CORE
!
ip routing
!
router isis CORE
   net 49.0001.0100.0100.0004.00
   is-type level-1
   !
   address-family ipv4 unicast
!
router multicast
   ipv4
      software-forwarding kernel
   !
   ipv6
      software-forwarding kernel
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
   mtu 9194
   no switchport
   ip address 10.1.16.5/31
   isis enable CORE
   isis bfd
   isis network point-to-point
!
interface Ethernet2
   description PtP to spine2
   mtu 9194
   no switchport
   ip address 10.1.16.11/31
   isis enable CORE
   isis bfd
   isis network point-to-point
!
interface Loopback0
   ip address 10.1.0.5/32
   isis enable CORE
!
ip routing
!
router isis CORE
   net 49.0001.0100.0100.0005.00
   is-type level-1
   !
   address-family ipv4 unicast
!
router multicast
   ipv4
      software-forwarding kernel
   !
   ipv6
      software-forwarding kernel
!
end
```
<br>

### 3. Проверка IS-IS и L3-связности

### spine1 neighbors

```
spine1#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
CORE      default  leaf1            L1   Ethernet1          P2P               UP    23          19
CORE      default  leaf2            L1   Ethernet2          P2P               UP    26          18
CORE      default  leaf3            L1   Ethernet3          P2P               UP    22          1E
```

<br>

### spine1 to spine2

```
spine1#ping 10.1.0.2 source 10.1.0.1
PING 10.1.0.2 (10.1.0.2) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.2: icmp_seq=1 ttl=63 time=9.14 ms
80 bytes from 10.1.0.2: icmp_seq=2 ttl=63 time=5.14 ms
80 bytes from 10.1.0.2: icmp_seq=3 ttl=63 time=4.39 ms
80 bytes from 10.1.0.2: icmp_seq=4 ttl=63 time=4.01 ms
80 bytes from 10.1.0.2: icmp_seq=5 ttl=63 time=5.67 ms

--- 10.1.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 49ms
rtt min/avg/max/mdev = 4.005/5.671/9.142/1.829 ms, ipg/ewma 12.245/7.357 ms
```

<br>

### spine1 to leaf1

```
spine1#ping 10.1.0.3 source 10.1.0.1
PING 10.1.0.3 (10.1.0.3) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.3: icmp_seq=1 ttl=64 time=3.74 ms
80 bytes from 10.1.0.3: icmp_seq=2 ttl=64 time=3.71 ms
80 bytes from 10.1.0.3: icmp_seq=3 ttl=64 time=1.83 ms
80 bytes from 10.1.0.3: icmp_seq=4 ttl=64 time=2.05 ms
80 bytes from 10.1.0.3: icmp_seq=5 ttl=64 time=1.99 ms

--- 10.1.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 27ms
rtt min/avg/max/mdev = 1.827/2.662/3.735/0.870 ms, ipg/ewma 6.769/3.148 ms
```

<br>

### spine1 to leaf2

```
spine1#ping 10.1.0.4 source 10.1.0.1
PING 10.1.0.4 (10.1.0.4) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.4: icmp_seq=1 ttl=64 time=7.36 ms
80 bytes from 10.1.0.4: icmp_seq=2 ttl=64 time=2.06 ms
80 bytes from 10.1.0.4: icmp_seq=3 ttl=64 time=2.02 ms
80 bytes from 10.1.0.4: icmp_seq=4 ttl=64 time=1.95 ms
80 bytes from 10.1.0.4: icmp_seq=5 ttl=64 time=2.60 ms

--- 10.1.0.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 32ms
rtt min/avg/max/mdev = 1.951/3.198/7.361/2.094 ms, ipg/ewma 7.941/5.219 ms
```

<br>

### spine1 to leaf3

```
spine1#ping 10.1.0.5 source 10.1.0.1
PING 10.1.0.5 (10.1.0.5) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.5: icmp_seq=1 ttl=64 time=6.74 ms
80 bytes from 10.1.0.5: icmp_seq=2 ttl=64 time=2.14 ms
80 bytes from 10.1.0.5: icmp_seq=3 ttl=64 time=2.20 ms
80 bytes from 10.1.0.5: icmp_seq=4 ttl=64 time=2.56 ms
80 bytes from 10.1.0.5: icmp_seq=5 ttl=64 time=2.63 ms

--- 10.1.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 26ms
rtt min/avg/max/mdev = 2.141/3.254/6.736/1.751 ms, ipg/ewma 6.506/4.948 ms
```

<br>

### spine2 neighbors

```
spine2#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
CORE      default  leaf1            L1   Ethernet1          P2P               UP    23          1E
CORE      default  leaf2            L1   Ethernet2          P2P               UP    28          19
CORE      default  leaf3            L1   Ethernet3          P2P               UP    26          22
```

<br>

### spine2 to leaf1

```
spine2#ping 10.1.0.3 source 10.1.0.2
PING 10.1.0.3 (10.1.0.3) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.3: icmp_seq=1 ttl=64 time=7.87 ms
80 bytes from 10.1.0.3: icmp_seq=2 ttl=64 time=2.06 ms
80 bytes from 10.1.0.3: icmp_seq=3 ttl=64 time=2.07 ms
80 bytes from 10.1.0.3: icmp_seq=4 ttl=64 time=1.80 ms
80 bytes from 10.1.0.3: icmp_seq=5 ttl=64 time=1.98 ms

--- 10.1.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 30ms
rtt min/avg/max/mdev = 1.801/3.154/7.868/2.358 ms, ipg/ewma 7.462/5.426 ms
```

<br>

### spine2 to leaf2

```
spine2#ping 10.1.0.4 source 10.1.0.2
PING 10.1.0.4 (10.1.0.4) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.4: icmp_seq=1 ttl=64 time=4.63 ms
80 bytes from 10.1.0.4: icmp_seq=2 ttl=64 time=2.68 ms
80 bytes from 10.1.0.4: icmp_seq=3 ttl=64 time=2.58 ms
80 bytes from 10.1.0.4: icmp_seq=4 ttl=64 time=2.34 ms
80 bytes from 10.1.0.4: icmp_seq=5 ttl=64 time=2.17 ms

--- 10.1.0.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 20ms
rtt min/avg/max/mdev = 2.170/2.879/4.630/0.893 ms, ipg/ewma 5.028/3.712 ms
```

<br>

### spine2 to leaf3

```
spine2#ping 10.1.0.5 source 10.1.0.2
PING 10.1.0.5 (10.1.0.5) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.5: icmp_seq=1 ttl=64 time=3.71 ms
80 bytes from 10.1.0.5: icmp_seq=2 ttl=64 time=1.84 ms
80 bytes from 10.1.0.5: icmp_seq=3 ttl=64 time=3.67 ms
80 bytes from 10.1.0.5: icmp_seq=4 ttl=64 time=1.68 ms
80 bytes from 10.1.0.5: icmp_seq=5 ttl=64 time=1.64 ms

--- 10.1.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 20ms
rtt min/avg/max/mdev = 1.637/2.506/3.709/0.966 ms, ipg/ewma 4.985/3.067 ms
```
