# **Топология**
<img width="1655" height="793" alt="image" src="https://github.com/user-attachments/assets/09f514ea-4070-4568-a1b6-adca6353c2d1" />

# **Адресное пространство**

Адресное пространство организовано согласно методике, описанной в презентации к занятию.

10.Dn.Sn.X

Где:\
Dn - диапазон значений в зависимости от номера ЦОД\
Sn - порядоковый номер спайн
X - адрес в порядке возрастания

Расчет для одного ЦОД, где

N=1\
Dn =[0..7]\
0,1 - loopback /32\
2,3 - p2p /31 основной и резервный\
4..7 - подсети сервисов

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

Сервисы /VLSM:

10.4.0.0/16
10.5.0.0/16
10.6.0.0/16
10.7.0.0/16

# **Конфигурация**
План работ:
- настройка интерфейсов loopback1,2
- настрока интерфейсов для p2p линков
- Выборочная проверка ping доступности Spine интерфесов с Leaf

Конфигурация интерфейсов перенесена на оборудование:

```
Spine1#sh ip int bri
                                                                        Address
Interface        IP Address       Status      Protocol           MTU    Owner
---------------- ---------------- ----------- ------------- ----------- -------
Ethernet1        10.2.1.0/31      up          up                1500
Ethernet2        10.2.1.2/31      up          up                1500
Ethernet3        10.2.1.4/31      up          up                1500
Loopback1        10.0.1.0/32      up          up               65535
Loopback2        10.1.1.0/32      up          up               65535
Management1      unassigned       down        down              1500

Spine2#sh ip int bri
                                                                        Address
Interface        IP Address       Status      Protocol           MTU    Owner
---------------- ---------------- ----------- ------------- ----------- -------
Ethernet1        10.2.2.0/31      up          up                1500
Ethernet2        10.2.2.2/31      up          up                1500
Ethernet3        10.2.2.4/31      up          up                1500
Loopback1        10.0.2.0/32      up          up               65535
Loopback2        10.1.2.0/32      up          up               65535
Management1      unassigned       down        down              1500

Leaf1#sh ip int bri
                                                                        Address
Interface        IP Address       Status      Protocol           MTU    Owner
---------------- ---------------- ----------- ------------- ----------- -------
Ethernet1        10.2.1.1/31      up          up                1500
Ethernet2        10.2.2.1/31      up          up                1500
Loopback1        10.0.1.1/32      up          up               65535
Loopback2        10.1.1.1/32      up          up               65535
Management1      unassigned       down        down              1500

Leaf2#sh ip int bri
                                                                        Address
Interface        IP Address       Status      Protocol           MTU    Owner
---------------- ---------------- ----------- ------------- ----------- -------
Ethernet1        10.2.1.3/31      up          up                1500
Ethernet2        10.2.2.3/31      up          up                1500
Loopback1        10.0.1.2/32      up          up               65535
Loopback2        10.1.1.2/32      up          up               65535
Management1      unassigned       down        down              1500

Leaf3#sh ip int bri
                                                                        Address
Interface        IP Address       Status      Protocol           MTU    Owner
---------------- ---------------- ----------- ------------- ----------- -------
Ethernet1        10.2.1.5/31      up          up                1500
Ethernet2        10.2.2.5/31      up          up                1500
Loopback1        10.0.1.3/32      up          up               65535
Loopback2        10.1.1.3/32      up          up               65535
Management1      unassigned       down        down              1500

Leaf1(config-if-Et2)#do ping 10.2.2.0
PING 10.2.2.0 (10.2.2.0) 72(100) bytes of data.
80 bytes from 10.2.2.0: icmp_seq=1 ttl=64 time=30.7 ms
80 bytes from 10.2.2.0: icmp_seq=2 ttl=64 time=14.0 ms
80 bytes from 10.2.2.0: icmp_seq=3 ttl=64 time=9.22 ms
80 bytes from 10.2.2.0: icmp_seq=4 ttl=64 time=2.06 ms
80 bytes from 10.2.2.0: icmp_seq=5 ttl=64 time=2.28 ms

--- 10.2.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 80ms
rtt min/avg/max/mdev = 2.060/11.671/30.722/10.536 ms, pipe 3, ipg/ewma 20.237/20.580 ms

Leaf2(config-if-Et1)#do ping 10.2.1.2
PING 10.2.1.2 (10.2.1.2) 72(100) bytes of data.
80 bytes from 10.2.1.2: icmp_seq=1 ttl=64 time=29.6 ms
80 bytes from 10.2.1.2: icmp_seq=2 ttl=64 time=21.3 ms
80 bytes from 10.2.1.2: icmp_seq=3 ttl=64 time=16.7 ms
80 bytes from 10.2.1.2: icmp_seq=4 ttl=64 time=2.31 ms
80 bytes from 10.2.1.2: icmp_seq=5 ttl=64 time=2.61 ms

--- 10.2.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 75ms
rtt min/avg/max/mdev = 2.316/14.526/29.604/10.678 ms, pipe 3, ipg/ewma 18.995/21.324 ms

Leaf3(config-if-Et1)#do ping 10.2.1.4
PING 10.2.1.4 (10.2.1.4) 72(100) bytes of data.
80 bytes from 10.2.1.4: icmp_seq=1 ttl=64 time=25.8 ms
80 bytes from 10.2.1.4: icmp_seq=2 ttl=64 time=11.8 ms
80 bytes from 10.2.1.4: icmp_seq=3 ttl=64 time=2.13 ms
80 bytes from 10.2.1.4: icmp_seq=4 ttl=64 time=2.27 ms
80 bytes from 10.2.1.4: icmp_seq=5 ttl=64 time=2.91 ms

--- 10.2.1.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 85ms
rtt min/avg/max/mdev = 2.139/9.010/25.852/9.183 ms, pipe 2, ipg/ewma 21.290/16.966 ms
```
