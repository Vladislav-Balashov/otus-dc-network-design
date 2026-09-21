### 1. Адресное пространство (IPAM)

#### Общие подсети

- **Loopback0:** `10.1.0.0/24` (для Router ID и Loopback-интерфейсов)
    
- **P2P Links (Underlay):** `10.1.16.0/20` (Point-to-Point линки между Spine и Leaf)
    
- **Infra / Services:** `10.1.32.0/19` (Инфраструктура и сервисы)

<br>

#### Loopback0

| **Устройство** | **IP-адрес / маска** |
| -------------- | -------------------- |
| **spine1**     | `10.1.0.1/32`        |
| **spine2**     | `10.1.0.2/32`        |
| **leaf1**      | `10.1.0.3/32`        |
| **leaf2**      | `10.1.0.4/32`        |
| **leaf3**      | `10.1.0.5/32`        |

<br>

#### PtP-подключения

| **Подсеть**     | **Spine устройство & интерфейс** | **IP Spine** | **Leaf устройство & интерфейс** | **IP Leaf**  |
| --------------- | -------------------------------- | ------------ | ------------------------------- | ------------ |
| `10.1.16.0/31`  | **spine1** Ethernet1             | `10.1.16.0`  | **leaf1** Ethernet1             | `10.1.16.1`  |
| `10.1.16.2/31`  | **spine1** Ethernet2             | `10.1.16.2`  | **leaf2** Ethernet1             | `10.1.16.3`  |
| `10.1.16.4/31`  | **spine1** Ethernet3             | `10.1.16.4`  | **leaf3** Ethernet1             | `10.1.16.5`  |
| `10.1.16.6/31`  | **spine2** Ethernet1             | `10.1.16.6`  | **leaf1** Ethernet2             | `10.1.16.7`  |
| `10.1.16.8/31`  | **spine2** Ethernet2             | `10.1.16.8`  | **leaf2** Ethernet2             | `10.1.16.9`  |
| `10.1.16.10/31` | **spine2** Ethernet3             | `10.1.16.10` | **leaf3** Ethernet2             | `10.1.16.11` |

<br>

#### Клиенты

| **Подсеть**    | **Spine устройство & интерфейс** | **IP Client** | **Leaf устройство & интерфейс** |
| -------------- | -------------------------------- | ------------- | ------------------------------- |
| `10.1.40.0/24` | **client1** Ethernet0            | `10.1.40.10`  | **leaf1** Ethernet12            |
| `10.1.50.0/24` | **client2** Ethernet0            | `10.1.50.10`  | **leaf2** Ethernet12            |
| `10.1.60.0/24` | **client3** Ethernet0            | `10.1.60.10`  | **leaf3** Ethernet11            |




<br>


### 2. Конфигурации устройств (Arista vEOS)

### spine1

```
service routing protocols model multi-agent
!
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
router bgp 65000
   router-id 10.1.0.1
   no bgp default ipv4-unicast
   neighbor LEAVES peer group
   neighbor LEAVES remote-as 65000
   neighbor LEAVES update-source Loopback0
   neighbor LEAVES route-reflector-client
   neighbor LEAVES send-community extended
   neighbor LEAVES maximum-routes 0
   neighbor 10.1.0.3 peer group LEAVES
   neighbor 10.1.0.4 peer group LEAVES
   neighbor 10.1.0.5 peer group LEAVES
   !
   address-family evpn
      neighbor LEAVES activate
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
<br>

### spine2

```
service routing protocols model multi-agent
!
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
router bgp 65000
   router-id 10.1.0.2
   no bgp default ipv4-unicast
   neighbor LEAVES peer group
   neighbor LEAVES remote-as 65000
   neighbor LEAVES update-source Loopback0
   neighbor LEAVES route-reflector-client
   neighbor LEAVES send-community extended
   neighbor LEAVES maximum-routes 0
   neighbor 10.1.0.3 peer group LEAVES
   neighbor 10.1.0.4 peer group LEAVES
   neighbor 10.1.0.5 peer group LEAVES
   !
   address-family evpn
      neighbor LEAVES activate
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
<br>

### leaf1

