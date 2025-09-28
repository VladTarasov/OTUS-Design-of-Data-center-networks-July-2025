# Занятие 11. VxLAN EVPN L2

## **План работы**
1. Дизайн
  - Схема
  - Адресное пространство
  - Протоколы и общие замечания
2. Конфигурация
3. Проверка работоспособности

## 1. Дизайн

### Схема

<img width="1126" height="819" alt="image" src="https://github.com/user-attachments/assets/3898583a-7995-46c6-a606-a0ab65e27782" />

### Адресное пространство
Распределение адресов для loopback интерфейсов /32:

| Loopback         | Spine1   | Spine2   | Leaf1    |Leaf2     |Leaf3     |
| ---------------- |:--------:| --------:|----------|----------|----------|    
| Lo1              | 10.0.1.0 | 10.0.2.0 | 10.0.1.1 | 10.0.1.2 | 10.0.1.3 |

Распределение адресов для p2p интерфейсов /31:

|  Узел/Интерфейс  | Leaf1   |Leaf2     |Leaf3     |Spine1    |Spine2    |
| ---------------- |:-------:| -------: |--------- |----------|----------|    
| Eth1             | 10.2.1.1| 10.2.1.3 | 10.2.1.5 | 10.2.1.0 | 10.2.2.0 |
| Eth2             | 10.2.2.1| 10.2.2.3 | 10.2.2.5 | 10.2.1.2 | 10.2.2.2 |
| Eth3             |         |          |          | 10.2.1.4 | 10.2.2.4 |

Суммированный маршрут для p2p соединений 10.2.0.0/15

### Протоколы и общие замечания
- Underlay сеть посроена на eBGP.
- BFD отключен для экономии ресурсов стенда.
- Для распространения BUM используем Ingress Replication.
- Модель EVPN сервиса VLAN-based.
- Организуем сервис L2VPN для узлов в VLAN10 VNI 10010.

## 2. Конфигурация
Необходимо сконфигурировать:
- send-community extended на каждом узле;
- соседство в address-family evpn между Leaf и Spine;
- VLAN 10 и access порт на каждом Leaf в сторону хоста;
- MAC-vrf VLAN 10: rd, rt import/export, редистрибьюцию изученных MAC в BGP;
- interface Vxlan, где настроить сопоставление VLAN 10 с VNI 10010. 

Ниже приведены конфигурации узлов (в части касающейся настроек VxLAN и EVPN для краткости).

