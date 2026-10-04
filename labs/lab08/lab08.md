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
| `10.1.60.0/24` | **client3** Ethernet0            | `10.1.60.10`  | **leaf3** Ethernet11 (VRF TENANT-A)         |
| `10.1.40.0/24` | **client4** Ethernet0            | `10.1.40.10`  | **leaf3** Ethernet12 (VRF TENANT-B)         |




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
vlan 10
   name VLAN10
!
vlan 30
   name VLAN30
!
vrf instance TENANT-A
!
vrf instance TENANT-B
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
   description 'PtP to router1 (TENANT-A)'
   no switchport
   vrf TENANT-A
   ip address 192.168.100.0/31
!
interface Ethernet4
   description 'PtP to router1 (TENANT-B)'
   no switchport
   vrf TENANT-B
   ip address 192.168.200.0/31
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
   switchport access vlan 10
!
interface Loopback0
   ip address 10.1.0.5/32
   ip ospf area 0.0.0.0
!
interface Management1
!
interface Vlan10
   description 'Gateway for client4'
   vrf TENANT-B
   ip address virtual 10.1.40.1/24
!
interface Vlan30
   description 'Gateway for client3'
   vrf TENANT-A
   ip address virtual 10.1.60.1/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 30 vni 10030
   vxlan vrf TENANT-A vni 50001
   vxlan vrf TENANT-B vni 50002
!
ip virtual-router mac-address 00:00:22:22:33:33
!
ip routing
ip routing vrf TENANT-A
ip routing vrf TENANT-B
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
      rd 10.1.0.5:10010
      route-target both 65000:10010
      redistribute learned
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
      neighbor 192.168.100.1 remote-as 65500
      redistribute connected
      !
      address-family ipv4
         neighbor 192.168.100.1 activate
         redistribute connected
   !
   vrf TENANT-B
      rd 10.1.0.5:50002
      route-target import evpn 65000:50002
      route-target export evpn 65000:50002
      router-id 10.1.0.5
      neighbor 192.168.200.1 remote-as 65500
      redistribute connected
      !
      address-family ipv4
         neighbor 192.168.200.1 activate
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


### leaf3 bgp summary (TENANT-A, TENANT-B)

```
leaf3#show ip bgp summary vrf TENANT-A
BGP summary information for VRF TENANT-A
Router identifier 10.1.0.5, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor      V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  192.168.100.1 4 65500            371       418    0    0 05:52:20 Estab   3      3
leaf3#
leaf3#show ip bgp summary vrf TENANT-B
BGP summary information for VRF TENANT-B
Router identifier 10.1.0.5, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor      V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  192.168.200.1 4 65500 
```


<br>

### leaf3 bgp routes (TENANT-A)

```
leaf3#show ip bgp vrf TENANT-A
BGP routing table information for VRF TENANT-A
Router identifier 10.1.0.5, local AS number 65000
Route status codes: s - suppressed contributor, * - valid, > - active, E - ECMP head, e - ECM                                                                                                P
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
                    % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI Origin Validation codes: V - valid, I - invalid, U - unknown
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Next                                                                                                hop

          Network                Next Hop              Metric  AIGP       LocPref Weight  Pat                                                                                                h
 * >      8.8.8.8/32             192.168.100.1         0       -          100     0       655                                                                                                00 i
 * >      9.9.9.9/32             192.168.100.1         0       -          100     0       655                                                                                                00 i
 * >      10.1.40.0/24           192.168.100.1         0       -          100     0       655                                                                                                00 65500 i
 * >Ec    10.1.50.0/24           10.1.0.4              0       -          100     0       i O                                                                                                r-ID: 10.1.0.4 C-LST: 10.1.0.1
 *  ec    10.1.50.0/24           10.1.0.3              0       -          100     0       i O                                                                                                r-ID: 10.1.0.3 C-LST: 10.1.0.1
 * >Ec    10.1.50.10/32          10.1.0.4              0       -          100     0       i O                                                                                                r-ID: 10.1.0.4 C-LST: 10.1.0.1
 *  ec    10.1.50.10/32          10.1.0.3              0       -          100     0       i O                                                                                                r-ID: 10.1.0.3 C-LST: 10.1.0.1
 * >      10.1.60.0/24           -                     -       -          -       0       i
 * >      192.168.100.0/31       -                     -       -          -       0       i
```



