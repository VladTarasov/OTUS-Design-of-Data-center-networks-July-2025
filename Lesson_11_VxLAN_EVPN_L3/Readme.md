# Занятие 12. VxLAN EVPN L3

## **План работы**
1. Дизайн
  - Схема
  - Адресное пространство
  - Протоколы и общие замечания
2. Конфигурация
3. Проверка работоспособности

## 1. Дизайн

### Схема
<img width="1787" height="1472" alt="image" src="https://github.com/user-attachments/assets/0195dbde-4298-46db-b9f6-5b7d5615aa89" />
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

### Протоколы и общие замечания
Задача - организовать между хостами LAN_10 и LAN_20 связность на уровне L3.
- Underlay сеть посроена на eBGP из предыдущих ДЗ.
- BFD отключен для экономии ресурсов стенда.
- Для распространения BUM используем Ingress Replication.
- Модель EVPN L2 сервиса VLAN-based.
- Модель EVPN L3 сервиса Edge-routed Briging Symetric IRB. Модели Briged Overlay и Edge-routed Briging Asymetric IRB не рассматриваются.

## 2. Конфигурация
В дополенение к конфигурации EVPN L2 Необходимо:\
На каждом Leaf
- создать VRF;
- включить ip routing для мозданного VRF;
- добавить в bgp процесс конфигурацию VRF, указать RD и политики import/export;
- в interface Vxlan1 настроить сопоставление VRF с VNI 100;
- создать SVI Vlan10, Vlan20, добавить из в VRF, указать IP как anycast gateway.

На Leaf2
- создать VLAN 20 и настроить access порт в сторону Host_20.1;
- добавить в bgp процесс конфигурацию MAC-vrf VLAN 20: rd auto, rt import/export настроены вручную, включена редистрибьюция изученных MAC в BGP;

Ниже приведены конфигурации узлов (в части касающейся настроек VxLAN и EVPN для краткости).

**Leaf1**
```
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
   !
   vrf S_IRB-100
      rd 10.0.1.1:100
      route-target import evpn 65102:100
      route-target import evpn 65103:100
      route-target export evpn 65101:100

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vrf S_IRB-100 vni 100

interface Vlan10
   vrf S_IRB-100
   ip address virtual 192.168.10.254/24
!
ip routing vrf S_IRB-100
```
**Leaf2**
```
vlan 20
   name LAN_20

vrf instance S_IRB-100

interface Ethernet4
   description =Host_20.1=
   switchport access vlan 20

interface Vlan10
   vrf S_IRB-100
   ip address virtual 192.168.10.254/24
!
interface Vlan20
   vrf S_IRB-100
   ip address virtual 192.168.20.254/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf S_IRB-100 vni 100
!
ip routing
ip routing vrf S_IRB-100

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
```
**Leaf3**
```
interface Vlan10
   vrf S_IRB-100
   ip address virtual 192.168.10.254/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vrf S_IRB-100 vni 100
!
ip routing
ip routing vrf S_IRB-100
!
ip prefix-list loopback
   seq 10 permit 10.0.1.3/32
!
route-map loopback permit 10
   match ip address prefix-list loopback
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
   !
   vrf S_IRB-100
      rd 10.0.1.3:100
      route-target import evpn 65101:100
      route-target import evpn 65102:100
      route-target export evpn 65103:100
```
## 3. Проверка работоспособности
Выполняются следующие условия:
- Базовый L2 в LAN_10 остался работоспособен.
- С хостов в LAN_10 доступен хост из LAN_20 и наоборот, доступны anycast gateway.
- После проверки доступности хостов на Leaf появляется по два маршрута route type 2 вида mac-only и mac-ip.

