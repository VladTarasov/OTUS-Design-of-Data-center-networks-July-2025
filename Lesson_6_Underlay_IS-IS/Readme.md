# **План работы**
1. Подготовка дизайна
  - Схема
  - Адресное пространство
  - Протоколы
2. Конфигурация
3. Проверка работоспособности

## 1. Поготовка дизайна
### Схема
<img width="816" height="676" alt="image" src="https://github.com/user-attachments/assets/8623f24d-1d66-49cc-b0db-d3c79f7c618c" />

### Адресное пространство
Распределение адресов для loopback интерфейсов /32:

| Loopback         | Spine1   | Spine2   | Leaf1    |Leaf2     |Leaf3     |
| ---------------- |:--------:| --------:|----------|----------|----------|    
| Lo1              | 10.0.1.0 | 10.0.2.0 | 10.0.1.1 | 10.0.1.2 | 10.0.1.3 |

Распределение адресов для p2p интерфейсов /31:

|                  | Leaf1   |Leaf2    |Leaf3    |Spine1   |Spine2   |
| ---------------- |:-------:| -------:|---------|---------|---------|    
| Spine1           | 10.2.1.0| 10.2.1.2| 10.2.1.4|         |         |
| Spine2           | 10.2.2.0| 10.2.2.2| 10.2.2.4|         |         |
| Leaf1            |         |         |         |10.2.1.1 |10.2.2.1 |
| Leaf2            |         |         |         |10.2.1.3 |10.2.2.3 |
| Leaf3            |         |         |         |10.2.1.5 |10.2.2.5 |

### Протоколы
В основу underlay положен протокол IS-IS, где:
- Так как рассматривается сеть с единственным POD, то настроена одна область 49.0001 и все интефейсы относятся к уровню L1. Нет необходимости в редистрибьюции.
- System-ID получен преобразованием ip-адреса loopback.
- Используется аутентификая md5.
- Связи между узлами имеют тип сети point-to-point.
- Не используется VRF, все настроено в GRT.
- Включен bfd для повышения отказоустойчивости.

## 2. Конфигурация

Ниже приведены конфигурации узлов.

**Spine1**
```
router isis POD1
   net 49.0001.0100.0000.1000.00
   is-type level-1
   authentication mode md5 level-1
   authentication key 7 YpAWPw+DfNM= level-1
   !
   address-family ipv4 unicast
      bfd all-interfaces

interface Ethernet1
   description =Leaf1_Eth1=
   no switchport
   ip address 10.2.1.0/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Ethernet2
   description =Leaf2_Eth1=
   no switchport
   ip address 10.2.1.2/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Ethernet3
   description =Leaf3_Eth1=
   no switchport
   ip address 10.2.1.4/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Loopback1
   ip address 10.0.1.0/32
   isis enable POD1
```
**Spine2**
```
router isis POD1
   net 49.0001.0100.0000.2000.00
   is-type level-1
   authentication mode md5 level-1
   authentication key 7 YpAWPw+DfNM= level-1
   !
   address-family ipv4 unicast
      bfd all-interfaces

interface Ethernet1
   description =Leaf1_Eth2=
   no switchport
   ip address 10.2.2.0/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Ethernet2
   description =Leaf2_Eth2=
   no switchport
   ip address 10.2.2.2/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Ethernet3
   description =Leaf3_Eth2=
   no switchport
   ip address 10.2.2.4/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Loopback1
   ip address 10.0.2.0/32
   isis enable POD1
```
**Leaf1**
```
router isis POD1
   net 49.0001.0100.0000.1001.00
   is-type level-1
   authentication mode md5 level-1
   authentication key 7 YpAWPw+DfNM= level-1
   !
   address-family ipv4 unicast
      bfd all-interfaces

interface Ethernet1
   description =Spine1_Eth1=
   no switchport
   ip address 10.2.1.1/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Ethernet2
   description =Spine2_Eth1=
   no switchport
   ip address 10.2.2.1/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Loopback1
   ip address 10.0.1.1/32
   isis enable POD1
```
**Leaf2**
```
router isis POD1
   net 49.0001.0100.0000.1002.00
   is-type level-1
   authentication mode md5 level-1
   authentication key 7 YpAWPw+DfNM= level-1
   !
   address-family ipv4 unicast
      bfd all-interfaces
interface Ethernet1
   description =Spine1_Eth2=
   no switchport
   ip address 10.2.1.3/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Ethernet2
   description =Spine2_Eth2=
   no switchport
   ip address 10.2.2.3/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Loopback1
   ip address 10.0.1.2/32
   isis enable POD1
```
**Leaf3**
```
router isis POD1
   net 49.0001.0100.0000.1003.00
   is-type level-1
   authentication mode md5 level-1
   authentication key 7 YpAWPw+DfNM= level-1
   !
   address-family ipv4 unicast
      bfd all-interfaces

interface Ethernet1
   description =Spine1_Eth3=
   no switchport
   ip address 10.2.1.5/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Ethernet2
   description =Spine2_Eth3=
   no switchport
   ip address 10.2.2.5/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable POD1
   isis bfd
   isis network point-to-point
interface Loopback1
   ip address 10.0.1.3/32
   isis enable POD1
```
## 3. Проверка работоспособности
Соблюдаеются следующие условия:
- На всех узлах установлены отношения соседства.
- База данных ISIS содержит LSP, полученные от соседей.
- В таблице маршрутизации присутствуют маршруты IS-IS к loopback-интерфейсам и стыковочным сетям, при этом присутствуют эквивалентные маршурты (ECMP). Коды описания маршрутов удалены из вывода для краткости.
- BFD peers в состоянии UP.
- С Leaf1 доступны loopback-интерфейсы остальных узлов.

