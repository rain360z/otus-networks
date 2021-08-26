## Обобщенное распределение адресов 
Главный офис   
 - Пользователи 10.0.0.0/15
 - Сервера 10.128.0.0/15
 - Сетевое оборудование 172.16.0.0/16
 
Филиалы   
 - Пользователи 10.1.0.0/16
 - Сервера 10.129.0.0/16
-  Сетевое оборудование 172.17.0.0/16

Свободные адреса 
10.2.0.0-10.127.255.255  
10.130.0.0-10.255.255.255  
172.18.4.0-172.31.255.255


Более подробно vPc

![alert-text](Pictures/Screenshot_1.png)

nxos27
```
feature vpc - включаем vpc
feature lacp -  for  the peerlink port channel
interface mgmt 0  - уже в VRF
ip address 10.224.102.1/30
no shut
exit

ping 10.224.10.2.2 vrf management

vpc domain 10
role priotity 10 - primary

peer-keepalive destination 10.224.102.2 source 10.224.102.1 vrf management
exit 

show vpc brief
status faile peer link has not been configured
```
![alert-text](Pictures/Screenshot_11.png)
```
interface eth 1/53-54
description **vPC Peer-Link**
channel-group 15 mode active
no shut
switchport

exit

int port-channel 15
description **vPC Peer-Link**
no shutdown
switchport
switchport mode trunk
vpc peer-link

show vpc brief
```
![alert-text](Pictures/Screenshot_12.png)
```
** Включим member
int eth1/20
channel-group 20 mode active
no sh
int port-channel 20
switchport
vpc 20

show vpc brief
** Если увидили id 20 это хорошо

** Настроим orphan port
interface eth 1/11
no sh
switch

show vpc orphan-ports


```

nxos38
```
feature vpc - включаем vpc
feature lacp -  for  the peerlink port channel
interface mgmt 0  - уже в VRF
ip address 10.224.102.2/30
no shut


ping 10.224.10.2.1 vrf management

vpc domain 10
role priotity 20 - secondery

peer-keepalive destination 10.224.102.1 source 10.224.102.2 vrf management
exit
show vpc brief

status faile peer link has not been configured

interface eth 1/53-54
description **vPC Peer-Link**
channel-group 15 mode active
no shut
switchport

exit

int port-channel 15
description **vPC Peer-Link**
no shutdown
switchport
switchport mode trunk
vpc peer-link


int eth1/20
channel-group 20 mode active
no sh
int port-channel 20
switchport
vpc 20
```

vPC и маршрутизация может возникнуть нестабильный peer.

VPC Подключен к 2 маршрутизаторам

Теория LAG это группаиз нескольких визических каналов, соединяющих два устройства. Каждый поток трафика назначается одному из физических каналов. 

VPC это расширеннаягрупа LAG, в которой используются два коммутатора Nexus вместо одного. Кажыдй поток трафика по-прежнему назначается одному каналу в vPC. Иногда трафик предназначенный для SWitch-1 попадает на SW2. SW Может иметь включенную функцию - одноранговый шлюз. Это позволяет ему отвечать от имени SW1. 

Рассмотрим как протоколы маршрутизации взаимодействуют с vPC.

Обычные протоколы маршрутизации, предполагают что ониподключены напрямую. Чтобы это обеспечить TTL выставляется 1. На протяжении процесса формирования соседей отправляются серии многоадресных и одноадресных пакетов. Если SW-2 получает некоторые из пакетов предназначенных для SW1, он, естественно уменьшает TTL. TTL теперь равен нулю, и пакет отбрасывается.  

Есть разные топологии overvPC VLAns
table: Routing Protocol Adjacencies support over physicalinterfaces

![alert-text](Pictures/Screenshot_2.png)

Первая топология,  которую мырассмотрим, - это когда маршшрутизаторы хотят взаимодействовать друг с другом.

![alert-text](Pictures/Screenshot_3.png)

Этот сценарий работат, поскольку маршрутизаторы подключены на уровне 2. Коммутаторы не уменьшают TTL пирингового трафика. Коммутаторы не взаимодействуют ни с чем. В этом  случае они используются дл транспорта. 




Но что если вы хотите, чтобыкоммутаторы участвовали вдинамической маршрутизации? В этом случае маршрутизаторне использует vPC. Вместо этого он подключен к сиротскому порту. Если вы хотите установить соединение с обоими коммутаторами, вам  может потребоватся дополнительный межкоммутаторный канал уровня 2.   
![alert-text](Pictures/Screenshot_4.png) 

Если вам действительно нужна эта ссылка, VLAN, используемую для пиринга, необходимо будет отсечь от однорангового канала, Чтобы принудительно передать трафик однорангового соединения по новому каналу.

![alert-text](Pictures/Screenshot_6.png)