**Spine1**
```
!
service routing protocols model multi-agent
!
router bgp 65100
   router-id 10.0.1.0
   maximum-paths 2 ecmp 2
   bgp listen range 10.2.0.0/15 peer-group Leaf_PG peer-filter Leaf
   neighbor Leaf_PG peer group
   neighbor Leaf_PG timers 3 9
   neighbor Leaf_PG password 7 n5V7FYyq00M=
   neighbor Leaf_PG send-community extended
   redistribute connected route-map loopback
   !
   address-family evpn
      neighbor Leaf_PG activate
```
**Spine2**
```
!
service routing protocols model multi-agent
!
router bgp 65100
   router-id 10.0.2.0
   maximum-paths 2 ecmp 2
   bgp listen range 10.2.0.0/15 peer-group Leaf_PG peer-filter Leaf
   neighbor Leaf_PG peer group
   neighbor Leaf_PG timers 3 9
   neighbor Leaf_PG password 7 n5V7FYyq00M=
   neighbor Leaf_PG send-community extended
   redistribute connected route-map loopback
   !
   address-family evpn
      neighbor Leaf_PG activate
```
**Leaf1**
```
!
vlan 10
   name LAN_10
!
interface Ethernet3
   description =Host1_Eth0=
   switchport access vlan 10
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
!
router bgp 65101
   router-id 10.0.1.1
   maximum-paths 2 ecmp 2
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
```
**Leaf2**
```
!
vlan 10
   name LAN_10
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
router bgp 65102
   router-id 10.0.1.2
   maximum-paths 2 ecmp 2
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
   address-family evpn
      neighbor Spine_PG activate
```
**Leaf3**
```
!
vlan 10
   name LAN_10
!
interface Ethernet3
   description =Host3_Eth0=
   switchport access vlan 10
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
!
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
```
## 3. Проверка работоспособности
Выполняются следующие условия:
- Между узлами установлены соседства в address-family evpn.
- На каждом Leaf присутствуют по два route type 3 маршрута до остальных Leaf с учетом ECMP.
- С каждого хоста доступны два других хоста.
- После проверки доступности хостов на Leaf появляется по два маршрута route type 2 с учетом ECMP.
```
Spine1#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.1.0, local AS number 65100
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.1 4 65101            276       273    0    0 00:11:21 Estab   1      1
  10.2.1.3 4 65102           2598      2591    0    0 01:50:21 Estab   1      1
  10.2.1.5 4 65103            276       273    0    0 00:11:21 Estab   1      1

Spine2#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.2.0, local AS number 65100
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.2.1 4 65101            285       286    0    0 00:11:46 Estab   1      1
  10.2.2.3 4 65102            289       285    0    0 00:11:45 Estab   1      1
  10.2.2.5 4 65103            288       284    0    0 00:11:46 Estab   1      1

Leaf1#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.0 4 65100           6414      6414    0    0 00:11:51 Estab   2      2
  10.2.2.0 4 65100           6404      6414    0    0 00:11:51 Estab   2      2

Leaf2#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.1.2, local AS number 65102
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.2 4 65100           6356      6354    0    0 01:50:55 Estab   2      2
  10.2.2.2 4 65100           6347      6355    0    0 00:11:54 Estab   2      2

Leaf3#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.1.3, local AS number 65103
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.4 4 65100           6306      6310    0    0 00:11:58 Estab   2      2
  10.2.2.4 4 65100           6309      6311    0    0 00:11:58 Estab   2      2

Leaf1#sh bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.1.1:10 imet 10.0.1.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.2:10 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:10 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.3:10 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:10 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65100 65103 i

Leaf2#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.1.2, local AS number 65102
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.2 4 65100           6356      6354    0    0 01:50:55 Estab   2      2
  10.2.2.2 4 65100           6347      6355    0    0 00:11:54 Estab   2      2
Leaf2#sh bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.0.1.2, local AS number 65102
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.1.1:10 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:10 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65100 65101 i
 * >      RD: 10.0.1.2:10 imet 10.0.1.2
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.3:10 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:10 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65100 65103 i

Leaf3#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.1.3, local AS number 65103
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.4 4 65100           6306      6310    0    0 00:11:58 Estab   2      2
  10.2.2.4 4 65100           6309      6311    0    0 00:11:58 Estab   2      2
Leaf3#sh bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.0.1.3, local AS number 65103
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.1.1:10 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:10 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65100 65101 i
 * >Ec    RD: 10.0.1.2:10 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:10 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 * >      RD: 10.0.1.3:10 imet 10.0.1.3
                                 -                     -       -       0       i
Host1> sh ip

NAME        : Host1[1]
IP/MASK     : 192.168.10.1/24
GATEWAY     : 255.255.255.0
DNS         :
MAC         : 00:50:79:66:68:09
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

Host1> ping 192.168.10.2

84 bytes from 192.168.10.2 icmp_seq=1 ttl=64 time=18.722 ms
84 bytes from 192.168.10.2 icmp_seq=2 ttl=64 time=58.870 ms
84 bytes from 192.168.10.2 icmp_seq=3 ttl=64 time=31.448 ms
84 bytes from 192.168.10.2 icmp_seq=4 ttl=64 time=43.022 ms
84 bytes from 192.168.10.2 icmp_seq=5 ttl=64 time=43.975 ms
^C
Host1> ping 192.168.10.3

84 bytes from 192.168.10.3 icmp_seq=1 ttl=64 time=17.017 ms
84 bytes from 192.168.10.3 icmp_seq=2 ttl=64 time=62.420 ms
84 bytes from 192.168.10.3 icmp_seq=3 ttl=64 time=43.796 ms
84 bytes from 192.168.10.3 icmp_seq=4 ttl=64 time=44.480 ms
^C
Host2> sh ip

NAME        : Host2[1]
IP/MASK     : 192.168.10.2/24
GATEWAY     : 192.168.10.254
DNS         :
MAC         : 00:50:79:66:68:07
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

Host2> ping 192.168.10.1

84 bytes from 192.168.10.1 icmp_seq=1 ttl=64 time=33.634 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=64 time=27.847 ms
84 bytes from 192.168.10.1 icmp_seq=3 ttl=64 time=54.868 ms
84 bytes from 192.168.10.1 icmp_seq=4 ttl=64 time=39.599 ms
^C
Host2> ping 192.168.10.3

84 bytes from 192.168.10.3 icmp_seq=1 ttl=64 time=11.714 ms
84 bytes from 192.168.10.3 icmp_seq=2 ttl=64 time=36.933 ms
84 bytes from 192.168.10.3 icmp_seq=3 ttl=64 time=55.969 ms
84 bytes from 192.168.10.3 icmp_seq=4 ttl=64 time=37.146 ms
^C

Host3> sh ip

NAME        : Host3[1]
IP/MASK     : 192.168.10.3/24
GATEWAY     : 192.168.10.254
DNS         :
MAC         : 00:50:79:66:68:08
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

Host3> ping 192.168.10.2

84 bytes from 192.168.10.2 icmp_seq=1 ttl=64 time=35.846 ms
84 bytes from 192.168.10.2 icmp_seq=2 ttl=64 time=30.707 ms
84 bytes from 192.168.10.2 icmp_seq=3 ttl=64 time=47.589 ms
84 bytes from 192.168.10.2 icmp_seq=4 ttl=64 time=65.995 ms
^C
Host3> ping 192.168.10.1

84 bytes from 192.168.10.1 icmp_seq=1 ttl=64 time=41.625 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=64 time=46.073 ms
84 bytes from 192.168.10.1 icmp_seq=3 ttl=64 time=53.189 ms
84 bytes from 192.168.10.1 icmp_seq=4 ttl=64 time=50.324 ms
^C

Leaf1#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.1.2:10 mac-ip 0050.7966.6807
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:10 mac-ip 0050.7966.6807
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6808
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6808
                                 10.0.1.3              -       100     0       65100 65103 i
 * >      RD: 10.0.1.1:10 mac-ip 0050.7966.6809
                                 -                     -       -       0       i

Leaf2#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.0.1.2, local AS number 65102
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.1.2:10 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6808
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6808
                                 10.0.1.3              -       100     0       65100 65103 i
 * >Ec    RD: 10.0.1.1:10 mac-ip 0050.7966.6809
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:10 mac-ip 0050.7966.6809
                                 10.0.1.1              -       100     0       65100 65101 i

Leaf3#sh bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.0.1.3, local AS number 65103
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.1.1:10 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:10 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65100 65101 i
 * >Ec    RD: 10.0.1.2:10 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:10 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 * >      RD: 10.0.1.3:10 imet 10.0.1.3
                                 -                     -       -       0       i
```
