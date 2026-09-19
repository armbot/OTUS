### Настройка маршрутизации в VxLAN EVPN (Edge-Routed Bridging, Symmetric IRB)

### Описание
- VxLAN EVPN L2-сеть взята из предыдущей работы - [Lab05. VxLAN EVPN L2](https://github.com/armbot/OTUS/tree/9505106f8681b0b35010acc5577b91d84ab4c4a9/VxLAN/lab05).
- На каждом Leaf производятся идентичные настройки для Symmetric IRB.

### Настройки
<details>
<summary> Leaf-1 </summary>

```
!
vrf instance TENANT-A
!
interface Vlan10
   vrf TENANT-A
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf TENANT-A
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan vrf TENANT-A vni 50001
!
ip virtual-router mac-address 00:00:22:22:33:33
!
ip routing vrf TENANT-A
!
!
router bgp 65000
   !
   vrf TENANT-A
      rd 172.16.0.3:50001
      route-target import evpn 65000:50001
      route-target export evpn 65000:50001
      redistribute connected
!
```
</details>
<details>
<summary> Leaf-2 </summary>

```
!
vrf instance TENANT-A
!
interface Vlan10
   vrf TENANT-A
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf TENANT-A
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan vrf TENANT-A vni 50001
!
ip virtual-router mac-address 00:00:22:22:33:33
!
ip routing vrf TENANT-A
!
!
router bgp 65000
   !
   vrf TENANT-A
      rd 172.16.0.4:50001
      route-target import evpn 65000:50001
      route-target export evpn 65000:50001
      redistribute connected
!
```
</details>
<details>
<summary> Leaf-3 </summary>

```
!
vrf instance TENANT-A
!
interface Vlan10
   vrf TENANT-A
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf TENANT-A
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan vrf TENANT-A vni 50001
!
ip virtual-router mac-address 00:00:22:22:33:33
!
ip routing vrf TENANT-A
!
!
router bgp 65000
   !
   vrf TENANT-A
      rd 172.16.0.5:50001
      route-target import evpn 65000:50001
      route-target export evpn 65000:50001
      redistribute connected
!
```
</details>

### Проверка работы
</details>
<details>
<summary> Проверка доступности VPC_1 (Leaf-1) <-> VPC_3 (Leaf-2) </summary>

```
VPC_1> ping 192.168.20.203

84 bytes from 192.168.20.203 icmp_seq=1 ttl=63 time=254.020 ms
84 bytes from 192.168.20.203 icmp_seq=2 ttl=63 time=55.640 ms
84 bytes from 192.168.20.203 icmp_seq=3 ttl=63 time=67.200 ms
84 bytes from 192.168.20.203 icmp_seq=4 ttl=63 time=67.751 ms
84 bytes from 192.168.20.203 icmp_seq=5 ttl=63 time=81.951 ms

VPC_1> trace 192.168.20.203  
trace to 192.168.20.203, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   43.572 ms  121.904 ms  47.688 ms
 2   *192.168.20.203   245.482 ms (ICMP type:3, code:3, Destination port unreachable)
```
</details>
<details>
<summary> Проверка доступности VPC_1 (Leaf-1) <-> VPC_4 (Leaf-3) </summary>

```
VPC_1> ping 192.168.20.204

84 bytes from 192.168.20.204 icmp_seq=1 ttl=63 time=246.132 ms
84 bytes from 192.168.20.204 icmp_seq=2 ttl=63 time=101.082 ms
84 bytes from 192.168.20.204 icmp_seq=3 ttl=63 time=88.055 ms
84 bytes from 192.168.20.204 icmp_seq=4 ttl=63 time=87.150 ms
84 bytes from 192.168.20.204 icmp_seq=5 ttl=63 time=198.263 ms

VPC_1> trace 192.168.20.204
trace to 192.168.20.204, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   53.539 ms  50.057 ms  49.352 ms
 2   *192.168.20.204   239.813 ms (ICMP type:3, code:3, Destination port unreachable)

#### #show bgp evpn route-type mac-ip vni 10010
```
</details>
<details>
<summary> Leaf-1#show bgp evpn route-type mac-ip vni 10010 </summary>

```
Leaf-1#show bgp evpn route-type mac-ip vni 10010
BGP routing table information for VRF default
Router identifier 172.16.0.3, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 172.16.0.3:10 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >      RD: 172.16.0.4:10 mac-ip 0050.7966.6807
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.4:10 mac-ip 5000.00af.d3f6
                                 172.16.0.4            -       100     0       i
```
</details>
<details>
<summary> Leaf-1#show mac address-table </summary>

```
Leaf-1#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0050.7966.6806    DYNAMIC     Et3        1       0:00:10 ago
  10    0050.7966.6807    DYNAMIC     Vx1        1       0:00:10 ago
  10    5000.00af.d3f6    DYNAMIC     Vx1        1       0:00:02 ago
```
</details>
