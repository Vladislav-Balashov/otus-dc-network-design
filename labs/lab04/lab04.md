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
!
interface Ethernet2
   description PtP to leaf2
   mtu 9194
   no switchport
   ip address 10.1.16.2/31
!
interface Ethernet3
   description PtP to leaf3
   mtu 9194
   no switchport
   ip address 10.1.16.4/31
!
interface Loopback0
   ip address 10.1.0.1/32
!
ip routing
!
route-map RM_REDIS_CON permit 10
   match interface Loopback0
!
router bgp 65000
   router-id 10.1.0.1
   maximum-paths 10 ecmp 10
   neighbor LEAVES peer group
   neighbor LEAVES bfd
   neighbor LEAVES timers 3 9
   neighbor 10.1.16.1 peer group LEAVES
   neighbor 10.1.16.1 remote-as 65001
   neighbor 10.1.16.3 peer group LEAVES
   neighbor 10.1.16.3 remote-as 65002
   neighbor 10.1.16.5 peer group LEAVES
   neighbor 10.1.16.5 remote-as 65003
   !
   address-family ipv4
      redistribute connected route-map RM_REDIS_CON
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
!
interface Ethernet2
   description PtP to leaf2
   mtu 9194
   no switchport
   ip address 10.1.16.8/31
!
interface Ethernet3
   description PtP to leaf3
   mtu 9194
   no switchport
   ip address 10.1.16.10/31
!
interface Loopback0
   ip address 10.1.0.2/32
!
ip routing
!
router bgp 65000
   router-id 10.1.0.2
   maximum-paths 10 ecmp 10
   neighbor LEAVES peer group
   neighbor LEAVES bfd
   neighbor LEAVES timers 3 9
   neighbor 10.1.16.7 peer group LEAVES
   neighbor 10.1.16.7 remote-as 65001
   neighbor 10.1.16.9 peer group LEAVES
   neighbor 10.1.16.9 remote-as 65002
   neighbor 10.1.16.11 peer group LEAVES
   neighbor 10.1.16.11 remote-as 65003
   !
   address-family ipv4
      redistribute connected route-map RM_REDIS_CON
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
!
interface Ethernet2
   description PtP to spine2
   mtu 9194
   no switchport
   ip address 10.1.16.7/31
!
interface Loopback0
   ip address 10.1.0.3/32
!
ip routing
!
router bgp 65001
   router-id 10.1.0.3
   maximum-paths 10 ecmp 10
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES bfd
   neighbor SPINES timers 3 9
   neighbor 10.1.16.0 peer group SPINES
   neighbor 10.1.16.6 peer group SPINES
   !
   address-family ipv4
      redistribute connected route-map RM_REDIS_CON
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
!
interface Ethernet2
   description PtP to spine2
   mtu 9194
   no switchport
   ip address 10.1.16.9/31
!
interface Loopback0
   ip address 10.1.0.4/32
!
ip routing
!
router bgp 65002
   router-id 10.1.0.4
   maximum-paths 10 ecmp 10
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES bfd
   neighbor SPINES timers 3 9
   neighbor 10.1.16.2 peer group SPINES
   neighbor 10.1.16.8 peer group SPINES
   !
   address-family ipv4
      redistribute connected route-map RM_REDIS_CON
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
!
interface Ethernet2
   description PtP to spine2
   mtu 9194
   no switchport
   ip address 10.1.16.11/31
!
interface Loopback0
   ip address 10.1.0.5/32
!
ip routing
!
router bgp 65003
   router-id 10.1.0.5
   maximum-paths 10 ecmp 10
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES bfd
   neighbor SPINES timers 3 9
   neighbor 10.1.16.4 peer group SPINES
   neighbor 10.1.16.10 peer group SPINES
   !
   address-family ipv4
      redistribute connected route-map RM_REDIS_CON
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

### 3. Проверка BGP и L3-связности

### spine1 bgp sessions

```
spine1#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.1.0.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor  V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.16.1 4 65001           4440      4436    0    0 00:10:13 Estab   1      1
  10.1.16.3 4 65002            447       456    0    0 00:10:13 Estab   1      1
  10.1.16.5 4 65003            349       349    0    0 00:10:13 Estab   1      1
```


<br>

### spine2 bgp sessions

```
spine2#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.1.0.2, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor   V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.16.7  4 65001            871       876    0    0 00:13:41 Estab   1      1
  10.1.16.9  4 65002            527       521    0    0 00:13:41 Estab   1      1
  10.1.16.11 4 65003            426       429    0    0 00:13:41 Estab   1      1
```


<br>

### spine1 to leaf1

