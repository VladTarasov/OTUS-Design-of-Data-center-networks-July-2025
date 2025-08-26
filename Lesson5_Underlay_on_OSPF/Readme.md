# **Диаграмма сети**

<img width="1739" height="1203" alt="image" src="https://github.com/user-attachments/assets/577bb2f3-46dc-43a5-b4a8-930646e7b515" />

# **Адресный план**

Распределение адресов для loopback интерфейсов /32:

| Loopback         | Spine1   | Spine2   | Leaf1    |Leaf2     |Leaf3     |
| ---------------- |:--------:| --------:|----------|----------|----------|    
| Lo1              | 10.0.1.0 | 10.0.2.0 | 10.0.1.1 | 10.0.1.2 | 10.0.1.3 |
| Lo2              | 10.1.1.0 | 10.1.2.0 | 10.1.1.1 | 10.1.1.2 | 10.1.1.3 |

Распределение адресов для p2p интерфейсов /31:

|                  | Leaf1   |Leaf2    |Leaf3    |Spine1   |Spine2   |
| ---------------- |:-------:| -------:|---------|---------|---------|    
| Spine1           | 10.2.1.0| 10.2.1.2| 10.2.1.4|         |         |
| Spine2           | 10.2.2.0| 10.2.2.2| 10.2.2.4|         |         |
| Leaf1            |         |         |         |10.2.1.1 |10.2.2.1 |
| Leaf2            |         |         |         |10.2.1.3 |10.2.2.3 |
| Leaf3            |         |         |         |10.2.1.5 |10.2.2.5 |

# **Конфигурация**

Общие замечания:
- На всех маршрутизаторах создается процесс ospf под номером 1.
- Так как рассматривается топология с единственным POD, то используется только backbone area 0.
- Loopback-интефейсы являются пассивными.
- Используется аутентификация md5.
- На соединениях Spine-Leaf используется тип сети point-to-point.
- На соединениях Spine-Leaf настроен BFD.
```
Spine1#sh run | s r ospf
interface Ethernet1
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 syd9D76nV5B/SVQeMxRyDg==
interface Ethernet2
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 nStT+wbPlNmDOUraWn0fVw==
interface Ethernet3
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 nStT+wbPlNmDOUraWn0fVw==
interface Loopback1
   ip ospf area 0.0.0.0
interface Loopback2
   ip ospf area 0.0.0.0
router ospf 1
   router-id 10.0.1.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000

Spine2#sh run | s r ospf
interface Ethernet1
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 syd9D76nV5B/SVQeMxRyDg==
interface Ethernet2
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 nStT+wbPlNmDOUraWn0fVw==
interface Ethernet3
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 nStT+wbPlNmDOUraWn0fVw==
interface Loopback1
   ip ospf area 0.0.0.0
interface Loopback2
   ip ospf area 0.0.0.0
router ospf 1
   router-id 10.0.2.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000

Leaf1#sh run | s r ospf
interface Ethernet1
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 syd9D76nV5B/SVQeMxRyDg==
interface Ethernet2
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 nStT+wbPlNmDOUraWn0fVw==
interface Loopback1
   ip ospf area 0.0.0.0
interface Loopback2
   ip ospf area 0.0.0.0
router ospf 1
   router-id 10.0.1.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000

Leaf2#sh run | s r ospf
interface Ethernet1
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 syd9D76nV5B/SVQeMxRyDg==
interface Ethernet2
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 nStT+wbPlNmDOUraWn0fVw==
interface Loopback1
   ip ospf area 0.0.0.0
interface Loopback2
   ip ospf area 0.0.0.0
router ospf 1
   router-id 10.0.1.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000

Leaf3#sh run | s r ospf
interface Ethernet1
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 syd9D76nV5B/SVQeMxRyDg==
interface Ethernet2
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 nStT+wbPlNmDOUraWn0fVw==
interface Loopback1
   ip ospf area 0.0.0.0
interface Loopback2
   ip ospf area 0.0.0.0
router ospf 1
   router-id 10.0.1.3
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
```

# **Проверка работоспособности**

