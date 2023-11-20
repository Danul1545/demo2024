# demo2024

## №1

## Описание задания.
Выполните базовую настройку всех устройств:

1) Соберите топологию согласно рисунку. Все устройства работают на OC Linux - Debian
2) Присвоить имена в соответствии с топологией
3) Рассчитайте IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактируйте таблицу.
4) Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.
5) Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.
  
## Топология №1.

![image](https://github.com/Danul1545/demo2024/assets/148867600/88bc13c3-8222-45fa-9848-3fe04e8da41d)

### Таблица №1 (разбита на подсети).

| Имя устройства | Итерфейс |  IPv4/IPv6   | Маска/Префикс |       Шлюз       |
| -------------- | -------- | ------------ | ------------- |    ----------    |
|                | ens 192  | 10.12.25.10  | /24           | 10.12.25.254     |
| ISP            | ens 224  | 192.168.0.166| /30           |                  |
|                | ens 256  | 192.168.0.162| /30           |                  |
| HQ-R           | ens 192  | 192.168.0.2  | /25           |                  |
|                | ens 224  | 192.168.0.165| /30           |                  |
| BR-R           | ens 192  | 192.168.0.132| /27           |                  |
|                | ens 224  | 192.168.0.161| /30           |                  |
| HQ-SRV         | ens 192  | 192.168.0.1  | /25           | 192.168.0.2      |
| BR-SRV         | ens 192  | 192.168.0.130| /27           | 192.168.0.132    |

### 1. Настройка интерфесов.

Просмотр существующих интерфейсов  ` ip a `

Заходим на файл  конфигурации интерфейсов.`nano /etc/network/interfaces`

#### Назначаем IP адреса для ISP в соотвествии с таблицей.
```
auto ens192
iface ens192 inet static
address 10.12.25.10
netmask 255.255.255.0
gateway 10.12.25.254

auto ens224
iface ens224 inet static
address 192.168.0.170
netmask 255.255.255.252

auto ens256
iface ens256 inet static
address 192.168.0.164
netmask 255.255.255.252
```
#### Для HQ-R
```
auto ens192
iface ens192 inet static
address 192.168.0.2
netmask 255.255.255.128

auto ens224
iface ens192 inet static
address 192.168.0.169
netmask 255.255.255.252
```
#### Для BR-R
```
auto ens192
iface ens192 inet static
address 192.168.0.132
netmask 255.255.255.224

auto ens224
iface ens192 inet static
address 192.168.0.163
netmask 255.255.255.252
```
#### Для HQ-SRV
```
auto ens192
iface ens192 inet static
address 192.168.0.1
netmask 255.255.255.128
```
#### Для BR-SRV
```
auto ens192
iface ens192 inet static
address 192.168.0.130
netmask 255.255.255.224
```

Сохраняем и выходим комбинацией клавиш: `Ctrl+S` и `Ctrl+X`

Перезагружаем сеть `systemctl restar networking`

#### Проверяем настройку командой
```
ip a
```
Должно выйти такое:



### Настройка NAT на ISP, HQ-R, BR-R

#### Устанавливаем зеркала для скачивания.
```
nano /etc/apt/sources.list
```
#### Добовляем в конец списка.
```
deb http://mirror.yandex.ru/debian/ stable main contrib non-free
```

#### Обновляем и устанавливваем протокол `iptables`
```
apt update
apt install iptables
```

#### Заходим на файл конфигурации.
```
nano /etc/sysctl.conf
```

#### Меняем строку как на фото.
![image](https://github.com/Danul1545/demo2024/assets/148867600/574307a7-748c-487f-b4e9-31ff31a90df7)

Проверяем что всё работает командой `sysctl -p`

#### Прописываем команду
```
iptables -A POSTROUTING -t nat -j MASQUERADE
```

## №1.2

### Описание задания.

Настроить внутренюю динамическую маршрутицацию по средствам FRR. Выберите и обоснуйте выбор протокола динамической маршрутицации из расчёта, что в дальнейшем сеть будет масштабиоваться.

#### Установка пакета frr.
```
apt update
apt-get install frr
```
После утановки проверим состояние frr  `systemctl status frr`

#### Заходим в конфигурацию файла frr.
```
nano /etc/frr/daemons
```

Меняем ___ospfd=no___ на:   ` ospfd=yes ` и перезапускаем его  `systemctl restart frr`

Заходим в среду роутера командой `vtysh`

#### Проверяем интеррфейсы.
```
sh int br
```

![image](https://github.com/Danul1545/demo2024/assets/148867600/6034adea-1fc6-4196-8e7a-6d25ff6cc885)

Заходим в терминал ospf командой `conf t` (configuration terminal), `router ospf`

#### Настраиваем ospf.
```
net 192.168.0.0/25 area 0
net 192.168.0.164/30 area 0
```

#### Проверим натройку.
```
sh ip ospf neighbor
```
#### Общая топология сети.

![image](https://github.com/Danul1545/demo2024/assets/148867600/85946a86-ea26-4c94-9bfa-60bf0495e4d2)

С HQ-SRV ДО BR-SRV `ping 192.168.0.130`

## №1.3

### Описание задания.

Настроить автоматическое распределение IP-адресов на роутере HQ-R.

#### Установка DHCP.
```
aot install isc-dhcp-server
```

#### заходим в режим конфигурации.
```
nano /etc/default/isc-dhcp-server
```

#### Указываем интерфейс в сторону Интернета.

![image](https://github.com/Danul1545/demo2024/assets/148867600/70439aac-94b0-4c48-9946-f9aa1c29a66e)

Настройка раздачи IP-адресов.
```
subnet 192.168.0.0 netmask 255.255.255.0 {
range 192.168.0.2 192.168.0.125;
option domain-name-servers 8.8.8.8, 8.8.4.4;
option routers 192.168.0.2;
}
```

#### Перезапускаем службу.
```
systemctl restart isc-dhcp-server.service
```

## №1.4

### Описание задания.

Настроить локальные учётные записи на всех устройствах в соотвествии с таблицей.

### Таблица №2 (локальные учётные записи)

| Учётная запись | Пароль   | Примечание            |
| :------------: | :------: | :-------------------: |
| Admin          | P@ssw0rd |  CLI, HQ-SRV, HQ-R    |
| Branch admin   | P@ssw0rd |  BR-SRV, BR-R         |
| Network admin  | P@ssw0rd |  HQ-R, BR-SRV, BR-SRV |




