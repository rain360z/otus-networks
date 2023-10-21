# Лабораторная работ 4.
## Построение Overlay на основе VxLAN EVPN для L2 связанности между клиентами .

Цель.
- Настроить BGP peering между Leaf и Spine в AF l2vpn evpn
- Настроить связанность между клиентами в первой зоне
- План работы, адресное пространство, схема сети, настройки - зафиксированы в документации


План работ:
1) Адресное пространство, настройку оборудования используем из четвёртой лабораторной.
2) Внесём изменения в схему.
3) Сконфигурируем оборудование.
4) Проверка работоспособности.

 
## 1. Распределение ip адресов.

Адресация из 2 лабораторной

Таблица адресов  
|Уровень| Оборудование | Интерфейс  | ip-адрес | Маска |  Маршрут по умолчанию |
|-------|--------------|------------|----------|-------|-----------------------|
|Leaf|L-NX9500_1 |e1/1|172.16.1.1|255.255.255.254|N/A|
|    |           |e1/2|172.16.1.3|255.255.255.254|N/A|
|    |           |lo  |1.1.1.1   |255.255.255.255|N/A|
|Leaf|L-NX9500_2 |e1/1|172.16.2.1|255.255.255.254|N/A|
|    |           |e1/2|172.16.2.3|255.255.255.254|N/A|
|    |           |lo  |1.1.1.2   |255.255.255.255|N/A|
|Leaf|L-NX9500_3 |e1/1|172.16.3.1|255.255.255.254|N/A|
|    |           |e1/2|172.16.3.3|255.255.255.254|N/A|
|    |           |lo  |1.1.1.3   |255.255.255.255|N/A|
|Spine|S-NX9500_1|e1/1|172.16.1.0|255.255.255.254|N/A|
|     |          |e1/2|172.16.2.0|255.255.255.254|N/A|
|     |          |e1/3|172.16.3.0|255.255.255.254|N/A|
|     |          |lo  |2.2.2.1   |255.255.255.255|N/A|
|Spine|S-NX9500_2|e1/1|172.16.1.2|255.255.255.254|N/A|
|     |          |e1/2|172.16.2.2|255.255.255.254|N/A|
|     |          |e1/3|172.16.3.2|255.255.255.254|N/A|  
|     |          |lo  |2.2.2.2   |255.255.255.255|N/A|

## 2. Внесём изменения в схему

Настроить loopback интерфесы для NVE интерфейсов.


| Hostname | ASN   |router-id        |  LO NVE       |
|----------|-------|-----------------|---------------|
|S-NX9500_1|64601  |2.2.2.1          |100.122.122.1  |
|S-NX9500_2|64601  |2.2.2.2          |100.122.122.2  |
|L-NX9500_1|65001  |1.1.1.1          |100.111.111.1  |
|L-NX9500_2|65002  |1.1.1.2          |100.111.111.2  |
|L-NX9500_3|65003  |1.1.1.3          |100.111.111.3  |





## 3 Сконфигурируем оборудование.


Настройка на LEAF
```
## Enable Function
feature vn-segment-vlan-based
feature nv overlay
nv overlay evpn
!

router bgp 65001
router-id 1.1.1.1
timers bgp 3 9
bestpath as-path multipath-relax
reconnect-interval 10
log-neighbor-changes
address-family l2vpn evpn
maximum-paths 10
 

 template peer SPINES
bfd
remote-as 65999
update-source loopback0
ebgp-multihop 2
timers 3 9
address-family l2vpn evpn
send-community
send-community extended
rewrite-evpn-rt-asn
neighbor 4.4.4.4
inherit peer SPINES


# Mapping VLAN to VXLAN VNI
vlan 10
name VLAN_10
vn-segment 10010
!
evpn
vni 10010l2
rd auto
route-target import auto
route-target export auto

## Создание и конфигурация NVE интерфейса и анонс VNI
interface nve1
no shutdown
host-reachability protocol bgp
source-interface loopback100
member vni 10010
ingress-replication protocol bgp
```

SPINE 

```
nv overlay evpn
!
route-map NH_UNCHANGED permit 10
set ip next-hop unchanged
!
router bgp 65999
router-id 4.4.4.4
timers bgp 3 9
reconnect-interval 10
log-neighbor-changes
address-family l2vpn evpn
maximum-paths 10
retain route-target all


router bgp 65999
neighbor 1.1.1.1
remote-as 65001
update-source loopback0
ebgp-multihop 5
address-family l2vpn evpn
send-community
send-community extended
rewrite-evpn-rt-asn
route-map NH_UNCHANGED out
```

```
show mac address-table static interface nve 1
```

## 4 Проверка работоспособности.