с одним или обоими коммутаторами через потерянный порт. Он так же может взаимодействовать с маршрутизатором, Подключеннымк vPC.

...


Представьте себе случай, когда у вас естькластер ASA. Рекомендуемый способ их подключения - с помощью VSS /vPC


![alert-text](Pictures/Screenshot_7.png)

+ Во первых, подумайте действительно ли нужна динамическая маршрутизацияна границе сети, вам может потребоваться только исходящий маршрутпо умолчанию и некоторые входящие статические маршруты.  
+ Если вам действительно нужна динамическая маршрутизация, вы можете рассмотреть возможность использовать BGP. Во многихслучаях это может быть не идеальным решением.
+ в третьих. В конфигурации vPC мы добавляем команду ``layer3 peer-router`` с обеих сторон. Это позволяет маршрутизаторам, подключенным к vPC, взаимодействовать с коммутаторами. Есть ограничения вверсиях, и линейных картах. 


ASA

CCL отдельный физический линк через коммутатор между node. Отдельная ip подсеть.





Настроим EIGRP(суммаризацию редистрибьюцию static)
```

redistribute static route-map Default
ip prefix-list Default seq 5 permit 0.0.0.0/0 
route-map Default permit 10
  match ip address prefix-list Default 
```

__________________________


Cisco ASA Cluster Configuration

![alert-text](Pictures/Screenshot_8.png)

simple cluster

ASA 1 master
interface mod to span


```
hostname ASA-1

cluster interface-mode spanned force

int port-channel 1
port-channel span-cluster
int gig0/0
no shu
channel-group 1 mode active
int gig0/1
no shu
channel-group 1 mode active


Show run int port1

int gig0/3
cluster group ASA-CLUSTER
local-unit ASA-1
claster-interface gig0/3 ip 2.2.2.1 255.255.255.0

priority 1
console-replicate

sh run claster
enable




```


ASA 2
```
hostname ASA-2

int gig0/3
cluster group ASA-CLUSTER
local-unit ASA-1
claster interface-mode spanned force

cluster group ASA-CLUSTER 
    local-unit ASA-2
    cluster-interface gig0/3 ip 2.2.2.2 255.255.255.0
    priority 2
    cosole-replicate
    sh run cluster

    enable

```


Зайдем на  ASA-1
```
show cluster info

```

Дополнительные настройки

ASA-1

```
int port1.7
vlan 7
no sh
int port1.8
no sh
exit

admin-context admin

context admin
allocate-interface Management0/0
allocate-interface port1.7 inside
allocate-interface port1.7 outside
config-url disk0:/admin-test2.cfg
exit


sh run ccontext 

changeto context admin
conf t
ip local pool MGM-POOL 192.168.1.2-192.168.1.3
interface management0/0
nameif mgmnt
ip address 192.168.1.1 255.255.255.0 cluster-pool MGMT
no sh
exit

```

ASA 2

```
changeto context admin
sh run int Management0/0
sh ip add
interface inside
ip add 1.1.1.1 255.255.255.0
security-level 100
nameif inside

no sh

interface outside 
nameif outside 
securit-level 0
ip address 1.1.2.1 255.255.255.0
no sh

sh int ip brie

```


![alert-text](Pictures/Screenshot_9.png)
посморимint ip brie на asa02
![alert-text](Pictures/Screenshot_10.png)

Проверим доступность до SVI VLAN 7 на VSS
Не работает.

Чтобы настромть SVI/eigrp надо его включить

# Настроим  cisco asa
Настроили failover

Прозрачный режим это уровень 2 используя BVI

Нужно настроить интерфейс управления. 

Нужно настроить самой фактической группе моста, а не физическим интерфейсам 

Cisco asa по  дефолту находится в режиме конфигурации, в нашем режиме нужно переключить на transperent mode.  
```
show firewall

Поменяем режим

firewall transparent
```
![alert-text](Pictures/Screenshot_13.png)  

``redundent`` режим не работает в режиме transparant

```
int g0/4
nameif outside
no sh
int g0/5
nameif outside
no sh

int g0/1
nameif inside
no sh
int g0/2
nameif inside
no sh

int BVI 1
ip add 172.16.4.67 255.255.255.240


int g0/4
bridge-group 2

int g0/5
bridge-group 2

int g0/1
bridge-group 2

int g0/2
bridge-group 2


```

Добавим винспектирование icmp



Создадим object для eigrp и 

```
policy-map global_policy
 class inspection_default
 inspect icmp
```



настроим Cisco asa в режиме routed

ASA# conf t
ASA(config)# int gi0/0
ASA(config-if)# nameif outside
(Этому имени интерфейса автоматически задается security level 0)
ASA(config-if)# ip address 210.210.1.2 255.255.255.0
ASA(config-if)# no sh

