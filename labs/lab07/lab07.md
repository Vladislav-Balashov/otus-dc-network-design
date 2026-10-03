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

| **Подсеть**    | **Spine устройство & интерфейс** | **IP Client** | **Leaf устройство & интерфейс**             |
| -------------- | -------------------------------- | ------------- | ------------------------------------------- |
| `10.1.50.0/24` | **client1** Ethernet1,Ethernet2  | `10.1.50.10`  | **leaf1** Ethernet12, **leaf2** Etherhent12 |
| `10.1.60.0/24` | **client3** Ethernet0            | `10.1.60.10`  | **leaf3** Ethernet11                        |




<br>


### 2. Конфигурации устройств (Arista vEOS)

### spine1

```
! Command: show running-config
! device: spine1 (vEOS-lab, EOS-4.33.1.1F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
username admin role network-admin secret sha512 $6$7ikBP6ofViD9b320$oWLQlO.k3kMwqQfzdlWlav7Cjn8dJ3hssfCfVHf1RDk/uypkzzirlvp.1Esskld2jnI3KZnaf6uIWTat3hrBl.
!
no service interface inactive port-id allocation disabled
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname spine1
!
spanning-tree mode mstp
!
system l1
   unsupported speed action error
   unsupported error-correction action error
!
interface Ethernet1
   description 'PtP to leaf1'
   mtu 9194
   no switchport
   ip address 10.1.16.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
!
interface Ethernet2
   description 'PtP to leaf2'
   mtu 9194
   no switchport
   ip address 10.1.16.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
!
interface Ethernet3
   description 'PtP to leaf3'
   mtu 9194
   no switchport
   ip address 10.1.16.4/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Ethernet9
!
interface Ethernet10
!
interface Ethernet11
!
interface Ethernet12
!
interface Loopback0
   ip address 10.1.0.1/32
   ip ospf area 0.0.0.0
!
interface Management1
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
! Command: show running-config
! device: spine2 (vEOS-lab, EOS-4.33.1.1F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
username admin role network-admin secret sha512 $6$Ozaoig6kk3YWMtsX$w5PJVDjBatCNvUJfYuw/IDPqH3wRRuP08G4vWLfF0DQXZYIil7hnSYNCHD0VnhvDcWqyfhwVAvPkhDb4YL3lW/
!
no service interface inactive port-id allocation disabled
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname spine2
!
spanning-tree mode mstp
!
system l1
   unsupported speed action error
   unsupported error-correction action error
!
interface Ethernet1
   description 'PtP to leaf1'
   mtu 9194
   no switchport
   ip address 10.1.16.6/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
!
interface Ethernet2
   description 'PtP to leaf2'
   mtu 9194
   no switchport
   ip address 10.1.16.8/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
!
interface Ethernet3
   description 'PtP to leaf3'
   mtu 9194
   no switchport
   ip address 10.1.16.10/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Ethernet9
!
interface Ethernet10
!
interface Ethernet11
!
interface Ethernet12
!
interface Loopback0
   ip address 10.1.0.2/32
   ip ospf area 0.0.0.0
!
interface Management1
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
! Command: show running-config
! device: leaf1 (vEOS-lab, EOS-4.33.1.1F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
username admin role network-admin secret sha512 $6$WOSdtlDEfLbfSrrz$m3Y0N8fbHA/TCANKsGrr/rVw/A1/azvFwDLe8T93QV.BQ5USnvBNw/7u4hyc1bk5K9xRVXa4c8ntAWxqxS/kY/
!
no service interface inactive port-id allocation disabled
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
link tracking group CORE-TRACKING
   recovery delay 1
!
hostname leaf1
!
spanning-tree mode mstp
!
system l1
   unsupported speed action error
   unsupported error-correction action error
!
vlan 20
   name VLAN20
!
vrf instance TENANT-A
!
interface Port-Channel1
   description 'Link to client1'
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:0001
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:00:01
   lacp system-id 1111.2222.3333
!
interface Ethernet1
   description 'PtP to spine1'
   mtu 9194
   no switchport
   ip address 10.1.16.1/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
   link tracking group CORE-TRACKING upstream
!
interface Ethernet2
   description 'PtP to spine2'
   mtu 9194
   no switchport
   ip address 10.1.16.7/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
   link tracking group CORE-TRACKING upstream
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Ethernet9
!
interface Ethernet10
!
interface Ethernet11
!
interface Ethernet12
   description 'Link to client1'
   switchport mode trunk
   channel-group 1 mode active
   link tracking group CORE-TRACKING downstream
!
interface Loopback0
   ip address 10.1.0.3/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan20
   description 'Gateway for client1'
   vrf TENANT-A
   ip address virtual 10.1.50.1/24
   ip virtual-router address 10.1.50.254
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
   vxlan vrf TENANT-A vni 50001
!
ip virtual-router mac-address 02:00:00:00:00:00
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
   vlan 20
      rd 10.1.0.3:10020
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
! Command: show running-config
! device: leaf2 (vEOS-lab, EOS-4.33.1.1F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
username admin role network-admin secret sha512 $6$W.8Bw2BpyjGiyz8r$46FChKw24YtZ55Mq/OJxBLYOKtjO8/cHa278bencLA3StDorvv966ux6muZbPX34.QINM9imWLjKM5jTFYMIM/
!
no service interface inactive port-id allocation disabled
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
link tracking group CORE-TRACKING
   recovery delay 1
!
hostname leaf2
!
spanning-tree mode mstp
!
system l1
   unsupported speed action error
   unsupported error-correction action error
!
vlan 20
   name VLAN20
!
vrf instance TENANT-A
!
interface Port-Channel1
   description 'Link to client1'
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:0001
      designated-forwarder election algorithm preference 50
      route-target import 00:00:00:00:00:01
   lacp system-id 1111.2222.3333
!
interface Ethernet1
   description 'PtP to spine1'
   mtu 9194
   no switchport
   ip address 10.1.16.3/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
   link tracking group CORE-TRACKING upstream
!
interface Ethernet2
   description 'PtP to spine2'
   mtu 9194
   no switchport
   ip address 10.1.16.9/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
   link tracking group CORE-TRACKING upstream
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Ethernet9
!
interface Ethernet10
!
interface Ethernet11
!
interface Ethernet12
   description 'Link to client1'
   switchport mode trunk
   channel-group 1 mode active
   link tracking group CORE-TRACKING downstream
!
interface Loopback0
   ip address 10.1.0.4/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan20
   description 'Gateway for client1'
   vrf TENANT-A
   ip address virtual 10.1.50.1/24
   ip virtual-router address 10.1.50.254
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
   vxlan vrf TENANT-A vni 50001
!
ip virtual-router mac-address 02:00:00:00:00:00
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
      rd 10.1.0.4:10020
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
! Command: show running-config
! device: leaf3 (vEOS-lab, EOS-4.33.1.1F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
username admin role network-admin secret sha512 $6$SAWDJyEmubwCOOyQ$QokSPBc3WdvAq2.M.ETzqvAHxhSnXG6OmPiExOzzGEyy2v6y0RFc/C.pO2O77Oggv7lz/aPZbCDlqy1rAAPdS.
!
no service interface inactive port-id allocation disabled
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname leaf3
!
spanning-tree mode mstp
!
system l1
   unsupported speed action error
   unsupported error-correction action error
!
vlan 30
   name VLAN30
!
vrf instance TENANT-A
!
interface Ethernet1
   description 'PtP to spine1'
   mtu 9194
   no switchport
   ip address 10.1.16.5/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
!
interface Ethernet2
   description 'PtP to spine2'
   mtu 9194
   no switchport
   ip address 10.1.16.11/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   no isis bfd
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Ethernet9
!
interface Ethernet10
!
interface Ethernet11
   description 'Link to client3'
   switchport access vlan 30
!
interface Ethernet12
   description 'Link to client4'
   switchport access vlan 30
!
interface Loopback0
   ip address 10.1.0.5/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan30
   description 'Gateway for client3'
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
      route-target both 65000:10030
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

### 3. Тесты


### leaf1 port-channel

```
leaf1#show port-channel detailed
Port Channel Port-Channel1 (Fallback State: Unconfigured):
Minimum links: unconfigured
Minimum speed: unconfigured
Current weight/Max weight: 1/16
  Active Ports:
     Port          Time Became Active    Protocol     Mode       Weight   State
    ------------- --------------------- ----------- ---------- ---------- -----
     Ethernet12    16:10:27              LACP         Active       1      Rx,Tx