```
Host1> sh ip

NAME        : Host1[1]
IP/MASK     : 192.168.10.1/24
GATEWAY     : 192.168.10.254
DNS         :
MAC         : 00:50:79:66:68:08
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

Host1> ping 192.168.10.2

84 bytes from 192.168.10.2 icmp_seq=1 ttl=64 time=13.113 ms
84 bytes from 192.168.10.2 icmp_seq=2 ttl=64 time=24.073 ms
84 bytes from 192.168.10.2 icmp_seq=3 ttl=64 time=13.521 ms
^C
Host1> ping 192.168.10.3

84 bytes from 192.168.10.3 icmp_seq=1 ttl=64 time=27.674 ms
84 bytes from 192.168.10.3 icmp_seq=2 ttl=64 time=15.178 ms
84 bytes from 192.168.10.3 icmp_seq=3 ttl=64 time=43.080 ms
^C
Host1> ping 192.168.10.254

84 bytes from 192.168.10.254 icmp_seq=1 ttl=64 time=3.335 ms
84 bytes from 192.168.10.254 icmp_seq=2 ttl=64 time=13.976 ms
84 bytes from 192.168.10.254 icmp_seq=3 ttl=64 time=6.883 ms
^C
Host1> ping 192.168.20.1

84 bytes from 192.168.20.1 icmp_seq=1 ttl=62 time=30.126 ms
84 bytes from 192.168.20.1 icmp_seq=2 ttl=62 time=55.615 ms
84 bytes from 192.168.20.1 icmp_seq=3 ttl=62 time=39.404 ms
^C
Host2> sh ip

NAME        : Host2[1]
IP/MASK     : 192.168.10.2/24
GATEWAY     : 192.168.10.254
DNS         :
MAC         : 00:50:79:66:68:06
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

Host2> ping 192.168.10.254

84 bytes from 192.168.10.254 icmp_seq=1 ttl=64 time=3.249 ms
84 bytes from 192.168.10.254 icmp_seq=2 ttl=64 time=14.964 ms
84 bytes from 192.168.10.254 icmp_seq=3 ttl=64 time=6.505 ms
84 bytes from 192.168.10.254 icmp_seq=4 ttl=64 time=3.812 ms
^C
Host2> ping 192.168.10.1

84 bytes from 192.168.10.1 icmp_seq=1 ttl=64 time=26.279 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=64 time=13.784 ms
^C
Host2> ping 192.168.10.3

84 bytes from 192.168.10.3 icmp_seq=1 ttl=64 time=26.191 ms
84 bytes from 192.168.10.3 icmp_seq=2 ttl=64 time=23.448 ms
84 bytes from 192.168.10.3 icmp_seq=3 ttl=64 time=13.494 ms
^C
Host2> ping 192.168.20.1

84 bytes from 192.168.20.1 icmp_seq=1 ttl=63 time=13.482 ms
84 bytes from 192.168.20.1 icmp_seq=2 ttl=63 time=13.925 ms
84 bytes from 192.168.20.1 icmp_seq=3 ttl=63 time=23.493 ms
84 bytes from 192.168.20.1 icmp_seq=4 ttl=63 time=4.552 ms
^C

Host3> sh ip

NAME        : Host3[1]
IP/MASK     : 192.168.10.3/24
GATEWAY     : 192.168.10.254
DNS         :
MAC         : 00:50:79:66:68:07
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

Host3 : 192.168.10.3 255.255.255.0 gateway 192.168.10.254

Host3> ping 192.168.10.254

84 bytes from 192.168.10.254 icmp_seq=1 ttl=64 time=4.879 ms
84 bytes from 192.168.10.254 icmp_seq=2 ttl=64 time=5.395 ms
^C
Host3> ping 192.168.10.2

84 bytes from 192.168.10.2 icmp_seq=1 ttl=64 time=30.958 ms
84 bytes from 192.168.10.2 icmp_seq=2 ttl=64 time=14.956 ms
84 bytes from 192.168.10.2 icmp_seq=3 ttl=64 time=11.289 ms
^C
Host3> ping 192.168.10.1

84 bytes from 192.168.10.1 icmp_seq=1 ttl=64 time=23.900 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=64 time=29.510 ms
84 bytes from 192.168.10.1 icmp_seq=3 ttl=64 time=11.642 ms
84 bytes from 192.168.10.1 icmp_seq=4 ttl=64 time=43.665 ms
^C

Host3> ping 192.168.20.1

84 bytes from 192.168.20.1 icmp_seq=1 ttl=62 time=32.981 ms
84 bytes from 192.168.20.1 icmp_seq=2 ttl=62 time=33.762 ms
84 bytes from 192.168.20.1 icmp_seq=3 ttl=62 time=15.492 ms
84 bytes from 192.168.20.1 icmp_seq=4 ttl=62 time=40.614 ms
^C
```
*sh bgp evpn*

