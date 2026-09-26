### Внешние маршруты в VxLAN EVPN (TYPE-5)

### Схема стенда

![lab_8_scheme.jpg](lab_8_scheme.jpg)

### Описание
- VxLAN EVPN сеть взята из работы - [Lab06. Edge-Routed Bridging (Anycast Gateway](https://github.com/armbot/OTUS/tree/5f4ad0bc44a3915d06a75d83844c7fd0ae5506cb/VxLAN/lab06/2).
- На каждом Leaf производятся идентичные настройки для Anycast Gateway (меняется только rd Loopback:50001).
- Проверяется 2 варианта распространения маршрутов о конечных клиентах: Type-5 (сети /24) и дополнительно Type-2 (сети /32).

### Настройки
#### Type-5:
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

#### Type-2:
Для работы через Type-2 из конфигурации удаляется строка "redistribute connected".

### Проверка работы
#### Type-5:
<details>
<summary> Leaf-1#show ip route vrf TENANT-A </summary>

```
Leaf-1#show ip route vrf TENANT-A

VRF: TENANT-A
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1...

Gateway of last resort is not set

 C        192.168.10.0/24 is directly connected, Vlan10
 B I      192.168.20.203/32 [200/0] via VTEP 172.16.0.4 VNI 50001 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B I      192.168.20.204/32 [200/0] via VTEP 172.16.0.5 VNI 50001 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 B I      192.168.20.0/24 [200/0] via VTEP 172.16.0.4 VNI 50001 router-mac 50:00:00:03:37:66 local-interface Vxlan1
                                  via VTEP 172.16.0.5 VNI 50001 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
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
 * >      RD: 172.16.0.3:50001 ip-prefix 192.168.10.0/24
                                 -                     -       -       0       i
 * >      RD: 172.16.0.4:50001 ip-prefix 192.168.10.0/24
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.4:50001 ip-prefix 192.168.20.0/24
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.5:50001 ip-prefix 192.168.20.0/24
                                 172.16.0.5            -       100     0       i
```
</details>
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
<summary> Проверка доступности VPC_1 (Leaf-1) <-> VPC_4 (Leaf-3) </summary>

```
VPC_1> ping 192.168.20.204 

84 bytes from 192.168.20.204 icmp_seq=1 ttl=62 time=50.378 ms
84 bytes from 192.168.20.204 icmp_seq=2 ttl=62 time=35.326 ms
84 bytes from 192.168.20.204 icmp_seq=3 ttl=62 time=45.327 ms
84 bytes from 192.168.20.204 icmp_seq=4 ttl=62 time=42.364 ms
84 bytes from 192.168.20.204 icmp_seq=5 ttl=62 time=41.609 ms

VPC_1> trace 192.168.20.204  
trace to 192.168.20.204, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   7.398 ms  9.612 ms  8.528 ms
 2   192.168.20.1   40.143 ms  27.631 ms  26.709 ms
 3   *192.168.20.204   48.012 ms (ICMP type:3, code:3, Destination port unreachable)
```
</details>

#### Type-2:
<details>
<summary> Leaf-1#show ip route vrf TENANT-A </summary>

```
Leaf-1#show ip route vrf TENANT-A

VRF: TENANT-A
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,

Gateway of last resort is not set

 C        192.168.10.0/24 is directly connected, Vlan10
 B I      192.168.20.203/32 [200/0] via VTEP 172.16.0.4 VNI 50001 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B I      192.168.20.204/32 [200/0] via VTEP 172.16.0.5 VNI 50001 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
```
</details>
<details>
<summary> Leaf-1#show bgp evpn </summary>

```
          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 172.16.0.3:10 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >      RD: 172.16.0.3:10 mac-ip 0050.7966.6806 192.168.10.101
                                 -                     -       -       0       i
 * >      RD: 172.16.0.4:10 mac-ip 0050.7966.6807
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.4:20 mac-ip 0050.7966.6808
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.4:20 mac-ip 0050.7966.6808 192.168.20.203
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.5:20 mac-ip 0050.7966.6809
                                 172.16.0.5            -       100     0       i
 * >      RD: 172.16.0.5:20 mac-ip 0050.7966.6809 192.168.20.204
                                 172.16.0.5            -       100     0       i
 * >      RD: 172.16.0.3:10 imet 172.16.0.3
                                 -                     -       -       0       i
 * >      RD: 172.16.0.4:10 imet 172.16.0.4
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.4:20 imet 172.16.0.4
                                 172.16.0.4            -       100     0       i
 * >      RD: 172.16.0.5:20 imet 172.16.0.5
                                 172.16.0.5            -       100     0       i
```
</details>
</details>
<details>
<summary> Проверка доступности VPC_1 (Leaf-1) <-> VPC_3 (Leaf-2) </summary>

```
VPC_1> ping 192.168.20.203

84 bytes from 192.168.20.203 icmp_seq=1 ttl=62 time=49.683 ms
84 bytes from 192.168.20.203 icmp_seq=2 ttl=62 time=49.400 ms
84 bytes from 192.168.20.203 icmp_seq=3 ttl=62 time=45.818 ms
84 bytes from 192.168.20.203 icmp_seq=4 ttl=62 time=37.500 ms
84 bytes from 192.168.20.203 icmp_seq=5 ttl=62 time=38.538 ms

VPC_1> trace 192.168.20.203
trace to 192.168.20.203, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   7.027 ms  7.656 ms  8.864 ms
 2   192.168.10.1   40.770 ms  56.893 ms  31.416 ms
 3   *192.168.20.203   46.650 ms (ICMP type:3, code:3, Destination port unreachable)
```
</details>
<details>
<summary> Проверка доступности VPC_1 (Leaf-1) <-> VPC_4 (Leaf-3) </summary>

```
VPC_1> ping 192.168.20.204 

84 bytes from 192.168.20.204 icmp_seq=1 ttl=62 time=69.863 ms
84 bytes from 192.168.20.204 icmp_seq=2 ttl=62 time=36.451 ms
84 bytes from 192.168.20.204 icmp_seq=3 ttl=62 time=44.148 ms
84 bytes from 192.168.20.204 icmp_seq=4 ttl=62 time=43.580 ms
84 bytes from 192.168.20.204 icmp_seq=5 ttl=62 time=41.887 ms

VPC_1> trace 192.168.20.204
trace to 192.168.20.204, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   8.327 ms  9.068 ms  10.270 ms
 2   192.168.20.1   32.358 ms  29.231 ms  29.078 ms
 3   *192.168.20.204   37.781 ms (ICMP type:3, code:3, Destination port unreachable)
```
</details>
