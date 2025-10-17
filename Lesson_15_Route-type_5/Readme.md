# Занятие 15. VxLAN. Оптимизация таблиц маршрутизации

## **План работы**
1. Дизайн
  - Схема
  - Адресное пространство
  - Протоколы и общие замечания
2. Конфигурация
3. Проверка работоспособности

## 1. Дизайн

### Схема
<img width="1853" height="1470" alt="image" src="https://github.com/user-attachments/assets/3923400b-b68b-4c6b-87ee-3ec879860c33" />

### Адресное пространство
Распределение адресов для loopback интерфейсов /32:

| Loopback         | Spine1   | Spine2   | Leaf1    |Leaf2     |Leaf3     |
| ---------------- |:--------:| --------:|----------|----------|----------|    
| Lo1              | 10.0.1.0 | 10.0.2.0 | 10.0.1.1 | 10.0.1.2 | 10.0.1.3 |

Распределение адресов для p2p интерфейсов фабрики /31:

|  Узел/Интерфейс  | Leaf1   |Leaf2     |Leaf3     |Spine1    |Spine2    |
| ---------------- |:-------:| -------: |--------- |----------|----------|    
| Eth1             | 10.2.1.1| 10.2.1.3 | 10.2.1.5 | 10.2.1.0 | 10.2.2.0 |
| Eth2             | 10.2.2.1| 10.2.2.3 | 10.2.2.5 | 10.2.1.2 | 10.2.2.2 |
| Eth3             |         |          |          | 10.2.1.4 | 10.2.2.4 |
| Eth4             |         |          |          | 10.2.1.4 | 10.2.2.4 |

Распределение адресов для подключения нагрузки к фабрике (Хосты, маршрутизаторы, FW):

VLAN 10 192.168.10.0/24 - Подсеть 10
VLAN 20 192.168.10.0/24 - Подсеть 20
VLAN -  192.168.255.0/24 le 30 - подсети для p2p L3 соединений с другим сетевым оборудованием (маршурутизаторы, FW)                                                                                                                                                                                                                                                                                                                                                                                                                                                         
### Протоколы и общие замечания
Задача - организовать хостами LAN_10 и LAN_20 связность на уровне L3 c внешними относительно фабрики сетями (эмуляция доступности Интернет).\
Решение - проанонсировать в фабрику маршрут по умолчанию, ведущий на пограничный маршрутизатор.
- Underlay сеть посроена на eBGP из предыдущих ДЗ.
- BFD отключен для экономии ресурсов стенда.
- Для распространения BUM используем Ingress Replication.
- Модель EVPN L2 сервиса VLAN-based.
- Модель EVPN L3 сервиса Edge-routed Briging Symetric IRB.
- Leaf3 выступает в качетсве boarder leaf на границе с Интернет.
- Связность с пограничным маршрутизатором построена на eBGP.

## 2. Конфигурация
В дополенение к конфигурации EVPN L3 VNI Необходимо:\
На Leaf 1 и 2
- В BGP для VRF S_IRB-100 включить redistribute connected;

На Leaf 3
- Перевести интерфейс Eth4 в режим L3, привязать его к VRF S_IRB-100, назначить IP-адрес.
- В BGP для VRF S_IRB-100 включить redistribute connected с фильтрацией по route-map (чтобы не анонсировать connected стык с RT-EDGE).
- В BGP для VRF S_IRB-100 добавить соседство c RT-EDGE и активировать его в сеймействе адресов ipv4.

На RT-EDGE
- Назначить IP-адрес L3 интерфейсу Gi0/0.
- Назначить IP-адрес L3 интерфейсу Loopback0.
- Запустить BGP процесс, задать соседство с Leaf3, проанонсировать маршрут по умолчанию.

Ниже приведены конфигурации узлов (в части касающейся добавленных настроек).

