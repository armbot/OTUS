### Настройка маршрутизации в VxLAN EVPN

### Цели
- обеспечить маршрутизацию в рамках Overlay VxLAN EVPN между клиентами следующими способами:
   - [Bridged Overlay](lab06/1);
   - Edge-Routed Bridging (Symmetric IRB);
   - Anycast Gateway.

### Схема стенда

![lab5_scheme.jpg](lab5_scheme.jpg)

### Описание
- VxLAN EVPN L2-сеть взята из предыдущей работы - [Lab05. Overlay на основе VxLAN EVPN для L2 связанности между клиентами](https://github.com/armbot/OTUS/tree/9505106f8681b0b35010acc5577b91d84ab4c4a9/VxLAN/lab05)
- AF l2vpn evpn настраивается только между Leaf (peer group LEAF_EVPN).

### Таблица IP-адресов клиентов

|Device|VLAN|IP Address|
|---|---|---|
VPC_1|10|192.168.10.101
VPC_2|10|192.168.10.102
VPC_3|20|192.168.20.203
VPC_4|20|192.168.20.204

### Настройки Leaf
<details>
<summary> Leaf-1 </summary>

```
hostname Leaf-1
!
vlan 10
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
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
!
ip routing
!
route-map REDISTRIBUTE_ONLY_LOOPBACKS permit 10
   match interface Loopback0
!
router bgp 65000
   router-id 172.16.0.3
   no bgp default ipv4-unicast
   maximum-paths 8 ecmp 8
   neighbor LEAF_EVPN peer group
   neighbor LEAF_EVPN remote-as 65000
   neighbor LEAF_EVPN update-source Loopback0
   neighbor LEAF_EVPN send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE password 7 p1iGcmS72bggHzKQpAB8dA==
   neighbor 10.0.1.1 peer group SPINE
   neighbor 10.0.1.3 peer group SPINE
   neighbor 172.16.0.4 peer group LEAF_EVPN
   neighbor 172.16.0.5 peer group LEAF_EVPN
   !
   vlan 10
      rd 1.1.1.1:10
      route-target both 65000:10010
      redistribute learned
   !
   address-family evpn
      neighbor LEAF_EVPN activate
   !
   address-family ipv4
      neighbor SPINE activate
      redistribute connected route-map REDISTRIBUTE_ONLY_LOOPBACKS
!
end
```
</details>
<details>
<summary> Leaf-2 </summary>

```
hostname Leaf-2
!
vlan 10,20
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
interface Loopback0
   description Router-ID
   ip address 172.16.0.4/32
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
!
ip routing
!
route-map REDISTRIBUTE_ONLY_LOOPBACKS permit 10
   match interface Loopback0
!
router bgp 65000
   router-id 172.16.0.4
   no bgp default ipv4-unicast
   maximum-paths 8 ecmp 8
   neighbor LEAF_EVPN peer group
   neighbor LEAF_EVPN remote-as 65000
   neighbor LEAF_EVPN update-source Loopback0
   neighbor LEAF_EVPN send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE password 7 p1iGcmS72bggHzKQpAB8dA==
   neighbor 10.0.2.1 peer group SPINE
   neighbor 10.0.2.3 peer group SPINE
   neighbor 172.16.0.3 peer group LEAF_EVPN
   neighbor 172.16.0.5 peer group LEAF_EVPN
   !
   vlan 10
      rd 2.2.2.2:10
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd 2.2.2.2:20
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor LEAF_EVPN activate
   !
   address-family ipv4
      neighbor SPINE activate
      redistribute connected route-map REDISTRIBUTE_ONLY_LOOPBACKS
!
end
```
</details>
<details>
<summary> Leaf-3 </summary>

```
hostname Leaf-3
!
vlan 20
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
interface Loopback0
   description Router-ID
   ip address 172.16.0.5/32
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
!
ip routing
!
route-map REDISTRIBUTE_ONLY_LOOPBACKS permit 10
   match interface Loopback0
!
!
router bgp 65000
   router-id 172.16.0.5
   no bgp default ipv4-unicast
   maximum-paths 8 ecmp 8
   neighbor LEAF_EVPN peer group
   neighbor LEAF_EVPN remote-as 65000
   neighbor LEAF_EVPN update-source Loopback0
   neighbor LEAF_EVPN send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE password 7 p1iGcmS72bggHzKQpAB8dA==
   neighbor 10.0.3.1 peer group SPINE
   neighbor 10.0.3.3 peer group SPINE
   neighbor 172.16.0.3 peer group LEAF_EVPN
   neighbor 172.16.0.4 peer group LEAF_EVPN
   !
   vlan 20
      rd 3.3.3.3:20
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor LEAF_EVPN activate
   !
   address-family ipv4
      neighbor SPINE activate
      redistribute connected route-map REDISTRIBUTE_ONLY_LOOPBACKS
!
end

```
</details>

### Проверка работы EVPN (Leaf-2)
#### #show bgp evpn summary
```
Leaf-2#show bgp evpn summary 
BGP summary information for VRF default
Router identifier 172.16.0.4, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor   V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.0.3 4 65000             62        68    0    0 00:44:35 Estab   2      2
  172.16.0.5 4 65000             55        57    0    0 00:35:53 Estab   1      1
```
#### #show bgp evpn
```
Leaf-2#show bgp evpn 
BGP routing table information for VRF default
Router identifier 172.16.0.4, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 1.1.1.1:10 mac-ip 0050.7966.6806
                                 172.16.0.3            -       100     0       i
 * >      RD: 2.2.2.2:10 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >      RD: 2.2.2.2:20 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >      RD: 3.3.3.3:20 mac-ip 0050.7966.6809
                                 172.16.0.5            -       100     0       i
 * >      RD: 1.1.1.1:10 imet 172.16.0.3
                                 172.16.0.3            -       100     0       i
 * >      RD: 2.2.2.2:10 imet 172.16.0.4
                                 -                     -       -       0       i
 * >      RD: 2.2.2.2:20 imet 172.16.0.4
                                 -                     -       -       0       i
 * >      RD: 3.3.3.3:20 imet 172.16.0.5
                                 172.16.0.5            -       100     0       i
```
#### #show mac address-table
```
Leaf-2#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0050.7966.6806    DYNAMIC     Vx1        1       0:01:19 ago
  10    0050.7966.6807    DYNAMIC     Et3        1       0:01:19 ago
  20    0050.7966.6808    DYNAMIC     Et4        1       0:03:39 ago
  20    0050.7966.6809    DYNAMIC     Vx1        1       0:03:39 ago
Total Mac Addresses for this criterion: 4
```

</details>
<details>
<summary> Проверка доступности VPC_1 (Leaf-1) <-> VPC_2 (Leaf-2) </summary>

```
VPC_1> ping 192.168.10.102

84 bytes from 192.168.10.102 icmp_seq=1 ttl=64 time=99.836 ms
84 bytes from 192.168.10.102 icmp_seq=2 ttl=64 time=46.479 ms
84 bytes from 192.168.10.102 icmp_seq=3 ttl=64 time=31.308 ms
84 bytes from 192.168.10.102 icmp_seq=4 ttl=64 time=48.919 ms
84 bytes from 192.168.10.102 icmp_seq=5 ttl=64 time=45.292 ms

```
</details>
<details>
<summary> Проверка доступности VPC_4 (Leaf-3) <-> VPC_3 (Leaf-2) </summary>

```

VPC_4> ping 192.168.20.203

84 bytes from 192.168.20.203 icmp_seq=1 ttl=64 time=133.484 ms
