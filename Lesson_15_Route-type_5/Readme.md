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
<img width="1833" height="1784" alt="image" src="https://github.com/user-attachments/assets/c0fe526a-7bfa-4354-a033-ad68b8b11e9b" />

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

VLAN 10 192.168.10.0/24 - Подсеть 10 vrf Client_1
VLAN 20 192.168.10.0/24 - Подсеть 20 vrf Client_1
VLAN 11 192.168.11.0/24 - Подсеть 10 vrf Client_2
VLAN 21 192.168.21.0/24 - Подсеть 20 vrf Client_2
VLAN -  192.168.255.0/24 le 30 - подсети для p2p L3 соединений с другим сетевым оборудованием (маршурутизаторы, FW)                                                                                                                                                                                                                                                                                                                                                                                                                                                         
### Протоколы и общие замечания
Задача - организовать связность между хостами, принадлежащими разным vrf, через внешнее относительно фабрики устройство.\
Решение - подключить к фабрике двумя линками в разных vrf маршрутизатор, построить ibgp соседсво с boarder leaf, назначив машрутизатор RR, для отражения маршрутов.
- Underlay сеть посроена на eBGP из предыдущих ДЗ.
- BFD отключен для экономии ресурсов стенда.
- Для распространения BUM используем Ingress Replication.
- Модель EVPN L2 сервиса VLAN-based.
- Модель EVPN L3 сервиса Edge-routed Briging Symetric IRB.
- Leaf3 выступает в качетсве boarder leaf.
- Связность с пограничным маршрутизатором построена на iBGP, маршрутизатор выступает в качетсве отражателя маршрутов, а leaf3 его клиентов в каждом vrf.

## 2. Конфигурация
В дополенение к конфигурации EVPN L3 VNI Необходимо:\
На Leaf 1 и 3
- Создать VRF Client_2, включить для него ip routing.
- Настроить интерфейс Eth4 в режим L3, привязать его к VRF Client_2.
- Настроить mac-vrf для vlan 11 и 21
- В BGP для VRF Client_2 настроить политики импорта/экпорта, выполнить redistribute connected.
- В интерфейсе vxlan выполнить маппинг vlan 11, 21 и vrf Client_2.

На RT-EDGE
- Назначить IP-адрес L3 интерфейсу Gi0/0.
- Назначить IP-адрес L3 интерфейсу Gi0/1.
- Запустить BGP процесс, задать соседство с Leaf3, в каждом линке, назначить соседей клиентами отражателя маршрутов, выполнить редистрибьюцию непосредственно подключенных сетей.

Ниже приведены конфигурации узлов (в части касающейся добавленных настроек).

