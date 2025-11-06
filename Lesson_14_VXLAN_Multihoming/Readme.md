# Занятие 14 VXLAN. Multihoming

## **Задание**
Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming.

## **План работы**
1. Дизайн
  - Схема
  - Адресное пространство
  - Протоколы и общие замечания
2. Конфигурация
3. Проверка работоспособности

## 1. Дизайн

### Схема
<img width="1883" height="1381" alt="image" src="https://github.com/user-attachments/assets/935e6833-8bde-427d-be8d-99af6b1909e3" />

### **Адресное пространство**

Распределение адресов для loopback интерфейсов /32:

| Loopback         | Spine1     | Spine2     | Leaf1        |Leaf2         |Leaf3         |Leaf4        |
| ---------------- |:----------:| ----------:|--------------|--------------|--------------|-------------|    
| Lo0              | 172.17.1.0 | 172.17.2.0 | 172.17.101.0 | 172.17.102.0 | 172.17.103.0 |172.17.104.0 |

Распределение адресов для p2p интерфейсов /31:

|  Узел/Интерфейс  | Leaf1       |Leaf2         |Leaf3         |Leaf4         |Spine1        |Spine2        |
| ---------------- |:-----------:| -----------: |------------- |--------------|--------------|--------------|    
| Eth1             | 172.17.201.1| 172.17.201.3 | 172.17.201.5 | 172.17.201.7 | 172.17.201.0 |172.17.202.0  |
| Eth2             | 172.17.202.1| 172.17.202.3 | 172.17.202.5 | 172.17.202.7 | 172.17.201.2 |172.17.202.2  |
| Eth3             |             |              |              |              | 172.17.201.4 |172.17.202.4  |
| Eth4             |             |              |              |              | 172.17.201.6 |172.17.202.6  |
| Vlan101          |172.17.250.2 |              |              | 172.17.201.4 |              |              |
| Vlan103          |172.17.250.6 |              |              | 172.17.201.4 |              |              |

Конечное оборудование (нагрузка)

10.2.10.0/24 VLAN 10 - подсеть контроллера домена\
  10.2.10.1 - адрес контроллера домена\
  10.2.10.254 - шлюз по умолчанию (Vlanif10 Anycast gateway на Leaf3-4)\
10.2.20.0/24 VLAN 20 - подсеть Web-сервера\
  10.2.20.1 - адрес контроллера домена\
  10.2.20.254 - шлюз по умолчанию (Port-channel1.20 на COD2-FW)\
172.17.250.0/31 VLAN 100 - линковая подсеть для соединения COD2-EDGE-R с COD2-FW.\
  172.17.250.0 - COD2-EDGE-R Port-channel1.100
  172.17.250.1 - COD2-FW Port-channel1.100
172.17.250.2/31 VLAN 101 - линковая подсеть для соединения COD2-EDGE-R с VRF Infrastructure.\
  172.17.250.2 - Leaf1 Vlan101
  172.17.250.3 - COD2-EDGE-R Port-channel1.101
172.17.250.4/31 VLAN 103 - линковая подсеть для соединения COD2-FW с VRF Infrastructure.\
  172.17.250.6 - Leaf1 Vlan103
  172.17.250.7 - COD2-FW Port-channel1.103

Распределение автономных систем\
65200 - Spine1-2\
65201-65204 - Leaf1-4\
65502 - COD2-EDGE-R, COD2-FW (другое сетевое оборудование)
Распределение VRF\
VRF - Infrastructure для размещения сегмента с контроллером домена и подобных ему сегментов.

Распределение (mapping) VNI\
vxlan vlan 20 vni 20020\
vxlan vlan 100 vni 700100\
vxlan vlan 101 vni 700101\
vxlan vlan 103 vni 700103\
vxlan vrf Infrastructure vni 777

