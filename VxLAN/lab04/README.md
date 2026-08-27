### Построение Underlay сети (iBGP)

### Цель
- настроить iBGP для Underlay сети.

### Схема стенда

![ISIS_scheme.jpg](ISIS_scheme.jpg)

### Таблица IP-адресов Loopback-ов

|Device|Interface|IP Address|
|---|---|---|
Spine-1|loopback 0|172.16.0.1/32
Spine-2|loopback 0|172.16.0.2/32
Leaf-1|loopback 0|172.16.0.3/32
Leaf-2|loopback 0|172.16.0.4/32
Leaf-3|loopback 0|172.16.0.5/32

### Таблица IP-адресов P2P-сетей

|Линк|IP Лифа|IP Спайна|Подсеть /31|
|---|---|---|---|
Leaf-1 → Spine-1|10.0.1.0/31|10.0.1.1/31|10.0.1.0/31
Leaf-1 → Spine-2|10.0.1.2/31|10.0.1.3/31|10.0.1.2/31

|Линк|IP Лифа|IP Спайна|Подсеть /31|
|---|---|---|---|
Leaf-2 → Spine-1|10.0.2.0/31|10.0.2.1/31|10.0.2.0/31
Leaf-2 → Spine-2|10.0.2.2/31|10.0.2.3/31|10.0.2.2/31

|Линк|IP Лифа|IP Спайна|Подсеть /31|
|---|---|---|---|
Leaf-3 → Spine-1|10.0.3.0/31|10.0.3.1/31|10.0.3.0/31
Leaf-3 → Spine-2|10.0.3.2/31|10.0.3.3/31|10.0.3.2/31

### Настройка ISIS
<details>
<summary> Spine-1 </summary>

```
hostname Spine-1
!
interface Ethernet1
   description to-Leaf-1
   mtu 9000
   no switchport
   ip address 10.0.1.1/31
   isis enable UNDERLAY
   isis circuit-type level-1
   isis network point-to-point
!
interface Ethernet2
   description to-Leaf-2
   mtu 9000
   no switchport
   ip address 10.0.2.1/31
   isis enable UNDERLAY
   isis circuit-type level-1
   isis network point-to-point
!
interface Ethernet3
   description to-Leaf-3
   mtu 9000
   no switchport
   ip address 10.0.3.1/31
   isis enable UNDERLAY
   isis circuit-type level-1
   isis network point-to-point
!
interface Loopback0
   description Router-ID
   ip address 172.16.0.1/32
   isis enable UNDERLAY
   isis passive
!
ip routing
!
router isis UNDERLAY
   net 49.0001.1111.1111.1111.00
   !
   address-family ipv4 unicast
!
end
```
</details>
<details>
<summary> Spine-2 </summary>

```
hostname Spine-2
!
interface Ethernet1
   description to-Leaf-1
   mtu 9000
   no switchport
   ip address 10.0.1.3/31
   isis enable UNDERLAY
   isis circuit-type level-1
   isis network point-to-point
!
interface Ethernet2
   description to-Leaf-2
   mtu 9000
   no switchport
   ip address 10.0.2.3/31
   isis enable UNDERLAY
   isis circuit-type level-1
   isis network point-to-point
!
interface Ethernet3
   description to-Leaf-3
   mtu 9000
   no switchport
   ip address 10.0.3.3/31
   isis enable UNDERLAY
   isis circuit-type level-1
   isis network point-to-point
!
interface Loopback0
   description Router-ID
   ip address 172.16.0.2/32
   isis enable UNDERLAY
!
ip routing
!
router isis UNDERLAY
   net 49.0001.2222.2222.2222.00
   !
   address-family ipv4 unicast
!
end
```
</details>
<details>
<summary> Leaf-1 </summary>

```
hostname Leaf-1
!
interface Ethernet1
   description to-Spine-1
   mtu 9000
   no switchport
   ip address 10.0.1.0/31
   isis enable UNDERLAY
   isis circuit-type level-1
   isis network point-to-point
!
interface Ethernet2
   description to-Spine-2
   mtu 9000
   no switchport
   ip address 10.0.1.2/31
   isis enable UNDERLAY
   isis circuit-type level-1
!
interface Loopback0
   description Router-ID
   ip address 172.16.0.3/32
   isis enable UNDERLAY
!
ip routing
!
router isis UNDERLAY
   net 49.0001.0001.0001.0001.00
   !
   address-family ipv4 unicast
!
end
```
</details>
<details>
<summary> Leaf-2 </summary>

```
hostname Leaf-2
!
interface Ethernet1
   description to-Spine-1
   mtu 9000
   no switchport
   ip address 10.0.2.0/31
   isis enable UNDERLAY
   isis circuit-type level-1
   isis network point-to-point
!
interface Ethernet2
   description to-Spine-2
   mtu 9000
   no switchport
   ip address 10.0.2.2/31
   isis enable UNDERLAY
   isis circuit-type level-1
   isis network point-to-point
!
interface Loopback0
   description Router-ID
   ip address 172.16.0.4/32
   isis enable UNDERLAY
   isis passive
!
ip routing
!
router isis UNDERLAY
   net 49.0001.0002.0002.0002.00
   !
   address-family ipv4 unicast
!
end
```
</details>
<details>
<summary> Leaf-3 </summary>

```
hostname Leaf-3
!
interface Ethernet1
   description to-Spine-1
   mtu 9000
   no switchport
   ip address 10.0.3.0/31
   isis enable UNDERLAY
   isis circuit-type level-1
   isis network point-to-point
!
interface Ethernet2
   description to-Spine-2
   mtu 9000
   no switchport
   ip address 10.0.3.2/31
   isis enable UNDERLAY
   isis circuit-type level-1
   isis network point-to-point
!
interface Loopback0
   description Router-ID
   ip address 172.16.0.5/32
   isis enable UNDERLAY
   isis passive
!
ip routing
!
router isis UNDERLAY
   net 49.0001.0003.0003.0003.00
   !
   address-family ipv4 unicast
!
end
```
</details>