**Leaf1**
```
vlan 10
   name LAN_10
!
vlan 11
   name LAN_11
!
vrf instance Client_1
!
vrf instance Client_2
!
interface Ethernet3
   description =Host1_Eth0=
   switchport access vlan 10
!
interface Ethernet4
   description =Host_11.1=
   switchport access vlan 11


interface Vlan10
   vrf Client_1
   ip address virtual 192.168.10.254/24
!
interface Vlan11
   vrf Client_2
   ip address virtual 192.168.11.254/24


interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 11 vni 10011
   vxlan vrf Client_1 vni 101
   vxlan vrf Client_2 vni 102

ip routing vrf Client_1
ip routing vrf Client_2

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
   vlan 11
      rd auto
      route-target import 65103:10021
      route-target export 65101:10011
      redistribute learned
   !
   address-family evpn
      neighbor Spine_PG activate
   !
   vrf Client_1
      rd 10.0.1.1:101
      route-target import evpn 65102:101
      route-target import evpn 65103:101
      route-target export evpn 65101:101
      redistribute connected
   !
   vrf Client_2
      rd 10.0.1.1:102
      route-target import evpn 65102:102
      route-target import evpn 65103:102
      route-target export evpn 65101:102
      redistribute connected
```
**Leaf2**
```
vlan 10
   name LAN_10
!
vlan 20
   name LAN_20
!
vrf instance Client_1
!
interface Ethernet3
   description =Host2_Eth0=
   switchport access vlan 10
!
interface Ethernet4
   description =Host_20.1=
   switchport access vlan 20

interface Vlan10
   vrf Client_1
   ip address virtual 192.168.10.254/24
!
interface Vlan20
   vrf Client_1
   ip address virtual 192.168.20.254/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf Client_1 vni 101
!
ip routing vrf Client_1
!
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
   vrf Client_1
      rd 10.0.1.2:101
      route-target import evpn 65101:101
      route-target import evpn 65103:101
      route-target export evpn 65102:101
      redistribute connected
```
**Leaf3**
```
vlan 10
   name LAN_10
!
vlan 21
   name LAN_21
!
vrf instance Client_1
!
vrf instance Client_2
!
interface Ethernet3
   description =Host3_Eth0=
   switchport access vlan 10
!
interface Ethernet4
   description =Host_21.1=
   switchport access vlan 21
!
interface Ethernet7
   description =RT-EDGE_Gi0/1=
   no switchport
   vrf Client_2
   ip address 192.168.255.2/31
!
interface Ethernet8
   description =RT-EDGE_Gi0/0=
   no switchport
   vrf Client_1
   ip address 192.168.255.0/31
!
interface Vlan10
   vrf Client_1
   ip address virtual 192.168.10.254/24
!
interface Vlan21
   vrf Client_2
   ip address virtual 192.168.21.254/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 21 vni 10021
   vxlan vrf Client_1 vni 101
   vxlan vrf Client_2 vni 102
!
ip routing vrf Client_1
ip routing vrf Client_2
!
router bgp 65103
   router-id 10.0.1.3
   maximum-paths 8 ecmp 8
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
   vlan 21
      rd auto
      route-target import 65101:10011
      route-target export 65103:10021
      redistribute learned
   !
   address-family evpn
      neighbor Spine_PG activate
   !
   vrf Client_1
      rd 10.0.1.3:101
      route-target import evpn 65101:101
      route-target import evpn 65102:101
      route-target export evpn 65103:101
      neighbor 192.168.255.1 remote-as 65103
      neighbor 192.168.255.1 route-map AS_path_clear out
      redistribute connected route-map C2B
      !
      address-family ipv4
         neighbor 192.168.255.1 activate
   !
   vrf Client_2
      rd 10.0.1.3:102
      route-target import evpn 65101:102
      route-target import evpn 65102:102
      route-target export evpn 65103:102
      neighbor 192.168.255.3 remote-as 65103
      neighbor 192.168.255.3 route-map AS_path_clear out
      redistribute connected route-map C2B_2
      !
      address-family ipv4
         neighbor 192.168.255.3 activate

route-map AS_path_clear permit 10
   set as-path match all replacement none
route-map C2B permit 10
   match ip address prefix-list C2B
route-map C2B_2 permit 10
   match ip address prefix-list C2B_2

ip prefix-list C2B
   seq 10 permit 192.168.10.0/24
   seq 20 permit 192.168.20.0/24
ip prefix-list C2B_2
   seq 10 permit 192.168.21.0/24
   seq 20 permit 192.168.11.0/24
```
**RT-EDGE**
```
interface GigabitEthernet0/0
 description =VxLAN_Fabric=
 ip address 192.168.255.1 255.255.255.254
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 description =VxLAN_Fabric_VRF_Client_2=
 ip address 192.168.255.3 255.255.255.254
 duplex auto
 speed auto
 media-type rj45

router bgp 65103
 bgp router-id 192.168.255.255
 bgp log-neighbor-changes
 redistribute connected
 neighbor 192.168.255.0 remote-as 65103
 neighbor 192.168.255.0 route-reflector-client
 neighbor 192.168.255.0 soft-reconfiguration inbound
 neighbor 192.168.255.2 remote-as 65103
 neighbor 192.168.255.2 route-reflector-client
 neighbor 192.168.255.2 soft-reconfiguration inbound

```

## 3. Проверка работоспособности
Выполняются следующие условия:
- На RT-EDGE присутствуют маршурты из обоих VRF.
- В таблице маршрутизации на каждом Leaf в каждом VRF присутствует маршруты из другого VRF.
- C хостов доступны хосты, находящиеся с ними в одном VLAN И VRF.
- C хостов доступны хосты, находящиеся с ними в разных VLAN И VRF.