Распределение Port-channel, DF, ESI identifier, lacp system-id\
Leaf1-2 (DF - Leaf1 preference 200)
  Po1 
    identifier 0000:0000:0000:0000:1201
    lacp system-id 0000.0000.1201
  Po2
    identifier 0000:0000:0000:0000:1202
    lacp system-id 0000.0000.1202
Leaf3-4 (DF - Leaf3 preference 200)
  Po1
     identifier 0000:0000:0000:0000:3401
     lacp system-id 0000.0000.3401
  Po2
     identifier 0000:0000:0000:0000:3402
     lacp system-id 0000.0000.3402

### Протоколы и общие замечания
- Underlay сеть посроена на eBGP.
- BFD отключен для экономии ресурсов стенда.
- Для распространения BUM используем Ingress Replication.
- Модель EVPN L2 сервиса VLAN-based.
- Модель EVPN L3 сервиса Edge-routed Briging Symetric IRB.

## 2. Конфигурация
- настройка хостов
- настрйока Multihoming
- настройка L2VNI
- настройка L3VNI

Конфигурации устройств в части настройки Multihoming приведены ниже.

```
Leaf1#sh run int po1
interface Port-Channel1
   description =COD2-FW=
   switchport trunk allowed vlan 20,100,103
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:1201
      designated-forwarder election algorithm preference 200
      route-target import 00:00:00:00:12:01
   lacp system-id 0000.0000.1201
Leaf1#sh run int po2
interface Port-Channel2
   description =COD2-EDGE-R=
   switchport trunk allowed vlan 100-101
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:1202
      designated-forwarder election algorithm preference 200
      route-target import 00:00:00:00:12:02
   lacp system-id 0000.0000.1202


Leaf2#sh run int po1
interface Port-Channel1
   description =COD2-FW=
   switchport trunk allowed vlan 20,100,103
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:1201
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:12:01
   lacp system-id 0000.0000.1201
Leaf2#sh run int po2
interface Port-Channel2
   description =COD2-EDGE-R=
   switchport trunk allowed vlan 100-101
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:1202
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:12:02
   lacp system-id 0000.0000.1202


Leaf3#sh run int po1
interface Port-Channel1
   description =WWW=
   switchport trunk allowed vlan 20
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:3401
      designated-forwarder election algorithm preference 200
      route-target import 00:00:00:00:34:01
   lacp system-id 0000.0000.3401
Leaf3#sh run int po2
interface Port-Channel2
   description =COD2-DC=
   switchport trunk allowed vlan 10
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:3402
      designated-forwarder election algorithm preference 200
      route-target import 00:00:00:00:34:02
   lacp system-id 0000.0000.3402


Leaf4#sh run int po1
interface Port-Channel1
   description =WWW=
   switchport trunk allowed vlan 20
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:3401
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:34:01
   lacp system-id 0000.0000.3401
Leaf4#sh run int po2
interface Port-Channel2
   description =COD2-DC=
   switchport trunk allowed vlan 10
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:3402
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:34:02
   lacp system-id 0000.0000.3402
```

## 3. Проверка работоспособности
Отключить по 1 downlink на DF Leaf1,3 в сторону хостов, убедиться, что связность восстановилась с минимальными потерями или не нарушилась вовсе.
Состояние до начала проверки.

Данные с сервера COD2-DC, что видим:
- порты в сторону Leaf3-4 UP;
- Port-channel 1 UP, оба линка bundled, active;
- SVI Vl10 UP;
- проходит ping до COD2-WWW.
В дампах на стороне COD2-DC видим, что весь трафик проходит через Leaf3.
В дампах на стороне COD2-WWW видим, что reqest проходят через leaf4, а reply через Leaf3.

<img width="3771" height="2113" alt="image" src="https://github.com/user-attachments/assets/fb61ecc8-c83d-4e53-a338-43fe3795e798" />

