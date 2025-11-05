Проектная работа\
Тема: **Разработка дизайна сети геораспределенного ЦОД компании с использованием VxLAN**\
Содержание:
- Постановка задачи
- Общая концепция решения
- Схема сети
- Адресный план
- Виртуальное моделирование

**Постановка задачи**\
  В геораспределенную структуру компании (центральный офис, филиалы, представительства, ЦОД) планируется добавить дублирующий центр обработки данных с целью обеспечения катастрофоустойчивости (полный выход из строя) основного. Таким образом непосредственно бизнес сервисы функционируют независимо друг от друга, но
  при этом обеспечивается репликация баз данных. файловых серверов, контроллеров домена и т.п., чтобы при запуске в работу второго ЦОД расхождения в актуальности данных (потеря данных) были минимальными. Дублирующий ЦОД располагается в удаленной географической точке (как минимум другой регион). Канал связи между ЦОД - L2VPN или Интернет, выделение оптического волокна или волны у провайдера услуг связи не предусматривается. Миграция виртуальных машин предполагается  только в пределах одного ЦОД. Непрерывное управление инфраструктурой и мониторинг обоих ЦОД должны обеспечиваться из Центрального офиса. 

**Общая концепция решения**\
Для решения поставленной задачи предлагается:
- Использовать топологию CLOS с двумя Spine коммутаторами и 4 Leaf, собранными в Multihoming пары.
- Одна из пар выполняет функции Boarder Leaf для подключения маршрутизаторов и межсетевых экранов.
- Нагрузка (конечные устройства) подключаются к Leaf с использованием LAG минимум двумя линиями.
- На базе топологии CLOS построить IP-фабрику и для работы Underlay применить протокол eBGP (выбор может быть изменен в пользу iBGP или протоклов IGP, если это рекомендует вендор).
- Для ускорения реакции протоколов маршрутизации на сбои использовать протокол BFD, а для балансировки нагрузки внутри фабрики включить ECMP (path > 8). 
- Overlay сеть построить на основе VxLAN, а в качетстве его Control Plane применить расширение BGP - EVPN.
- Поэтапно придти к архитектуре 2-х ЦОД объединенных в Multipod. Два промежуточных этапа - это построение фабрики во втором ЦОД и соединение ее с первым, затем построение фабрики в первом ЦОД.
- Связать площадки по L3. На каждой площадке использовать уникальную адресацию... (?)
- IT-инфраструктуру ЦОД поместить в отдельный VRF, с возможность горизонтального взаимодействия между собой и инфрастурктурой первого ЦОД без участия FW. На этапе Multifabric стараемся не использовать много VRF из-за сложностей дальнейшего масштабирования.
- Взаимодействие инфраструктуры с сервисами и Интернет осуществляется через FW.
- В случае сжатых сроков и необходимости предоставлять сервисы L2 возможен маневр - при достаточном MTU в DCI один коммутатор из состава Boarder Leaf можно разместить в ЦОД1, затерминировав на нем подсети IT-инфраструктуры и построив через него Multipod. В дальнейшем развивать эту схему.

Плюсы данного решения:
- Обеспечивается минимум двойное резервирование на уровне: соединительных линий, узлов сети и путей прохождения трафика (включая возможность балансировки нагрузки).
- Присутствует возможность балансировки нагрузки в IP-фабрике за счет ECMP.
- Отсутствует L2 сегмент, и следовательно вероятность образования петель и broadcast-штормов, нет необходимости в настройке STP, прощу и безопаснее проводить работы по коммутации.
- Отсутствет стекирование, у каждого устройства независимый Control Plane, сделовательно проще выполнять работы по обслуживанию (обновлению ПО).
- Простота масштабирования, добавления новых узлова сети и даже ЦОД (в случае Multipod).
- Проста и удобство конфигурации: после построения Underlay и Overlay изменения вносятся только на Leaf, что создает благоприятные условия для внедрения элементов автоматизации.
- В случае использования eBGP имеем единый протокол для Underlay и Overlay, что также упрощает конфигурацию.

 Недостатки решения:
 - Необходимость в большем количестве коммутаторов в стравнении с текущим Tier3.
 - Необходимось в более компетентных инженерах.
 - И как следствие первых двух пунктов вероятность значительного удорожания и увеличения сроков реализации проекта в целом.

**Схема сети**
0-й этап Tier3