```


<br>

### leaf1 route type 1

```
leaf1#show bgp evpn route-type auto-discovery
BGP routing table information for VRF default
Router identifier 10.1.0.3, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.4:10020 auto-discovery 0 0000:0000:0000:0000:0001
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.1
 *  ec    RD: 10.1.0.4:10020 auto-discovery 0 0000:0000:0000:0000:0001
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.2
 * >Ec    RD: 10.1.0.4:1 auto-discovery 0000:0000:0000:0000:0001
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.1
 *  ec    RD: 10.1.0.4:1 auto-discovery 0000:0000:0000:0000:0001
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.2
```


<br>

### leaf1 route type 4

```
leaf1#show bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.1.0.3, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.4:1 ethernet-segment 0000:0000:0000:0000:0001 10.1.0.4
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.1
 *  ec    RD: 10.1.0.4:1 ethernet-segment 0000:0000:0000:0000:0001 10.1.0.4
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.2
```


<br>


### leaf1 core-tracking

```
leaf1#show link tracking group detail
Link State Group: CORE-TRACKING Status: up
Upstream Interfaces : Ethernet2 Ethernet1
Downstream Interfaces : Ethernet12
Number of times disabled : 0
Last disabled never
```


<br>


### leaf2 port-channel

```
leaf2#show port-channel detailed
Port Channel Port-Channel1 (Fallback State: Unconfigured):
Minimum links: unconfigured
Minimum speed: unconfigured
Current weight/Max weight: 1/16
  Active Ports:
     Port          Time Became Active    Protocol     Mode       Weight   State
    ------------- --------------------- ----------- ---------- ---------- -----
     Ethernet12    16:08:37              LACP         Active       1      Rx,Tx