**Leaf1**
```
router bgp 65101
   router-id 10.0.1.1
   maximum-paths 8 ecmp 8
   neighbor Spine_PG peer group
   neighbor Spine_PG remote-as 65100
   neighbor Spine_PG timers 3 9
   neighbor Spine_PG password 7 iV7A4HcTNGo=
   neighbor Spine_PG send-community extended
   neighbor 10.2.1.0 peer group Spine_PG
   neighbor 10.2.2.0 peer group Spine_PG
   redistribute connected route-map loopback
   !
   vlan 10
      rd auto
      route-target import 65102:10010
      route-target import 65103:10010
      route-target export 65101:10010
      redistribute learned
   !
   address-family evpn
      neighbor Spine_PG activate
   !
   vrf S_IRB-100
      rd 10.0.1.1:100
      route-target import evpn 65102:100
      route-target import evpn 65103:100
      route-target export evpn 65101:100
      redistribute connected
```
**Leaf2**
```
router bgp 65102
   router-id 10.0.1.2
   maximum-paths 8 ecmp 8
   neighbor Spine_PG peer group
   neighbor Spine_PG remote-as 65100
   neighbor Spine_PG timers 3 9
   neighbor Spine_PG password 7 iV7A4HcTNGo=
   neighbor Spine_PG send-community extended
   neighbor 10.2.1.2 peer group Spine_PG
   neighbor 10.2.2.2 peer group Spine_PG
   redistribute connected route-map loopback
   !
   vlan 10
      rd auto
      route-target import 65101:10010
      route-target import 65103:10010
      route-target export 65102:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target import 65101:10020
      route-target import 65103:10020
      route-target export 65102:10020
      redistribute learned
   !
   address-family evpn
      neighbor Spine_PG activate
   !
   vrf S_IRB-100
      rd 10.0.1.2:100
      route-target import evpn 65101:100
      route-target import evpn 65103:100
      route-target export evpn 65102:100
      redistribute connected
```
**Leaf3**
```
interface Ethernet4
   description =RT-EDGE_Gi0/0=
   no switchport
   vrf S_IRB-100
   ip address 192.168.255.0/31

router bgp 65103
   router-id 10.0.1.3
   maximum-paths 2 ecmp 2
   neighbor Spine_PG peer group
   neighbor Spine_PG remote-as 65100
   neighbor Spine_PG timers 3 9
   neighbor Spine_PG password 7 iV7A4HcTNGo=
   neighbor Spine_PG send-community extended
   neighbor 10.2.1.4 peer group Spine_PG
   neighbor 10.2.2.4 peer group Spine_PG
   redistribute connected route-map loopback
   !
   vlan 10
      rd auto
      route-target import 65101:10010
      route-target import 65102:10010
      route-target export 65103:10010
      redistribute learned
   !
   address-family evpn
      neighbor Spine_PG activate
   !
   vrf S_IRB-100
      rd 10.0.1.3:100
      route-target import evpn 65101:100
      route-target import evpn 65102:100
      route-target export evpn 65103:100
      neighbor 192.168.255.1 remote-as 1
      redistribute connected route-map C2B
      !
      address-family ipv4
         neighbor 192.168.255.1 activate

route-map C2B permit 10
   match ip address prefix-list C2B

ip prefix-list C2B
   seq 10 permit 192.168.10.0/24
```
**RT-EDGE**
```
interface Loopback0
 description =Internet_Simulation=
 ip address 8.8.8.8 255.255.255.255

interface GigabitEthernet0/0
 description =VxLAN_Fabric=
 ip address 192.168.255.1 255.255.255.254
 duplex auto
 speed auto
 media-type rj45

router bgp 1
 bgp router-id 8.8.8.8
 bgp log-neighbor-changes
 redistribute static route-map D2B
 neighbor 192.168.255.0 remote-as 65103
 neighbor 192.168.255.0 soft-reconfiguration inbound
 default-information originate

ip route 0.0.0.0 0.0.0.0 Null0

ip prefix-list D2B seq 5 permit 0.0.0.0/0

route-map D2B permit 10
 match ip address prefix-list D2B
```

## 3. Проверка работоспособности
Выполняются следующие условия:
- Установлено BGP соседство между RT-EDGE и Leaf3.
- На каждом Leaf присутствуют маршурты EVPN route-type 5.
- В таблице маршрутизации S_IRB-100 на каждом Leaf присутствует маршрут по умолчанию, ведущий на RT-EDGE.
- В таблице маршрутизации RT-EDGE присутствует маршруты в LAN_10 и LAN_20.
- С хостов в LAN_10 и LAN_20 доступен адрес 8.8.8.8, эмулирующий Интернет.

