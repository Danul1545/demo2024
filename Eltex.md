
## Команды для новичков
- Configure – используется для входа в режим полной конфигурации.
- Hostname – используется для изменения имени хоста.
- Commit – используется для применения изменений конфигурации устройства.
- Confirm – используется для сохранения изменений конфигурации устройства.
## Создание нового пользователя.
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

Проверка соединения осуществляется с помощью команды «ping».
Проверка в случае, если ISP1 и ISP2 работают в штатном режиме:

![image](https://github.com/Danul1545/demo2024/assets/148867600/26f7df5b-539a-4996-b758-f16ab7154d70)

Если произойдет авария у провайдера, который активен на R1, то R1 выдаст сообщение, что на gi1/0/1 пропало соединение, а также переключится на gi1/0/2:

![image](https://github.com/Danul1545/demo2024/assets/148867600/a12312d4-a022-4ed2-81be-1a4a69642c1c)

Но соединение все равно успешно:

![image](https://github.com/Danul1545/demo2024/assets/148867600/54e3ad0a-33f1-44ea-a090-f0b8969b4c74)

## Port Security.

Существует несколько режимов работы port security.

1) По умолчанию настроен режим lock. В режиме lock все динамически изученные mac-адреса переходят в состояние static. В данном режиме сохраняет в файл текущие динамически изученные адреса, связанные с интерфейсом, и запрещает обучение новым адресам и старение уже изученных адресов
2) Режим max-address удаляет текущие динамически изученные адреса, связанные с интерфейсом. Разрешено изучение максимального количества адресов на порту. Повторное изучение и старение разрешены.
3) Режим secure-delete-on-reset удаляет текущие динамически изученные адреса, связанные с интерфейсом. Разрешено изучение максимального количества адресов на порту. Повторное изучение и старение запрещены. Адреса сохраняются до перезагрузки.
4) Режим secure-permanent удаляет текущие динамически изученные адреса, связанные с интерфейсом. Разрешено изучение максимального количества адресов на порту. Повторное изучение и старение запрещены. Адреса сохраняются при перезагрузке

#### Пример настройки port-security в режиме lock.

Данный режим также добавляет все MAC-адреса статически в конфигурацию show run:
Включить функцию защиты на интерфейсе:

```
SW1(config-if)# switchport port-security enable
```

Настройка MAC-адреса в port-security в ручном режиме. После ввода команды настройка отображается как в выводе show run, так и в выводе show mac-address-table, с пометкой Static в типе записи.

```
SW1(config)# mac-address-table static unicast aa:bb:cc:dd:00:11 vlan X interface gi1/0/x
```

#### Пример настройки port-security в режиме max-address.

Установить режим ограничения изучения максимального количества MAC-адресов:

```
SW1(config-if)# switchport port-security mode max-addresses
```

Задать максимальное количество адресов, которое может изучить порт, например, 10:

```
SW1(config-if)# switchport port-security mac-limit 10
```

Включить функцию защиты на интерфейсе:

```
SW1(config-if)# switchport port-security enable
```

#### Пример настройки port-security в режиме secure-delete-on-reset.

Установить режим ограничения изучения максимального количества MAC-адресов:

```
SW1(config-if)# switchport port-security mode secure-delete-on-reset
```

Задать максимальное количество адресов, которое может изучить порт, например, 2:
 
```
SW1(config-if)# switchport port-security mac-limit 2
```

Включить функцию защиты на интерфейсе:
 
```
SW1(config-if)# switchport port-security enable
```

После настройки порта на нем возможно изучение 2х новых мак-адресов.

#### Пример настройки port-security в режиме secure-permanent.
Установить режим ограничения изучения максимального количества MAC-адресов:

```
SW1(config-if)# switchport port-security mode secure-permanent
```

Задать максимальное количество адресов, которое может изучить порт, например, 2:

```
SW1(config-if)# switchport port-security mac-limit 2
```

Включить функцию защиты на интерфейсе:

```
SW1(config-if)# switchport port-security enable
```

После настройки порта на нем возможно изучение 2х новых мак-адресов. Адреса сохранятся после перезагрузки.
Настройка режима реагирования возможна в 3х режимах:

- protect – в данном режиме оповещения о нарушении безопасности нет. Включает перехват МАС-адресов, которые должны быть отброшены, на CPU, после чего MAC-адреса помечаются как заблокированные и отбрасываются в течение aging-time.  Режим по умолчанию.
- restrict – в данном режиме при нарушении безопасности отправляется SYSLOG-сообщение на SYSLOG-сервер.
- discard-shutdown – в данном режиме кадры с неизученными МАС-адресами источника отбрасываются, порт отключается. Поддерживается начиная с версии ПО 10.3.3.

Пример настройки:

```
SW1(config-if)# switchport port-security violation restrict
```

## Списки доступа ACL.
Настроим список доступа для фильтрации по подсетям:

```
R1(config)# ip aceess-list extended white
R1(config-acl)# rule 1
R1(config-acl-rule)# action permit
R1(config-acl-rule)# match protocol any
R1(config-acl-rule)# match source-address 192.168.20.0 255.255.255.0
R1(config-acl-rule)# match destination-address any
R1(config-acl-rule)# enable
```

Применим список доступа на интерфейс gi1/0/1 и сохраним конфигурацию:

```
R1(config)# interface gi1/0/1
R1(config)# service-acl input white
```

Просмотреть детальную информацию о списке доступа возможно через команду:

```
R1(config)# do show ip access-list white
```

## Удаленный мониторинг SNMP.
Настройка SNMPv3
Включаем SNMP-сервер:

```
R1(config)# snmp-server
```

оздаем пользователя SNMPv3:

```
R1(config)# snmp-server user admin
```

Определим режим безопасности:

```
R1(snmp-user)# authentication access priv
```

Определим алгоритм аутентификации для SNMPv3-запросов:

```
R1(snmp-user)# authentication algorithm md5
```

Установим пароль для аутентификации SNMPv3-запросов:

```
R1(snmp-user)# authentication key ascii-text P@ssw0rd
```

Определим алгоритм шифрования передаваемых данных:

```
R1(snmp-user)# privacy algorithm aes128
```

Установим пароль для шифрования передаваемых данных:

```
R1(snmp-user)# privacy key ascii-text P@ssw0rd
```

Активируем SNMPv3-пользователя:

```
R1(snmp-user)# enable
R1(snmp-user)# exit
```

Определяем сервер-приемник Trap-PDU сообщений:

```
R1(config)# snmp-server host 172.16.1.10
```

Добавляем данный маршрутизатор в список любой программы для мониторинга локальной сети, и получаем результат:

![image](https://github.com/Danul1545/demo2024/assets/148867600/2736ba13-3c30-4b9c-a3bd-ec2cdc23c7c7)

## Настройка GRE- и IPSec-туннелей.
Настройка GRE-туннеля
Создадим туннель GRE 1 на R1:

```
R1(config)# tunnel gre 1
```

Укажем локальный и удаленный шлюз (IP-адреса интерфейсов, граничащих с WAN):
 
```
R1(config-gre)# local address 1.1.1.2
R1(config-gre)# remote address 2.2.2.2
```

Укажем IP-адрес туннеля 172.16.1.1/30:

```
R1(config-gre)# ip add 172.16.1.1/30
```

Включим туннель и выйдем:

```
R1(config-gre)# enable
```

На маршрутизаторе должен быть создан маршрут до локальной сети партнера. В качестве интерфейса назначения указываем ранее созданный туннель GRE:

```
R1(config)# ip route 172.16.1.0/30 tunnel gre 1
```

Сделаем то же на маршрутизаторе R2, с некоторыми отличиями:

```
R2(config)# tunnel gre 1
R2(config-gre)# local address 2.2.2.2
R2(config-gre)# remote address 1.1.1.2
R2(config-gre)# ip add 172.16.1.2/30
R2(config-gre)# enable
R2(config-gre)# exit
R2(config)# ip route 172.16.1.0/30 tunnel gre 1
```

Состояние туннеля можно посмотреть командой:`R1# show tunnels status gre 1`
Конфигурацию туннеля можно посмотреть командой: - `R1# show tunnels configuration gre 1`

### Настройка IPsec-туннеля на R1
Настроим IPsec на туннеле GRE 1. Создадим профиль протокола IKE. В профиле укажем группу Диффи-Хэллмана 2, алгоритм шифрования AES 128 bit, алгоритм аутентификации MD5. Данные параметры безопасности используются для защиты IKE-соединения:

```
R1(config)# security ike proposal ike_prop
R1(config-ike-proposal)# authentication algorithm md5
R1(config-ike-proposal)# encryption algorithm aes128
R1(config-ike-proposal)# dh-group 2
```

Создадим политику протокола IKE. В политике указывается список профилей протокола IKE, по которым могут согласовываться узлы и ключ аутентификации:

```
R1(config)# security ike policy ike_pol1
R1(config-ike-policy)# pre-shared-key ascii-text P@ssw0rd
R1(config-ike-policy)# proposal ike_prop1
```

Создадим шлюз протокола IKE. В данном профиле указывается GRE-туннель, политика, версия протокола и режим перенаправления трафика в туннель:

```
R1(config)# security ike gateway ike_gw1
R1(config-ike-gw)# ike-policy ike_pol1
R1(config-ike-gw)# local address 1.1.1.2
R1(config-ike-gw)# local network 1.1.1.2/32 protocol gre
R1(config-ike-gw)# remote address 2.2.2.2
R1(config-ike-gw)# remote network 2.2.2.2/32 protocol gre
R1(config-ike-gw)# mode policy_based
```

Создадим профиль параметров безопасности для IPsec-туннеля. В профиле укажем группу Диффи-Хэллмана 2, алгоритм шифрования AES 128 bit, алгоритм аутентификации MD5. Данные параметры безопасности используются для защиты IPsec-туннеля:

```
R1(config)# security ipsec proposal ipsec_prop1
R1(config-ipsec-proposal)# authentication algorithm md5
R1(config-ipsec-proposal)# encryption algorithm aes128
R1(config-ipsec-proposal)# pfs dh-group 2
```

Создадим политику для IPsec-туннеля. В политике указывается список профилей IPsec-туннеля, по которым могут согласовываться узлы:

```
R1(config)# security ipsec policy ipsec_pol1
R1(config-ipsec-policy)# proposal ipsec_prop1
```

Создадим IPsec VPN. В VPN указывается шлюз IKE-протокола, политика IP sec-туннеля, режим обмена ключами и способ установления соединения. После ввода всех параметров включим туннель командой enable и сохраним конфигурацию:

```
R1(config)# security ipsec vpn ipsec1
R1(config-ipsec-vpn)# ike establish-tunnel route
R1(config-ipsec-vpn)# ike gateway ike_gw1
R1(config-ipsec-vpn)# ike ipsec-policy ipsec_pol1
R1(config-ipsec-vpn)# enable
```

Делаем то же для R2, но с небольшими отличиями:

```
R2(config)# security ike proposal ike_prop
R2(config-ike-proposal)# authentication algorithm md5
R2(config-ike-proposal)# encryption algorithm aes128
R2(config-ike-proposal)# dh-group 2
R2(config-ike-proposal)# exit
R2(config)# security ike policy ike_pol1
R2(config-ike-policy)# pre-shared-key ascii-text P@ssw0rd
R2(config-ike-policy)# proposal ike_prop1
R2(config-ike-policy)# exit
R2(config)# security ike gateway ike_gw1
R2(config-ike-gw)# ike-policy ike_pol1
R2(config-ike-gw)# local address 2.2.2.2
R2(config-ike-gw)# local network 2.2.2.2/32 protocol gre
R2(config-ike-gw)# remote address 1.1.1.2
R2(config-ike-gw)# remote network 1.1.1.2/32 protocol gre
R2(config-ike-gw)# mode policy_based
R2(config-ike-gw)# exit
R2(config)# security ipsec proposal ipsec_prop1
R2(config-ipsec-proposal)# authentication algorithm md5
R2(config-ipsec-proposal)# encryption algorithm aes128
R2(config-ipsec-proposal)# pfs dh-group 2
R2(config-ipsec-proposal)# exit
R2(config)# security ipsec policy ipsec_pol1
R2(config-ipsec-policy)# proposal ipsec_prop1
R2(config-ipsec-policy)# exit
R2(config)# security ipsec vpn ipsec1
R2(config-ipsec-vpn)# ike establish-tunnel route
R2(config-ipsec-vpn)# ike gateway ike_gw1
R2(config-ipsec-vpn)# ike ipsec-policy ipsec_pol1
R2(config-ipsec-vpn)# enable
```