```


<br>

### leaf2 route type 1

```
leaf2#show bgp evpn route-type auto-discovery
BGP routing table information for VRF default
Router identifier 10.1.0.4, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.0.4:10020 auto-discovery 0 0000:0000:0000:0000:0001
                                 -                     -       -       0       i
 * >      RD: 10.1.0.4:1 auto-discovery 0000:0000:0000:0000:0001
                                 -                     -       -       0       i
```


<br>

### leaf2 route type 4

```
leaf2#show bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.1.0.4, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.0.4:1 ethernet-segment 0000:0000:0000:0000:0001 10.1.0.4
                                 -                     -       -       0       i
```



<br>

### leaf2 core-tracking

```
leaf2#show link tracking group detail
Link State Group: CORE-TRACKING Status: up
Upstream Interfaces : Ethernet2 Ethernet1
Downstream Interfaces : Ethernet12
Number of times disabled : 0
Last disabled never
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
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Next                                                                                                hop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.4:50001 ip-prefix 10.1.50.0/24
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.                                                                                                0.4 C-LST: 10.1.0.1
 *  ec    RD: 10.1.0.4:50001 ip-prefix 10.1.50.0/24
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.                                                                                                0.4 C-LST: 10.1.0.2
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

 B I      10.1.50.10/32 [200/0]
           via VTEP 10.1.0.4 VNI 50001 router-mac 0c:7d:8f:9c:5b:5f local-interface Vxlan1
 B I      10.1.50.0/24 [200/0]
           via VTEP 10.1.0.4 VNI 50001 router-mac 0c:7d:8f:9c:5b:5f local-interface Vxlan1
 C        10.1.60.0/24
           directly connected, Vlan30
```


<br>

### client1 bonding

```
[admin@client1] > /interface/bonding/monitor-slaves bond1
Flags: A - active; P - partner
 AP port=ether1 key=15 flags="A-GSCD--" partner-sys-id=11:11:22:22:33:33
     partner-sys-priority=32768 partner-key=1 partner-flags="A-GSCD--"

 AP port=ether2 key=15 flags="A-GSCD--" partner-sys-id=11:11:22:22:33:33
     partner-sys-priority=32768 partner-key=1 partner-flags="A-GSCD--"
```


<br>

### client1 ping

```
[admin@client1] > /ip address/print
Columns: ADDRESS, NETWORK, INTERFACE
# ADDRESS        NETWORK    INTERFACE
0 10.1.50.10/24  10.1.50.0  vlan20-bond1
[admin@client1] > /ip route print
Flags: D - DYNAMIC; A - ACTIVE; c - CONNECT, s - STATIC
Columns: DST-ADDRESS, GATEWAY, ROUTING-TABLE, DISTANCE
#     DST-ADDRESS   GATEWAY       ROUTING-TABLE  DISTANCE
0  As 0.0.0.0/0     10.1.50.254   main                  1
  DAc 10.1.50.0/24  vlan20-bond1  main                  0
