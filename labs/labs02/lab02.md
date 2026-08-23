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
   mtu 9194
   no switchport
   ip address 10.1.16.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description PtP to leaf2
   mtu 9194
   no switchport
   ip address 10.1.16.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description PtP to leaf3
   mtu 9194
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
   bfd default
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
```

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
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description PtP to leaf2
   mtu 9194
   no switchport
   ip address 10.1.16.8/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description PtP to leaf3
   mtu 9194
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
   bfd default
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
```

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
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description PtP to spine2
   mtu 9194
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
   bfd default
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
```

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
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description PtP to spine2
   mtu 9194
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
   bfd default
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
```

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
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description PtP to spine2
   mtu 9194
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
   bfd default
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
```


### 3. Проверка OSPF и L3-связности

### spine1 neighbors

```
spine1#show ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.1.0.3        1        default  0   FULL                   00:00:36    10.1.16.1       Ethernet1
10.1.0.4        1        default  0   FULL                   00:00:34    10.1.16.3       Ethernet2
10.1.0.5        1        default  0   FULL                   00:00:38    10.1.16.5       Ethernet3
```

### spine1 to spine2
```
spine1#ping 10.1.0.2 source 10.1.0.1
PING 10.1.0.2 (10.1.0.2) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.2: icmp_seq=1 ttl=63 time=26.4 ms
80 bytes from 10.1.0.2: icmp_seq=2 ttl=63 time=7.66 ms
80 bytes from 10.1.0.2: icmp_seq=3 ttl=63 time=4.65 ms
80 bytes from 10.1.0.2: icmp_seq=4 ttl=63 time=4.05 ms
80 bytes from 10.1.0.2: icmp_seq=5 ttl=63 time=4.32 ms

--- 10.1.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 87ms
rtt min/avg/max/mdev = 4.050/9.410/26.372/8.579 ms, pipe 2, ipg/ewma 21.874/17.528 ms
```

### spine1 to leaf1
```
spine1#ping 10.1.0.3 source 10.1.0.1
PING 10.1.0.3 (10.1.0.3) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.3: icmp_seq=1 ttl=64 time=3.98 ms
80 bytes from 10.1.0.3: icmp_seq=2 ttl=64 time=1.78 ms
80 bytes from 10.1.0.3: icmp_seq=3 ttl=64 time=1.77 ms
80 bytes from 10.1.0.3: icmp_seq=4 ttl=64 time=1.98 ms
80 bytes from 10.1.0.3: icmp_seq=5 ttl=64 time=3.99 ms

--- 10.1.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 17ms
rtt min/avg/max/mdev = 1.766/2.699/3.991/1.054 ms, ipg/ewma 4.354/3.368 ms
```


### spine1 to leaf2
```
spine1#ping 10.1.0.4 source 10.1.0.1
PING 10.1.0.4 (10.1.0.4) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.4: icmp_seq=1 ttl=64 time=5.75 ms
80 bytes from 10.1.0.4: icmp_seq=2 ttl=64 time=1.85 ms
80 bytes from 10.1.0.4: icmp_seq=3 ttl=64 time=1.53 ms
80 bytes from 10.1.0.4: icmp_seq=4 ttl=64 time=1.54 ms
80 bytes from 10.1.0.4: icmp_seq=5 ttl=64 time=1.74 ms

--- 10.1.0.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 22ms
rtt min/avg/max/mdev = 1.533/2.481/5.746/1.636 ms, ipg/ewma 5.381/4.055 ms
```


### spine1 to leaf3
```
spine1#ping 10.1.0.5 source 10.1.0.1
PING 10.1.0.5 (10.1.0.5) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.5: icmp_seq=1 ttl=64 time=7.75 ms
80 bytes from 10.1.0.5: icmp_seq=2 ttl=64 time=1.94 ms
80 bytes from 10.1.0.5: icmp_seq=3 ttl=64 time=1.77 ms
80 bytes from 10.1.0.5: icmp_seq=4 ttl=64 time=1.99 ms
80 bytes from 10.1.0.5: icmp_seq=5 ttl=64 time=2.24 ms

--- 10.1.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 30ms
rtt min/avg/max/mdev = 1.767/3.139/7.752/2.311 ms, ipg/ewma 7.496/5.374 ms
```

<br>

### spine2 neighbors

```
spine2#show ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.1.0.3        1        default  0   FULL                   00:00:35    10.1.16.7       Ethernet1
10.1.0.4        1        default  0   FULL                   00:00:32    10.1.16.9       Ethernet2
10.1.0.5        1        default  0   FULL                   00:00:35    10.1.16.11      Ethernet3
```

<br>
### spine2 to leaf1
```
spine2#ping 10.1.0.3 source 10.1.0.2
PING 10.1.0.3 (10.1.0.3) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.3: icmp_seq=1 ttl=64 time=3.95 ms
80 bytes from 10.1.0.3: icmp_seq=2 ttl=64 time=1.74 ms
80 bytes from 10.1.0.3: icmp_seq=3 ttl=64 time=1.53 ms
80 bytes from 10.1.0.3: icmp_seq=4 ttl=64 time=1.59 ms
80 bytes from 10.1.0.3: icmp_seq=5 ttl=64 time=1.43 ms

--- 10.1.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 17ms
rtt min/avg/max/mdev = 1.433/2.046/3.947/0.955 ms, ipg/ewma 4.251/2.958 ms
```

### spine2 to leaf2
```
spine2#ping 10.1.0.4 source 10.1.0.2
PING 10.1.0.4 (10.1.0.4) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.4: icmp_seq=1 ttl=64 time=3.94 ms
80 bytes from 10.1.0.4: icmp_seq=2 ttl=64 time=1.63 ms
80 bytes from 10.1.0.4: icmp_seq=3 ttl=64 time=1.85 ms
80 bytes from 10.1.0.4: icmp_seq=4 ttl=64 time=1.72 ms
80 bytes from 10.1.0.4: icmp_seq=5 ttl=64 time=1.60 ms

--- 10.1.0.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 20ms
rtt min/avg/max/mdev = 1.603/2.147/3.935/0.898 ms, ipg/ewma 5.040/3.008 ms
```


### spine2 to leaf3
```
spine2#ping 10.1.0.5 source 10.1.0.2
PING 10.1.0.5 (10.1.0.5) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.5: icmp_seq=1 ttl=64 time=3.83 ms
80 bytes from 10.1.0.5: icmp_seq=2 ttl=64 time=1.73 ms
80 bytes from 10.1.0.5: icmp_seq=3 ttl=64 time=1.91 ms
80 bytes from 10.1.0.5: icmp_seq=4 ttl=64 time=1.75 ms
80 bytes from 10.1.0.5: icmp_seq=5 ttl=64 time=1.52 ms

--- 10.1.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 17ms
rtt min/avg/max/mdev = 1.517/2.148/3.832/0.851 ms, ipg/ewma 4.148/2.955 ms
```