```
Spine1#sh isis neighbors

Instance  VRF      System Id        Type Interface
POD1      default  Leaf1            L1   Ethernet1
POD1      default  Leaf2            L1   Ethernet2
POD1      default  Leaf3            L1   Ethernet3

Spine1#sh isis database

IS-IS Instance: POD1 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS
    Spine1.00-00                 14  27516  1109    165 L1
    Leaf1.00-00                  16  45379  1030    140 L1
    Leaf2.00-00                   5  63584  1078    140 L1
    Leaf3.00-00                   5  55655  1113    140 L1
    Spine2.00-00                 15   7605  1110    165 L1

Spine1#sh bfd peers
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Ty
--------- ----------- ----------- -------------------- ----
10.2.1.1  4265968219  3242325992        Ethernet1(13)  norm
10.2.1.3  2135220427  1048789604        Ethernet2(14)  norm
10.2.1.5  2576563537   575178923        Ethernet3(15)  norm

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

Spine1#sh ip route isis

 I L1     10.0.1.1/32 [115/20] via 10.2.1.1, Ethernet1
 I L1     10.0.1.2/32 [115/20] via 10.2.1.3, Ethernet2
 I L1     10.0.1.3/32 [115/20] via 10.2.1.5, Ethernet3
 I L1     10.0.2.0/32 [115/30] via 10.2.1.1, Ethernet1
                               via 10.2.1.3, Ethernet2
                               via 10.2.1.5, Ethernet3
 I L1     10.2.2.0/31 [115/20] via 10.2.1.1, Ethernet1
 I L1     10.2.2.2/31 [115/20] via 10.2.1.3, Ethernet2
 I L1     10.2.2.4/31 [115/20] via 10.2.1.5, Ethernet3


Spine2#sh isis neighbors

Instance  VRF      System Id        Type Interface
POD1      default  Leaf1            L1   Ethernet1
POD1      default  Leaf2            L1   Ethernet2
POD1      default  Leaf3            L1   Ethernet3
Spine2#sh isis database

IS-IS Instance: POD1 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS
    Spine1.00-00                 14  27516  1051    165 L1
    Leaf1.00-00                  16  45379   972    140 L1
    Leaf2.00-00                   5  63584  1019    140 L1
    Leaf3.00-00                   5  55655  1055    140 L1
    Spine2.00-00                 15   7605  1052    165 L1

pine2#sh bfd peers
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Ty
--------- ----------- ----------- -------------------- ----
10.2.2.1  3259045032  3188542071        Ethernet1(13)  norm
10.2.2.3  1935409532  3555654101        Ethernet2(14)  norm
10.2.2.5  1635225613  1376246084        Ethernet3(15)  norm

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

Spine2#sh ip route isis

 I L1     10.0.1.0/32 [115/30] via 10.2.2.1, Ethernet1
                               via 10.2.2.3, Ethernet2
                               via 10.2.2.5, Ethernet3
 I L1     10.0.1.1/32 [115/20] via 10.2.2.1, Ethernet1
 I L1     10.0.1.2/32 [115/20] via 10.2.2.3, Ethernet2
 I L1     10.0.1.3/32 [115/20] via 10.2.2.5, Ethernet3
 I L1     10.2.1.0/31 [115/20] via 10.2.2.1, Ethernet1
 I L1     10.2.1.2/31 [115/20] via 10.2.2.3, Ethernet2
 I L1     10.2.1.4/31 [115/20] via 10.2.2.5, Ethernet3


Leaf1#sh isis neighbors

Instance  VRF      System Id        Type Interface
POD1      default  Spine1           L1   Ethernet1
POD1      default  Spine2           L1   Ethernet2
Leaf1#sh isis database

IS-IS Instance: POD1 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS
    Spine1.00-00                 14  27516  1032    165 L1
    Leaf1.00-00                  16  45379   953    140 L1
    Leaf2.00-00                   5  63584  1001    140 L1
    Leaf3.00-00                   5  55655  1036    140 L1
    Spine2.00-00                 15   7605  1033    165 L1

Leaf1#sh bfd peers
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Ty
--------- ----------- ----------- -------------------- ----
10.2.1.0  3242325992  4265968219        Ethernet1(13)  norm
10.2.2.0  3188542071  3259045032        Ethernet2(14)  norm

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

Leaf1#sh ip route isis

 I L1     10.0.1.0/32 [115/20] via 10.2.1.0, Ethernet1
 I L1     10.0.1.2/32 [115/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 I L1     10.0.1.3/32 [115/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 I L1     10.0.2.0/32 [115/20] via 10.2.2.0, Ethernet2
 I L1     10.2.1.2/31 [115/20] via 10.2.1.0, Ethernet1
 I L1     10.2.1.4/31 [115/20] via 10.2.1.0, Ethernet1
 I L1     10.2.2.2/31 [115/20] via 10.2.2.0, Ethernet2
 I L1     10.2.2.4/31 [115/20] via 10.2.2.0, Ethernet2


Leaf2#sh isis neighbors

Instance  VRF      System Id        Type Interface
POD1      default  Spine1           L1   Ethernet1
POD1      default  Spine2           L1   Ethernet2
Leaf2#sh isis database

IS-IS Instance: POD1 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS
    Spine1.00-00                 14  27516  1023    165 L1
    Leaf1.00-00                  16  45379   986    140 L1
    Leaf2.00-00                   5  63584   992    140 L1
    Leaf3.00-00                   5  55655  1027    140 L1
    Spine2.00-00                 15   7605  1024    165 L1

Leaf2#sh bfd peers
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Ty
--------- ----------- ----------- -------------------- ----
10.2.1.2  1048789604  2135220427        Ethernet1(13)  norm
10.2.2.2  3555654101  1935409532        Ethernet2(14)  norm

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

Leaf2#sh ip route isis

 I L1     10.0.1.0/32 [115/20] via 10.2.1.2, Ethernet1
 I L1     10.0.1.1/32 [115/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 I L1     10.0.1.3/32 [115/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 I L1     10.0.2.0/32 [115/20] via 10.2.2.2, Ethernet2
 I L1     10.2.1.0/31 [115/20] via 10.2.1.2, Ethernet1
 I L1     10.2.1.4/31 [115/20] via 10.2.1.2, Ethernet1
 I L1     10.2.2.0/31 [115/20] via 10.2.2.2, Ethernet2
 I L1     10.2.2.4/31 [115/20] via 10.2.2.2, Ethernet2


Leaf3#sh isis neighbors

Instance  VRF      System Id        Type Interface
POD1      default  Spine1           L1   Ethernet1
POD1      default  Spine2           L1   Ethernet2
Leaf3#sh isis database

IS-IS Instance: POD1 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS
    Spine1.00-00                 14  27516  1006    165 L1
    Leaf1.00-00                  16  45379  1006    140 L1
    Leaf2.00-00                   5  63584  1006    140 L1
    Leaf3.00-00                   5  55655  1010    140 L1
    Spine2.00-00                 15   7605  1007    165 L1

Leaf3#sh bfd peers
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Ty
--------- ----------- ----------- -------------------- ----
10.2.1.4   575178923  2576563537        Ethernet1(13)  norm
10.2.2.4  1376246084  1635225613        Ethernet2(14)  norm

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

Leaf3#sh ip route isis

 I L1     10.0.1.0/32 [115/20] via 10.2.1.4, Ethernet1
 I L1     10.0.1.1/32 [115/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 I L1     10.0.1.2/32 [115/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 I L1     10.0.2.0/32 [115/20] via 10.2.2.4, Ethernet2
 I L1     10.2.1.0/31 [115/20] via 10.2.1.4, Ethernet1
 I L1     10.2.1.2/31 [115/20] via 10.2.1.4, Ethernet1
 I L1     10.2.2.0/31 [115/20] via 10.2.2.4, Ethernet2
 I L1     10.2.2.2/31 [115/20] via 10.2.2.4, Ethernet2


Leaf1#ping 10.0.1.0
PING 10.0.1.0 (10.0.1.0) 72(100) bytes of data.
80 bytes from 10.0.1.0: icmp_seq=1 ttl=64 time=9.68 ms
80 bytes from 10.0.1.0: icmp_seq=2 ttl=64 time=1.88 ms
80 bytes from 10.0.1.0: icmp_seq=3 ttl=64 time=2.32 ms
80 bytes from 10.0.1.0: icmp_seq=4 ttl=64 time=2.39 ms
80 bytes from 10.0.1.0: icmp_seq=5 ttl=64 time=2.18 ms

--- 10.0.1.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 34m
rtt min/avg/max/mdev = 1.887/3.695/9.687/3.001 ms, ipg/ewma
Leaf1#ping 10.0.2.0
PING 10.0.2.0 (10.0.2.0) 72(100) bytes of data.
80 bytes from 10.0.2.0: icmp_seq=1 ttl=64 time=6.70 ms
80 bytes from 10.0.2.0: icmp_seq=2 ttl=64 time=2.42 ms
80 bytes from 10.0.2.0: icmp_seq=3 ttl=64 time=2.79 ms
80 bytes from 10.0.2.0: icmp_seq=4 ttl=64 time=2.34 ms
80 bytes from 10.0.2.0: icmp_seq=5 ttl=64 time=2.63 ms

--- 10.0.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 24m
rtt min/avg/max/mdev = 2.341/3.378/6.701/1.669 ms, ipg/ewma
Leaf1#ping 10.0.1.2
PING 10.0.1.2 (10.0.1.2) 72(100) bytes of data.
80 bytes from 10.0.1.2: icmp_seq=1 ttl=63 time=11.3 ms
80 bytes from 10.0.1.2: icmp_seq=2 ttl=63 time=5.78 ms
80 bytes from 10.0.1.2: icmp_seq=3 ttl=63 time=6.02 ms
80 bytes from 10.0.1.2: icmp_seq=4 ttl=63 time=5.46 ms
80 bytes from 10.0.1.2: icmp_seq=5 ttl=63 time=5.71 ms

--- 10.0.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 43m
rtt min/avg/max/mdev = 5.468/6.874/11.386/2.263 ms, ipg/ewm
Leaf1#ping 10.0.1.3
PING 10.0.1.3 (10.0.1.3) 72(100) bytes of data.
80 bytes from 10.0.1.3: icmp_seq=1 ttl=63 time=12.6 ms
80 bytes from 10.0.1.3: icmp_seq=2 ttl=63 time=4.89 ms
80 bytes from 10.0.1.3: icmp_seq=3 ttl=63 time=5.11 ms
80 bytes from 10.0.1.3: icmp_seq=4 ttl=63 time=4.64 ms
80 bytes from 10.0.1.3: icmp_seq=5 ttl=63 time=5.53 ms

--- 10.0.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 44m
rtt min/avg/max/mdev = 4.640/6.575/12.696/3.075 ms, pipe 2,
```
