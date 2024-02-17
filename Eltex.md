## Создание нового пользователя

- Configure – используется для входа в режим полной конфигурации.
- Hostname – используется для изменения имени хоста.
- Commit – используется для применения изменений конфигурации устройства.
- Confirm – используется для сохранения изменений конфигурации устройства.

```
Router# configure
Router(config)# username user
Router(config-user)# password user 
Router(config-user)# privilege 1
Router(config-user)# exit
Router(config)# exit
Router# commit
Router# confirm
```

### Сброс устройства к заводским настройкам
```
Router# copy system:default-config system:candidate-config
```

### Узнать интерфейсы роутера 
- Show ip interface – просмотр информации об интерфейсах.
- Interface <интерфейс> – переход к настройкам интерфейса.
- Ip address <адрес/маска> – настройка IP-адреса интерфейса.
- Ip firewall disable – отключает firewall на интерфейсе.

```
show ip interfaces
show interfaces status
```

## Настройка IP-адресации
```
Router# configure
Router(config)# interface gi1/0/1
Router(config-if-gi)# ip firewall disable
Router(config-if-gi)# ip address 192.168.0.210/24
Router(config-if-gi)# exit
Router(config)# exit
Router# commit
Router# confirm
```

## Создание и удаление VLAN на коммутаторе
```
Switch# configure
Switch(config)# vlan 2
Switch(config-vlan)# name test
```

### VLAN создается на коммутаторе только посредством команды vlan.\
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
