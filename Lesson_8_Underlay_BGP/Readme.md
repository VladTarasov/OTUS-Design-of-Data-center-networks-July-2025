# Занятие 8. Построение Underlay сети (BGP)

## **План работы**
1. Дизайн
  - Схема
  - Адресное пространство
  - Протоколы и общие замечания
2. Конфигурация
3. Проверка работоспособности

## 1. Дизайн

### Схема

<img width="846" height="518" alt="image" src="https://github.com/user-attachments/assets/5d279c77-82b5-4028-8d78-6e31ba98fa30" />

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
- Underlay сеть строим на eBGP.
- Для компактности конфигурации используем динамическую peer-group на стороне Spine, адреса соседей и номера их AS задаем через диапазоны (bgp listen range и peer-filter).
- Используем аутентификацию.
- Настраивам мнимальные таймеры 3 9.
- Все Spine находятся в одной AS 65100, каждый Leaf находятся в своей AS из диапазона 65101-65103. Принцип нумерации 65m00 - для Spine, где m - номер POD от 0 до 9; 65mNN для Leaf, где NN - порядковый номер Leaf от 00 до 99 (на случай масштабирования).
- Для анонсированя loopback интерфейсов, используем редистрибьюцию connected сетей в комбинации с ip prefix-lists и route-map.
- Топология с единственным POD.
- Все настроено в GRT, не используется VRF.
- Используем BFD.
- Работает ECMP.

## 2. Конфигурация

Ниже приведены конфигурации узлов.

**Spine1**
```
router bgp 65100
   router-id 10.0.1.0
   bgp listen range 10.2.0.0/15 peer-group Leaf_PG peer-filter Leaf
   neighbor Leaf_PG peer group
   neighbor Leaf_PG bfd
   neighbor Leaf_PG timers 3 9
   neighbor Leaf_PG password 7 n5V7FYyq00M=
   redistribute connected route-map loopback

peer-filter Leaf
      10 match as-range 65101-65103 result accept

ip prefix-list loopback
    seq 10 permit 10.0.1.0/32

route-map loopback permit 10
   match ip address prefix-list loopback
```
**Spine2**
```
router bgp 65100
   router-id 10.0.2.0
   bgp listen range 10.2.0.0/15 peer-group Leaf_PG peer-filter Leaf
   neighbor Leaf_PG peer group
   neighbor Leaf_PG bfd
   neighbor Leaf_PG timers 3 9
   neighbor Leaf_PG password 7 n5V7FYyq00M=
   redistribute connected route-map loopback

peer-filter Leaf
      10 match as-range 65101-65103 result accept

ip prefix-list loopback
    seq 10 permit 10.0.2.0/32

route-map loopback permit 10
   match ip address prefix-list loopback
```
**Leaf1**
```
router bgp 65101
   router-id 10.0.1.1
   maximum-paths 2 ecmp 2
   neighbor Spine_PG peer group
   neighbor Spine_PG remote-as 65100
   neighbor Spine_PG bfd
   neighbor Spine_PG timers 3 9
   neighbor Spine_PG password 7 iV7A4HcTNGo=
   neighbor 10.2.1.0 peer group Spine_PG
   neighbor 10.2.2.0 peer group Spine_PG
   redistribute connected route-map loopback

ip prefix-list loopback
    seq 10 permit 10.0.1.1/32

route-map loopback permit 10
   match ip address prefix-list loopback
```
**Leaf2**
```
router bgp 65102
   router-id 10.0.1.2
   maximum-paths 2 ecmp 2
   neighbor Spine_PG peer group
   neighbor Spine_PG remote-as 65100
   neighbor Spine_PG bfd
   neighbor Spine_PG timers 3 9
   neighbor Spine_PG password 7 iV7A4HcTNGo=
   neighbor 10.2.1.2 peer group Spine_PG
   neighbor 10.2.2.2 peer group Spine_PG
   redistribute connected route-map loopback

ip prefix-list loopback
   seq 10 permit 10.0.1.2/32

route-map loopback permit 10
   match ip address prefix-list loopback
```
**Leaf3**
```
router bgp 65103
   router-id 10.0.1.3
   maximum-paths 2 ecmp 2
   neighbor Spine_PG peer group
   neighbor Spine_PG remote-as 65100
   neighbor Spine_PG bfd
   neighbor Spine_PG timers 3 9
   neighbor Spine_PG password 7 iV7A4HcTNGo=
   neighbor 10.2.1.4 peer group Spine_PG
   neighbor 10.2.2.4 peer group Spine_PG
   redistribute connected route-map loopback

ip prefix-list loopback
   seq 10 permit 10.0.1.3/32

route-map loopback permit 10
   match ip address prefix-list loopback
```
