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
### Адресное пространство
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

```

```
