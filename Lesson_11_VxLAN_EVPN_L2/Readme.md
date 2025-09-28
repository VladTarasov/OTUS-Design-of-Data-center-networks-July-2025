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
- На каждеом Leaf присутствуют по два route type 3 маршрута до остальных Leaf.
```
Spine1#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.1.0, local AS number 65100
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.1 4 65101            619       620    0    0 00:26:12 Estab   1      1
  10.2.1.3 4 65102            622       621    0    0 00:26:10 Estab   1      1
  10.2.1.5 4 65103            306       304    0    0 00:12:36 Estab   0      0

Spine2#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.2.0, local AS number 65100
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.2.1 4 65101            643       634    0    0 00:26:48 Estab   1      1
  10.2.2.3 4 65102            641       636    0    0 00:26:45 Estab   1      1
  10.2.2.5 4 65103            628       630    0    0 00:26:28 Estab   0      0

Leaf1#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.0 4 65100            428       429    0    0 00:18:03 Estab   1      1
  10.2.2.0 4 65100            431       437    0    0 00:18:04 Estab   1      1

Leaf2#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.1.2, local AS number 65102
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.2 4 65100            599       599    0    0 00:25:12 Estab   1      1
  10.2.2.2 4 65100            598       604    0    0 00:25:12 Estab   1      1

Leaf3#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 10.0.1.3, local AS number 65103
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.4 4 65100            381       380    0    0 00:02:09 Estab   2      2
  10.2.2.4 4 65100            371       370    0    0 00:15:26 Estab   2      2




```
