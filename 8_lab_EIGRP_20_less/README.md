

# EIGRP

Задание:
Настроить EIGRP в С.-Петербург; 
Использовать named EIGRP

В офисе С.-Петербург настроить EIGRP^
+ R32 получает только маршрут по-умолчанию
+ R16-17 анонсируют только суммарные префиксы
+ Использовать EIGRP named-mode для настройки сети
+ Настройка осуществляется одновременно для IPv4 и IPv6

Настроим named EIGRP ipv4 на R17.
Будут настроены следующие настройки:
+ включим настройки процесс eigrp;
+ все интерфейсы переведем в пассивный режим, для того чтобы небыли отправлены hello пакеты, на интерфейсах где будет включен eigrp;
+ назначим router-id
+ включим на интерфейсах eigrp
+ выключим на интерфейсах

```
R17(config)# router eigrp E
R17(config-router)# shutdown
R17(config-router)# address-family ipv4 unicast autonomus-system 1
R17(config-router-af)# af-interface default 
R17(config-router-af-interface)# passive interface
R17(config-router-af-interface)# exit
R17(config-router-af)# eigrp router-id 0.0.0.17
R17(config-router-af)# network 10.128.255.0 0.0.0.1
R17(config-router-af)# network 10.128.255.4 0.0.0.1
R17(config-router-af)# network 10.128.255.8 0.0.0.1
R17(config-router-af)# network 10.128.255.10 0.0.0.1
R17(config-router-af)# network 10.128.255.12 0.0.0.1
R17(config-router-af)# af-interface e0/0
R17(config-router-af-interface)# no passive-interface
R17(config-router-af)# af-interface e0/1
R17(config-router-af-interface)# no passive-interface
R17(config-router-af)# af-interface e0/2
R17(config-router-af-interface)# no passive-interface
R17(config-router-af)# af-interface e0/3
R17(config-router-af-interface)# no passive-interface
R17(config-router-af)# af-interface e1/1
R17(config-router-af-interface)# no passive-interface
R17(config-router-af-interface)# exit
R17(config-router-af)# exit
R17(config-router)# no shutdown











```