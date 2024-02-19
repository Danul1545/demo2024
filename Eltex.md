## Создание нового пользователя.

- Configure – используется для входа в режим полной конфигурации.
- Hostname – используется для изменения имени хоста.
- Commit – используется для применения изменений конфигурации устройства.
- Confirm – используется для сохранения изменений конфигурации устройства.

```
Router(config)# username user
Router(config-user)# password user 
Router(config-user)# privilege 1
Router(config-user)# exit
Router(config)# exit
```

### Сброс устройства к заводским настройкам.

```
Router# copy system:default-config system:candidate-config
```

### Узнать интерфейсы роутера.

- Show ip interface – просмотр информации об интерфейсах.
- Interface <интерфейс> – переход к настройкам интерфейса.
- Ip address <адрес/маска> – настройка IP-адреса интерфейса.
- Ip firewall disable – отключает firewall на интерфейсе.

```
show ip interfaces
show interfaces status
```

## Настройка IP-адресации.

```
Router(config)# interface gi1/0/1
Router(config-if-gi)# ip firewall disable
Router(config-if-gi)# ip address 192.168.0.210/24
```

## Создание и удаление VLAN на коммутаторе.

```
Switch(config)# vlan 2
Switch(config-vlan)# name test
```

### VLAN создается на коммутаторе только посредством команды vlan.

```
Switch(config)# interface gi1/0/1
Switch(config-if-gi)# mode switchport (для виртуального Eltex)
Switch(config-if-gi)# switchport mode access
Switch(config-if-gi)# switchport access vlan 2
```

### Для создания Trunk портов, нужно выбрать режим trunk.

- Mode switchport – определяет порт как порт коммутатора.
- Switchport mode access – меняет режим порта коммутатора на «access».
- Switchport mode trunk – меняет режим порта коммутатора на «trunk».
- Switchport access vlan <номер_vlan> – присваивает порту access номер vlan.
- Switchport trunk native-vlan <номер_vlan> – определяет vlan управления для trunk порта.
- Show vlan – показывает созданные vlan и их принадлежность интерфейсам.

``` 
Switch(config-if-gi)# switchport mode trunk
Switch(config-if-gi)# switchport trunk native-vlan 2
Switch# show vlan
```

### Настройка статического маршрута.

Для того, чтобы настроить такой маршрут, нужно сначала заглянуть в таблицу маршрутизации - `R1# show ip route`

