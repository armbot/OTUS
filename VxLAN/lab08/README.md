### Внешние маршруты в VxLAN EVPN (TYPE-5)

### Схема стенда

![lab_8_scheme.jpg](lab_8_scheme.jpg)

### Описание
- VxLAN EVPN сеть взята из работы - [Lab06. Edge-Routed Bridging (Anycast Gateway)](https://github.com/armbot/OTUS/tree/5f4ad0bc44a3915d06a75d83844c7fd0ae5506cb/VxLAN/lab06/2).
- Каждому VLAN соответствует свой VRF: VLAN 10 - VRF "A", VLAN 20 - VRF "B". Без внешней маршрутизации доступ из VLAN 10 в VLAN 20 невозможен.
- Добавлен Router, анонсирующий маршрут по умолчанию в каждый VRF через eBGP стыки (VRF "A" - VLAN 100, 101; VRF "B" - VLAN 200, 201).

### Настройки
<details>
<summary> Leaf-1 </summary>

```
service routing protocols model multi-agent
!
hostname Leaf-1
!
vlan 10
!
vrf instance TENANT-A
!
interface Ethernet1
   description to-Spine-1
   mtu 9000
   no switchport
   ip address 10.0.1.0/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet2
   description to-Spine-2
   mtu 9000
   no switchport
   ip address 10.0.1.2/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet3
   switchport access vlan 10
   spanning-tree portfast
!
interface Loopback0
   description Router-ID
   ip address 172.16.0.3/32
!
interface Vlan10
   vrf TENANT-A
   ip address virtual 192.168.10.1/24
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
route-map REDISTRIBUTE_ONLY_LOOPBACKS permit 10
   match interface Loopback0
!
router bgp 65000
   router-id 172.16.0.3
   no bgp default ipv4-unicast
   maximum-paths 8 ecmp 8
   neighbor LEAF_OVERLAY peer group
   neighbor LEAF_OVERLAY remote-as 65000
   neighbor LEAF_OVERLAY update-source Loopback0
   neighbor LEAF_OVERLAY send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE password 7 p1iGcmS72bggHzKQpAB8dA==
   neighbor 10.0.1.1 peer group SPINE
   neighbor 10.0.1.3 peer group SPINE
   neighbor 172.16.0.4 peer group LEAF_OVERLAY
   neighbor 172.16.0.5 peer group LEAF_OVERLAY
   !
   vlan 10
      rd 172.16.0.3:10
      route-target both 65000:10010
      redistribute learned
   !
   address-family evpn
      neighbor LEAF_OVERLAY activate
   !
   address-family ipv4
      neighbor SPINE activate
      redistribute connected route-map REDISTRIBUTE_ONLY_LOOPBACKS
   !
   vrf TENANT-A
      rd 172.16.0.3:50001
      route-target import evpn 65000:50001
      route-target export evpn 65000:50001
!
end
```
</details>
<details>
<summary> Leaf-2 </summary>

```
service routing protocols model multi-agent
!
hostname Leaf-2
!
spanning-tree mode mstp
!
vlan 10,20,100,200
!
vrf instance TENANT-A
!
vrf instance TENANT-B
!
interface Ethernet1
   description to-Spine-1
   mtu 9000
   no switchport
   ip address 10.0.2.0/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet2
   description to-Spine-2
   mtu 9000
   no switchport
   ip address 10.0.2.2/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet3
   switchport access vlan 10
   spanning-tree portfast
!
interface Ethernet4
   switchport access vlan 20
   spanning-tree portfast
!
interface Ethernet8
   description TO_Router
   switchport mode trunk
!
interface Loopback0
   description Router-ID
   ip address 172.16.0.4/32
!
interface Vlan10
   vrf TENANT-A
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf TENANT-B
   ip address virtual 192.168.20.1/24
!
interface Vlan100
   description P-to-P_Router_VRF_A
   vrf TENANT-A
   ip address 10.0.2.100/31
!
interface Vlan200
   description P-to-P_Router_VRF_B
   vrf TENANT-B
   ip address 10.0.2.200/31
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf TENANT-A vni 50001
   vxlan vrf TENANT-B vni 50002
!
ip virtual-router mac-address 00:00:22:22:33:33
!
ip routing
ip routing vrf TENANT-A
ip routing vrf TENANT-B
!
route-map REDISTRIBUTE_ONLY_LOOPBACKS permit 10
   match interface Loopback0
!
router bgp 65000
   router-id 172.16.0.4
   no bgp default ipv4-unicast
   maximum-paths 8 ecmp 8
   neighbor LEAF_OVERLAY peer group
   neighbor LEAF_OVERLAY remote-as 65000
   neighbor LEAF_OVERLAY update-source Loopback0
   neighbor LEAF_OVERLAY send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE password 7 p1iGcmS72bggHzKQpAB8dA==
   neighbor 10.0.2.1 peer group SPINE
   neighbor 10.0.2.3 peer group SPINE
   neighbor 172.16.0.3 peer group LEAF_OVERLAY
   neighbor 172.16.0.5 peer group LEAF_OVERLAY
   !
   vlan 10
      rd 172.16.0.4:10
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd 172.16.0.4:20
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor LEAF_OVERLAY activate
   !
   address-family ipv4
      neighbor SPINE activate
      no neighbor 10.0.2.201 activate
      redistribute connected route-map REDISTRIBUTE_ONLY_LOOPBACKS
   !
   vrf TENANT-A
      rd 172.16.0.4:50001
      route-target import evpn 65000:50001
      route-target export evpn 65000:50001
      neighbor 10.0.2.101 remote-as 65001
      !
      address-family ipv4
         neighbor 10.0.2.101 activate
         redistribute connected
   !
   vrf TENANT-B
      rd 172.16.0.4:50002
      route-target import evpn 65000:50002
      route-target export evpn 65000:50002
      neighbor 10.0.2.201 remote-as 65001
      !
      address-family ipv4
         neighbor 10.0.2.201 activate
         redistribute connected
!
end
```
</details>
<details>
<summary> Leaf-3 </summary>

```
service routing protocols model multi-agent
!
hostname Leaf-3
!
vlan 10,20,101,201
!
vrf instance TENANT-A
!
vrf instance TENANT-B
!
interface Ethernet1
   description to-Spine-1
   mtu 9000
   no switchport
   ip address 10.0.3.0/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet2
   description to-Spine-2
   mtu 9000
   no switchport
   ip address 10.0.3.2/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet3
   switchport access vlan 20
   spanning-tree portfast
!
interface Ethernet8
   description TO_Router
   switchport mode trunk
!
interface Loopback0
   description Router-ID
   ip address 172.16.0.5/32
!
interface Vlan10
   vrf TENANT-A
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf TENANT-B
   ip address virtual 192.168.20.1/24
!
interface Vlan101
   description P-to-P_Router_VRF_A
   vrf TENANT-A
   ip address 10.0.3.100/31
!
interface Vlan201
   description P-to-P_Router_VRF_B
   vrf TENANT-B
   ip address 10.0.3.200/31
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf TENANT-A vni 50001
   vxlan vrf TENANT-B vni 50002
!
ip virtual-router mac-address 00:00:22:22:33:33
!
ip routing
ip routing vrf TENANT-A
ip routing vrf TENANT-B
!
route-map REDISTRIBUTE_ONLY_LOOPBACKS permit 10
   match interface Loopback0
!
router bgp 65000
   router-id 172.16.0.5
   no bgp default ipv4-unicast
   maximum-paths 8 ecmp 8
   neighbor LEAF_OVERLAY peer group
   neighbor LEAF_OVERLAY remote-as 65000
   neighbor LEAF_OVERLAY update-source Loopback0
   neighbor LEAF_OVERLAY send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE password 7 p1iGcmS72bggHzKQpAB8dA==
   neighbor 10.0.3.1 peer group SPINE
   neighbor 10.0.3.3 peer group SPINE
   neighbor 172.16.0.3 peer group LEAF_OVERLAY
   neighbor 172.16.0.4 peer group LEAF_OVERLAY
   !
   vlan 10
      rd 172.16.0.5:10
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd 172.16.0.5:20
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor LEAF_OVERLAY activate
   !
   address-family ipv4
      neighbor SPINE activate
      redistribute connected route-map REDISTRIBUTE_ONLY_LOOPBACKS
   !
   vrf TENANT-A
      rd 172.16.0.5:50001
      route-target import evpn 65000:50001
      route-target export evpn 65000:50001
      neighbor 10.0.3.101 remote-as 65001
      !
      address-family ipv4
         neighbor 10.0.3.101 activate
         redistribute connected
   !
   vrf TENANT-B
      rd 172.16.0.5:50002
      route-target import evpn 65000:50002
      route-target export evpn 65000:50002
      neighbor 10.0.3.201 remote-as 65001
      !
      address-family ipv4
         neighbor 10.0.3.201 activate
         redistribute connected
!
end
```
</details>

<details>
<summary> Router </summary>

```
service routing protocols model multi-agent
!
hostname Router
!
vlan 100-101,200-201
!
interface Ethernet1
   description To_LEAF-2
   switchport trunk allowed vlan 100,200
   switchport mode trunk
!
interface Ethernet2
   description To_LEAF-3
   switchport trunk allowed vlan 101,201
   switchport mode trunk
!
interface Vlan100
   no autostate
   ip address 10.0.2.101/31
!
interface Vlan101
   no autostate
   ip address 10.0.3.101/31
!
interface Vlan200
   no autostate
   ip address 10.0.2.201/31
!
interface Vlan201
   no autostate
   ip address 10.0.3.201/31
!
ip routing
!
ip route 0.0.0.0/0 Null0
!
router bgp 65001
   no bgp default ipv4-unicast
   maximum-paths 2 ecmp 2
   neighbor LEAF peer group
   neighbor LEAF remote-as 65000
   neighbor 10.0.2.100 peer group LEAF
   neighbor 10.0.2.200 peer group LEAF
   neighbor 10.0.3.100 peer group LEAF
   neighbor 10.0.3.200 peer group LEAF
   !
   address-family ipv4
      neighbor LEAF activate
      redistribute static
!
end
```
</details>


### Проверка работы
<details>
<summary> Router#show ip route bgp </summary>

```
Router#show ip route bgp 

VRF: default

 B E      192.168.10.0/24 [200/0] via 10.0.2.100, Vlan100
                                  via 10.0.3.100, Vlan101
 B E      192.168.20.0/24 [200/0] via 10.0.2.200, Vlan200
                                  via 10.0.3.200, Vlan201
```
</details>
<details>
<summary> Leaf-1#show bgp evpn route-type ip-prefix ipv4 </summary>

```
Leaf-1#show bgp evpn route-type ip-prefix ipv4

BGP routing table information for VRF default
Router identifier 172.16.0.3, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 172.16.0.4:50001 ip-prefix 0.0.0.0/0
                                 172.16.0.4            -       100     0       65001 ?
 * >      RD: 172.16.0.4:50002 ip-prefix 0.0.0.0/0
                                 172.16.0.4            -       100     0       65001 ?
 * >      RD: 172.16.0.5:50001 ip-prefix 0.0.0.0/0
                                 172.16.0.5            -       100     0       65001 ?
 * >      RD: 172.16.0.5:50002 ip-prefix 0.0.0.0/0
                                 172.16.0.5            -       100     0       65001 ?
 * >      RD: 172.16.0.4:50001 ip-prefix 10.0.2.100/31
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.4:50002 ip-prefix 10.0.2.200/31
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.5:50001 ip-prefix 10.0.3.100/31
                                 172.16.0.5            -       100     0       i
 * >      RD: 172.16.0.5:50002 ip-prefix 10.0.3.200/31
                                 172.16.0.5            -       100     0       i
 * >      RD: 172.16.0.4:50001 ip-prefix 192.168.10.0/24
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.5:50001 ip-prefix 192.168.10.0/24
                                 172.16.0.5            -       100     0       i
 * >      RD: 172.16.0.4:50002 ip-prefix 192.168.20.0/24
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.5:50002 ip-prefix 192.168.20.0/24
                                 172.16.0.5            -       100     0       i
```
</details>

<details>
<summary> Проверка доступности VPC_1 (Leaf-1) <-> VPC_3 (Leaf-2) </summary>

```
VPC_1> ping 192.168.20.203  

84 bytes from 192.168.20.203 icmp_seq=1 ttl=62 time=73.365 ms
84 bytes from 192.168.20.203 icmp_seq=2 ttl=62 time=100.237 ms
84 bytes from 192.168.20.203 icmp_seq=3 ttl=62 time=37.407 ms
84 bytes from 192.168.20.203 icmp_seq=4 ttl=62 time=42.458 ms
84 bytes from 192.168.20.203 icmp_seq=5 ttl=62 time=40.282 ms

VPC_1> trace 192.168.20.203
trace to 192.168.20.203, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   7.651 ms  7.490 ms  9.355 ms
 2   192.168.10.1   39.109 ms  24.719 ms  29.490 ms
 3   *192.168.20.203   38.096 ms (ICMP type:3, code:3, Destination port unreachable)
```
</details>
<details>
<summary> Проверка доступности VPC_1 (VLAN 10, VRF "A") <-> VPC_4 (VLAN 20, VRF "B") </summary>

```
VPC_1> ping 192.168.20.204 

84 bytes from 192.168.20.204 icmp_seq=1 ttl=60 time=129.876 ms
84 bytes from 192.168.20.204 icmp_seq=2 ttl=60 time=205.674 ms
84 bytes from 192.168.20.204 icmp_seq=3 ttl=60 time=95.852 ms
84 bytes from 192.168.20.204 icmp_seq=4 ttl=60 time=115.654 ms
84 bytes from 192.168.20.204 icmp_seq=5 ttl=60 time=100.957 ms

VPC_1> trace 192.168.20.204
trace to 192.168.20.204, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   13.353 ms  7.088 ms  6.434 ms
 2   192.168.10.1   33.735 ms  45.509 ms  32.694 ms
 3   10.0.2.101   51.964 ms  52.602 ms  48.664 ms
 4   10.0.2.200   68.788 ms  56.640 ms  112.691 ms
 5   10.0.3.200   211.652 ms  70.499 ms  88.227 ms
 6   *192.168.20.204   102.048 ms (ICMP type:3, code:3, Destination port unreachable)
```
</details>