```
spine1#ping 10.1.0.3 source 10.1.0.1
PING 10.1.0.3 (10.1.0.3) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.3: icmp_seq=1 ttl=64 time=6.34 ms
80 bytes from 10.1.0.3: icmp_seq=2 ttl=64 time=2.06 ms
80 bytes from 10.1.0.3: icmp_seq=3 ttl=64 time=2.25 ms
80 bytes from 10.1.0.3: icmp_seq=4 ttl=64 time=2.03 ms
80 bytes from 10.1.0.3: icmp_seq=5 ttl=64 time=1.75 ms

--- 10.1.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 26ms
rtt min/avg/max/mdev = 1.750/2.885/6.339/1.733 ms, ipg/ewma 6.431/4.544 ms

```

<br>

### spine1 to leaf2

```
spine1#ping 10.1.0.4 source 10.1.0.1
PING 10.1.0.4 (10.1.0.4) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.4: icmp_seq=1 ttl=64 time=4.72 ms
80 bytes from 10.1.0.4: icmp_seq=2 ttl=64 time=1.58 ms
80 bytes from 10.1.0.4: icmp_seq=3 ttl=64 time=1.53 ms
80 bytes from 10.1.0.4: icmp_seq=4 ttl=64 time=1.45 ms
80 bytes from 10.1.0.4: icmp_seq=5 ttl=64 time=2.09 ms

--- 10.1.0.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 20ms
rtt min/avg/max/mdev = 1.448/2.273/4.718/1.242 ms, ipg/ewma 4.968/3.464 ms
```

<br>

### spine1 to leaf3

```
spine1#ping 10.1.0.5 source 10.1.0.1
PING 10.1.0.5 (10.1.0.5) from 10.1.0.1 : 72(100) bytes of data.
80 bytes from 10.1.0.5: icmp_seq=1 ttl=64 time=3.65 ms
80 bytes from 10.1.0.5: icmp_seq=2 ttl=64 time=2.02 ms
80 bytes from 10.1.0.5: icmp_seq=3 ttl=64 time=2.20 ms
80 bytes from 10.1.0.5: icmp_seq=4 ttl=64 time=2.71 ms
80 bytes from 10.1.0.5: icmp_seq=5 ttl=64 time=1.69 ms

--- 10.1.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 21ms
rtt min/avg/max/mdev = 1.690/2.453/3.652/0.684 ms, ipg/ewma 5.242/3.027 ms
```


<br>

### spine2 to leaf1

```
spine2#ping 10.1.0.3 source 10.1.0.2
PING 10.1.0.3 (10.1.0.3) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.3: icmp_seq=1 ttl=64 time=6.60 ms
80 bytes from 10.1.0.3: icmp_seq=2 ttl=64 time=1.85 ms
80 bytes from 10.1.0.3: icmp_seq=3 ttl=64 time=1.64 ms
80 bytes from 10.1.0.3: icmp_seq=4 ttl=64 time=1.58 ms
80 bytes from 10.1.0.3: icmp_seq=5 ttl=64 time=1.99 ms

--- 10.1.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 25ms
rtt min/avg/max/mdev = 1.577/2.732/6.604/1.941 ms, ipg/ewma 6.242/4.604 ms
```

<br>

### spine2 to leaf2

```
spine2#ping 10.1.0.4 source 10.1.0.2
PING 10.1.0.4 (10.1.0.4) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.4: icmp_seq=1 ttl=64 time=4.39 ms
80 bytes from 10.1.0.4: icmp_seq=2 ttl=64 time=1.80 ms
80 bytes from 10.1.0.4: icmp_seq=3 ttl=64 time=1.65 ms
80 bytes from 10.1.0.4: icmp_seq=4 ttl=64 time=1.85 ms
80 bytes from 10.1.0.4: icmp_seq=5 ttl=64 time=1.61 ms

--- 10.1.0.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 18ms
rtt min/avg/max/mdev = 1.609/2.258/4.386/1.067 ms, ipg/ewma 4.568/3.283 ms
```

<br>

### spine2 to leaf3

```
spine2#ping 10.1.0.5 source 10.1.0.2
PING 10.1.0.5 (10.1.0.5) from 10.1.0.2 : 72(100) bytes of data.
80 bytes from 10.1.0.5: icmp_seq=1 ttl=64 time=5.65 ms
80 bytes from 10.1.0.5: icmp_seq=2 ttl=64 time=1.65 ms
80 bytes from 10.1.0.5: icmp_seq=3 ttl=64 time=1.58 ms
80 bytes from 10.1.0.5: icmp_seq=4 ttl=64 time=3.18 ms
80 bytes from 10.1.0.5: icmp_seq=5 ttl=64 time=1.59 ms

--- 10.1.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 23ms
rtt min/avg/max/mdev = 1.578/2.727/5.650/1.583 ms, ipg/ewma 5.648/4.146 ms
```


<br>

### leaf1 to leaf2

