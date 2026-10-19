### Проектная работа - "Применение VxLAN EVPN в кампусной сети доступа предприятия"

### Задание
- Развернуть стенд сети доступа с применением VxLAN EVPN для заданной структуры (схемы):
![task.jpg](task.jpg)

#### Дополнительные требования
- В качестве внутреннего протокола маршрутизации Underlay-сети использовать iBGP.
- Использовать минимальное кол-во соединение между ЦОД и РЦОД.
- Добиться максимальной изоляции, подлкючаемых хостов (клиентов).

### Схема стенда
![stand.jpg](stand.jpg)

### Описание
На стенде используется минимальное кол-во узлов для имитации работы заданной сети.
#### LEAF
- Не являются клиентами RR SPINE. Получают маршруты LOOPBACK только от BORDER LEAF. Связность между LEAF на уровне Underlay-сети отсутствует, что соответствует требованию максимальной изоляции клиентов.
- Имеют основной и резервный маршруты (через другой ЦОД) до каждого BORDER LEAF на случай выхода из строя одного из SPINE, к которым они подключены.
#### BORDER LEAF
- Добавлены дополнительные линки для обеспечения связности между ЦОД и РЦОД.
- Являются RR для BORDER LEAF в противоположном ЦОД-е.
- На каждой паре настроен EVPN ETHERNET-SEGMENT для подключения FW агрегированным L2-каналом (LACP).
- Добавлены дополнительные правила фильтрации маршрутной информации ("костыли") для избегания петель и стабильной работы сети.
#### SPINE
- Участвуют только в L3-взаимодествии Underlay-сети (only bgp address-family ipv4) .
- Являются RR для BORDER LEAF.
#### FW
- Подключаются к паре BORDER LEAF в каждом ЦОД агрегированным L2-каналом (LACP). Между BORDER LEAF в одном ЦОД настроен MuLtihoming.
- Настроены VRRP-группы для шлюзов каждой сети (VLAN 10, VLAN 20). Схема работы: Active-Standby.
- Включена маршрутизация между VLAN 10 и VLAN 20.

### Настройки
<details>
<summary> LEAF-1 </summary>

```
!
service routing protocols model multi-agent
!
hostname LEAF-1
!
vlan 10
!
interface Ethernet1
   description SPINE-1-1
   mtu 9000
   no switchport
   ip address 10.0.1.0/31
!
interface Ethernet2
   description SPINE-2-1
   mtu 9000
   no switchport
   ip address 10.0.1.2/31
!
interface Ethernet3
   switchport access vlan 10
   spanning-tree portfast
!
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
   timers bgp 1 3
   maximum-paths 2 ecmp 2
   neighbor BORDER_LEAF_OVERLAY peer group
   neighbor BORDER_LEAF_OVERLAY remote-as 65000
   neighbor BORDER_LEAF_OVERLAY update-source Loopback0
   neighbor BORDER_LEAF_OVERLAY send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor 10.0.1.1 peer group SPINE
   neighbor 10.0.1.3 peer group SPINE
   neighbor 172.16.1.1 peer group BORDER_LEAF_OVERLAY
   neighbor 172.16.1.2 peer group BORDER_LEAF_OVERLAY
   neighbor 172.16.1.3 peer group BORDER_LEAF_OVERLAY
   neighbor 172.16.1.4 peer group BORDER_LEAF_OVERLAY
   !
   vlan 10
      rd 172.16.0.3:10
      route-target both 65000:10010
      redistribute learned
   !
   address-family evpn
      neighbor BORDER_LEAF_OVERLAY activate
   !
   address-family ipv4
      neighbor SPINE activate
      redistribute connected route-map REDISTRIBUTE_ONLY_LOOPBACKS
!
end
```
</details>
<details>
<summary> LEAF-2 </summary>