```
RT-EDGE#sh ip route
Gateway of last resort is 0.0.0.0 to network 0.0.0.0

S*    0.0.0.0/0 is directly connected, Null0
      8.0.0.0/32 is subnetted, 1 subnets
C        8.8.8.8 is directly connected, Loopback0
B     192.168.10.0/24 [20/0] via 192.168.255.0, 00:26:28
B     192.168.20.0/24 [20/0] via 192.168.255.0, 00:26:53
      192.168.255.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.255.0/31 is directly connected, GigabitEthernet0/0
L        192.168.255.1/32 is directly connected, GigabitEthernet0/0

RT-EDGE# sh ip bgp sum
BGP router identifier 8.8.8.8, local AS number 1
BGP table version is 13, main routing table version 13
3 network entries using 432 bytes of memory
3 path entries using 240 bytes of memory
3/3 BGP path/bestpath attribute entries using 456 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1176 total bytes of memory
BGP activity 7/4 prefixes, 7/4 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.255.0   4        65103      70      41       13    0    0 00:32:48        2

Leaf1#sh ip route vrf S_IRB-100

VRF: S_IRB-100
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

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 10.0.1.3 VNI 100 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1

 B E      192.168.10.2/32 [200/0] via VTEP 10.0.1.2 VNI 100 router-mac 50:ed:8c:5a:89:4d local-interface Vxlan1
 B E      192.168.10.3/32 [200/0] via VTEP 10.0.1.3 VNI 100 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 C        192.168.10.0/24 is directly connected, Vlan10
 B E      192.168.20.1/32 [200/0] via VTEP 10.0.1.2 VNI 100 router-mac 50:ed:8c:5a:89:4d local-interface Vxlan1
 B E      192.168.20.0/24 [200/0] via VTEP 10.0.1.2 VNI 100 router-mac 50:ed:8c:5a:89:4d local-interface Vxlan1

Leaf1#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.1.3:100 ip-prefix 0.0.0.0/0
                                 10.0.1.3              -       100     0       65100 65103 1 ?
 *  ec    RD: 10.0.1.3:100 ip-prefix 0.0.0.0/0
                                 10.0.1.3              -       100     0       65100 65103 1 ?
 * >      RD: 10.0.1.1:100 ip-prefix 192.168.10.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.2:100 ip-prefix 192.168.10.0/24
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:100 ip-prefix 192.168.10.0/24
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.3:100 ip-prefix 192.168.10.0/24
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:100 ip-prefix 192.168.10.0/24
                                 10.0.1.3              -       100     0       65100 65103 i
 * >Ec    RD: 10.0.1.2:100 ip-prefix 192.168.20.0/24
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:100 ip-prefix 192.168.20.0/24
                                 10.0.1.2              -       100     0       65100 65102 i

Leaf2#sh ip route vrf S_IRB-100

VRF: S_IRB-100
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

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 10.0.1.3 VNI 100 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1

 B E      192.168.10.1/32 [200/0] via VTEP 10.0.1.1 VNI 100 router-mac 50:2d:e4:c4:d8:0b local-interface Vxlan1
 B E      192.168.10.3/32 [200/0] via VTEP 10.0.1.3 VNI 100 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 C        192.168.10.0/24 is directly connected, Vlan10
 C        192.168.20.0/24 is directly connected, Vlan20

Leaf2#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.0.1.2, local AS number 65102
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.1.3:100 ip-prefix 0.0.0.0/0
                                 10.0.1.3              -       100     0       65100 65103 1 ?
 *  ec    RD: 10.0.1.3:100 ip-prefix 0.0.0.0/0
                                 10.0.1.3              -       100     0       65100 65103 1 ?
 * >Ec    RD: 10.0.1.1:100 ip-prefix 192.168.10.0/24
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:100 ip-prefix 192.168.10.0/24
                                 10.0.1.1              -       100     0       65100 65101 i
 * >      RD: 10.0.1.2:100 ip-prefix 192.168.10.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.3:100 ip-prefix 192.168.10.0/24
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:100 ip-prefix 192.168.10.0/24
                                 10.0.1.3              -       100     0       65100 65103 i
 * >      RD: 10.0.1.2:100 ip-prefix 192.168.20.0/24
                                 -                     -       -       0       i

Leaf3#sh ip route vrf S_IRB-100

VRF: S_IRB-100
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

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via 192.168.255.1, Ethernet4

 C        192.168.10.0/24 is directly connected, Vlan10
 B E      192.168.20.0/24 [200/0] via VTEP 10.0.1.2 VNI 100 router-mac 50:ed:8c:5a:89:4d local-interface Vxlan1
 C        192.168.255.0/31 is directly connected, Ethernet4

Leaf3#sh bgp evpn route-type ip-prefix ?
  A.B.C.D/E          IPv4 address prefix
  A:B:C:D:E:F:G:H/I  IPv6 address prefix
  ipv4               Limit address family to IPv4
  ipv6               Limit address family to IPv6

Leaf3#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.0.1.3, local AS number 65103
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.1.3:100 ip-prefix 0.0.0.0/0
                                 -                     0       100     0       1 ?
 * >Ec    RD: 10.0.1.1:100 ip-prefix 192.168.10.0/24
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:100 ip-prefix 192.168.10.0/24
                                 10.0.1.1              -       100     0       65100 65101 i
 * >Ec    RD: 10.0.1.2:100 ip-prefix 192.168.10.0/24
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:100 ip-prefix 192.168.10.0/24
                                 10.0.1.2              -       100     0       65100 65102 i
 * >      RD: 10.0.1.3:100 ip-prefix 192.168.10.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.2:100 ip-prefix 192.168.20.0/24
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:100 ip-prefix 192.168.20.0/24
                                 10.0.1.2              -       100     0       65100 65102 i

Host10.1> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=253 time=44.366 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=253 time=66.071 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=253 time=51.542 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=253 time=60.859 ms
^C

Host10.2> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=253 time=22.374 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=253 time=60.272 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=253 time=54.826 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=253 time=60.699 ms
^C

Host_20.1> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=253 time=16.266 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=253 time=72.715 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=253 time=52.996 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=253 time=66.942 ms
^C

Host10.3> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=254 time=18.736 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=254 time=35.828 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=254 time=24.056 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=254 time=28.825 ms
^C
```
