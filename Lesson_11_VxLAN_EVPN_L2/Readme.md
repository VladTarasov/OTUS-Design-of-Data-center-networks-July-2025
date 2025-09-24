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

## 2. Конфигурация

Ниже приведены конфигурации узлов в части касающейся настроек VxLAN и EVPN.

**Spine1**
```
```
**Spine2**
```
```
**Leaf1**
```
```
**Leaf2**
```
```
**Leaf3**
```
```


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

Leaf3#sh bgp evpn
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


```