```
!
service routing protocols model multi-agent
!
hostname LEAF-2
!
vlan 10,20
!
interface Ethernet1
   description SPINE-1-1
   mtu 9000
   no switchport
   ip address 10.0.1.4/31
!
interface Ethernet2
   description SPINE-2-1
   mtu 9000
   no switchport
   ip address 10.0.1.6/31
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
   timers bgp 1 3
   maximum-paths 2 ecmp 2
   neighbor BORDER_LEAF_OVERLAY peer group
   neighbor BORDER_LEAF_OVERLAY remote-as 65000
   neighbor BORDER_LEAF_OVERLAY update-source Loopback0
   neighbor BORDER_LEAF_OVERLAY send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor 10.0.1.5 peer group SPINE
   neighbor 10.0.1.7 peer group SPINE
   neighbor 172.16.1.1 peer group BORDER_LEAF_OVERLAY
   neighbor 172.16.1.2 peer group BORDER_LEAF_OVERLAY
   neighbor 172.16.1.3 peer group BORDER_LEAF_OVERLAY
   neighbor 172.16.1.4 peer group BORDER_LEAF_OVERLAY
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
      neighbor BORDER_LEAF_OVERLAY activate
   !
   address-family ipv4
      neighbor SPINE activate
      redistribute connected route-map REDISTRIBUTE_ONLY_LOOPBACKS
!
end
```
</details>
<details>
<summary> BORDER-LEAF-1-1 </summary>

```
!
service routing protocols model multi-agent
!
hostname BORDER-LEAF-1-1
!
vlan 10,20
!
interface Port-Channel1
   description TO_Router
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:0001
      designated-forwarder election algorithm preference 20
      route-target import 00:00:00:00:00:01
   lacp system-id 1111.2222.3333
!
interface Ethernet1
   description to-Spine-1
   mtu 9000
   no switchport
   ip address 10.0.2.0/31
!
interface Ethernet5
   description BORDER_LEAF_2_1
   mtu 9000
   no switchport
   ip address 10.10.0.0/31
!
interface Ethernet6
   description BORDER_LEAF_2_2
   mtu 9000
   no switchport
   ip address 10.10.0.2/31
!
interface Ethernet8
   description TO_Router
   switchport mode trunk
   channel-group 1 mode active
!
interface Loopback0
   description Router-ID
   ip address 172.16.1.1/32
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
!
ip routing
!
route-map LEAF_+_LOOPBACKS permit 10
   description LOCAL_LOOPBACK
   match interface Loopback0
!
route-map LEAF_+_LOOPBACKS permit 100
   description LEAFs_LOOPBACKs
   match local-preference 100
!
route-map NO_LP_50 deny 50
   match local-preference 50
!
route-map NO_LP_50 permit 100
!
route-map REDISTRIBUTE_ONLY_LOOPBACKS permit 10
   match interface Loopback0
!
route-map SET_LP_50 permit 50
   set local-preference 50
!
route-map SET_LP_75_LOOPBACK permit 75
   match interface Loopback0
   set local-preference 75
!
route-map SET_LP_75_LOOPBACK permit 100
!
router bgp 65000
   router-id 172.16.1.1
   no bgp default ipv4-unicast
   timers bgp 1 3
   maximum-paths 4 ecmp 4
   bgp listen range 172.16.0.0/24 peer-group LEAF_OVERLAY remote-as 65000
   neighbor BORDER_LEAF_DC_2 peer group
   neighbor BORDER_LEAF_DC_2 remote-as 65000
   neighbor BORDER_LEAF_DC_2 next-hop-self
   neighbor BORDER_LEAF_DC_2 route-reflector-client
   neighbor BORDER_LEAF_OVERLAY peer group
   neighbor BORDER_LEAF_OVERLAY remote-as 65000
   neighbor BORDER_LEAF_OVERLAY update-source Loopback0
   neighbor BORDER_LEAF_OVERLAY send-community extended
   neighbor LEAF_OVERLAY peer group
   neighbor LEAF_OVERLAY remote-as 65000
   neighbor LEAF_OVERLAY update-source Loopback0
   neighbor LEAF_OVERLAY send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE next-hop-self
   neighbor 10.0.2.1 peer group SPINE
   neighbor 10.10.0.1 peer group BORDER_LEAF_DC_2
   neighbor 10.10.0.3 peer group BORDER_LEAF_DC_2
   neighbor 172.16.1.2 peer group BORDER_LEAF_OVERLAY
   neighbor 172.16.1.3 peer group BORDER_LEAF_OVERLAY
   neighbor 172.16.1.4 peer group BORDER_LEAF_OVERLAY
   !
   vlan 10
      rd 172.16.1.1:10
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd 172.16.1.1:20
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor BORDER_LEAF_OVERLAY activate
      neighbor LEAF_OVERLAY activate
   !
   address-family ipv4
      neighbor BORDER_LEAF_DC_2 activate
      neighbor BORDER_LEAF_DC_2 route-map SET_LP_50 in
      neighbor BORDER_LEAF_DC_2 route-map LEAF_+_LOOPBACKS out
      neighbor SPINE activate
      neighbor SPINE route-map NO_LP_50 in
      neighbor SPINE route-map SET_LP_75_LOOPBACK out
      redistribute connected route-map REDISTRIBUTE_ONLY_LOOPBACKS
!
end
```
</details>