<br>

### leaf3 bgp routes (TENANT-B)

```
leaf3#show ip bgp vrf TENANT-B
BGP routing table information for VRF TENANT-B
Router identifier 10.1.0.5, local AS number 65000
Route status codes: s - suppressed contributor, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
                    % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI Origin Validation codes: V - valid, I - invalid, U - unknown
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >      8.8.8.8/32             192.168.200.1         0       -          100     0       65500 i
 * >      9.9.9.9/32             192.168.200.1         0       -          100     0       65500 i
 * >      10.1.40.0/24           -                     -       -          -       0       i
 * >      10.1.50.0/24           192.168.200.1         0       -          100     0       65500 65500 i
 * >      10.1.50.10/32          192.168.200.1         0       -          100     0       65500 65500 i
 * >      10.1.60.0/24           192.168.200.1         0       -          100     0       65500 65500 i
 * >      192.168.200.0/31       -                     -       -          -       0       i
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
 * >      RD: 10.1.0.5:50001 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       65500 i
 * >      RD: 10.1.0.5:50002 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       65500 i
 * >      RD: 10.1.0.5:50001 ip-prefix 9.9.9.9/32
                                 -                     -       100     0       65500 i
 * >      RD: 10.1.0.5:50002 ip-prefix 9.9.9.9/32
                                 -                     -       100     0       65500 i
 * >      RD: 10.1.0.5:50001 ip-prefix 10.1.40.0/24
                                 -                     -       100     0       65500 65500 i
 * >      RD: 10.1.0.5:50002 ip-prefix 10.1.40.0/24
                                 -                     -       -       0       i
 * >      RD: 10.1.0.3:50001 ip-prefix 10.1.50.0/24
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.0.1
 * >      RD: 10.1.0.4:50001 ip-prefix 10.1.50.0/24
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.0.1
 * >      RD: 10.1.0.5:50002 ip-prefix 10.1.50.0/24
                                 -                     -       100     0       65500 65500 i
 * >      RD: 10.1.0.5:50002 ip-prefix 10.1.50.10/32
                                 -                     -       100     0       65500 65500 i
 * >      RD: 10.1.0.5:50001 ip-prefix 10.1.60.0/24
                                 -                     -       -       0       i
 * >      RD: 10.1.0.5:50002 ip-prefix 10.1.60.0/24
                                 -                     -       100     0       65500 65500 i
 * >      RD: 10.1.0.5:50001 ip-prefix 192.168.100.0/31
                                 -                     -       -       0       i
 * >      RD: 10.1.0.5:50002 ip-prefix 192.168.200.0/31
                                 -                     -       -       0       i
```


<br>


### client1 ping

