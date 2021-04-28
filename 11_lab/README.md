#  IPv4/v6

## OSPF

Цель:
Настроить OSPF офисе Москва Разделить сеть на зоны Настроить фильтрацию между зонами

Маршрутизаторы R14-R15 находятся в зоне 0 - backbone
Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по-умолчанию
Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию
Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101
Настройка для IPv6 повторяет логику IPv4
План работы и изменения зафиксированы в документации   

###  Задание: 
1. Настроить OSPF IPv4    
 1.1 Задать на схеме Area и Router ID
 1.2 Настроить работу OSPF на всех маршрутизаторах в сети
 1.3 Поставить фильтр на AREA 101, маршрутизаторы получают только маршрут по умолчанию.
 1.4 Поставить фильтр 102, маршрутизаторы получают все маршруты, кроме маршрутов в зону 101
2. Настроить OSPF IPv6
3. Подумать над оптимизацией


###  Решение:

### 1. Настроить OSPF IPv4   

### 1.1. Задать на схеме Area и Router ID

СХЕМА  
![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_1.png)

### 2. Настроить работу OSPF на всех маршрутизаторах в сети

Для более наглядной фильтрации настраим ospf везде

2.1 Настроим работу OSFP на R14
 
Команды для настройки OSPF на примере R14

Запустим процесс ospf и зададим уникальный router-id:

``` R14(config)# router ospf 4  
    R14(config-router)# router-id 0.0.0.14   
```

Отключим отправку hello пакетов на всех интерфейсах  
``` R14(config-router)#passive-interface default ```

Теперь укажем интерфейсы, на которых мы будем отправлять hello пакеты. Для R1 это e0/0 и e0/1, e0/3:     

``` R14(config-router)#no passive-interface fa0/0   
    R14(config-router)#no passive-interface fa0/1  
```
Текущий маршрутизатор имет роль ASBR/
ASBR имеет доступ до провайдера internet. Укажем другим маршрутизаторам в Автономной системе,
что мы занаем маршрут по умолчанию.   

```  R14(config-router)#default-information originate  ```

Настроим работу OSPF на интерфейсе e0/3. Укажем что между двумя маршрутизаторами 
в области 101 point-to-point сеть.
``` 
R14(config-if)#int e0/3
R14(config-if)# ip ospf 4 area 101
R14(config-if)# ip ospf network point-to-point

```
Настроим e0/0 и e0/1

``` 
R14(config-if)#int e0/0
R14(config-if)# ip ospf 4 area 0
R14(config-if)# ip ospf network point-to-point
R14(config-if)#int e0/1
R14(config-if)# ip ospf 4 area 0
R14(config-if)# ip ospf network point-to-point
```

В соответствии с зонами настроим остальные маршрутизаторы.

### 1.2 Настроить работу OSPF на всех маршрутизаторах в сети

Проверим в LSDB кажой AREA Router Link States .
AREA 0 
![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_2.png)

AREA 101  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_3.png)

AREA 102  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_4.png)  

AREA 10  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_5.png)

### 1.3 Поставить фильтр на AREA 101, маршрутизаторы получают только маршрут по умолчанию.

Под данные требования приходит Total Stub Area

Посмотирм текущие маршруты в AREA 101

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_6.png)

После настройки зоны должны остаться присоединенные сети и Default route

В случае зоны Total area stub. Необходимо на маршрутизаторах в зоне 101 прописать слежующие настройки:

ABR

```   
(config-router)# area 101 stub no-summary   
```

На всех маршрутизаторах зоны

``` area 1 stub ```

Посмотрим маршруты на R19 

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_7.png)

###1.4 Поставить фильтр 102, маршрутизаторы получают все маршруты, кроме маршрутов в зону 101

К данной зоне применим LSA Filter, чтобы маршруты из сети 101 обявлялись в зоне 102

Проверим текущие маршруты в AREA 102

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_8.png) 

Необходимо добавить фильтр на ABR граничащие с зоной 102

``` 
    (config)# ip prefix-list non101 seq 5 dany 10.127.255.0/28   
    (config)# ip prefix-list non101 seq 5 dany 10.127.255.224/28
     
    (config-router)#area 0 filter-list prefix out   
```

т.о мы блокруем сети выходящие из area 0 с данными прейфиксами.

Проверим текущие маршруты в AREA 102

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_9.png)

видим что в таблице маршрутизации из других зон нету сетей из AREA 101

2. Настроить OSPF IPv6
2.1 Настроить Total stub Area 101
Настроим OSPF на R14

```
    R14(config)# ipv6 unicast-routing
    R14(config)# ipv6 router ospf 6
    R14(config-rtr)# router-id 0.0.0.4
    R14(config)# interface range e0/0-1
    R14(config-if)# ipv6 ospf 6 area 10
    R14(config)# interface e0/3
    R14(config-if)# ipv6 ospf 6 area 10
```

Проверим соседства во всех зонах

Area 101

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_10.png)


AREA 0 

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_11.png)

AREA 102  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_12.png)

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_13.png)

2.1 Настроить Total stub Area 101

Настройки аналогичные с ipv4

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_14.png)


2.3 Настроим фильтр на зоне 102

```
R15(config)# ipv6 prefix-list non101 seq 4 deny 1:1:1:1::/64
R15(config)# ipv6 prefix-list non101 seq 4 deny 1:1:1:15::/64
R15(config-rtr)# area 102 filter-list prefix non101 in
```
![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_15.png)

Исходя из таблица маршрутизации, у нас вместе с AREA 101 пропали маршруты и AREA 0

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_16.png)

Настройка ospf ipv6 закончена.

Выявлены проблемные места

1) В AREA 101 пропали маршруты и AREA 0

2) Не работает маршрутизация на L3 коммутаторах 
Проверка работоспособности межвлановой маршрутизации между vlan 100 и 99.
Проверка доступности SW4 (10.127.255.98) c сетей пользователей

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_17.png)

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_18.png)
Настройки vPS  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_19.png)

Настройки SW3

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_20.png)

Tracert до 10.127.255.1


![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_21.png)

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_22.png)

Очень странный

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_23.png)

C VPC335 нету доступа до узла 10.127.255.65

С маршрутизатора доступ до есть 

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_24.png) 

Доступа нет и дальше шлюза трафик не идет
![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_25.png)




