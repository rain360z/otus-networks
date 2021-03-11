# 2_Лабораторная_работа_Конфигурирование_Роутера на палке. Межлановая маршрутизация  


### Задания
1. Постройка стенада сети и базовая настройка уструйств
2. Создание VLANs и назначить(Assign) порте коммутатора.
3. Конфигурировать 802.1Q транк между коммутаторами
4. Конфигурирование межвлановой маршрутизации на маршрутизаторе
5. Проверка межвлановой маршрутизации

Решение

Таблица адресов  
| Оборудование | Интерфейс  | ip-адрес | Маска |  Маршрут по умолчанию |
|--------------|------------|----------|-------|-----------------------|
|R1|G0/0/1.3| 192.168.3.1| 255.255.255.0| N/A|
||G0/01.4|192.168.4.1|255.255.255.0|N/A|
||G0/0/1.8|N/A|N/A|N/A|
|S1|  VLAN 3|192.168.3.11|255.255.255.0|192.168.3.1|
|S2| VLAN

Таблица VLAN

| VLAN| Name | Назначение интерфейсов|
|----|-------|-----------------------|
| 3 |	Management |	S1: VLAN 3 S2: VLAN 3 S1: F0/6|
| 4 |	Operations |	S2: F0/18 |
| 7 |	ParkingLot |	S1: F0/2-4, F0/7-24, G0/1-2  S2 F0/2-17, F0/19-24, G0/1-2 |
| 8 |	Native |	N/A |




#### Решение
#### Часть 1. Постройка стенда сети и базовая настройка устройств.

### Соберем стенд как указанно в задании. 
![alert text](https://github.com/rain360z/otus-networks/blob/main/2_lab/pictures/1.JPG)

#### Конфигурируем базовые настройки на оборудовании:

На примере рассмотрим конфигурирование роутера рассмотрим что включают в себя базовые настройки:

+ Назначение имени роутеру
+ Выключение DNS lookup
+ Назначение пароля для привилегированного режима
+ Назначение пароля при подключении через консоль        
+ Назначим в качестве пороля для удаленного подключения к vty(virtual teletype) 
+ Включение службы шифрования паролей
+ Создание банера
+ Настройка время на роуторе
+ Сохраним текущую конфигурацию в startup конфигурацию.

   ``` 
     Router>enable
     Router#conf terminal
     Router(config)#hostname R1
     R1(config)#no ip domain-lookup
     R1(config)#enable password class
     
     R1(config)#line console 0
     R1(config-line)#password class
     R1(config-line)#login
     
     R1(config)#line vty 0 4
     R1(config-line)#password class
     R1(config-line)#login
     
     R1(config)#service password-encryption
     
     R1(config)#banner login "anyone unauthorized access is prohibited."
     
     R1#clock set 18:14:15 08 march 2021
     
     R1#copy running-config startup-config

Все файлы изменений приведены [здесь](configs/). 
 
Конфигурируем аналогично базовые настройки на коммутаторах.
#### Конфигурируем PC
На основе таблицы зададим IP адрес, маску и шлюз по умолчанию

#### Часть 2. Создание VLANs на обоих коммутаторах

На примере одного коммутатора рассмотрим:
+ Создание  VLANs в соответствии с таблицей
+ Настройку SVI портов
+ Назначение неиспользуемым портам native vlan, в данном случает отдельного VLANz ParkingLot
+ Настройка SSHv2

``` S1(config)# vlan 3
    S1(config-vlan)# name Managment
    
    S1(config)# interface vlan 3
    S2(config-if)#ip address 192.168.3.12 255.255.255.0
    S1(config-if)# no shutdown
    
    S1(config)#ip default-gateway 192.168.3.1
    
    S1(config)#interface range f0/2-4, f0/7-24, g0/1-2
    S1(config-if-range)#switchport mode access 
    S1(config-if-range)#switchport access vlan 7```

    S1(config)#ip domain name S1
    S1(config)#crypto key generate rsa general-keys modulus 2048
    S1(config)#username adm password adm
    S1(config)#line vty 0 4
    S1(config-line)#transport input ssh
    S1(config-line)#login local
    S1(config)#ip ssh version 2
 ```
Создадим другие VLANs и назначим их на конкретне интерфейсы
Все файлы изменений приведены [здесь](configs/).

#### Часть 3. Создадание 802.1Q Trunk между коммутаторами

Вручную настроить транк на коммутаторе S1

``` 
   S1(config)#interface f0/1
   
   S1(config-if)#switchport mode trunk
   S1(config-if)#switchport trunk native vlan 8
   S1(config-if)#switchport trunk allowed vlan 3,4,7,8
   
```
Зададим аналогичные настройки на коммутаторе S2 и S1 в сторону роутера.
Все файлы изменений приведены [здесь](configs/).

#### Часть 4. Насройка мевлановой маршрутизации на роутере.

Включим интерфейс роутера G0/0/1. Настроим сабынтерфейсы с использованием 802.1Q.

```
R1(config)#interface gigabitEthernet 0/0/1
R1(config-if)#no shutdown

R1(config-if)#interface gigabitEthernet 0/0/1.3
R1(config-subif)#Description "Default Gateway for Managment(3)"
R1(config-subif)#encapsulation dot1Q 3
R1(config-subif)#ip address 192.168.3.1 255.255.255.0
```
Все файлы изменений приведены [здесь](configs/).

Закончили конфигурирование оборудования. Проверка роботоспособности:
+ пинг от PC-A До Шлюза
  ``` C:\>ping 192.168.3.1

      Pinging 192.168.3.1 with 32 bytes of data:

      Reply from 192.168.3.1: bytes=32 time<1ms TTL=255
      Reply from 192.168.3.1: bytes=32 time=1ms TTL=255
      Reply from 192.168.3.1: bytes=32 time<1ms TTL=255

      Ping statistics for 192.168.3.1:
      Packets: Sent = 3, Received = 3, Lost = 0 (0% loss),
      Approximate round trip times in milli-seconds:
      Minimum = 0ms, Maximum = 1ms, Average = 0ms
  ```
+ Пинг от PC-A до PC-B
 ```C:\>ping 192.168.4.3

Pinging 192.168.4.3 with 32 bytes of data:

Request timed out.
Reply from 192.168.4.3: bytes=32 time<1ms TTL=127
Reply from 192.168.4.3: bytes=32 time=10ms TTL=127
Reply from 192.168.4.3: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.4.3:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 10ms, Average = 3ms
```

+ Пинг от PC-A до S2
``` C:\>ping 192.168.3.12

Pinging 192.168.3.12 with 32 bytes of data:

Reply from 192.168.3.12: bytes=32 time=12ms TTL=255
Reply from 192.168.3.12: bytes=32 time<1ms TTL=255
Reply from 192.168.3.12: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.3.12:
    Packets: Sent = 3, Received = 3, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 12ms, Average = 4ms
```
+ Пустm tracert из PC-B  до PC-A
``` C:\>tracert 192.168.3.3

Tracing route to 192.168.3.3 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.4.1
  2   0 ms      2 ms      10 ms     192.168.3.3
```
Промежуточным адресом является адрес шлюза

    
    
    
 










   