[admin@client1] > ping 10.1.50.254 count=4
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 10.1.50.254                                56  64 3ms181us
    1 10.1.50.254                                56  64 1ms799us
    2 10.1.50.254                                56  64 1ms904us
    3 10.1.50.254                                56  64 2ms31us
    sent=4 received=4 packet-loss=0% min-rtt=1ms799us avg-rtt=2ms228us
   max-rtt=3ms181us

[admin@client1] > ping 10.1.60.10 count=4
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 10.1.60.10                                 56  62 26ms377us
    1 10.1.60.10                                 56  62 10ms541us
    2 10.1.60.10                                 56  62 7ms859us
    3 10.1.60.10                                 56  62 7ms186us
    sent=4 received=4 packet-loss=0% min-rtt=7ms186us avg-rtt=12ms990us
   max-rtt=26ms377us
```


<br>

### client3 ping

```
client3> show ip

NAME        : client3[1]
IP/MASK     : 10.1.60.10/24
GATEWAY     : 10.1.60.1
DNS         :
MAC         : 00:50:79:66:68:02
LPORT       : 20142
RHOST:PORT  : 127.0.0.1:20143
MTU         : 1500

client3> ping 10.1.60.1 -c 4

84 bytes from 10.1.60.1 icmp_seq=1 ttl=64 time=8.563 ms
84 bytes from 10.1.60.1 icmp_seq=2 ttl=64 time=2.797 ms
84 bytes from 10.1.60.1 icmp_seq=3 ttl=64 time=11.471 ms
84 bytes from 10.1.60.1 icmp_seq=4 ttl=64 time=1.883 ms

client3> ping 10.1.50.10 -c 4

84 bytes from 10.1.50.10 icmp_seq=1 ttl=62 time=23.261 ms
84 bytes from 10.1.50.10 icmp_seq=2 ttl=62 time=8.586 ms
84 bytes from 10.1.50.10 icmp_seq=3 ttl=62 time=8.596 ms
84 bytes from 10.1.50.10 icmp_seq=4 ttl=62 time=8.849 ms
```


<br>

### leaf1 core-tracking test

```
leaf1#show interfaces Ethernet 1,2 status
Port       Name            Status       Vlan     Duplex Speed  Type            Flags Encapsulation
Et1        'PtP to spine1' notconnect   routed   full   1G     EbraTestPhyPort
Et2        'PtP to spine2' notconnect   routed   full   1G     EbraTestPhyPort

leaf1#show port-channel detailed
Port Channel Port-Channel1 (Fallback State: Unconfigured):
Minimum links: unconfigured
Minimum speed: unconfigured
Current weight/Max weight: 0/16
  No Active Ports
  Configured, but inactive ports:
       Port             Time Became Inactive    Reason
    ---------------- -------------------------- -----------------------------
       Ethernet12       18:12:21                link down in LACP negotiation

leaf1#show link tracking group detail
Link State Group: CORE-TRACKING Status: down
Upstream Interfaces : Ethernet2 Ethernet1
Downstream Interfaces : Ethernet12
Number of times disabled : 1
Last disabled 0:05:38 ago
```


<br>

### client1 core-tracking test

```
[admin@client1] > /interface/bonding/monitor-slaves bond1
Flags: A - active; P - partner
    port=ether1 key=15 flags="A-G---F-"

 AP port=ether2 key=15 flags="A-GSCD--" partner-sys-id=11:11:22:22:33:33
     partner-sys-priority=32768 partner-key=1 partner-flags="A-GSCD--"

[admin@client1] > ping 10.1.50.254 count=4
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 10.1.50.254                                56  64 3ms326us
    1 10.1.50.254                                56  64 2ms69us
    2 10.1.50.254                                56  64 2ms497us
    3 10.1.50.254                                56  64 2ms164us
    sent=4 received=4 packet-loss=0% min-rtt=2ms69us avg-rtt=2ms514us
   max-rtt=3ms326us

[admin@client1] > ping 10.1.60.10 count=4
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 10.1.60.10                                 56  62 39ms855us
    1 10.1.60.10                                 56  62 8ms286us
    2 10.1.60.10                                 56  62 8ms52us
    3 10.1.60.10                                 56  62 8ms15us
    sent=4 received=4 packet-loss=0% min-rtt=8ms15us avg-rtt=16ms52us
   max-rtt=39ms855us

```