```
[admin@client1] > ip address/print
Columns: ADDRESS, NETWORK, INTERFACE
# ADDRESS        NETWORK    INTERFACE
0 10.1.50.10/24  10.1.50.0  vlan20-bond1
[admin@client1] > ip route print
Flags: D - DYNAMIC; A - ACTIVE; c - CONNECT, s - STATIC
Columns: DST-ADDRESS, GATEWAY, ROUTING-TABLE, DISTANCE
#     DST-ADDRESS   GATEWAY       ROUTING-TABLE  DISTANCE
0  As 0.0.0.0/0     10.1.50.254   main                  1
  DAc 10.1.50.0/24  vlan20-bond1  main                  0
[admin@client1] > ping 8.8.8.8 count=4
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 8.8.8.8                                    56 106 21ms502us
    1 8.8.8.8                                    56 106 13ms662us
    2 8.8.8.8                                    56 106 14ms185us
    3 8.8.8.8                                    56 106 15ms618us
    sent=4 received=4 packet-loss=0% min-rtt=13ms662us avg-rtt=16ms241us
   max-rtt=21ms502us

[admin@client1] > ping 9.9.9.9 count=4
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 9.9.9.9                                    56  51 65ms839us
    1 9.9.9.9                                    56  51 53ms556us
    2 9.9.9.9                                    56  51 56ms676us
    3 9.9.9.9                                    56  51 53ms620us
    sent=4 received=4 packet-loss=0% min-rtt=53ms556us avg-rtt=57ms422us
   max-rtt=65ms839us

[admin@client1] > ping 10.1.60.10 count=4
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 10.1.60.10                                 56  62 37ms398us
    1 10.1.60.10                                 56  62 9ms173us
    2 10.1.60.10                                 56  62 7ms526us
    3 10.1.60.10                                 56  62 7ms519us
    sent=4 received=4 packet-loss=0% min-rtt=7ms519us avg-rtt=15ms404us
   max-rtt=37ms398us

[admin@client1] > ping 10.1.40.10 count=4
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 10.1.40.10                                 56  60 18ms878us
    1 10.1.40.10                                 56  60 14ms801us
    2 10.1.40.10                                 56  60 10ms86us
    3 10.1.40.10                                 56  60 9ms836us
    sent=4 received=4 packet-loss=0% min-rtt=9ms836us avg-rtt=13ms400us
   max-rtt=18ms878us
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
LPORT       : 20152
RHOST:PORT  : 127.0.0.1:20153
MTU         : 1500

client3> ping 8.8.8.8 -c 4

84 bytes from 8.8.8.8 icmp_seq=1 ttl=107 time=9.604 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=107 time=10.130 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=107 time=9.635 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=107 time=13.886 ms

client3> ping 9.9.9.9 -c 4

84 bytes from 9.9.9.9 icmp_seq=1 ttl=52 time=49.643 ms
84 bytes from 9.9.9.9 icmp_seq=2 ttl=52 time=60.336 ms
84 bytes from 9.9.9.9 icmp_seq=3 ttl=52 time=48.027 ms
84 bytes from 9.9.9.9 icmp_seq=4 ttl=52 time=47.785 ms

client3> ping 10.1.50.10 -c 4

84 bytes from 10.1.50.10 icmp_seq=1 ttl=62 time=14.821 ms
84 bytes from 10.1.50.10 icmp_seq=2 ttl=62 time=7.406 ms
84 bytes from 10.1.50.10 icmp_seq=3 ttl=62 time=7.411 ms
84 bytes from 10.1.50.10 icmp_seq=4 ttl=62 time=8.614 ms

client3> ping 10.1.40.10 -c 4

84 bytes from 10.1.40.10 icmp_seq=1 ttl=61 time=9.423 ms
84 bytes from 10.1.40.10 icmp_seq=2 ttl=61 time=5.346 ms
84 bytes from 10.1.40.10 icmp_seq=3 ttl=61 time=5.908 ms
84 bytes from 10.1.40.10 icmp_seq=4 ttl=61 time=5.040 ms

```


<br>

### client4 ping

```
client4> show ip

NAME        : client4[1]
IP/MASK     : 10.1.40.10/24
GATEWAY     : 10.1.40.1
DNS         :
MAC         : 00:50:79:66:68:03
LPORT       : 20150
RHOST:PORT  : 127.0.0.1:20151
MTU         : 1500

client4> ping 8.8.8.8 -c 4

84 bytes from 8.8.8.8 icmp_seq=1 ttl=107 time=14.187 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=107 time=9.618 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=107 time=10.465 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=107 time=9.303 ms

client4> ping 9.9.9.9 -c 4

84 bytes from 9.9.9.9 icmp_seq=1 ttl=52 time=53.826 ms
84 bytes from 9.9.9.9 icmp_seq=2 ttl=52 time=47.700 ms
84 bytes from 9.9.9.9 icmp_seq=3 ttl=52 time=47.799 ms
84 bytes from 9.9.9.9 icmp_seq=4 ttl=52 time=48.712 ms

client4> ping 10.1.50.10 -c 4

84 bytes from 10.1.50.10 icmp_seq=1 ttl=60 time=16.607 ms
84 bytes from 10.1.50.10 icmp_seq=2 ttl=60 time=12.463 ms
84 bytes from 10.1.50.10 icmp_seq=3 ttl=60 time=10.881 ms
84 bytes from 10.1.50.10 icmp_seq=4 ttl=60 time=9.807 ms

client4> ping 10.1.60.10 -c 4

84 bytes from 10.1.60.10 icmp_seq=1 ttl=61 time=9.362 ms
84 bytes from 10.1.60.10 icmp_seq=2 ttl=61 time=5.233 ms
84 bytes from 10.1.60.10 icmp_seq=3 ttl=61 time=5.384 ms
84 bytes from 10.1.60.10 icmp_seq=4 ttl=61 time=4.891 ms
```


