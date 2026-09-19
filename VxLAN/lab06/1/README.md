### Настройка маршрутизации в VxLAN EVPN (Bridged Overlay)

### Схема стенда

![lab6_1_scheme.jpg](lab6_1_scheme.jpg)

### Описание
- VxLAN EVPN L2-сеть взята из предыдущей работы - [Lab05. Overlay на основе VxLAN EVPN для L2 связанности между клиентами](https://github.com/armbot/OTUS/tree/9505106f8681b0b35010acc5577b91d84ab4c4a9/VxLAN/lab05).
- Добавлен элемент Router с функцией маршрутизации между подсетями (Router-on-Stick).
- На Router настроены шлюзы сетей 192.168.10.1 и 192.168.20.1.

### Настройки
<details>
<summary> Leaf-2 </summary>

```
!
interface Ethernet8
   switchport mode trunk
!
```
</details>
<details>
<summary> Router </summary>

```
!
hostname Router
!
vlan 10,20
!
interface Ethernet1
   switchport mode trunk
!
interface Vlan10
   no autostate
   ip address 192.168.10.1/24
!
interface Vlan20
   no autostate
   ip address 192.168.20.1/24
!
ip routing
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