### Проверка работы протокола ISIS
#### Leaf-1
```
Leaf-1#show isis neighbors 
Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id          
UNDERLAY  default  Spine-1          L1   Ethernet1          P2P               UP    22          0E                  
UNDERLAY  default  Spine-2          L1   Ethernet2          P2P               UP    28          0E  
```
```
Leaf-1#show isis database 
IS-IS Instance: UNDERLAY VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    Leaf-1.00-00                  7  42142  1182    122 L2 <>
    Leaf-2.00-00                  9  11274  1188    122 L2 <>
    Leaf-3.00-00                  8   3104   613    122 L2 <>
    Spine-1.00-00                15  40816   341    147 L2 <>
    Spine-2.00-00                10  58043   550    147 L2 <>
  IS-IS Level 2 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    Leaf-1.00-00                 13    791  1127    164 L2 <>
```
```
Leaf-1#show ip route isis

 I L1     10.0.2.0/31 [115/20] via 10.0.1.1, Ethernet1
 I L1     10.0.2.2/31 [115/20] via 10.0.1.3, Ethernet2
 I L1     10.0.3.0/31 [115/20] via 10.0.1.1, Ethernet1
 I L1     10.0.3.2/31 [115/20] via 10.0.1.3, Ethernet2
 I L1     172.16.0.1/32 [115/20] via 10.0.1.1, Ethernet1
 I L1     172.16.0.2/32 [115/20] via 10.0.1.3, Ethernet2
 I L1     172.16.0.4/32 [115/30] via 10.0.1.1, Ethernet1
                                 via 10.0.1.3, Ethernet2
 I L1     172.16.0.5/32 [115/30] via 10.0.1.1, Ethernet1
                                 via 10.0.1.3, Ethernet2
```
<details>
<summary> Проверка доступности Spine-1 </summary>

```
Leaf-1#ping 172.16.0.1 source loopback 0
PING 172.16.0.1 (172.16.0.1) from 172.16.0.3 : 72(100) bytes of data.
80 bytes from 172.16.0.1: icmp_seq=1 ttl=64 time=18.1 ms
80 bytes from 172.16.0.1: icmp_seq=2 ttl=64 time=21.1 ms
80 bytes from 172.16.0.1: icmp_seq=3 ttl=64 time=14.4 ms
80 bytes from 172.16.0.1: icmp_seq=4 ttl=64 time=7.27 ms
80 bytes from 172.16.0.1: icmp_seq=5 ttl=64 time=8.41 ms

--- 172.16.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 73ms
rtt min/avg/max/mdev = 7.276/13.890/21.155/5.389 ms, pipe 2, ipg/ewma 18.410/15.653 ms
```
</details>
<details>
<summary> Проверка доступности Spine-2 </summary>

```
Leaf-1#ping 172.16.0.2 source loopback 0
PING 172.16.0.2 (172.16.0.2) from 172.16.0.3 : 72(100) bytes of data.
80 bytes from 172.16.0.2: icmp_seq=1 ttl=64 time=16.7 ms
80 bytes from 172.16.0.2: icmp_seq=2 ttl=64 time=19.8 ms
80 bytes from 172.16.0.2: icmp_seq=3 ttl=64 time=14.4 ms
80 bytes from 172.16.0.2: icmp_seq=4 ttl=64 time=12.1 ms
80 bytes from 172.16.0.2: icmp_seq=5 ttl=64 time=9.53 ms

--- 172.16.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 66ms
rtt min/avg/max/mdev = 9.538/14.515/19.806/3.558 ms, pipe 2, ipg/ewma 16.586/15.344 ms
```
</details>
<details>
<summary> Проверка доступности Leaf-2 </summary>

```
Leaf-1#ping 172.16.0.4 source loopback 0
PING 172.16.0.4 (172.16.0.4) from 172.16.0.3 : 72(100) bytes of data.
80 bytes from 172.16.0.4: icmp_seq=1 ttl=63 time=67.3 ms
80 bytes from 172.16.0.4: icmp_seq=2 ttl=63 time=62.0 ms
80 bytes from 172.16.0.4: icmp_seq=3 ttl=63 time=62.0 ms
80 bytes from 172.16.0.4: icmp_seq=4 ttl=63 time=84.4 ms
80 bytes from 172.16.0.4: icmp_seq=5 ttl=63 time=82.7 ms

--- 172.16.0.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 50ms
rtt min/avg/max/mdev = 62.065/71.757/84.490/9.908 ms, pipe 5, ipg/ewma 12.746/70.227 ms
```
</details>
<details>
<summary> Проверка доступности Leaf-3 </summary>

```
Leaf-1#ping 172.16.0.5 source loopback 0
PING 172.16.0.5 (172.16.0.5) from 172.16.0.3 : 72(100) bytes of data.
80 bytes from 172.16.0.5: icmp_seq=1 ttl=63 time=40.6 ms
80 bytes from 172.16.0.5: icmp_seq=2 ttl=63 time=48.5 ms
80 bytes from 172.16.0.5: icmp_seq=3 ttl=63 time=40.6 ms
80 bytes from 172.16.0.5: icmp_seq=4 ttl=63 time=30.3 ms
80 bytes from 172.16.0.5: icmp_seq=5 ttl=63 time=92.9 ms

--- 172.16.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 108ms