![image](https://github.com/Danul1545/demo2024/assets/148867600/170aa992-7b97-4410-95cb-cfa6bb922960)

В данной таблице указаны все статические и динамические маршруты для соединения со всей сетью. «C – connected» означает непосредственно подключенные к этому маршрутизатору сети, а именно 192.168.1.0/24. Такая сеть есть, поэтому первый эхо-запрос прошел успешно. А вот 192.168.2.0/24 тут нет. Значит, придется указывать такой маршрут вручную, с помощью команды «ip route <IP-адрес сети или хоста> <IP-адрес соседнего маршрутизатора или же выходной интерфейс>»:
```
R1(config)# ip route 192.168.2.0/24 192.168.1.1
```

### Настройка динамической маршрутизации.

- Router ospf <идентификатор> – создает лист OSPF с идентификатором.
- Area <идентификатор_области> – создает область с идентификатором.
- Enable – включает область и OSPF.
- Ip ospf instance <идентификатор> – указывает интерфейсу, какой использовать идентификатор OSPF.
- Ip ospf area <идентификатор_области> – указывает интерфейсу, какой использовать идентификатор области OSPF.
- Ip ospf – настраивает интерфейс на работу с OSPF.

Для настройки протокола OSPF с одной областью, необходимо для начала создать лист OSPF, область и включить их на всех маршрутизаторах:

```
R1(config)# router ospf 10
R1(config-ospf)# area 1.1.1.1
R1(config-ospf-area)# enable
R1(config-ospf-area)# exit
R1(config-ospf)# enable
```

После этого нужно назначить данную область интерфейсам, которые подключены к другим маршрутизаторам (в данном случае все):

```
R1# configure
R1(config)# interface gi1/0/1
R1(config-if-gi)# ip ospf instance 10
R1(config-if-gi)# ip ospf area 1.1.1.1
R1(config-if-gi)# ip ospf
R1(config-if-gi)# exit
```

## Настройка DHCP-сервера.

```
R1(config)# ip dhcp-server pool LAN (создает пул для DHCP-сервера с именем LAN)
R1(config-dhcp-server)# network 192.168.0.0/24 (добавляет сеть, из которой нужно раздавать IP-адреса)
R1(config-dhcp-server)# address-range 192.168.0.10-192.168.0.100 (указывает диапазон адресов от 10 адреса в сети до 100)
R1(config-dhcp-server)# default-router 192.168.0.1 (указывает шлюз по-умолчанию для клиентских ПК)
R1(config-dhcp-server)# dns-server 1.1.1.1 (указывает DNS-сервер для клиентских ПК)
R1(config-dhcp-server)# excluded-address-range 192.168.0.15-192.168.0.20 (указывает IP-адреса, которые выдавать нельзя)
R1(config-dhcp-server)# max-lease-time 0:0:5 (указывает время, в течение которого выдается данный IP-адрес в формате D:H:M, в конкретном случае на 5 минут)
R1(config-dhcp-server)# exit
R1(config)# ip dhcp-server (включает службу DHCP)
```

## Настройка Source NAT

Настроим доступ пользователей локальной сети 192.168.0.0/24 к публичной сети с использованием функции Source NAT, через IP-адрес внешней сети 10.0.0.2. Настройку начнем с создания зон TRUST и UNTRUST, который обозначают LAN и WAN сегмент соответственно:

```
R1(config)# security zone UNTRUST
R1(config-zone)# exit
R1(config)# security zone TRUST
R1(config-zone)# exit
```

На интерфейсах укажем зоны:

```
R1(config)# int gi1/0/2
R1(config-if-gi)# security-zone TRUST
R1(config-if-gi)# exit
R1(config)# int gi1/0/1
R1(config-if-gi)# security-zone UNTRUST
```

Для конфигурирования SNAT и настройки правил зон безопасности потребуется создать профиль адресов локальной сети «LOCAL_NET», включающий адреса, которым разрешен выход в публичную сеть, и профиль адреса публичной сети «WAN_NET»:

```
R1(config)# object-group network LOCAL_NET
R1(config-object-group-network)# ip address-range 192.168.0.1-192.168.0.254
R1(config-object-group-network)# exit
R1(config)# object-group network WAN_NET
R1(config-object-group-network)# ip address-range 10.0.0.2
```

Для пропуска трафика из зоны «TRUST» в зону «UNTRUST» создадим пару зон и добавим правила, разрешающие проходить трафику в этом направлении. Дополнительно включена проверка адреса источника данных на принадлежность к диапазону адресов «LOCAL_NET» для соблюдения ограничения на выход в публичную сеть. Действие правил разрешается командой enable:

```
R1(config)# security zone-pair TRUST UNTRUST
R1(config-zone-pair)# rule 1
R1(config-zone-pair-rule)# match source-address LOCAL_NET
R1(config-zone-pair-rule)# action permit
R1(config-zone-pair-rule)# enable
R1(config-zone-pair-rule)# exit
R1(config-zone-pair)# exit
```

Конфигурируем сервис SNAT. Первым шагом создаётся IP-адрес публичной сети (WAN), используемых для сервиса SNAT:

```
R1(config)# nat sourсe
R1(config-snat)# pool WAN_ADDRESS
R1(config-snat-pool)# ip address-range 10.0.0.2
R1(config-snat-pool)# exit
```

Создаём набор правил SNAT. В атрибутах набора укажем, что правила применяются только для пакетов, направляющихся в публичную сеть – в зону «UNTRUST». Правила включают проверку адреса источника данных на принадлежность к пулу «LOCAL_NET»:

```
R1(config-snat)# ruleset SNAT
R1(config-snat-ruleset)# to zone UNTRUST
R1(config-snat-ruleset)# rule 1
R1(config-snat-rule)# match source-address LOCAL_NET
R1(config-snat-rule)# action source-nat pool WAN_ADDRESS
R1(config-snat-rule)# enable
```

Для того, чтобы маршрутизатор R1 синхронизировал время с провайдером ISP, нужно настроить NTP:

```
R1(config)# ntp enable
R1(config)# ntp server 10.0.0.1
```

## Настройка MultiWAN на маршрутизаторе организации.

Настройка MultiWAN на R1. Настроим маршрутизацию:

```
R1(config)# ip route 8.8.8.8/32 wan load-balance rule 1
```

Создадим список для проверки целостности соединения:

```
R1(config)# wan load-balance target-list google
R1(config-target-list)# target 1
```

Зададим адрес для проверки, включим проверку указанного адреса и выйдем:

```
R1(config-wan-target)# ip address 8.8.8.8
R1(config-wan-target)# enable
```

Настроим интерфейсы. В режиме конфигурирования интерфейса gi1/0/1 указываем nexthop:

```
R1(config)# interface gi1/0/1
R1(config-if)# wan load-balance nexthop 1.1.1.1
```

В режиме конфигурирования интерфейса gi1/0/1 указываем список целей для проверки соединения:

```
R1(config-if)# wan load-balance target-list google
```

В режиме конфигурирования интерфейса gi1/0/1 включаем WAN-режим и выходим:

```
R1(config-if)# wan load-balance enable
```

В режиме конфигурирования интерфейса gi1/0/2 указываем nexthop:

```
R1(config)# interface gi1/0/2
R1(config-if)# wan load-balance nexthop 2.2.2.1
R1(config-if)# wan load-balance target-list google
```

В режиме конфигурирования интерфейса gi1/0/2 включаем WAN-режим и выходим:

```
R1(config-if)# wan load-balance enable
``` 

Создадим правило WAN:

```
R1(config)# wan load-balance rule 1
```

Укажем участвующие интерфейсы:

```
R1(config-wan-rule)# outbound interface gi1/0/1
R1(config-wan-rule)# outbound interface gi1/0/2
```

Включим созданное правило балансировки и выйдем из режима конфигурирования правила:

```
R1(config-wan-rule)# enable
```

Функция MultiWAN также может работать в режиме резервирования, в котором трафик будет направляться в активный интерфейс c наибольшим весом. Включить данный режим можно следующими командами:

```
R1(config)# wan load-balance rule 1
R1(config-wan-rule)# failover
```