Создали зоны

Нужно настроить фильтры BGP AS1 и AS7



Настроим NAT (Object / autoNAT)

nat для разрешения выхода в хостов в интернет

Для настройки такой NAT необходимо создать сетевой объект, представляющий подсеть inside, а также объект, представляющий подсеть DMZ.  В каждом из этих объектов настройте правило динамического преобразования сетевых адресов, которое будет выполнять трансляцию адресов портов (PAT) этих клиентов, поскольку они проходят от соответствующих интерфейсов к интерфейсу outside.

Эта конфигурация выглядит примерно так:

object network inside-subnet
 subnet 192.168.0.0 255.255.255.0
 nat (inside,outside) dynamic interface
!
object network dmz-subnet
 subnet 192.168.1.0 255.255.255.0
 nat (dmz,outside) dynamic interface

Шаг 2. Настройка NAT для доступа к веб-серверу из Интернета

Эта конфигурация выглядит примерно так:

object network webserver-external-ip  

 host 198.51.100.101
!
object network webserver
 host 192.168.1.100  

 nat (dmz,outside) static webserver-external-ip service tcp www www


ACL позволяют переопределить поведение системы безопасности по умолчанию.

Поскольку трафик от outside к DMZ запрещен ASA в текущей конфигурации, пользователи в Интернете не смогут получить доступ к веб-серверу, несмотря на конфигурацию NAT на шаге 

access-list outside_acl extended permit tcp any object webserver eq www
!  
access-group outside_acl in interface outside

_________________________
Вот такая настройка не сработала, проработать с nx

```
FW-DELTACONFIG(config)#
host object network OBJ_NAT_SERVER
host 192.168.10.200
nat (inside,outside) static interface
```
___________________________________
Тест 3

Включили инспектирование и маршурт написали на R4 пинг есть с nx25

ASA
```
object network SERVER-INTARNET-PUBLIC  
host 46.46.46.2  
exit  
object network SERVER-INTRANET-OUTSIDE  
subnet 10.128.0.0 255.255.0  
nat (inside2,outside) static SERVER-INTRANET-PUBLIC  
```
Анононсы

1. Организовать отказоустойчивую сеть пользователей
2. На границе локальной сети внедрить кластер из межсетевых экранов cisco asa.
3. Обеспечить пользователям доступ в интернет. 
4. Создать DMZ зону.
5. Для филиалов организовать DMVPN с IPSec


## 1. Организовать отказоустойчивую сеть пользователей










____________________________
Настройка EiGRP

На Active node
```
router eigrp 1
    router eigrp 1
    eigrp router-id 0.0.0.40
    passive-interface default
    no passive-interface inside 
    no passive-interface inside2
    network 172.16.8.0 255.255.255.240
    network 172.16.8.32 255.255.255.240
! Суммаризация
interface GigabitEthernet0/1
    summary-address eigrp 1 0.0.0.0 0.0.0.0
interface GigabitEthernet0/2
    summary-address eigrp 1 0.0.0.0 0.0.0.0
```
ASA

```
! НЕ работает
object network USER-INTARNET-PUBLIC  
    host 46.46.46.2  
    exit  
object network USER-INTRANET-OUTSIDE  
    host 10.128.0.1  
    nat (inside2,outside2) static USER-INTRANET-PUBLIC 
    nat (inside,outside2) static USER-INTRANET-PUBLIC 
!  PAT не работает
object network OBJ_NAT_LAN
subnet 172.16.0.0 255.255.0.0
nat (inside,outside2) dynamic interface
```

В ASA можно включить встроенный packet tracer и проверить трафик.

```
ASAv39(config)# Packet-tracer input inside icmp 172.16.8.2 0 8 46.46.46.1

Phase: 1
Type: ACCESS-LIST
Subtype: 
Result: ALLOW
Config:
Implicit Rule
Additional Information:
MAC Access list

Phase: 2
Type: ROUTE-LOOKUP
Subtype: Resolve Egress Interface
Result: ALLOW
Config:
Additional Information:
found next-hop 46.46.46.1 using egress ifc  outside2

Phase: 3
Type: NAT
Subtype: 
Result: ALLOW
Config:
object network OBJ_NAT_LAN
 nat (inside,outside2) dynamic interface
Additional Information:

Phase: 4
Type: NAT
Subtype: per-session
Result: ALLOW
Config:
Additional Information:

Phase: 5
Type: IP-OPTIONS
Subtype: 
Result: ALLOW
Config:
Additional Information:

Result:
input-interface: inside
input-status: up
input-line-status: up
output-interface: outside2
output-status: up
output-line-status: up
Action: drop
Drop-reason: (nat-xlate-failed) NAT failed

ASAv39(config)# Packet-tracer input inside icmp 172.16.8.2 0 8 46.46.46.2

Result:
input-interface: inside
input-status: up
input-line-status: up
Action: drop
Drop-reason: (no-route) No route to host

```