```
service routing protocols model multi-agent
!
hostname leaf1
!
spanning-tree mode mstp
!
vlan 10
   name VLAN10
!
vrf instance TENANT-A
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
interface Ethernet12
   description client1
   switchport access vlan 10
!
interface Loopback0
   ip address 10.1.0.3/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   description Gateway for client1
   vrf TENANT-A
   ip address virtual 10.1.40.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vrf TENANT-A vni 50001
!
ip virtual-router mac-address 00:00:22:22:33:33
!
ip routing
ip routing vrf TENANT-A
!
router bgp 65000
   router-id 10.1.0.3
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 4
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES update-source Loopback0
   neighbor SPINES send-community extended
   neighbor 10.1.0.1 peer group SPINES
   neighbor 10.1.0.2 peer group SPINES
   !
   vlan 10
      rd 10.1.0.3:10
      route-target both 65000:10010
      redistribute learned
   !
   address-family evpn
      neighbor SPINES activate
   !
   address-family ipv4
      no neighbor SPINES activate
   !
   vrf TENANT-A
      rd 10.1.0.3:50001
      route-target import evpn 65000:50001
      route-target export evpn 65000:50001
      router-id 10.1.0.3
      redistribute connected
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
<br>

### leaf2

```
service routing protocols model multi-agent
!
hostname leaf2
!
spanning-tree mode mstp
!
vlan 20
   name VLAN20
!
vrf instance TENANT-A
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
interface Ethernet12
   description client2
   switchport access vlan 20
!
interface Loopback0
   ip address 10.1.0.4/32
   ip ospf area 0.0.0.0
!
interface Vlan20
   description Gateway for client2
   vrf TENANT-A
   ip address virtual 10.1.50.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
   vxlan vrf TENANT-A vni 50001
!
ip virtual-router mac-address 00:00:22:22:33:33
!
ip routing
ip routing vrf TENANT-A
!
router bgp 65000
   router-id 10.1.0.4
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 4
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES update-source Loopback0
   neighbor SPINES send-community extended
   neighbor 10.1.0.1 peer group SPINES
   neighbor 10.1.0.2 peer group SPINES
   !
   vlan 20
      rd 10.1.0.4:20
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor SPINES activate
   !
   address-family ipv4
      no neighbor SPINES activate
   !
   vrf TENANT-A
      rd 10.1.0.4:50001
      route-target import evpn 65000:50001
      route-target export evpn 65000:50001
      router-id 10.1.0.4
      redistribute connected
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
<br>

### leaf3

```
service routing protocols model multi-agent
!
hostname leaf3
!
spanning-tree mode mstp
!
vlan 30
   name VLAN30
!
vrf instance TENANT-A
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
interface Ethernet11
   description client3
   switchport access vlan 30
!
interface Loopback0
   ip address 10.1.0.5/32
   ip ospf area 0.0.0.0
!
interface Vlan30
   description Gateway for client3
   vrf TENANT-A
   ip address virtual 10.1.60.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 30 vni 10030
   vxlan vrf TENANT-A vni 50001
!
ip virtual-router mac-address 00:00:22:22:33:33
!
ip routing
ip routing vrf TENANT-A
!
router bgp 65000
   router-id 10.1.0.5
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 4
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES update-source Loopback0
   neighbor SPINES send-community extended
   neighbor 10.1.0.1 peer group SPINES
   neighbor 10.1.0.2 peer group SPINES
   !
   vlan 30
      rd 10.1.0.5:10030
      route-target both 65000:10010
      redistribute learned
   !
   address-family evpn
      neighbor SPINES activate
   !
   address-family ipv4
      no neighbor SPINES activate
   !
   vrf TENANT-A
      rd 10.1.0.5:50001
      route-target import evpn 65000:50001
      route-target export evpn 65000:50001
      router-id 10.1.0.5
      redistribute connected
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
<br>

### 3. Проверка EVPN и связности


### leaf1 evpn summary

```
leaf1#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.3, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.1 4 65000            314       304    0    0 04:07:43 Estab   4      4
  10.1.0.2 4 65000            305       305    0    0 04:07:16 Estab   4      4
```


<br>

### leaf1 route type 5

```
leaf1#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.0.3, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.0.3:50001 ip-prefix 10.1.40.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.4:50001 ip-prefix 10.1.50.0/24
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.2
 *  ec    RD: 10.1.0.4:50001 ip-prefix 10.1.50.0/24
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.1
 * >Ec    RD: 10.1.0.5:50001 ip-prefix 10.1.60.0/24
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.2
 *  ec    RD: 10.1.0.5:50001 ip-prefix 10.1.60.0/24
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.1