```
COD2-DC#sh int desc
Interface                      Status         Protocol Description
Gi0/0                          up             up       =Leaf3_Eth8=
Gi0/1                          up             up       =Leaf4_Eth8=
Gi0/2                          down           down
Gi0/3                          down           down
Gi1/0                          down           down
Gi1/1                          down           down
Gi1/2                          down           down
Gi1/3                          down           down
Po1                            up             up       =MH_Leaf3-4_Po2=
Vl10                           up             up       =DC=

COD2-DC#sh ip int bri
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  up                    up
GigabitEthernet0/1     unassigned      YES unset  up                    up
GigabitEthernet0/2     unassigned      YES unset  down                  down
GigabitEthernet0/3     unassigned      YES unset  down                  down
GigabitEthernet1/0     unassigned      YES unset  down                  down
GigabitEthernet1/1     unassigned      YES unset  down                  down
GigabitEthernet1/2     unassigned      YES unset  down                  down
GigabitEthernet1/3     unassigned      YES unset  down                  down
Port-channel1          unassigned      YES unset  up                    up
Vlan10                 10.2.10.1       YES NVRAM  up                    up

COD2-DC#sh int vlan 10
Vlan10 is up, line protocol is up
  Hardware is Ethernet SVI, address is 5036.7300.800a (bia 5036.7300.800a)
  Description: =DC=
  Internet address is 10.2.10.1/24
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive not supported
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 01:11:40, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     6303 packets input, 718484 bytes, 0 no buffer
     Received 0 broadcasts (0 IP multicasts)
     0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     4858 packets output, 553704 bytes, 0 underruns
     0 output errors, 0 interface resets
     0 unknown protocol drops
     0 output buffer failures, 0 output buffers swapped out

COD2-DC#sh etherchannel detail
                Channel-group listing:
                ----------------------

Group: 1
----------
Group state = L2
Ports: 2   Maxports = 4
Port-channels: 1 Max Port-channels = 4
Protocol:   LACP
Minimum Links: 0


                Ports in the group:
                -------------------
Port: Gi0/0
------------

Port state    = Up Mstr Assoc In-Bndl
Channel group = 1           Mode = Active          Gcchange = -
Port-channel  = Po1         GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/0     SA      bndl      32768         0x1       0x1     0x1         0x3D

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SA      32768     0000.0000.3402  18s    0x0    0x2    0x8     0x3D

Age of the port in the current state: 0d:01h:14m:24s

Port: Gi0/1
------------

Port state    = Up Mstr Assoc In-Bndl
Channel group = 1           Mode = Active          Gcchange = -
Port-channel  = Po1         GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/1     SA      bndl      32768         0x1       0x1     0x2         0x3D

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/1     SA      32768     0000.0000.3402  21s    0x0    0x2    0x8     0x3D

Age of the port in the current state: 0d:01h:29m:46s

                Port-channels in the group:
                ---------------------------

Port-channel: Po1    (Primary Aggregator)

------------

Age of the Port-channel   = 0d:01h:29m:56s
Logical slot/port   = 16/0          Number of ports = 2
HotStandBy port = null
Port state          = Port-channel Ag-Inuse
Protocol            =   LACP
Port security       = Disabled
Load share deferral = Disabled

Ports in the Port-channel:

Index   Load   Port     EC state        No of bits
------+------+------+------------------+-----------
  0     00     Gi0/0    Active             0
  0     00     Gi0/1    Active             0

Time since last port bundled:    0d:01h:14m:24s    Gi0/0
Time since last port Un-bundled: 0d:01h:20m:10s    Gi0/0
```
Данные с сервера COD2-WWW, что видим:
- порты в сторону Leaf3-4 UP;
- Port-channel 1 UP, оба линка bundled, active;
- SVI Vl20 UP;
- проходит ping до COD2-DC.
В дампах на стороне COD2-WWW видим, что request проходит через leaf3, а replay через Leaf4.
В дампах на стороне COD2-DC видим,  что весь трафик проходит через Leaf3.
```
COD2-WWW#sh int desc
Interface                      Status         Protocol Description
Gi0/0                          up             up       =Leaf3_Eth7=
Gi0/1                          up             up       =Leaf4_Eth7=
Gi0/2                          down           down
Gi0/3                          down           down
Gi1/0                          down           down
Gi1/1                          down           down
Gi1/2                          down           down
Gi1/3                          down           down
Po1                            up             up       =MH_Leaf3-4_Po1=
Vl20                           up             up       =WWW_SRV=
COD2-WWW#sh ip int bri
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  up                    up
GigabitEthernet0/1     unassigned      YES unset  up                    up
GigabitEthernet0/2     unassigned      YES unset  down                  down
GigabitEthernet0/3     unassigned      YES unset  down                  down
GigabitEthernet1/0     unassigned      YES unset  down                  down
GigabitEthernet1/1     unassigned      YES unset  down                  down
GigabitEthernet1/2     unassigned      YES unset  down                  down
GigabitEthernet1/3     unassigned      YES unset  down                  down
Port-channel1          unassigned      YES unset  up                    up
Vlan20                 10.2.20.1       YES TFTP   up                    up

COD2-WWW#sh int vlan20
Vlan20 is up, line protocol is up
  Hardware is Ethernet SVI, address is 5017.8a00.8014 (bia 5017.8a00.8014)
  Description: =WWW_SRV=
  Internet address is 10.2.20.1/24
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive not supported
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 01:17:18, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     3133 packets input, 357108 bytes, 0 no buffer
     Received 0 broadcasts (0 IP multicasts)
     0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     3164 packets output, 358968 bytes, 0 underruns
     0 output errors, 0 interface resets
     0 unknown protocol drops
     0 output buffer failures, 0 output buffers swapped out

COD2-WWW#sh etherchannel detail
                Channel-group listing:
                ----------------------

Group: 1
----------
Group state = L2
Ports: 2   Maxports = 4
Port-channels: 1 Max Port-channels = 4
Protocol:   LACP
Minimum Links: 0


                Ports in the group:
                -------------------
Port: Gi0/0
------------

Port state    = Up Mstr Assoc In-Bndl
Channel group = 1           Mode = Active          Gcchange = -
Port-channel  = Po1         GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/0     SA      bndl      32768         0x1       0x1     0x1         0x3D

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SA      32768     0000.0000.3401  27s    0x0    0x1    0x7     0x3D

Age of the port in the current state: 0d:01h:19m:29s

Port: Gi0/1
------------

Port state    = Up Mstr Assoc In-Bndl
Channel group = 1           Mode = Active          Gcchange = -
Port-channel  = Po1         GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/1     SA      bndl      32768         0x1       0x1     0x2         0x3D

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/1     SA      32768     0000.0000.3401  26s    0x0    0x1    0x7     0x3D

Age of the port in the current state: 0d:01h:38m:16s

                Port-channels in the group:
                ---------------------------

Port-channel: Po1    (Primary Aggregator)

------------

Age of the Port-channel   = 0d:05h:48m:49s
Logical slot/port   = 16/0          Number of ports = 2
HotStandBy port = null
Port state          = Port-channel Ag-Inuse
Protocol            =   LACP
Port security       = Disabled
Load share deferral = Disabled

Ports in the Port-channel:

Index   Load   Port     EC state        No of bits
------+------+------+------------------+-----------
  0     00     Gi0/0    Active             0
  0     00     Gi0/1    Active             0

Time since last port bundled:    0d:01h:19m:29s    Gi0/0
Time since last port Un-bundled: 0d:01h:25m:15s    Gi0/0
```
<img width="3787" height="2148" alt="image" src="https://github.com/user-attachments/assets/b688f83d-4af5-4dc3-9b8a-356c87ce35d0" />