PAT

```
object network Inside-Subnet
    subnet 172.16.8.0 255.255.255.240
object network Internet-PAT
    host 46.46.46.1
    exit
! не сработало
!nat (inside,outside2) source dynamic Inside-Subnet Internet-PAT
! не сработало
nat source dynamic Inside-Subnet Internet-PAT 

```

Забросим пока нат

VPN 

Нужно помнить
• EIGRP по умолчанию использует правило split horizon  
• EIGRP меняет next hop IP address  

```
• Настройка Hub
HQ(config)#interface tunnel100
HQ(config-if)#ip nhrp redirect
• Настройка Spoke
HQ(config)#interface tunnel100
HQ(config-if)#ip nhrp shortcut
```

```
Конфигурация тунеля
interface tunnel100
ip address 10.64.0.1 255.255.255.0
ip mtu 1400
sysopt connection tcpmss 1360
ip nhrp network-id 100
ip nhrp map multicast dynamic
ip nhrp redirect
tunnel mode gre multipoint
tunnel source FastEthernet0/0
Опционально
interface tunnel100
ip nhrp authentication OTUS
ip nhrp holdtimer 600
```

```
Конфигурация тунеля
interface tunnel100
ip address 10.64.0.2 255.255.255.0
ip mtu 1400
ip tcp adjust-mss 1360
ip nhrp network-id 100
ip nhrp map 10.64.0.1 100.0.0.1
ip nhrp map multicast 100.0.0.1
ip nhrp nhs 10.64.0.1
ip nhrp shortcut
tunnel mode gre multipoint
tunnel source FastEthernet0/0
Опционально
interface tunnel100
ip nhrp authentication OTUS
ip nhrp holdtimer 600
ip nhrp registration no-unique
```
eigrp 

```
Конфигурация HQ (Hub)
!
router eigrp 100
no auto-summary
network 10.64.0.0 0.0.0.255
network 10.0.0.0 0.0.0.255
!
interface tunnel00
ip summary-address eigrp 100 0.0.0.0 0.0.0.0 *
!
Конфигурация BR1 (Spoke)
!
router eigrp 100
no auto-summary
network 10.64.0.0 0.0.0.255
network 10.5.0.0 0.0.0.255
!
```
![alert-text](Pictures/Screenshot_14.png) 

Asa 


FFFFF

domain-name otus.ru
http server enable



29.07.2021

Проблемы скорее всего из-за суммаризации 
Или из-за того что находились в одной зоне, а этот трафик блокируется

Рабочик конфиг
```
object network OBJ_NAT_LAN
 subnet 172.16.0.0 255.255.0.0
object network OBJ_NAT_LAN2
 subnet 172.16.0.0 255.255.255.0
object network OBJ_NAT_LAN
 nat (inside,outside2) dynamic interface
object network OBJ_NAT_LAN2
 nat (inside2,outside2) dynamic interface
```

тестирую доступ по телнет до R4 46,46,46,1 Работает

```
object network OBJ_NAT_LAN
 subnet 172.16.0.0 255.255.0.0
 nat (inside,outside2) source dynamic PAT

object network OBJ_NAT_LAN2
 subnet 172.16.0.0 255.255.255.0

 object network PAT
    host 46.46.46.2
```
C cуммаризацией тоже работает




_______________________________________________________________


## 3. Обеспечить пользователям доступ в интернет. 

Настроим PAT для пользователей из сети 10.128.0.0/17 в 46.46.46.1

```
object network OBJ_NAT_LAN
 subnet 10.128.0.0 255.255.128.0
 nat (inside,outside2) source dynamic PAT

object network OBJ_NAT_LAN2
 subnet 10.128.0.0 255.255.128.0
 nat (inside,outside2) source dynamic PAT

 object network PAT
    host 46.46.46.2
```






## 4. Создать DMZ зону.



## 5. Для филиалов организовать IPSec

ip addess сети пользователе
Пользователи 10.1.0.0/16


R7
```
interface tunnel  100
    tunnel mode gre
    ip address 10.254.254.10 255.255.255.0
    tunnel source 33.33.33.17
    tunnel destination 46.46.46.2
    ip mtu 1400
    ip tcp adjust-mss 1360
```

ASA 
```
interface tunnel  100
    ip address 10.254.254.1 255.255.255.0
    tunnel source 46.46.46.2
mtu 14000
sysopt connection reclassify-vpn 
```

IPSec


```
crypto ikev1 enable outside 
crypto ikev1 policy 2
encryption aes
hash sha
group 2
authentication pre-share
life 86400

```