<img width="1674" height="2385" alt="image" src="https://github.com/user-attachments/assets/98c0bd63-10d7-4391-95e8-8aaf3bd13788" />

1-й этап SingleFabric

<img width="2817" height="1986" alt="image" src="https://github.com/user-attachments/assets/fede1edc-a445-4df9-bafb-25e9cd216e23" />

2-й этап MultiFabric

<img width="2199" height="1518" alt="image" src="https://github.com/user-attachments/assets/d3ed4546-5c99-4ef0-87e4-aa26380c58a8" />

3-й этап MultiPod

<img width="3109" height="2106" alt="image" src="https://github.com/user-attachments/assets/5054c8d7-1e68-4425-8c2f-72d305bccb51" />

**Адресный план**

ЦОД1

IP-адреса\
172.16.0.0/16 le 32 - аресация сетевой инфраструктуры ЦОД1.
- 172.16.A.X/32 - адресация для loopback-интерфейсов Spine, где A = 1-99 - порядковый номер Spine, X = 0-255 - порядоковый номер loopback-интерфейса.
- 172.16.B.X/32 - адресация для loopback-интерфейсов Leaf, где B = 101-199 - порядковый номер Leaf, X = 0-255 - порядоковый номер loopback-интерфейса.
- 172.16.C.X/31 - адресация для p2p-cоединений Spine-Leaf, где c = 201-210 - порядковый номер Spine, X = 0-255.
- 172.16.250.0/24 - адресация для соединений фабрики с другими сетевыми устройствами и других сетевых устройств между собой.
- 172.16.255.X/32 - адресация для loopback-интерфейсов других сетевых устройств, где X = 0-255.
10.1.0.0/16 le 29 - адресация для сервисов ЦОД1.

Номера автономных систем\
65100 - номер автономной системы для Spine.
65101-65199 - номера автономных систем для Leaf, где 1-99 - порядковые номера Leaf.
65501 - автономная системя для пользовательских сервисов и служб.

VRF\
Infrastructure - для оборудования IT-инфраструктуры ЦОД.\
Services - для пользовательских сервисов и служб (номинально, реально GRT).

VNI\
100 - для сегментов с оборудованием и сервисами IT-инфраструктуры ЦОД.\
200 - для сегментов пользовательских сервисов и служб.\
777 - Simetric IRB

ЦОД2 

IP-адреса\
172.17.0.0/16 le 32 - аресация сетевой инфраструктуры ЦОД2.
- 172.17.A.X/32 - адресация для loopback-интерфейсов Spine, где A = 1-99 - порядковый номер Spine, X = 0-255 - порядоковый номер loopback-интерфейса.
- 172.17.B.X/32 - адресация для loopback-интерфейсов Leaf, где B = 101-199 - порядковый номер Leaf, X = 0-255 - порядоковый номер loopback-интерфейса.
- 172.17.C.X/31 - адресация для p2p-cоединений Spine-Leaf, где c = 201-210 - порядковый номер Spine, X = 0-255.
- 172.17.250.0/24 - адресация для соединений фабрики с другими сетевыми устройствами и других сетевых устройств между собой.
- 172.17.255.X/32 - адресация для loopback-интерфейсов других сетевых устройств, где X = 0-255.
10.2.0.0/16 le 29 - адресация для сервисов ЦОД2.

Номера автономных систем\
65200 - номер автономной системы для Spine.\
65201-65299 - номера автономных систем для Leaf, где 1-99 - порядковые номера Leaf.
65502 - автономная системя для пользовательских сервисов и служб.

VRF\
Infrastructure - для оборудования и сервисов IT-инфраструктуры ЦОД.
Services - для пользовательских сервисов и служб (номинально, реально GRT).

VNI\
100X - для сегментов с оборудованием и сервисами IT-инфраструктуры ЦОД, где X - номер VLAN.
200Y - для сегментов пользовательских сервисов и служб, где Y - номер VLAN.
777 - Simetric IRB
700 - для стыковочных VLAN (FW outside)

Другое\
192.168.0.0/16 - адресация для небольших удаленных площадок и RAVPN.
10.G.0.0/16 le 29 - адресация для сервисов на крупных площаках, где G = 3-255 - порядоковый номер (условный код) площадки.
10.0.0.0/16-  адресация для каналов L2, S2S VPN и т.п.
mtu = 9000 - на ip-линках внутри фибрики.


**Виртуальное моделирование**