Состояние таблиц маршрутизации EVPN

```
Leaf1#sh bgp evpn vni 20020
BGP routing table information for VRF default
Router identifier 172.17.101.0, local AS number 65201
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 172.17.101.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.102.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.102.0          -       100     0       65200 65202 i
 *  ec    RD: 172.17.102.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.102.0          -       100     0       65200 65202 i
 * >Ec    RD: 172.17.103.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.102.0:20 mac-ip 001e.e5dc.47c0
                                 172.17.102.0          -       100     0       65200 65202 i
 *  ec    RD: 172.17.102.0:20 mac-ip 001e.e5dc.47c0
                                 172.17.102.0          -       100     0       65200 65202 i
 * >Ec    RD: 172.17.103.0:20 mac-ip 5017.8a00.8014
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:20 mac-ip 5017.8a00.8014
                                 172.17.103.0          -       100     0       65200 65203 i
 * >      RD: 172.17.101.0:20 imet 172.17.101.0
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.102.0:20 imet 172.17.102.0
                                 172.17.102.0          -       100     0       65200 65202 i
 *  ec    RD: 172.17.102.0:20 imet 172.17.102.0
                                 172.17.102.0          -       100     0       65200 65202 i
 * >Ec    RD: 172.17.103.0:20 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:20 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:20 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:20 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i

Leaf2#sh bgp evpn vni 20020
BGP routing table information for VRF default
Router identifier 172.17.102.0, local AS number 65202
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 172.17.101.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.101.0          -       100     0       65200 65201 i
 *  ec    RD: 172.17.101.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.101.0          -       100     0       65200 65201 i
 * >      RD: 172.17.102.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.103.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.104.0          -       100     0       65200 65204 i
 * >      RD: 172.17.102.0:20 mac-ip 001e.e5dc.47c0
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.103.0:20 mac-ip 5017.8a00.8014
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:20 mac-ip 5017.8a00.8014
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.101.0:20 imet 172.17.101.0
                                 172.17.101.0          -       100     0       65200 65201 i
 *  ec    RD: 172.17.101.0:20 imet 172.17.101.0
                                 172.17.101.0          -       100     0       65200 65201 i
 * >      RD: 172.17.102.0:20 imet 172.17.102.0
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.103.0:20 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:20 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:20 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:20 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i

Leaf3#sh bgp evpn vni 20020
BGP routing table information for VRF default
Router identifier 172.17.103.0, local AS number 65203
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 172.17.101.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.101.0          -       100     0       65200 65201 i
 *  ec    RD: 172.17.101.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.101.0          -       100     0       65200 65201 i
 * >Ec    RD: 172.17.102.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.102.0          -       100     0       65200 65202 i
 *  ec    RD: 172.17.102.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.102.0          -       100     0       65200 65202 i
 * >      RD: 172.17.103.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.104.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.102.0:20 mac-ip 001e.e5dc.47c0
                                 172.17.102.0          -       100     0       65200 65202 i
 *  ec    RD: 172.17.102.0:20 mac-ip 001e.e5dc.47c0
                                 172.17.102.0          -       100     0       65200 65202 i
 * >      RD: 172.17.103.0:20 mac-ip 5017.8a00.8014
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.101.0:20 imet 172.17.101.0
                                 172.17.101.0          -       100     0       65200 65201 i
 *  ec    RD: 172.17.101.0:20 imet 172.17.101.0
                                 172.17.101.0          -       100     0       65200 65201 i
 * >Ec    RD: 172.17.102.0:20 imet 172.17.102.0
                                 172.17.102.0          -       100     0       65200 65202 i
 *  ec    RD: 172.17.102.0:20 imet 172.17.102.0
                                 172.17.102.0          -       100     0       65200 65202 i
 * >      RD: 172.17.103.0:20 imet 172.17.103.0
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.104.0:20 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:20 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i

Leaf4#sh bgp evpn vni 20020
BGP routing table information for VRF default
Router identifier 172.17.104.0, local AS number 65204
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 172.17.101.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.101.0          -       100     0       65200 65201 i
 *  ec    RD: 172.17.101.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.101.0          -       100     0       65200 65201 i
 * >Ec    RD: 172.17.102.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.102.0          -       100     0       65200 65202 i
 *  ec    RD: 172.17.102.0:20 auto-discovery 0 0000:0000:0000:0000:1201
                                 172.17.102.0          -       100     0       65200 65202 i
 * >Ec    RD: 172.17.103.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 172.17.103.0          -       100     0       65200 65203 i
 * >      RD: 172.17.104.0:20 auto-discovery 0 0000:0000:0000:0000:3401
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.102.0:20 mac-ip 001e.e5dc.47c0
                                 172.17.102.0          -       100     0       65200 65202 i
 *  ec    RD: 172.17.102.0:20 mac-ip 001e.e5dc.47c0
                                 172.17.102.0          -       100     0       65200 65202 i
 * >Ec    RD: 172.17.103.0:20 mac-ip 5017.8a00.8014
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:20 mac-ip 5017.8a00.8014
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.101.0:20 imet 172.17.101.0
                                 172.17.101.0          -       100     0       65200 65201 i
 *  ec    RD: 172.17.101.0:20 imet 172.17.101.0
                                 172.17.101.0          -       100     0       65200 65201 i
 * >Ec    RD: 172.17.102.0:20 imet 172.17.102.0
                                 172.17.102.0          -       100     0       65200 65202 i
 *  ec    RD: 172.17.102.0:20 imet 172.17.102.0
                                 172.17.102.0          -       100     0       65200 65202 i
 * >Ec    RD: 172.17.103.0:20 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:20 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 * >      RD: 172.17.104.0:20 imet 172.17.104.0
                                 -                     -       -       0       i


Leaf1#sh bgp evpn vni 10010
BGP routing table information for VRF default
Router identifier 172.17.101.0, local AS number 65201
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 172.17.103.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i

Leaf2#sh bgp evpn vni 10010
BGP routing table information for VRF default
Router identifier 172.17.102.0, local AS number 65202
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 172.17.103.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i

Leaf3#sh bgp evpn vni 10010
BGP routing table information for VRF default
Router identifier 172.17.103.0, local AS number 65203
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 172.17.103.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 * >      RD: 172.17.103.0:10 mac-ip 5036.7300.800a
                                 -                     -       -       0       i
 * >      RD: 172.17.103.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 * >      RD: 172.17.103.0:10 imet 172.17.103.0
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i

Leaf4#sh bgp evpn vni 10010
BGP routing table information for VRF default
Router identifier 172.17.104.0, local AS number 65204
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 172.17.103.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.103.0          -       100     0       65200 65203 i
 * >      RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.103.0          -       100     0       65200 65203 i
 * >      RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 * >      RD: 172.17.104.0:10 imet 172.17.104.0
                                 -                     -       -       0       i
```
Симмитируем ситуацию выхода из строя Leaf3 путем выключения линков 7 и 8.