```
RT-EDGE#sh ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

B     192.168.10.0/24 [200/0] via 192.168.255.0, 01:01:45
B     192.168.11.0/24 [200/0] via 192.168.255.2, 00:01:50
B     192.168.20.0/24 [200/0] via 192.168.255.0, 00:02:22
B     192.168.21.0/24 [200/0] via 192.168.255.2, 01:01:46


Leaf1#sh ip route vrf Client_1 bgp

VRF: Client_1
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

 B E      192.168.11.0/24 [200/0] via VTEP 10.0.1.3 VNI 101 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.20.0/24 [200/0] via VTEP 10.0.1.2 VNI 101 router-mac 50:ed:8c:5a:89:4d local-interface Vxlan1
 B E      192.168.21.0/24 [200/0] via VTEP 10.0.1.3 VNI 101 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.255.0/31 [200/0] via VTEP 10.0.1.3 VNI 101 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.255.2/31 [200/0] via VTEP 10.0.1.3 VNI 101 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1

Leaf1#sh ip route vrf Client_2 bgp

VRF: Client_2
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

 B E      192.168.10.1/32 [200/0] via VTEP 10.0.1.3 VNI 102 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.10.0/24 [200/0] via VTEP 10.0.1.3 VNI 102 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.20.1/32 [200/0] via VTEP 10.0.1.3 VNI 102 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.20.0/24 [200/0] via VTEP 10.0.1.3 VNI 102 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.21.1/32 [200/0] via VTEP 10.0.1.3 VNI 102 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.21.0/24 [200/0] via VTEP 10.0.1.3 VNI 102 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.255.0/31 [200/0] via VTEP 10.0.1.3 VNI 102 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.255.2/31 [200/0] via VTEP 10.0.1.3 VNI 102 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1


Leaf2#sh ip route vrf Client_1 bgp

VRF: Client_1
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

 B E      192.168.11.0/24 [200/0] via VTEP 10.0.1.3 VNI 101 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.21.0/24 [200/0] via VTEP 10.0.1.3 VNI 101 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.255.0/31 [200/0] via VTEP 10.0.1.3 VNI 101 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1
 B E      192.168.255.2/31 [200/0] via VTEP 10.0.1.3 VNI 101 router-mac 50:1e:49:b9:09:14 local-interface Vxlan1


Leaf3#sh ip route vrf Client_1 bgp

VRF: Client_1
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

 B I      192.168.11.0/24 [200/0] via 192.168.255.1, Ethernet8
 B E      192.168.20.0/24 [200/0] via VTEP 10.0.1.2 VNI 101 router-mac 50:ed:8c:5a:89:4d local-interface Vxlan1
 B I      192.168.21.0/24 [200/0] via 192.168.255.1, Ethernet8
 B I      192.168.255.2/31 [200/0] via 192.168.255.1, Ethernet8


Leaf3#sh ip route vrf Client_2 bgp

VRF: Client_2
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

 B I      192.168.10.1/32 [200/0] via 192.168.255.3, Ethernet7
 B I      192.168.10.0/24 [200/0] via 192.168.255.3, Ethernet7
 B E      192.168.11.1/32 [200/0] via VTEP 10.0.1.1 VNI 102 router-mac 02:00:00:be:5f:db local-interface Vxlan1
 B E      192.168.11.0/24 [200/0] via VTEP 10.0.1.1 VNI 102 router-mac 02:00:00:be:5f:db local-interface Vxlan1
 B I      192.168.20.1/32 [200/0] via 192.168.255.3, Ethernet7
 B I      192.168.20.0/24 [200/0] via 192.168.255.3, Ethernet7
 B I      192.168.255.0/31 [200/0] via 192.168.255.3, Ethernet7


Host10.1 : 192.168.10.1 255.255.255.0 gateway 192.168.10.254

Host10.1> ping 192.168.10.2

84 bytes from 192.168.10.2 icmp_seq=1 ttl=64 time=18.156 ms
84 bytes from 192.168.10.2 icmp_seq=2 ttl=64 time=11.556 ms
84 bytes from 192.168.10.2 icmp_seq=3 ttl=64 time=11.307 ms
84 bytes from 192.168.10.2 icmp_seq=4 ttl=64 time=11.704 ms
84 bytes from 192.168.10.2 icmp_seq=5 ttl=64 time=11.157 ms

Host10.1> ping 192.168.10.3

84 bytes from 192.168.10.3 icmp_seq=1 ttl=64 time=12.837 ms
84 bytes from 192.168.10.3 icmp_seq=2 ttl=64 time=11.153 ms
84 bytes from 192.168.10.3 icmp_seq=3 ttl=64 time=11.575 ms
84 bytes from 192.168.10.3 icmp_seq=4 ttl=64 time=10.622 ms
84 bytes from 192.168.10.3 icmp_seq=5 ttl=64 time=11.153 ms

Host10.1> ping 192.168.20.1

84 bytes from 192.168.20.1 icmp_seq=1 ttl=62 time=22.408 ms
84 bytes from 192.168.20.1 icmp_seq=2 ttl=62 time=12.031 ms
84 bytes from 192.168.20.1 icmp_seq=3 ttl=62 time=12.060 ms
84 bytes from 192.168.20.1 icmp_seq=4 ttl=62 time=152.140 ms
84 bytes from 192.168.20.1 icmp_seq=5 ttl=62 time=13.976 ms

Host10.1> ping 192.168.21.1

84 bytes from 192.168.21.1 icmp_seq=1 ttl=60 time=48.978 ms
84 bytes from 192.168.21.1 icmp_seq=2 ttl=60 time=27.201 ms
84 bytes from 192.168.21.1 icmp_seq=3 ttl=60 time=17.419 ms
84 bytes from 192.168.21.1 icmp_seq=4 ttl=60 time=15.079 ms
84 bytes from 192.168.21.1 icmp_seq=5 ttl=60 time=19.589 ms

Host10.1> ping 192.168.11.1

84 bytes from 192.168.11.1 icmp_seq=1 ttl=59 time=39.796 ms
84 bytes from 192.168.11.1 icmp_seq=2 ttl=59 time=22.510 ms
84 bytes from 192.168.11.1 icmp_seq=3 ttl=59 time=23.814 ms
84 bytes from 192.168.11.1 icmp_seq=4 ttl=59 time=25.684 ms
84 bytes from 192.168.11.1 icmp_seq=5 ttl=59 time=22.929 ms

Host_11.1 : 192.168.11.1 255.255.255.0 gateway 192.168.11.254

Host_11.1> ping 192.168.21.1

84 bytes from 192.168.21.1 icmp_seq=1 ttl=62 time=44.302 ms
84 bytes from 192.168.21.1 icmp_seq=2 ttl=62 time=33.509 ms
84 bytes from 192.168.21.1 icmp_seq=3 ttl=62 time=11.143 ms
84 bytes from 192.168.21.1 icmp_seq=4 ttl=62 time=11.949 ms
84 bytes from 192.168.21.1 icmp_seq=5 ttl=62 time=17.991 ms

Host_11.1> ping 192.168.20.1

84 bytes from 192.168.20.1 icmp_seq=1 ttl=59 time=46.203 ms
84 bytes from 192.168.20.1 icmp_seq=2 ttl=59 time=33.089 ms
84 bytes from 192.168.20.1 icmp_seq=3 ttl=59 time=23.320 ms
84 bytes from 192.168.20.1 icmp_seq=4 ttl=59 time=22.649 ms
84 bytes from 192.168.20.1 icmp_seq=5 ttl=59 time=33.501 ms

Host_11.1> ping 192.168.10.1

84 bytes from 192.168.10.1 icmp_seq=1 ttl=59 time=29.501 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=59 time=21.465 ms
84 bytes from 192.168.10.1 icmp_seq=3 ttl=59 time=24.123 ms
84 bytes from 192.168.10.1 icmp_seq=4 ttl=59 time=22.928 ms
84 bytes from 192.168.10.1 icmp_seq=5 ttl=59 time=24.408 ms
```