Смоделируем первый этап реализации решения, где основной ЦОД имеет архитектуру Tier3, а дублирующий CLOS. Для простоты номинальную роль межсетевых экранов будут выполнять маршрутизаторы, при этом какие-либо правила фильтрации трафика отсутствуют, роль серверов во втором ЦОД выполняют коммутаторы, отключен BFD, не стоится S2S IPSEC. Основной ЦОД также представлен в упрощенном виде без реализации отказоустойчивости на L2 и L1 (без ипользования Stack, LAG) и полноценной проработки iBGP между CORE-SW и FW. Доступ в Интернет иммитируется по типу услуги IP-access, вопросы наличия у оргнизации собственной или арендованной автономной системы опускаются.
Схема виртуальной модели приведена на скриншоте ниже.

Конфигурации представлены в виде отдельных файлов в папке проекта.

Проверка работоспособности
Выполняются следующие условия:
- Связность между VRF внутри ЦОД осуществляется через межсетевой экран (трассировка).
- Связность между VRF Infrastructure разных ЦОД осуществляется через пограничные маршрутизаторы без участия межсетевого экрана (трассировка).
- Связность между VRF Services разных ЦОД отсутствует (трассировка).
- Доступ в Интернет в каждом ЦОД из всех VRF осущетсвляется только через межсетевой экран (трассировка).

Проверка ЦОД2
```
COD2-WWW#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is 10.2.20.254 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.2.20.254
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.2.20.0/24 is directly connected, Vlan20
L        10.2.20.1/32 is directly connected, Vlan20
COD2-WWW#ping 10.2.20.254
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.20.254, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 20/65/237 ms
COD2-WWW#ping 10.2.10.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.10.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 40/46/56 ms
COD2-WWW#ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 46/47/49 ms
COD2-WWW#trac
COD2-WWW#traceroute 10.2.10.1
Type escape sequence to abort.
Tracing the route to 10.2.10.1
VRF info: (vrf in name/id, vrf out name/id)
  1 10.2.20.254 34 msec 20 msec 18 msec
  2 172.17.250.6 67 msec 30 msec 39 msec
  3 10.2.10.254 54 msec 37 msec 33 msec
  4 10.2.10.1 46 msec *  218 msec
COD2-WWW#traceroute 8.8.8.8
Type escape sequence to abort.
Tracing the route to 8.8.8.8
VRF info: (vrf in name/id, vrf out name/id)
  1 10.2.20.254 30 msec 18 msec 18 msec
  2 172.17.250.0 27 msec 26 msec 24 msec
  3 192.168.175.2 31 msec 28 msec 27 msec
  4  *  *

COD2-DC#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is 10.2.10.254 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.2.10.254
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.2.10.0/24 is directly connected, Vlan10
L        10.2.10.1/32 is directly connected, Vlan10
COD2-DC#ping 10.2.10.254
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.10.254, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/30/107 ms
COD2-DC#ping 10.2.20.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.20.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 38/67/102 ms
COD2-DC#ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 44/67/137 ms
COD2-DC#trace
COD2-DC#traceroute 10.2.20.1
Type escape sequence to abort.
Tracing the route to 10.2.20.1
VRF info: (vrf in name/id, vrf out name/id)
  1 10.2.10.254 14 msec 5 msec 7 msec
  2 172.17.250.2 25 msec 18 msec 19 msec
  3 172.17.250.7 53 msec 30 msec 25 msec
  4 10.2.20.1 45 msec *  46 msec
COD2-DC#traceroute 8.8.8.8
Type escape sequence to abort.
Tracing the route to 8.8.8.8
VRF info: (vrf in name/id, vrf out name/id)
  1 10.2.10.254 34 msec 11 msec 10 msec
  2 172.17.250.2 29 msec 18 msec 17 msec
  3 172.17.250.7 35 msec 58 msec 44 msec
  4 172.17.250.0 50 msec 42 msec 39 msec
  5 192.168.175.2 49 msec 43 msec 43 msec
  6  *
```

Проверка в ЦОД1

