VPN. GRE. DmVPN

Цель:
1)Настроить GRE между офисами Москва и С.-Петербург 
2)Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги

## 1. Настроить GRE между офисами Москва и С.-Петербург 



Для настройки GRE

GRE tunnel будем использовать между R15 и R18 на loopback интерфейсах.

План:
1) Выбрать адресацию
2) Настроить Loopback  
3) Анонсировать peer BGP выбранные сети, если это небыло сделано до этого.
4) Настроить interface Tunnel 50 на R15 и R18
5) Настроить статический маршрут и редистрибьюцию.
6) Проверить доступность

Сеть туннеля 172.31.0.0/24

R15 ip адресс туннеля 172.31.0.1 255.255.255.0
R18 ip адресс туннеля 172.31.0.2 255.255.255.0
Туннель виртуальный L3 интерфейс, через который будет происходить маршрутизаия.


ip адресс source/destination, с которого будет строиться виртуальный туннель.

R15 ip 5.5.5.50/24
R18 ip 42.42.42.50/24

Приступим к настройке

R15

```
interface loopback 50
    ip address 5.5.5.50 255.255.255.255
    no sh
    exit
interface tunnel50
    tunnel mode gre ip
    ip address 172.31.0.1 255.255.255.0
    tunnel destination 42.42.42.50
    tunnel source 5.5.5.50
    keepalive 5
    ip mtu 1400
    ip tcp adjust-mss 1360
! keepalive 5 чтобы туннель не поднимался если недоступен destination

```

R18

```
interface loopback 50
    ip address 42.42.42.50 255.255.255.255
    no sh
    exit
interface tunnel50
    tunnel mode gre ip
    ip address 172.31.0.2 255.255.255.0
    tunnel destination 5.5.5.50 
    tunnel source 42.42.42.50
    keepalive 60
```


![](Pictures/Screenshot_1.png)

Доступ до туннельного адреса R15 мы имеем. И сеть туннеля непосредственно к нам подключена.

Перейдем к настройке маршрутизации. Предположим, что пользователи Москвы и Петербурга должны иметь с друг другом связь 

R15 redistribute статического маршрута в OSPF

```
ip route 10.128.0.0 255.255.255.0 172.31.0.2

route-map route-map-redistribute
    match ip address Peter
    exit
ip access-list standard Peter
    permit 10.128.0.0 0.0.0.255

router ospf 4
    redistribute static subnets route-map route-map-redistribute
```

SW3 Москва

![](Pictures/Screenshot_2.png)


R18 redistribute статического маршрута в EIGRP

```
ip route 10.0.0.0 255.255.255.128 172.31.0.1

route-map route-map-redistribute
    match ip address Moscow
    exit
ip access-list standard Moscow
    permit 10.0.0.0 0.0.0.127

router eigrp E
    address-family ipv4 unicast autonomous-system 1
    topology base

```

Из-за суммарногомаршрута там ни чего не прилетает на R18.


Доступ от сотрудников Московы до сотрудников Петербурга есть.

![](Pictures/Screenshot_3.png)

## 2. Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги



План:

1. Выберем Архитектуру  сети  DMVPN и Фазу
2. Выбрать адресацию
3. Настроить Loopback/Интерфейсы
4. Анонсировать peer BGP выбранные сети, если это небыло сделано до этого.
5. Настроить interface Tunnel 
6. Выбрать протокол динамической маршрутизации (Попробуй использовать VRF)
7. Проверить доступность

1. В качесиве HUB(NHS) Будут выступать R14, R15. Так как у нас более приоритетный маршрут через Ламас, более  корректным будет использовать Схему DMVPN "Dual Hub, Single Cloud"

![](Pictures/Screenshot_4.png)


