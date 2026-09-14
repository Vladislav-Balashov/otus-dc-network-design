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
| `10.1.50.0/24` | **client1** Ethernet0            | `10.1.50.10`  | **leaf1** Ethernet12            |
| `10.1.50.0/24` | **client2** Ethernet0            | `10.1.50.20`  | **leaf2** Ethernet12            |
| `10.1.50.0/24` | **client3** Ethernet0            | `10.1.50.30`  | **leaf3** Ethernet11            |




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
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
!
ip routing
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
vlan 10
   name VLAN10
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
   switchport access vlan 10
!
interface Loopback0
   ip address 10.1.0.4/32
   ip ospf area 0.0.0.0
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
!
ip routing
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
   vlan 10
      rd 10.1.0.4:10
      route-target both 65000:10010
      redistribute learned
   !
   address-family evpn
      neighbor SPINES activate
   !
   address-family ipv4
      no neighbor SPINES activate
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
vlan 10
   name VLAN10
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
   switchport access vlan 10
!
interface Loopback0
   ip address 10.1.0.5/32
   ip ospf area 0.0.0.0
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
!
ip routing
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
   vlan 10
      rd 10.1.0.5:10
      route-target both 65000:10010
      redistribute learned
   !
   address-family evpn
      neighbor SPINES activate
   !
   address-family ipv4
      no neighbor SPINES activate
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

### spine1 evpn summary

```
spine1#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.3 4 65000            105       111    0    0 01:22:43 Estab   2      2
  10.1.0.4 4 65000            104       109    0    0 01:22:43 Estab   2      2
  10.1.0.5 4 65000             60        66    0    0 00:46:09 Estab   2      2
```


<br>

### spine1 route type 2

```
spine1#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.0.3:10 mac-ip 0050.7966.6800
                                 10.1.0.3              -       100     0       i
 * >      RD: 10.1.0.4:10 mac-ip 0050.7966.6801
                                 10.1.0.4              -       100     0       i
 * >      RD: 10.1.0.5:10 mac-ip 0050.7966.6802
                                 10.1.0.5              -       100     0       i

```


<br>

### spine1 route type 3

```
spine1#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.0.3:10 imet 10.1.0.3
                                 10.1.0.3              -       100     0       i
 * >      RD: 10.1.0.4:10 imet 10.1.0.4
                                 10.1.0.4              -       100     0       i
 * >      RD: 10.1.0.5:10 imet 10.1.0.5
                                 10.1.0.5              -       100     0       i
```


<br>

### spine2 evpn summary

```
spine2#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.2, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.3 4 65000            190       191    0    0 02:32:01 Estab   1      1
  10.1.0.4 4 65000            191       191    0    0 02:32:01 Estab   1      1
  10.1.0.5 4 65000            146       149    0    0 01:58:02 Estab   1      1
```


<br>

### spine2 route type 2

```
spine2#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.2, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.0.3:10 mac-ip 0050.7966.6800
                                 10.1.0.3              -       100     0       i
 * >      RD: 10.1.0.4:10 mac-ip 0050.7966.6801
                                 10.1.0.4              -       100     0       i
 * >      RD: 10.1.0.5:10 mac-ip 0050.7966.6802
                                 10.1.0.5              -       100     0       i

```


<br>

### spine2 route type 3

```
spine2#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.1.0.2, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.0.3:10 imet 10.1.0.3
                                 10.1.0.3              -       100     0       i
 * >      RD: 10.1.0.4:10 imet 10.1.0.4
                                 10.1.0.4              -       100     0       i
 * >      RD: 10.1.0.5:10 imet 10.1.0.5
                                 10.1.0.5              -       100     0       i
```

<br>

### leaf1 evpn summary

```
leaf1#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.3, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.1 4 65000            201       194    0    0 02:37:23 Estab   2      2
  10.1.0.2 4 65000            195       194    0    0 02:34:46 Estab   2      2
```


<br>

### leaf1 route type 2

```
leaf1#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.3, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.0.3:10 mac-ip 0050.7966.6800
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.4:10 mac-ip 0050.7966.6801
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.1
 *  ec    RD: 10.1.0.4:10 mac-ip 0050.7966.6801
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.2
 * >Ec    RD: 10.1.0.5:10 mac-ip 0050.7966.6802
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.2
 *  ec    RD: 10.1.0.5:10 mac-ip 0050.7966.6802
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.1
```


<br>

### leaf1 route type 3

```
leaf1#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.1.0.3, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.0.3:10 imet 10.1.0.3
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.4:10 imet 10.1.0.4
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.1
 *  ec    RD: 10.1.0.4:10 imet 10.1.0.4
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.2
 * >Ec    RD: 10.1.0.5:10 imet 10.1.0.5
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.1
 *  ec    RD: 10.1.0.5:10 imet 10.1.0.5
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.2
```



<br>

### leaf1 vteps

```
leaf1#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.1.0.4       flood
10.1.0.5       flood

Total number of remote VTEPS:  2
```



<br>


### leaf2 evpn summary

```
leaf2#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.4, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.1 4 65000            226       218    0    0 02:58:40 Estab   4      4
  10.1.0.2 4 65000            223       221    0    0 02:56:03 Estab   4      4
```


<br>

### leaf2 route type 2

```
leaf2#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.4, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.3:10 mac-ip 0050.7966.6800
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.2
 *  ec    RD: 10.1.0.3:10 mac-ip 0050.7966.6800
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.1
 * >      RD: 10.1.0.4:10 mac-ip 0050.7966.6801
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.5:10 mac-ip 0050.7966.6802
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.2
 *  ec    RD: 10.1.0.5:10 mac-ip 0050.7966.6802
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.1
```


<br>

### leaf2 route type 3