```
COD1_WWW> sh ip

NAME        : COD1_WWW[1]
IP/MASK     : 10.1.20.1/24
GATEWAY     : 10.1.20.254
DNS         :
MAC         : 00:50:79:66:68:0c
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

COD1_WWW> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=126 time=30.210 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=126 time=32.802 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=126 time=27.129 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=126 time=25.962 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=126 time=29.150 ms

COD1_WWW>  ping 10.1.10.1

84 bytes from 10.1.10.1 icmp_seq=1 ttl=62 time=5.888 ms
84 bytes from 10.1.10.1 icmp_seq=2 ttl=62 time=2.791 ms
84 bytes from 10.1.10.1 icmp_seq=3 ttl=62 time=3.051 ms
84 bytes from 10.1.10.1 icmp_seq=4 ttl=62 time=2.656 ms
84 bytes from 10.1.10.1 icmp_seq=5 ttl=62 time=2.867 ms

COD1_WWW> ping 10.2.20.1

10.2.20.1 icmp_seq=1 timeout
10.2.20.1 icmp_seq=2 timeout
10.2.20.1 icmp_seq=3 timeout

COD1_WWW> trace 10.1.10.1
trace to 10.1.10.1, 8 hops max, press Ctrl+C to stop
 1   10.1.20.254   1.122 ms  0.919 ms  0.635 ms
 2   172.16.250.1   3.121 ms  2.798 ms  3.331 ms
 3   *10.1.10.1   2.363 ms (ICMP type:3, code:3, Destination port unreachable)

COD1_WWW> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   10.1.20.254   1.226 ms  1.102 ms  0.677 ms
 2   172.16.250.3   3.243 ms  3.622 ms  2.394 ms
 3   192.168.175.2   4.000 ms  2.986 ms  3.043 ms


COD1_DC> sh ip

NAME        : COD1_DC[1]
IP/MASK     : 10.1.10.1/24
GATEWAY     : 10.1.10.254
DNS         :
MAC         : 00:50:79:66:68:0b
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

COD1_DC> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=125 time=30.242 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=125 time=37.106 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=125 time=29.721 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=125 time=31.474 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=125 time=28.121 ms

COD1_DC> ping 10.1.20.1

84 bytes from 10.1.20.1 icmp_seq=1 ttl=62 time=4.404 ms
84 bytes from 10.1.20.1 icmp_seq=2 ttl=62 time=30.565 ms
84 bytes from 10.1.20.1 icmp_seq=3 ttl=62 time=1.983 ms
84 bytes from 10.1.20.1 icmp_seq=4 ttl=62 time=2.320 ms
84 bytes from 10.1.20.1 icmp_seq=5 ttl=62 time=1.986 ms

COD1_DC> ping 10.2.10.1

84 bytes from 10.2.10.1 icmp_seq=1 ttl=250 time=143.709 ms
84 bytes from 10.2.10.1 icmp_seq=2 ttl=250 time=20.751 ms
84 bytes from 10.2.10.1 icmp_seq=3 ttl=250 time=26.359 ms
84 bytes from 10.2.10.1 icmp_seq=4 ttl=250 time=23.325 ms
84 bytes from 10.2.10.1 icmp_seq=5 ttl=250 time=23.115 ms

COD1_DC> trace 10.1.20.1
trace to 10.1.20.1, 8 hops max, press Ctrl+C to stop
 1   10.1.10.254   1.832 ms  1.853 ms  2.294 ms
 2   172.16.250.0   2.472 ms  3.319 ms  2.823 ms
 3   *10.1.20.1   4.607 ms (ICMP type:3, code:3, Destination port unreachable)

COD1_DC> trace 10.2.10.1
trace to 10.2.10.1, 8 hops max, press Ctrl+C to stop
 1   10.1.10.254   1.974 ms  1.868 ms  1.339 ms
 2   172.16.250.5   4.393 ms  4.535 ms  3.443 ms
 3   10.0.0.2   4.599 ms  7.393 ms  2.513 ms
 4   172.17.250.2   7.771 ms  6.675 ms  6.644 ms
 5   10.2.10.254   25.096 ms  17.823 ms  18.588 ms
 6   *10.2.10.1   30.369 ms (ICMP type:3, code:3, Destination port unreachable)  *

COD1_DC> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   10.1.10.254   3.110 ms  1.374 ms  3.093 ms
 2   172.16.250.0   3.524 ms  3.556 ms  2.850 ms
 3   172.16.250.3   4.983 ms  2.953 ms  4.816 ms
 4   192.168.175.2   11.739 ms  4.736 ms  6.803 ms
^C 5     *

COD1_DC> ping 10.2.20.1

10.2.20.1 icmp_seq=1 timeout
10.2.20.1 icmp_seq=2 timeout
10.2.20.1 icmp_seq=3 timeout
10.2.20.1 icmp_seq=4 timeout
10.2.20.1 icmp_seq=5 timeout

```