Проверяем соблюдение следующих условий:
- На каждом Spine установлены соседсва с Leaf1-3, состояние FULL.
- В таблице маршрутизации присутствует несколько маршурутов до looback-интерфейсов и маршруты до p2p-линковых подсетей.
- Со Spine1 доступны все loopback-интерфейсы Spine2 и Leaf1-3.

```
Spine1#sh ip ospf nei
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.1.1        1        default  0   FULL                   00:00:30    10.2.1.1        Ethernet1
10.0.1.2        1        default  0   FULL                   00:00:35    10.2.1.3        Ethernet2
10.0.1.3        1        default  0   FULL                   00:00:38    10.2.1.5        Ethernet3

Spine1#sh ip route ospf

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 O        10.0.1.1/32 [110/20] via 10.2.1.1, Ethernet1
 O        10.0.1.2/32 [110/20] via 10.2.1.3, Ethernet2
 O        10.0.1.3/32 [110/20] via 10.2.1.5, Ethernet3
 O        10.0.2.0/32 [110/30] via 10.2.1.1, Ethernet1
                               via 10.2.1.3, Ethernet2
                               via 10.2.1.5, Ethernet3
 O        10.1.1.1/32 [110/20] via 10.2.1.1, Ethernet1
 O        10.1.1.2/32 [110/20] via 10.2.1.3, Ethernet2
 O        10.1.1.3/32 [110/20] via 10.2.1.5, Ethernet3
 O        10.1.2.0/32 [110/30] via 10.2.1.1, Ethernet1
                               via 10.2.1.3, Ethernet2
                               via 10.2.1.5, Ethernet3
 O        10.2.2.0/31 [110/20] via 10.2.1.1, Ethernet1
 O        10.2.2.2/31 [110/20] via 10.2.1.3, Ethernet2
 O        10.2.2.4/31 [110/20] via 10.2.1.5, Ethernet3

Spine2#sh ip ospf ne
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.1.1        1        default  0   FULL                   00:00:35    10.2.2.1        Ethernet1
10.0.1.2        1        default  0   FULL                   00:00:32    10.2.2.3        Ethernet2
10.0.1.3        1        default  0   FULL                   00:00:35    10.2.2.5        Ethernet3

Spine2#sh ip route ospf

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 O        10.0.1.0/32 [110/30] via 10.2.2.1, Ethernet1
                               via 10.2.2.3, Ethernet2
                               via 10.2.2.5, Ethernet3
 O        10.0.1.1/32 [110/20] via 10.2.2.1, Ethernet1
 O        10.0.1.2/32 [110/20] via 10.2.2.3, Ethernet2
 O        10.0.1.3/32 [110/20] via 10.2.2.5, Ethernet3
 O        10.1.1.0/32 [110/30] via 10.2.2.1, Ethernet1
                               via 10.2.2.3, Ethernet2
                               via 10.2.2.5, Ethernet3
 O        10.1.1.1/32 [110/20] via 10.2.2.1, Ethernet1
 O        10.1.1.2/32 [110/20] via 10.2.2.3, Ethernet2
 O        10.1.1.3/32 [110/20] via 10.2.2.5, Ethernet3
 O        10.2.1.0/31 [110/20] via 10.2.2.1, Ethernet1
 O        10.2.1.2/31 [110/20] via 10.2.2.3, Ethernet2
 O        10.2.1.4/31 [110/20] via 10.2.2.5, Ethernet3

Leaf1#sh ip ospf nei
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.1.0        1        default  0   FULL                   00:00:35    10.2.1.0        Ethernet1
10.0.2.0        1        default  0   FULL                   00:00:29    10.2.2.0        Ethernet2

Leaf1#sh ip route ospf

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 O        10.0.1.0/32 [110/20] via 10.2.1.0, Ethernet1
 O        10.0.1.2/32 [110/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 O        10.0.1.3/32 [110/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 O        10.0.2.0/32 [110/20] via 10.2.2.0, Ethernet2
 O        10.1.1.0/32 [110/20] via 10.2.1.0, Ethernet1
 O        10.1.1.2/32 [110/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 O        10.1.1.3/32 [110/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 O        10.1.2.0/32 [110/20] via 10.2.2.0, Ethernet2
 O        10.2.1.2/31 [110/20] via 10.2.1.0, Ethernet1
 O        10.2.1.4/31 [110/20] via 10.2.1.0, Ethernet1
 O        10.2.2.2/31 [110/20] via 10.2.2.0, Ethernet2
 O        10.2.2.4/31 [110/20] via 10.2.2.0, Ethernet2

Leaf2#sh ip ospf nei
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.1.0        1        default  0   FULL                   00:00:30    10.2.1.2        Ethernet1
10.0.2.0        1        default  0   FULL                   00:00:33    10.2.2.2        Ethernet2

Leaf2# sh ip route ospf

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 O        10.0.1.0/32 [110/20] via 10.2.1.2, Ethernet1
 O        10.0.1.1/32 [110/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 O        10.0.1.3/32 [110/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 O        10.0.2.0/32 [110/20] via 10.2.2.2, Ethernet2
 O        10.1.1.0/32 [110/20] via 10.2.1.2, Ethernet1
 O        10.1.1.1/32 [110/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 O        10.1.1.3/32 [110/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 O        10.1.2.0/32 [110/20] via 10.2.2.2, Ethernet2
 O        10.2.1.0/31 [110/20] via 10.2.1.2, Ethernet1
 O        10.2.1.4/31 [110/20] via 10.2.1.2, Ethernet1
 O        10.2.2.0/31 [110/20] via 10.2.2.2, Ethernet2
 O        10.2.2.4/31 [110/20] via 10.2.2.2, Ethernet2

Leaf3#sh ip ospf nei
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.1.0        1        default  0   FULL                   00:00:32    10.2.1.4        Ethernet1
10.0.2.0        1        default  0   FULL                   00:00:30    10.2.2.4        Ethernet2

Leaf3#sh ip route ospf

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 O        10.0.1.0/32 [110/20] via 10.2.1.4, Ethernet1
 O        10.0.1.1/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.0.1.2/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.0.2.0/32 [110/20] via 10.2.2.4, Ethernet2
 O        10.1.1.0/32 [110/20] via 10.2.1.4, Ethernet1
 O        10.1.1.1/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.1.1.2/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.1.2.0/32 [110/20] via 10.2.2.4, Ethernet2
 O        10.2.1.0/31 [110/20] via 10.2.1.4, Ethernet1
 O        10.2.1.2/31 [110/20] via 10.2.1.4, Ethernet1
 O        10.2.2.0/31 [110/20] via 10.2.2.4, Ethernet2
 O        10.2.2.2/31 [110/20] via 10.2.2.4, Ethernet2


Spine1#ping 10.0.2.0
PING 10.0.2.0 (10.0.2.0) 72(100) bytes of data.
80 bytes from 10.0.2.0: icmp_seq=1 ttl=63 time=10.4 ms
80 bytes from 10.0.2.0: icmp_seq=2 ttl=63 time=4.21 ms
80 bytes from 10.0.2.0: icmp_seq=3 ttl=63 time=4.36 ms
80 bytes from 10.0.2.0: icmp_seq=4 ttl=63 time=4.30 ms
80 bytes from 10.0.2.0: icmp_seq=5 ttl=63 time=4.08 ms

--- 10.0.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 39ms
rtt min/avg/max/mdev = 4.084/5.474/10.413/2.472 ms, ipg/ewma 9.772/7.855 ms
Spine1#ping 10.1.2.0
PING 10.1.2.0 (10.1.2.0) 72(100) bytes of data.
80 bytes from 10.1.2.0: icmp_seq=1 ttl=63 time=11.3 ms
80 bytes from 10.1.2.0: icmp_seq=2 ttl=63 time=4.45 ms
80 bytes from 10.1.2.0: icmp_seq=3 ttl=63 time=5.22 ms
80 bytes from 10.1.2.0: icmp_seq=4 ttl=63 time=5.11 ms
80 bytes from 10.1.2.0: icmp_seq=5 ttl=63 time=4.60 ms

--- 10.1.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 42ms
rtt min/avg/max/mdev = 4.456/6.150/11.362/2.623 ms, ipg/ewma 10.518/8.667 ms
Spine1#ping 10.0.1.1
PING 10.0.1.1 (10.0.1.1) 72(100) bytes of data.
80 bytes from 10.0.1.1: icmp_seq=1 ttl=64 time=6.62 ms
80 bytes from 10.0.1.1: icmp_seq=2 ttl=64 time=2.70 ms
80 bytes from 10.0.1.1: icmp_seq=3 ttl=64 time=2.33 ms
80 bytes from 10.0.1.1: icmp_seq=4 ttl=64 time=2.09 ms
80 bytes from 10.0.1.1: icmp_seq=5 ttl=64 time=2.31 ms

--- 10.0.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 26ms
rtt min/avg/max/mdev = 2.094/3.215/6.624/1.716 ms, ipg/ewma 6.679/4.851 ms
Spine1#ping 10.1.1.1
PING 10.1.1.1 (10.1.1.1) 72(100) bytes of data.
80 bytes from 10.1.1.1: icmp_seq=1 ttl=64 time=3.19 ms
80 bytes from 10.1.1.1: icmp_seq=2 ttl=64 time=2.22 ms
80 bytes from 10.1.1.1: icmp_seq=3 ttl=64 time=2.49 ms
80 bytes from 10.1.1.1: icmp_seq=4 ttl=64 time=2.04 ms
80 bytes from 10.1.1.1: icmp_seq=5 ttl=64 time=2.21 ms

--- 10.1.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 15ms
rtt min/avg/max/mdev = 2.047/2.434/3.190/0.406 ms, ipg/ewma 3.989/2.796 ms
Spine1#ping 10.0.1.2
PING 10.0.1.2 (10.0.1.2) 72(100) bytes of data.
80 bytes from 10.0.1.2: icmp_seq=1 ttl=64 time=6.33 ms
80 bytes from 10.0.1.2: icmp_seq=2 ttl=64 time=2.06 ms
80 bytes from 10.0.1.2: icmp_seq=3 ttl=64 time=2.08 ms
80 bytes from 10.0.1.2: icmp_seq=4 ttl=64 time=2.54 ms
80 bytes from 10.0.1.2: icmp_seq=5 ttl=64 time=2.07 ms

--- 10.0.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 23ms
rtt min/avg/max/mdev = 2.066/3.022/6.337/1.667 ms, ipg/ewma 5.990/4.625 ms
Spine1#ping 10.1.1.2
PING 10.1.1.2 (10.1.1.2) 72(100) bytes of data.
80 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=3.93 ms
80 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=2.70 ms
80 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=2.41 ms
80 bytes from 10.1.1.2: icmp_seq=4 ttl=64 time=2.04 ms
80 bytes from 10.1.1.2: icmp_seq=5 ttl=64 time=2.13 ms

--- 10.1.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 16ms
rtt min/avg/max/mdev = 2.046/2.646/3.932/0.685 ms, ipg/ewma 4.183/3.253 ms
Spine1#ping 10.0.1.3
PING 10.0.1.3 (10.0.1.3) 72(100) bytes of data.
80 bytes from 10.0.1.3: icmp_seq=1 ttl=64 time=6.80 ms
80 bytes from 10.0.1.3: icmp_seq=2 ttl=64 time=3.17 ms
80 bytes from 10.0.1.3: icmp_seq=3 ttl=64 time=2.53 ms
80 bytes from 10.0.1.3: icmp_seq=4 ttl=64 time=2.02 ms
80 bytes from 10.0.1.3: icmp_seq=5 ttl=64 time=3.14 ms

--- 10.0.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 25ms
rtt min/avg/max/mdev = 2.022/3.535/6.807/1.691 ms, ipg/ewma 6.283/5.112 ms
Spine1#ping 10.1.1.3
PING 10.1.1.3 (10.1.1.3) 72(100) bytes of data.
80 bytes from 10.1.1.3: icmp_seq=1 ttl=64 time=3.23 ms
80 bytes from 10.1.1.3: icmp_seq=2 ttl=64 time=2.43 ms
80 bytes from 10.1.1.3: icmp_seq=3 ttl=64 time=2.24 ms
80 bytes from 10.1.1.3: icmp_seq=4 ttl=64 time=2.10 ms
80 bytes from 10.1.1.3: icmp_seq=5 ttl=64 time=2.16 ms

--- 10.1.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 15ms
rtt min/avg/max/mdev = 2.103/2.434/3.235/0.415 ms, ipg/ewma 3.936/2.814 ms
```