```
leaf1#ping 10.1.0.4 source 10.1.0.3
PING 10.1.0.4 (10.1.0.4) from 10.1.0.3 : 72(100) bytes of data.
80 bytes from 10.1.0.4: icmp_seq=1 ttl=63 time=6.68 ms
80 bytes from 10.1.0.4: icmp_seq=2 ttl=63 time=4.15 ms
80 bytes from 10.1.0.4: icmp_seq=3 ttl=63 time=4.51 ms
80 bytes from 10.1.0.4: icmp_seq=4 ttl=63 time=4.06 ms
80 bytes from 10.1.0.4: icmp_seq=5 ttl=63 time=3.48 ms

--- 10.1.0.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 39ms
rtt min/avg/max/mdev = 3.479/4.575/6.677/1.101 ms, ipg/ewma 9.844/5.572 ms
```


<br>

### leaf1 to leaf3

```
leaf1#ping 10.1.0.5 source 10.1.0.3
PING 10.1.0.5 (10.1.0.5) from 10.1.0.3 : 72(100) bytes of data.
80 bytes from 10.1.0.5: icmp_seq=1 ttl=63 time=9.41 ms
80 bytes from 10.1.0.5: icmp_seq=2 ttl=63 time=10.0 ms
80 bytes from 10.1.0.5: icmp_seq=3 ttl=63 time=5.44 ms
80 bytes from 10.1.0.5: icmp_seq=4 ttl=63 time=5.57 ms
80 bytes from 10.1.0.5: icmp_seq=5 ttl=63 time=8.86 ms

--- 10.1.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 53ms
rtt min/avg/max/mdev = 5.440/7.862/10.027/1.960 ms, ipg/ewma 13.133/8.595 ms
```


<br>

### leaf2 to leaf3

```
leaf2#ping 10.1.0.5 source 10.1.0.4
PING 10.1.0.5 (10.1.0.5) from 10.1.0.4 : 72(100) bytes of data.
80 bytes from 10.1.0.5: icmp_seq=1 ttl=63 time=8.01 ms
80 bytes from 10.1.0.5: icmp_seq=2 ttl=63 time=4.02 ms
80 bytes from 10.1.0.5: icmp_seq=3 ttl=63 time=3.32 ms
80 bytes from 10.1.0.5: icmp_seq=4 ttl=63 time=3.91 ms
80 bytes from 10.1.0.5: icmp_seq=5 ttl=63 time=3.69 ms

--- 10.1.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 32ms
rtt min/avg/max/mdev = 3.321/4.590/8.006/1.724 ms, ipg/ewma 8.047/6.236 ms
```

<br>


### leaf1 routing table

```
leaf1#show ip route

VRF: default
Source Codes:
       C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route,
       CL - CBF Leaked Route

Gateway of last resort is not set

 B E      10.1.0.1/32 [200/0]
           via 10.1.16.0, Ethernet1
 B E      10.1.0.2/32 [200/0]
           via 10.1.16.6, Ethernet2
 C        10.1.0.3/32
           directly connected, Loopback0
 B E      10.1.0.4/32 [200/0]
           via 10.1.16.0, Ethernet1
           via 10.1.16.6, Ethernet2
 B E      10.1.0.5/32 [200/0]
           via 10.1.16.0, Ethernet1
           via 10.1.16.6, Ethernet2
 C        10.1.16.0/31
           directly connected, Ethernet1
 C        10.1.16.6/31
           directly connected, Ethernet2
```


<br>


### leaf2 routing table

```
leaf2#show ip route

VRF: default
Source Codes:
       C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route,
       CL - CBF Leaked Route

Gateway of last resort is not set

 B E      10.1.0.1/32 [200/0]
           via 10.1.16.2, Ethernet1
 B E      10.1.0.2/32 [200/0]
           via 10.1.16.8, Ethernet2
 B E      10.1.0.3/32 [200/0]
           via 10.1.16.2, Ethernet1
           via 10.1.16.8, Ethernet2
 C        10.1.0.4/32
           directly connected, Loopback0
 B E      10.1.0.5/32 [200/0]
           via 10.1.16.2, Ethernet1
           via 10.1.16.8, Ethernet2
 C        10.1.16.2/31
           directly connected, Ethernet1
 C        10.1.16.8/31
           directly connected, Ethernet2
```

<br>


### leaf3 routing table

```
leaf3#show ip route

VRF: default
Source Codes:
       C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route,
       CL - CBF Leaked Route

Gateway of last resort is not set

 B E      10.1.0.1/32 [200/0]
           via 10.1.16.4, Ethernet1
 B E      10.1.0.2/32 [200/0]
           via 10.1.16.10, Ethernet2
 B E      10.1.0.3/32 [200/0]
           via 10.1.16.4, Ethernet1
           via 10.1.16.10, Ethernet2
 B E      10.1.0.4/32 [200/0]
           via 10.1.16.4, Ethernet1
           via 10.1.16.10, Ethernet2
 C        10.1.0.5/32
           directly connected, Loopback0
 C        10.1.16.4/31
           directly connected, Ethernet1
 C        10.1.16.10/31
           directly connected, Ethernet2
```
