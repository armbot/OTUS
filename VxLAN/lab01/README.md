### Проектирование адресного пространства

### Цель
- собрать схему CLOS и распределить адресное пространство.

### Схема стенда

![lab1_scheme.jpg](lab1_scheme.jpg)

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

### Проверка IP-связности между интерфейсами Loopback 0
#### Leaf-1
```
Leaf-1#show ip route 
 C        10.0.1.0/31 is directly connected, Ethernet1
 C        10.0.1.2/31 is directly connected, Ethernet2
 B I      172.16.0.1/32 [200/0] via 10.0.1.1, Ethernet1
 B I      172.16.0.2/32 [200/0] via 10.0.1.3, Ethernet2
 C        172.16.0.3/32 is directly connected, Loopback0
 B I      172.16.0.4/32 [200/0] via 10.0.1.1, Ethernet1
                                via 10.0.1.3, Ethernet2
 B I      172.16.0.5/32 [200/0] via 10.0.1.1, Ethernet1
                                via 10.0.1.3, Ethernet2
```

<details>
<summary> Проверка доступности Spine-1 </summary>

```
Leaf-1#ping 172.16.0.1 source loopback 0
PING 172.16.0.1 (172.16.0.1) from 172.16.0.3 : 72(100) bytes of data.
80 bytes from 172.16.0.1: icmp_seq=1 ttl=64 time=8.96 ms
80 bytes from 172.16.0.1: icmp_seq=2 ttl=64 time=11.9 ms
80 bytes from 172.16.0.1: icmp_seq=3 ttl=64 time=11.7 ms
80 bytes from 172.16.0.1: icmp_seq=4 ttl=64 time=8.79 ms
80 bytes from 172.16.0.1: icmp_seq=5 ttl=64 time=11.2 ms
```