<br>

### client1 traceroute to client4

```
[admin@client1] > tool traceroute 10.1.40.10
ADDRESS                          LOSS SENT    LAST     AVG    BEST   WORST
                                 100%    5 timeout
10.1.60.1                          0%    4  13.8ms    18.4     6.5    45.9
192.168.100.1                      0%    4   8.1ms     8.1     6.8    10.1
192.168.200.0                      0%    4   8.2ms     7.9     7.4     8.2
10.1.40.10                         0%    4  10.6ms    13.1     8.6    17.2
```


<br>

### client3 traceroute to client4

```
client3> trace 10.1.40.10
trace to 10.1.40.10, 8 hops max, press Ctrl+C to stop
 1     *  *  *
 2   192.168.100.1   4.073 ms  3.008 ms  6.122 ms
 3   192.168.200.0   6.251 ms  3.662 ms  4.371 ms
 4   *10.1.40.10   5.129 ms (ICMP type:3, code:3, Destination port unreachable)
```


<br>


### client4 traceroute to client1 and client3

```
client4> trace 10.1.50.10
trace to 10.1.50.10, 8 hops max, press Ctrl+C to stop
 1     *  *  *
 2   192.168.200.1   3.445 ms  1.938 ms  1.760 ms
 3   192.168.100.0   4.734 ms  3.461 ms  3.313 ms
 4   10.1.50.1   16.697 ms  9.316 ms  9.157 ms
 5   *10.1.50.10   11.746 ms (ICMP type:3, code:3, Destination port unreachable)

client4> trace 10.1.60.10
trace to 10.1.60.10, 8 hops max, press Ctrl+C to stop
 1     *  *  *
 2   192.168.200.1   5.082 ms  4.139 ms  2.693 ms
 3   192.168.100.0   3.978 ms  3.642 ms  3.507 ms
 4   *10.1.60.10   7.645 ms (ICMP type:3, code:3, Destination port unreachable)
```


<br>


### router1 config

```
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
set [ find default-name=ether6 ] disable-running-check=no
set [ find default-name=ether7 ] disable-running-check=no
set [ find default-name=ether8 ] disable-running-check=no
/port
set 0 name=serial0
/routing bgp instance
add as=65500 disabled=no name=bgp-instance-1 router-id=192.168.100.1
/ip address
add address=192.168.100.1/31 interface=ether1 network=192.168.100.0
add address=172.31.99.150/24 interface=ether8 network=172.31.99.0
add address=192.168.200.1/31 interface=ether2 network=192.168.200.0
/ip firewall address-list
add address=9.9.9.9 list=bgp-networks
add address=8.8.8.8 list=bgp-networks
/ip firewall nat
add action=masquerade chain=srcnat out-interface=ether8
/ip route
add dst-address=8.8.8.8/32 gateway=172.31.99.254
add dst-address=9.9.9.9/32 gateway=172.31.99.254
/routing bgp connection
add as=65500 connect=yes instance=bgp-instance-1 listen=yes local.address=\
    192.168.100.1 .role=ebgp name=bgp1 output.as-override=yes .network=\
    bgp-networks .redistribute=bgp remote.address=192.168.100.0/32 .as=65000
add as=65500 connect=yes instance=bgp-instance-1 listen=yes local.address=\
    192.168.200.1 .role=ebgp name=bgp2 output.as-override=yes .network=\
    bgp-networks .redistribute=bgp remote.address=192.168.200.0/32 .as=65000
/system identity
set name=router1
```