```
leaf2#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.1.0.4, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.3:10 imet 10.1.0.3
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.1
 *  ec    RD: 10.1.0.3:10 imet 10.1.0.3
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.2
 * >      RD: 10.1.0.4:10 imet 10.1.0.4
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.5:10 imet 10.1.0.5
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.1
 *  ec    RD: 10.1.0.5:10 imet 10.1.0.5
                                 10.1.0.5              -       100     0       i Or-ID: 10.1.0.5 C-LST: 10.1.0.2
```


<br>

### leaf2 vteps

```
leaf2#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.1.0.3       flood
10.1.0.5       flood

Total number of remote VTEPS:  2
```


<br>


### leaf3 evpn summary

```
leaf3#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.5, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.1 4 65000            190       181    0    0 02:25:35 Estab   4      4
  10.1.0.2 4 65000            188       180    0    0 02:25:34 Estab   4      4
```


<br>

### leaf3 route type 2

```
leaf3#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.5, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.3:10 mac-ip 0050.7966.6800
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.2
 *  ec    RD: 10.1.0.3:10 mac-ip 0050.7966.6800
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.1
 * >Ec    RD: 10.1.0.4:10 mac-ip 0050.7966.6801
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.2
 *  ec    RD: 10.1.0.4:10 mac-ip 0050.7966.6801
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.1
 * >      RD: 10.1.0.5:10 mac-ip 0050.7966.6802
                                 -                     -       -       0       i
```


<br>

### leaf3 route type 3

```
leaf3#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.1.0.5, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.3:10 imet 10.1.0.3
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.1
 *  ec    RD: 10.1.0.3:10 imet 10.1.0.3
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.2
 * >Ec    RD: 10.1.0.4:10 imet 10.1.0.4
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.1
 *  ec    RD: 10.1.0.4:10 imet 10.1.0.4
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.2
 * >      RD: 10.1.0.5:10 imet 10.1.0.5
                                 -                     -       -       0       i
```

<br>

### leaf3 vteps

```
leaf3#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.1.0.3       flood, unicast
10.1.0.4       flood, unicast

Total number of remote VTEPS:  2
```


<br>

### client1 ping test

```
client1> show ip

NAME        : client1[1]
IP/MASK     : 10.1.50.10/24
GATEWAY     : 10.1.50.1
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 20124
RHOST:PORT  : 127.0.0.1:20125
MTU         : 1500

client1> ping 10.1.50.20 -c 4

84 bytes from 10.1.50.20 icmp_seq=1 ttl=64 time=19.538 ms
84 bytes from 10.1.50.20 icmp_seq=2 ttl=64 time=7.338 ms
84 bytes from 10.1.50.20 icmp_seq=3 ttl=64 time=7.490 ms
84 bytes from 10.1.50.20 icmp_seq=4 ttl=64 time=6.224 ms

client1> ping 10.1.50.30 -c 4

84 bytes from 10.1.50.30 icmp_seq=1 ttl=64 time=8.430 ms
84 bytes from 10.1.50.30 icmp_seq=2 ttl=64 time=7.021 ms
84 bytes from 10.1.50.30 icmp_seq=3 ttl=64 time=6.472 ms
84 bytes from 10.1.50.30 icmp_seq=4 ttl=64 time=7.064 ms
```


<br>

### client2 ping test

```
client2> show ip

NAME        : client2[1]
IP/MASK     : 10.1.50.20/24
GATEWAY     : 10.1.50.1
DNS         :
MAC         : 00:50:79:66:68:01
LPORT       : 20126
RHOST:PORT  : 127.0.0.1:20127
MTU         : 1500

client2> ping 10.1.50.10 -c 4

84 bytes from 10.1.50.10 icmp_seq=1 ttl=64 time=5.976 ms
84 bytes from 10.1.50.10 icmp_seq=2 ttl=64 time=5.468 ms
84 bytes from 10.1.50.10 icmp_seq=3 ttl=64 time=29.592 ms
84 bytes from 10.1.50.10 icmp_seq=4 ttl=64 time=6.370 ms

client2> ping 10.1.50.30 -c 4

84 bytes from 10.1.50.30 icmp_seq=1 ttl=64 time=5.811 ms
84 bytes from 10.1.50.30 icmp_seq=2 ttl=64 time=6.113 ms
84 bytes from 10.1.50.30 icmp_seq=3 ttl=64 time=8.097 ms
84 bytes from 10.1.50.30 icmp_seq=4 ttl=64 time=6.548 ms

```


<br>

### client3 ping test

```
client3> show ip

NAME        : client3[1]
IP/MASK     : 10.1.50.30/24
GATEWAY     : 10.1.50.1
DNS         :
MAC         : 00:50:79:66:68:02
LPORT       : 20154
RHOST:PORT  : 127.0.0.1:20155
MTU         : 1500

client3> ping 10.1.50.10 -c 4

84 bytes from 10.1.50.10 icmp_seq=1 ttl=64 time=6.498 ms
84 bytes from 10.1.50.10 icmp_seq=2 ttl=64 time=56.164 ms
84 bytes from 10.1.50.10 icmp_seq=3 ttl=64 time=6.696 ms
84 bytes from 10.1.50.10 icmp_seq=4 ttl=64 time=6.336 ms

client3> ping 10.1.50.20 -c 4

84 bytes from 10.1.50.20 icmp_seq=1 ttl=64 time=7.966 ms
84 bytes from 10.1.50.20 icmp_seq=2 ttl=64 time=24.026 ms
84 bytes from 10.1.50.20 icmp_seq=3 ttl=64 time=8.681 ms
84 bytes from 10.1.50.20 icmp_seq=4 ttl=64 time=7.530 ms
```
