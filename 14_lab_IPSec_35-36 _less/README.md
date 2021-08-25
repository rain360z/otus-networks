# IPSec over DmVPN

Цель:
1) Настроить GRE поверх IPSec между офисами Москва и С.-Петербург  
2) Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги

3) Далее планируется настроить GRE поверх iPSec используя сертификаты.

## 1. Настроить GRE поверх IPSec между офисами Москва и С.-Петербург 

Настроим GRE поверх IPSec используя pre-share key

R15
```
!Настройка 1 фазы

crypto isakmp policy 10
    authentication pre-share


crypto isakmp key IPSec address 42.42.42.50

! Настройка 2 фазы
! Устанавливаем алгоритм шифрования 

crypto ipsec transform-set ENC_ALG esp-aes 128 

crypto ipsec profile OTUS
    set transform-set ENC_ALG


int t50
tunnel protection ipsec profile OTUS

show crypto isakmp sa
```

R18

```
crypto isakmp policy 10
    authentication pre-share
crypto isakmp key IPSec address 5.5.5.50

crypto ipsec transform-set ENC_ALG esp-aes 128 

crypto ipsec profile OTUS
    set transform-set ENC_ALG

! phase 2
int t50
tunnetl protection ipsec profile OTUS
```

![alert-text](Pictures/Screenshot_1.png) 

## 2. Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги

Настроим DMVPN поверх IPSec используя pre-share key

R15
```
crypto isakmp policy 20
    authentication pre-share
crypto isakmp key 0 isakmpkey address 0.0.0.0 0.0.0.0

crypto ipsec transform-set ENC_ALG esp-aes 128 

crypto ipsec profile OTUS
    set transform-set ENC_ALG

int t100
tunnel protection ipsec profile OTUS

show crypto isakmp sa
```

R28

```
crypto isakmp policy 10
    authentication pre-share
crypto isakmp key IPSec address 5.5.5.50

crypto ipsec transform-set ENC_ALG esp-aes 128 

crypto ipsec profile OTUS
    set transform-set ENC_ALG

int t100
tunnetl protection ipsec profile OTUS
```

R27

```
crypto isakmp policy 10
    authentication pre-share
crypto isakmp key IPSec address 5.5.5.50

crypto ipsec transform-set ENC_ALG esp-aes 128 

crypto ipsec profile OTUS
    set transform-set ENC_ALG

int t100
tunnetl protection ipsec profile OTUS
```

Соединение установилось. 

![alert-text](Pictures/Screenshot_2.png)

![alert-text](Pictures/Screenshot_3.png) 

Доступ есть из Лабытанги в Чокурдах

![alert-text](Pictures/Screenshot_4.png) 