```


<br>

### leaf1 route vrf

```
leaf1#show ip route vrf TENANT-A

VRF: TENANT-A
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

 C        10.1.40.0/24
           directly connected, Vlan10
 B I      10.1.50.0/24 [200/0]
           via VTEP 10.1.0.4 VNI 50001 router-mac 0c:7d:8f:9c:5b:5f local-interface Vxlan1
 B I      10.1.60.0/24 [200/0]
           via VTEP 10.1.0.5 VNI 50001 router-mac 0c:e9:0c:a2:27:b9 local-interface Vxlan1
```


<br>


### leaf2 evpn summary

```
leaf2#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.4, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.1 4 65000            297       281    0    0 03:49:37 Estab   4      4
  10.1.0.2 4 65000            291       282    0    0 03:49:37 Estab   4      4
```


<br>


### leaf2 route type 5

```
leaf2#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.0.4, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.3:50001 ip-prefix 10.1.40.0/24
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.2
 *  ec    RD: 10.1.0.3:50001 ip-prefix 10.1.40.0/24
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.1
 * >      RD: 10.1.0.4:50001 ip-prefix 10.1.50.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.5:50001 ip-prefix 10.1.60.0/24
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.2
 *  ec    RD: 10.1.0.5:50001 ip-prefix 10.1.60.0/24
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.1
```


<br>


### leaf2 route vrf

```
leaf2#show ip route vrf TENANT-A

VRF: TENANT-A
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

 B I      10.1.40.0/24 [200/0]
           via VTEP 10.1.0.3 VNI 50001 router-mac 0c:9c:6c:03:e2:d5 local-interface Vxlan1
 C        10.1.50.0/24
           directly connected, Vlan20
 B I      10.1.60.0/24 [200/0]
           via VTEP 10.1.0.5 VNI 50001 router-mac 0c:e9:0c:a2:27:b9 local-interface Vxlan1
```



<br>


### leaf3 evpn summary

```
leaf3#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.5, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd Pf                                                                                                             xAcc
  10.1.0.1 4 65000             72        59    0    0 00:41:42 Estab   4      4
  10.1.0.2 4 65000             66        59    0    0 00:41:43 Estab   4      4
```


<br>


### leaf3 route type 5

```
leaf3#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.0.5, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.3:50001 ip-prefix 10.1.40.0/24
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.2
 *  ec    RD: 10.1.0.3:50001 ip-prefix 10.1.40.0/24
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.1
 * >Ec    RD: 10.1.0.4:50001 ip-prefix 10.1.50.0/24
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.2
 *  ec    RD: 10.1.0.4:50001 ip-prefix 10.1.50.0/24
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.1
 * >      RD: 10.1.0.5:50001 ip-prefix 10.1.60.0/24
                                 -                     -       -       0       i
```


<br>


### leaf3 route vrf

```
leaf3#show ip route vrf TENANT-A

VRF: TENANT-A
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

 B I      10.1.40.0/24 [200/0]
           via VTEP 10.1.0.3 VNI 50001 router-mac 0c:9c:6c:03:e2:d5 local-interface Vxlan1
 B I      10.1.50.0/24 [200/0]
           via VTEP 10.1.0.4 VNI 50001 router-mac 0c:7d:8f:9c:5b:5f local-interface Vxlan1
 C        10.1.60.0/24
           directly connected, Vlan30
```


<br>


### client1 ping test

```
client1> show ip

NAME        : client1[1]
IP/MASK     : 10.1.40.10/24
GATEWAY     : 10.1.40.1
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 20124
RHOST:PORT  : 127.0.0.1:20125
MTU         : 1500

client1> ping 10.1.40.1 -c 4

84 bytes from 10.1.40.1 icmp_seq=1 ttl=64 time=2.978 ms
84 bytes from 10.1.40.1 icmp_seq=2 ttl=64 time=2.557 ms
84 bytes from 10.1.40.1 icmp_seq=3 ttl=64 time=2.220 ms
84 bytes from 10.1.40.1 icmp_seq=4 ttl=64 time=3.039 ms

