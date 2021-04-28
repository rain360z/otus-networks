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
[1. Настроить OSPF IPv4](https://github.com/rain360z/otus-networks/tree/main/11_lab#1-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B8%D1%82%D1%8C-ospf-ipv4)  
  [1.1 Поставить фильтр на AREA 101, маршрутизаторы получают только маршрут по умолчанию.](https://github.com/rain360z/otus-networks/tree/main/11_lab#11-%D0%BF%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%B8%D1%82%D1%8C-%D1%84%D0%B8%D0%BB%D1%8C%D1%82%D1%80-%D0%BD%D0%B0-area-101-%D0%BC%D0%B0%D1%80%D1%88%D1%80%D1%83%D1%82%D0%B8%D0%B7%D0%B0%D1%82%D0%BE%D1%80%D1%8B-%D0%BF%D0%BE%D0%BB%D1%83%D1%87%D0%B0%D1%8E%D1%82-%D1%82%D0%BE%D0%BB%D1%8C%D0%BA%D0%BE-%D0%BC%D0%B0%D1%80%D1%88%D1%80%D1%83%D1%82-%D0%BF%D0%BE-%D1%83%D0%BC%D0%BE%D0%BB%D1%87%D0%B0%D0%BD%D0%B8%D1%8E)  
  [1.2 Поставить фильтр 102, маршрутизаторы получают все маршруты, кроме маршрутов в зону 101.](https://github.com/rain360z/otus-networks/tree/main/11_lab#12-%D0%BF%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%B8%D1%82%D1%8C-%D1%84%D0%B8%D0%BB%D1%8C%D1%82%D1%80-102-%D0%BC%D0%B0%D1%80%D1%88%D1%80%D1%83%D1%82%D0%B8%D0%B7%D0%B0%D1%82%D0%BE%D1%80%D1%8B-%D0%BF%D0%BE%D0%BB%D1%83%D1%87%D0%B0%D1%8E%D1%82-%D0%B2%D1%81%D0%B5-%D0%BC%D0%B0%D1%80%D1%88%D1%80%D1%83%D1%82%D1%8B-%D0%BA%D1%80%D0%BE%D0%BC%D0%B5-%D0%BC%D0%B0%D1%80%D1%88%D1%80%D1%83%D1%82%D0%BE%D0%B2-%D0%B2-%D0%B7%D0%BE%D0%BD%D1%83-101)  
[2. Настроить OSPF IPv6](https://github.com/rain360z/otus-networks/tree/main/11_lab#2-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B8%D1%82%D1%8C-ospf-ipv6)  
[3. Выявление проблемных мест](https://github.com/rain360z/otus-networks/tree/main/11_lab#3-%D0%B2%D1%8B%D1%8F%D0%B2%D0%BB%D0%B5%D0%BD%D1%8B-%D0%BF%D1%80%D0%BE%D0%B1%D0%BB%D0%B5%D0%BC%D0%BD%D1%8B%D0%B5-%D0%BC%D0%B5%D1%81%D1%82%D0%B0)  


###  Решение:

### 1. Настроить OSPF IPv4   

СХЕМА  
![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_1.png)

Настроим работу OSPF на всех маршрутизаторах в сети

Для более наглядной фильтрации настраим ospf везде

#### Настроим работу OSFP на R14
 
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
Все файлы изменений приведены [здесь](Configs/).
В соответствии с зонами настроим остальные маршрутизаторы.

Проверим в LSDB каждой AREA Router Link States.
AREA 0 
![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_2.png)

AREA 101  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_3.png)

AREA 102  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_4.png)  

AREA 10  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_5.png)

### 1.1 Поставить фильтр на AREA 101, маршрутизаторы получают только маршрут по умолчанию.

Под данные требованиям проходит Total Stub Area

Посмотирм текущие маршруты в AREA 101

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_6.png)

После настройки зоны должны остаться присоединенные сети и Default route

В случае зоны Total area stub. Необходимо на маршрутизаторах в зоне 101 прописать следующие настройки:

ABR

```   
(config-router)# area 101 stub no-summary   
```

На всех маршрутизаторах зоны

``` (config-router)# area 101 stub ```

Посмотрим маршруты на R19 

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_7.png)  

Все файлы изменений приведены [здесь](configs/). 

### 1.2 Поставить фильтр 102, маршрутизаторы получают все маршруты, кроме маршрутов в зону 101

К данной зоне применим LSA Filter, чтобы маршруты из сети 101 обявлялись в зоне 102

Проверим текущие маршруты в AREA 102

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_8.png) 

Необходимо добавить фильтр на ABR граничащие с зоной 102

``` 
    (config)# ip prefix-list non101 seq 5 dany 10.127.255.0/28   
    (config)# ip prefix-list non101 seq 5 dany 10.127.255.224/28
     
    (config-router)#area 0 filter-list prefix out   
```

т.о мы блокруем сети выходящие из area 0 с данными префиксами.  

Все файлы изменений приведены [здесь](configs/). 

Проверим текущие маршруты в AREA 102

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_9.png)

видим что в таблице маршрутизации из других зон нету сетей из AREA 101

### 2. Настроить OSPF IPv6


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

Настроим Total stub Area 101

Настройки аналогичные с ipv4

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_14.png)


Настроим фильтр на зоне 102

```
R15(config)# ipv6 prefix-list non101 seq 4 deny 1:1:1:1::/64
R15(config)# ipv6 prefix-list non101 seq 5 deny 1:1:1:15::/64
R15(config-rtr)# area 102 filter-list prefix non101 in
```
![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_16.png)


Исходя из таблица маршрутизации, у нас вместе с AREA 101 пропали маршруты и AREA 0

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_15.png)

Настройка ospf ipv6 закончена.  

Все файлы изменений приведены [здесь](configs/). 

### 3. Выявлены проблемные места

1) В AREA 101 пропали маршруты и AREA 0  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_15.png)   

R15 

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_23.png)  
   
R13  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_24.png) 


2) Не работает маршрутизация на L3 коммутаторах 


Проверка доступности SW4 (10.127.255.177) c узла 10.0.0.10  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_17.png)   
 
Настройки VPCs   

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_19.png)

Настройки SW3  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_20.png)

С SW3 доступ до маршрутизатора доступ есть

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_18.png)

проверка достпуа с VPCs:      
+ SW3: SVI-10.0.0.1
+ SW3: e0/1-10.127.255.194
+ SW4: e0/0-10.127.255.161  

![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_21.png)  
  
trace SW4: e0/1 10.127.255.177  


![](https://github.com/rain360z/otus-networks/blob/main/11_lab/Pictures/Screenshot_22.png)