Только лишь отключения линков на стороне Leaf3-4 оказалось недостаточно, поэтому пришлось также выключить соответствующие порты на стороне серверов.
В противном случае сервера продолжали отправлять трафик в упавшие линки. При этом не мопогало даже удаление линков между устройствами.

После устранения особенностей, связанных с виртуализацией, наблюдаем восстановление связности между хостами COD2-DC и COD2-www.

<img width="1349" height="664" alt="image" src="https://github.com/user-attachments/assets/a809a06a-06ab-4baa-91d1-fce81dbd2a89" />

<img width="4771" height="2175" alt="image" src="https://github.com/user-attachments/assets/c2afc86a-7278-43e9-9482-4a9ec15408fd" />

<img width="1855" height="1166" alt="image" src="https://github.com/user-attachments/assets/41098118-22b4-422a-bc5b-6ff3a6e99ed1" />

```
Leaf3(config)#int po 1
Leaf3(config-if-Po1)#shut
Leaf3(config-if-Po1)#exit
Leaf3(config)#int port-Channel 2
Leaf3(config-if-Po2)#shut
Leaf3(config-if-Po2)#end
Leaf3#sh int desc
Interface                      Status         Protocol           Description
Et1                            up             up                 =Spine1_Eth3=
Et2                            up             up                 =Spine2_Eth3=
Et3                            up             up
Et4                            up             up
Et5                            up             up
Et6                            up             up
Et7                            admin down     down               =WWW_Gi0/0=
Et8                            admin down     down               =COD2-DC_Gi0/0=
Lo0                            up             up                 Underlay
Ma1                            down           down
Po1                            admin down     down               =WWW=
Po2                            admin down     down               =COD2-DC=
Vl10                           up             up                 =WWW_Anycast_GW=
Vl4094                         up             up
Vx1                            up             up
Leaf3#

COD2-DC#sh int desc
*Nov  6 15:02:02.962: %SYS-5-CONFIG_I: Configured from consoleping 10.2.20.1 repeat 1000000
Type escape sequence to abort.
Sending 1000000, 100-byte ICMP Echos to 10.2.20.1, timeout is 2 seconds:
......................................................................
...............................................
Success rate is 0 percent (0/117)
COD2-DC#ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 52/75/110 ms

COD2-WWW#ping 10.2.10.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.10.1, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
COD2-WWW#ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 44/81/140 ms

Leaf1#sh bgp evpn vni 10010
BGP routing table information for VRF default
Router identifier 172.17.101.0, local AS number 65201
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i

Leaf2#sh bgp evpn vni 10010
BGP routing table information for VRF default
Router identifier 172.17.102.0, local AS number 65202
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 * >Ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i


Leaf3#sh bgp evpn vni 10010
BGP routing table information for VRF default
Router identifier 172.17.103.0, local AS number 65203
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a
                                 172.17.104.0          -       100     0       65200 65204 i
 * >Ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 172.17.104.0          -       100     0       65200 65204 i
 * >      RD: 172.17.103.0:10 imet 172.17.103.0
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i
 *  ec    RD: 172.17.104.0:10 imet 172.17.104.0
                                 172.17.104.0          -       100     0       65200 65204 i
Leaf3#sh mac address-table vlan 10
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    5036.7300.800a    DYNAMIC     Vx1        1       0:08:31 ago
Total Mac Addresses for this criterion: 1

          Multicast Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       ----        -----
Total Mac Addresses for this criterion: 0


Leaf4#sh bgp evpn vni 10010
BGP routing table information for VRF default
Router identifier 172.17.104.0, local AS number 65204
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 172.17.104.0:10 auto-discovery 0 0000:0000:0000:0000:3402
                                 -                     -       -       0       i
 * >      RD: 172.17.104.0:10 mac-ip 5036.7300.800a
                                 -                     -       -       0       i
 * >      RD: 172.17.104.0:10 mac-ip 5036.7300.800a 10.2.10.1
                                 -                     -       -       0       i
 * >Ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 *  ec    RD: 172.17.103.0:10 imet 172.17.103.0
                                 172.17.103.0          -       100     0       65200 65203 i
 * >      RD: 172.17.104.0:10 imet 172.17.104.0
                                 -                     -       -       0       i
Leaf4#sh mac address-table vlan 10
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    5036.7300.800a    DYNAMIC     Po2        1       0:06:15 ago
Total Mac Addresses for this criterion: 1

          Multicast Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       ----        -----
Total Mac Addresses for this criterion: 0

COD2-WWW(config)#int gi
COD2-WWW(config)#int gigabitEthernet 0/0
COD2-WWW(config-if)#shut
COD2-WWW(config-if)#
*Nov  6 15:04:11.479: %LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to administratively down
COD2-WWW(config-if)#
COD2-WWW(config-if)#

COD2-DC#ping 10.2.20.1 repeat 1000000
Type escape sequence to abort.
Sending 1000000, 100-byte ICMP Echos to 10.2.20.1, timeout is 2 seconds:
...................................................!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!.
Success rate is 90 percent (517/569), round-trip min/avg/max = 32/53/414 ms
```

