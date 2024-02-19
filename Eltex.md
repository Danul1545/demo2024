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