client1> ping 10.1.50.10 -c 4

84 bytes from 10.1.50.10 icmp_seq=1 ttl=62 time=44.905 ms
84 bytes from 10.1.50.10 icmp_seq=2 ttl=62 time=9.628 ms
84 bytes from 10.1.50.10 icmp_seq=3 ttl=62 time=9.370 ms
84 bytes from 10.1.50.10 icmp_seq=4 ttl=62 time=11.815 ms

client1> ping 10.1.60.10 -c 4

84 bytes from 10.1.60.10 icmp_seq=1 ttl=62 time=39.346 ms
84 bytes from 10.1.60.10 icmp_seq=2 ttl=62 time=9.574 ms
84 bytes from 10.1.60.10 icmp_seq=3 ttl=62 time=8.218 ms
84 bytes from 10.1.60.10 icmp_seq=4 ttl=62 time=10.564 ms
```


<br>


### client2 ping test

```
client2> show ip

NAME        : client2[1]
IP/MASK     : 10.1.50.10/24
GATEWAY     : 10.1.50.1
DNS         :
MAC         : 00:50:79:66:68:01
LPORT       : 20126
RHOST:PORT  : 127.0.0.1:20127
MTU         : 1500

client2> ping 10.1.50.1 -c 4

84 bytes from 10.1.50.1 icmp_seq=1 ttl=64 time=7.739 ms
84 bytes from 10.1.50.1 icmp_seq=2 ttl=64 time=2.603 ms
84 bytes from 10.1.50.1 icmp_seq=3 ttl=64 time=2.545 ms
84 bytes from 10.1.50.1 icmp_seq=4 ttl=64 time=3.544 ms

client2> ping 10.1.40.10 -c 4

84 bytes from 10.1.40.10 icmp_seq=1 ttl=62 time=22.144 ms
84 bytes from 10.1.40.10 icmp_seq=2 ttl=62 time=7.571 ms
84 bytes from 10.1.40.10 icmp_seq=3 ttl=62 time=9.139 ms
84 bytes from 10.1.40.10 icmp_seq=4 ttl=62 time=10.216 ms

client2> ping 10.1.60.10 -c 4

84 bytes from 10.1.60.10 icmp_seq=1 ttl=62 time=37.524 ms
84 bytes from 10.1.60.10 icmp_seq=2 ttl=62 time=12.103 ms
84 bytes from 10.1.60.10 icmp_seq=3 ttl=62 time=9.378 ms
84 bytes from 10.1.60.10 icmp_seq=4 ttl=62 time=9.158 ms
```


<br>


### client3 ping test

```
client3> show ip

NAME        : client3[1]
IP/MASK     : 10.1.60.10/24
GATEWAY     : 10.1.60.1
DNS         :
MAC         : 00:50:79:66:68:02
LPORT       : 20154
RHOST:PORT  : 127.0.0.1:20155
MTU         : 1500

client3> ping 10.1.60.1 -c 4

84 bytes from 10.1.60.1 icmp_seq=1 ttl=64 time=2.479 ms
84 bytes from 10.1.60.1 icmp_seq=2 ttl=64 time=2.770 ms
84 bytes from 10.1.60.1 icmp_seq=3 ttl=64 time=2.506 ms
84 bytes from 10.1.60.1 icmp_seq=4 ttl=64 time=1.938 ms

client3> ping 10.1.40.10 -c 4

84 bytes from 10.1.40.10 icmp_seq=1 ttl=62 time=23.166 ms
84 bytes from 10.1.40.10 icmp_seq=2 ttl=62 time=7.247 ms
84 bytes from 10.1.40.10 icmp_seq=3 ttl=62 time=6.577 ms
84 bytes from 10.1.40.10 icmp_seq=4 ttl=62 time=8.923 ms

client3> ping 10.1.50.10 -c 4

84 bytes from 10.1.50.10 icmp_seq=1 ttl=62 time=21.708 ms
84 bytes from 10.1.50.10 icmp_seq=2 ttl=62 time=10.900 ms
84 bytes from 10.1.50.10 icmp_seq=3 ttl=62 time=7.041 ms
84 bytes from 10.1.50.10 icmp_seq=4 ttl=62 time=8.853 ms
```