<details>
<summary> BORDER-LEAF-1-2 </summary>

```
!
service routing protocols model multi-agent
!
hostname BORDER-LEAF-1-2
!
vlan 10,20
!
interface Port-Channel1
   description TO_Router
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:0001
      designated-forwarder election algorithm preference 50
      route-target import 00:00:00:00:00:01
   lacp system-id 1111.2222.3333
!
interface Ethernet1
   description to-Spine-1
   mtu 9000
   no switchport
   ip address 10.0.3.0/31
!
!
interface Ethernet5
   description BORDER-LEAF-2-2
   mtu 9000
   no switchport
   ip address 10.10.0.6/31
!
interface Ethernet6
   description BORDER-LEAF-2-1
   mtu 9000
   no switchport
   ip address 10.10.0.4/31
!
interface Ethernet8
   switchport mode trunk
   channel-group 1 mode active
!
interface Loopback0
   description Router-ID
   ip address 172.16.1.2/32
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
!
ip routing
!
route-map LEAF_+_LOOPBACKS permit 10
   description LOCAL_LOOPBACK
   match interface Loopback0
!
route-map LEAF_+_LOOPBACKS permit 100
   description LEAFs_LOOPBACKs
   match local-preference 100
!
route-map NO_LP_50 deny 50
   match local-preference 50
!
route-map NO_LP_50 permit 100
!
route-map REDISTRIBUTE_ONLY_LOOPBACKS permit 10
   match interface Loopback0
!
route-map SET_LP_50 permit 50
   set local-preference 50
!
route-map SET_LP_75_LOOPBACK permit 75
   match interface Loopback0
   set local-preference 75
!
route-map SET_LP_75_LOOPBACK permit 100
!
router bgp 65000
   router-id 172.16.1.2
   no bgp default ipv4-unicast
   timers bgp 1 3
   maximum-paths 4 ecmp 4
   bgp listen range 172.16.0.0/24 peer-group LEAF_OVERLAY remote-as 65000
   neighbor BORDER_LEAF_2 peer group
   neighbor BORDER_LEAF_DC_2 peer group
   neighbor BORDER_LEAF_DC_2 remote-as 65000
   neighbor BORDER_LEAF_DC_2 next-hop-self
   neighbor BORDER_LEAF_DC_2 route-reflector-client
   neighbor BORDER_LEAF_OVERLAY peer group
   neighbor BORDER_LEAF_OVERLAY remote-as 65000
   neighbor BORDER_LEAF_OVERLAY update-source Loopback0
   neighbor BORDER_LEAF_OVERLAY send-community extended
   neighbor LEAF_OVERLAY peer group
   neighbor LEAF_OVERLAY remote-as 65000
   neighbor LEAF_OVERLAY update-source Loopback0
   neighbor LEAF_OVERLAY send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE next-hop-self
   neighbor 10.0.3.1 peer group SPINE
   neighbor 10.10.0.5 peer group BORDER_LEAF_DC_2
   neighbor 10.10.0.7 peer group BORDER_LEAF_DC_2
   neighbor 172.16.1.1 peer group BORDER_LEAF_OVERLAY
   neighbor 172.16.1.3 peer group BORDER_LEAF_OVERLAY
   neighbor 172.16.1.4 peer group BORDER_LEAF_OVERLAY
   !
   vlan 10
      rd 172.16.1.2:10
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd 172.16.1.2:20
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor BORDER_LEAF_OVERLAY activate
      neighbor LEAF_OVERLAY activate
   !
   address-family ipv4
      neighbor BORDER_LEAF_DC_2 activate
      neighbor BORDER_LEAF_DC_2 route-map SET_LP_50 in
      neighbor BORDER_LEAF_DC_2 route-map LEAF_+_LOOPBACKS out
      neighbor SPINE activate
      neighbor SPINE route-map NO_LP_50 in
      neighbor SPINE route-map SET_LP_75_LOOPBACK out
      redistribute connected route-map REDISTRIBUTE_ONLY_LOOPBACKS
!
end
```
</details>

<details>
<summary> BORDER-LEAF-2-1 </summary>

```

```
</details>

<details>
<summary> BORDER-LEAF-2-2 </summary>

```

```
</details>

<details>
<summary> FW-1-1 </summary>

```

```
</details>

<details>
<summary> FW-2-1 </summary>

```

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