```
Leaf1#sh bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.1.2:10 mac-ip 0050.7966.6806
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:10 mac-ip 0050.7966.6806
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.2:10 mac-ip 0050.7966.6806 192.168.10.2
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:10 mac-ip 0050.7966.6806 192.168.10.2
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6807
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6807
                                 10.0.1.3              -       100     0       65100 65103 i
 * >Ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6807 192.168.10.3
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6807 192.168.10.3
                                 10.0.1.3              -       100     0       65100 65103 i
 * >      RD: 10.0.1.1:10 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >      RD: 10.0.1.1:10 mac-ip 0050.7966.6808 192.168.10.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.2:20 mac-ip 0050.7966.6809
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:20 mac-ip 0050.7966.6809
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.2:20 mac-ip 0050.7966.6809 192.168.20.1
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:20 mac-ip 0050.7966.6809 192.168.20.1
                                 10.0.1.2              -       100     0       65100 65102 i
 * >      RD: 10.0.1.1:10 imet 10.0.1.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.2:10 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:10 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.2:20 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:20 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.3:10 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:10 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65100 65103 i

Leaf2#sh bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.1.2, local AS number 65102
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.1.2:10 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >      RD: 10.0.1.2:10 mac-ip 0050.7966.6806 192.168.10.2
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6807
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6807
                                 10.0.1.3              -       100     0       65100 65103 i
 * >Ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6807 192.168.10.3
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:10 mac-ip 0050.7966.6807 192.168.10.3
                                 10.0.1.3              -       100     0       65100 65103 i
 * >Ec    RD: 10.0.1.1:10 mac-ip 0050.7966.6808
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:10 mac-ip 0050.7966.6808
                                 10.0.1.1              -       100     0       65100 65101 i
 * >Ec    RD: 10.0.1.1:10 mac-ip 0050.7966.6808 192.168.10.1
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:10 mac-ip 0050.7966.6808 192.168.10.1
                                 10.0.1.1              -       100     0       65100 65101 i
 * >      RD: 10.0.1.2:20 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >      RD: 10.0.1.2:20 mac-ip 0050.7966.6809 192.168.20.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.1:10 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:10 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65100 65101 i
 * >      RD: 10.0.1.2:10 imet 10.0.1.2
                                 -                     -       -       0       i
 * >      RD: 10.0.1.2:20 imet 10.0.1.2
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.3:10 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65100 65103 i
 *  ec    RD: 10.0.1.3:10 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65100 65103 i

Leaf3#sh bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.1.3, local AS number 65103
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.1.2:10 mac-ip 0050.7966.6806
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:10 mac-ip 0050.7966.6806
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.2:10 mac-ip 0050.7966.6806 192.168.10.2
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:10 mac-ip 0050.7966.6806 192.168.10.2
                                 10.0.1.2              -       100     0       65100 65102 i
 * >      RD: 10.0.1.3:10 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >      RD: 10.0.1.3:10 mac-ip 0050.7966.6807 192.168.10.3
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.1:10 mac-ip 0050.7966.6808
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:10 mac-ip 0050.7966.6808
                                 10.0.1.1              -       100     0       65100 65101 i
 * >Ec    RD: 10.0.1.1:10 mac-ip 0050.7966.6808 192.168.10.1
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:10 mac-ip 0050.7966.6808 192.168.10.1
                                 10.0.1.1              -       100     0       65100 65101 i
 * >Ec    RD: 10.0.1.2:20 mac-ip 0050.7966.6809
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:20 mac-ip 0050.7966.6809
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.2:20 mac-ip 0050.7966.6809 192.168.20.1
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:20 mac-ip 0050.7966.6809 192.168.20.1
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.1:10 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65100 65101 i
 *  ec    RD: 10.0.1.1:10 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65100 65101 i
 * >Ec    RD: 10.0.1.2:10 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:10 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 * >Ec    RD: 10.0.1.2:20 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 *  ec    RD: 10.0.1.2:20 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65100 65102 i
 * >      RD: 10.0.1.3:10 imet 10.0.1.3
                                 -                     -       -       0       i
```
