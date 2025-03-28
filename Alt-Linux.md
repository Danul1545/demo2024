# ALT - LINUX    


<details>
<summary>Базова настройка всех устройств</summary>

__Цель задания:__  
Выполнить базовую настройку всех устройств:  
    a.Собрать топологию согласно рисунку. Все устройства работают на OC Linux - ALT  
            - ISP - Альт Сервер 10.2 (CLI)  
            - CLI - Альт Рабочая станция 10.2 (GUI)  
            - HQ-R - Альт Сервер 10.2 (CLI)  
            - HQ-SRV - Альт Сервер 10.2 (GUI)  
            - BR-R - Альт Сервер 10.2 (CLI)  
            - BR-SRV - Альт Сервер 10.2 (CLI)  
    b.Присвоить имена в соответствии с топологией  
    c.Рассчитать IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактировать таблицу.  
    d.Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустить этот пункт.  
    e.Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустить этот пункт.  

### Топология сети

![Снимок экрана 2023-12-06 143215](https://github.com/Danul1545/demo2024/assets/148867600/24a47534-3ea4-4205-83db-53da5d0edea6)

### Таблица сети (разбитая на подсети)

| Имя устройства | Интерфейсы |  IPv4/IPv6    | Маска/Префикс |      Шлюз      |
| :------------: | :--------: | :----------:  | :----------:  | :------------: |
|                | ens192     | 10.12.25.10   | /24           | 10.12.13.254   |
| ISP            | ens224     | 192.168.0.170 | /30           |                |
|                | ens256     | 192.168.0.162 | /30           |                |
|                | ens161     | 1.1.1.3       | /30           |                |
| HQ-R           | ens192     | 192.168.0.2   | /25           |                |
|                | ens224     | 192.168.0.169 | /30           | 192.168.0.164  | 
| BR-R           | ens192     | 192.168.0.130 | /27           |                |
|                | ens224     | 192.168.0.161 | /30           | 192.168.0.164  |
| HQ-SRV         | ens192     | 192.168.0.1   | /25           | 192.168.0.2    |
| BR-SRV         | ens192     | 192.168.0.129 | /27           | 192.168.0.130  |
| CLI            | ens192     | 1.1.1.2       | /30           | 1.1.1.3        |



### Выполнение задания  
### Метод, если интерфейсы были добалены до полной установки системы
Для начала узнал, какие интерфейсы есть на `ISP`:
```
ip a
```
После этого приступил к настройке статической маршрутизации  
Открыл файл options для нужного интерфейса:  
```
vim /etc/net/ifaces/ens192/options
```
Там поменял все как на примере:   
```
BOOTPROTO=static
TYPE=eth
CONFIG_WIRELESS=no
SYSTEMD_BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=no
```
После этого задал нужный адрес на интрефейс:  
```
echo xxx.xxx.xxx.xxx/xx > /etc/net/ifaces/ensxxx/ipv4address
```
Если нужно добавить шлюз по умолчанию, то нужна эта команда:  
```
echo default via xxx.xxx.xxx.xxx > /etc/net/ifaces/xxx/ipv4route
```
`Вместо x, нужно вставить IP-адрес и номер интерфейса`  
Если нужно указать информацию о DNS-сервере, прописываем команду:  
```
echo nameserver 8.8.8.8 > /etc/resolv.conf
```
После этого перезагружаем сетевую службу:  
```
service network restart
```
И смотрим результат:  
```
ip a
```
Если на интерфейсе показывается 2 IP-адреса то  нужно отключить NetworkManager командой:
```
systemctl disable network.service NetworkManager
```
---
### Добовление интерфейсов.  
```
mkdir /etc/net/ifaces/xxx
```
`Вместо x пишется нужный интерфейс`

Далее в данной папку нужно создать файл `options` со следующими параметрами:
```
BOOTPROTO=static
TYPE=eth
CONFIG_WIRELESS=no
SYSTEMD_BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=no
```
После этого задается нужный адрес на интрефейс:  
```
echo xxx.xxx.xxx.xxx/xx > /etc/net/ifaces/ensxxx/ipv4address
```
Если нужно добавить шлюз по умолчанию, то нужна эта команда:  
```
echo default via xxx.xxx.xxx.xxx > /etc/net/ifaces/xxx/ipv4route
```
`Вместо x, нужно вставить IP-адрес и номер интерфейса` 

Если нужно указать информацию о DNS-сервере, прописываем команду:  
```
echo nameserver 8.8.8.8 > /etc/resolv.conf
```
После этого перезагружаем сетевую службу:  
```
service network restart
```
И смотрим результат:  
```
ip a
```
Если на интерфейсе показывается 2 IP-адреса то  нужно отключить NetworkManager командой:
```
systemctl disable network.service NetworkManager
```

### Всё тоже самое повторил на других интерфейсах

## Настройка тунеля между HQ-R и BR-R.

Создаём новый интерфейс.
```
mkdir /etc/net/ifaces/tun1
```
Заходим в настройки итерфейса
```
vim /etc/net/ifaces/tun1/options
```
Пишем такие настройки:

```
TYPE=iptun
TUNTUPE=gre
TUNLOCAL=xxx.xxx.xxx.xxx
TUNREMOUTE=xxx.xxx.xxx.xxx
TUNOPTIONS='ttl 64'
HOST=ensxx
```
вместо x пишем ip и интерфейс 2 роутеров.

`TUNLOCAL`- ip данного роутера, а`TUNREMOUTE` - второго к которму мы делаем тунель.

Назначаем ip для тунеля:
```
echo 172.16.100.1/24 > /etc/net/ifaces/tun1/ipv4address
```
Также назначаем ipv6:
```
echo 2001:5::1/64 > /etc/net/ifaces/tun1/ipv6address
```
И перезапускаем сеть.
```
systemctl restart network
```
### Всё тоже самое повторяем на втором роутере.

после настройки проверяем командой `ping`

![image](https://github.com/Danul1545/demo2024/assets/148867600/cf6fddea-e2d4-45c9-8f5c-1be4ebf6c637)

</details>
    
<details>
    <summary>NAT с помощью firewalld ISP,HQ-R,BR-R</summary>
    
Отключить NetworkManager:
```
systemctl disable network.service NetworkManager
```
Настройки интерфейсов должны быть такими:
```
NM_CONTROLLED=no
DISABLED=no
```
Установка firewalld:
```
apt-get -y install firewalld
```
Автозагрузка:
```
systemctl enable --now firewalld
```
Правила к исходящим пакетам:
```
firewall-cmd --permanent --zone=public --add-interface=ens33
```
Правила к входящим пакетам:
```
firewall-cmd --permanent --zone=trusted --add-interface=ens34
```
Включение NAT:
```
firewall-cmd --permanent --zone=public --add-masquerade
```
Сохранение правил:
```
firewall-cmd --reload
```
    <details>
    
NAT 2 способ ISP,HQ-R,BR-R:
Правило:
```
iptables -A POSTROUTING -t nat -j MASQUERADE
```
Применение правил, работает только до перезагрузки:
```
iptables-save
```
Сохранение правил:
```
nano /etc/net/scripts/nat
```
```
#!/bin/sh
/sbin/iptables -A POSTROUTING -t nat -j MASQUERADE
```
```
chmod +x /etc/net/scripts/nat
```
Автозагрузка:
```
service iptables enable
```
</details>


<details><summary>Маршрутизация через frr</summary>

Настройте внутреннюю динамическую маршрутизацию по средствам FRR. Выберите и обоснуйте выбор протокола динамической маршрутизации из расчёта, что в дальнейшем сеть будет масштабироваться.  
a. Составьте топологию сети L3.  

Установка пакета:
```
apt-get -y install frr
```
Автозагрузка:
```
systemctl enable --now frr
```
Включение демона службы ospf:
```
nano /etc/frr/daemons
```
```
ospfd=yes
```
```
systemctl restart frr
```
Вход в среду роутера:
```
vtysh
```
Показать интерфейсы:
```
sh in br
```
|Interface|Status|VRF|Adresses|
|:----:|:-:|:------:|:--------------:|
|ens224|up |default |192.168.0.162/30|
|ens192|up |default |192.168.0.129/27|
|lo    |up |default |                |

Активировать ospf:
```
router ospf
```
Вводим СЕТИ:
```
net 192.168.0.160/30 area 0
net 192.168.0.128/27 area 0
```
Показать соседей:
```
do sh ip ospf neighbor
```
СОХРАНИТЬ КОНФИГИ:
```
do w
```

![image](https://github.com/abdurrah1m/DEMO2024/assets/148451230/a39631c1-a683-47d2-a63a-4bbb93d7556a)
</details>

<details><summary>Раздача ip-адресов через dhcp</summary>

Настройте автоматическое распределение IP-адресов на роутере HQ-R.  
a. Учтите, что у сервера должен быть зарезервирован адрес.

Установка пакета:
```
apt-get -y install dhcp-server
```
`/etc/sysconfig/dhcpd`, указываю интерфейс внутренней сети:
```
DHCPDARGS=ens19
```
Копирую образец:
```
cp /etc/dhcp/dhcpd.conf.sample /etc/dhcp/dhcpd.conf
```
`/etc/dhcp/dhcpd.conf` параметры раздачи:
```
ddns-update-style-none;

subnet 192.168.0.0 netmask 255.255.255.128 {
        option routers                  192.168.0.1;
        option subnet-mask              255.255.255.128;
        option domain-name-servers      8.8.8.8, 8.8.4.4;

        range dynamic-bootp 192.168.0.20 192.168.0.50;
        default-lease-time 21600;
        max-lease-time 43200;
}
```
```
systemctl restart dhcpd
```
```
systemctl status dhcpd.service
```
Автозагрузка:
```
chkconfig dhcpd on
service dhcpd start
```
HQ-SRV (клиент):
```
nano /etc/net/ifaces/ens18/ipv4address
```
```
#192.168.0.40
```
```
nano /etc/net/ifaces/ens18/options
```
```
BOOTROTO=dhcp
TYPE=eth
NM_CONTROLLED=yes
DISABLED=no
CONFIG_IPV4=yes
```
```
service network restart
```
```
ens18:
    inet 192.168.0.38/25 brd 192.168.0.127
```
</details>

<details><summary>Добавление пользователей на виртуальные машины</summary>

Настройте локальные учётные записи на всех устройствах в соответствии с таблицей.

|Учётная запись|Пароль|Примечание|
|:--------------:|:------:|:----------------:|
|Admin           |P@ssw0rd|CLI, HQ-SRV       |
|Branch admin    |P@ssw0rd|BR-SRV, BR-R      |
|Network admin   |P@ssw0rd|HQ-R, BR-R, HQ-SRV|

Пользователь `admin` на `HQ-SRV`
```
adduser admin
```
```
usermod -aG root admin
```
```
passwd admin
P@ssw0rd
P@ssw0rd
```
```
nano /etc/passwd
```
```
admin:x:0:501::/home/admin:/bin/bash
```
</details>

<details><summary>Измерьте пропускную способность сети между двумя узлами</summary>


Измерьте пропускную способность сети между двумя узлами HQ-R-ISP по средствам утилиты iperf 3. Предоставьте описание пропускной способности канала со скриншотами.

```
apt-get -y install iperf3
```
ISP как сервер:
> если надо открыть порт
>```
>iptables -A INPUT -p tcp --dport 5201 -j ACCEPT
>```
```
iperf3 -s
```
HQ-R:
```
iperf3 -c 192.168.0.161 -f M
```
```
[ID] Interval      Transfer   Bitrate        Retr Cwnd
[ 5] 0.00-1.00 sec 345 MBytes 344 MBytes/sec    0 538 KBytes
[ 5] 1.00-2.00 sec 338 MBytes 338 MBytes/sec    0 676 KBytes
[ 5] 3.00-4.00 sec 341 MBytes 341 MBytes/sec    0 749 KBytes
```
</details>

<details><summary>backup скрипты для сохранения конфигурации сетевых устройств</summary>

Составьте backup скрипты для сохранения конфигурации сетевых устройств, а именно HQ-R BR-R. Продемонстрируйте их работу.

Заход в планировщик заданий:
```
EDITOR=nano crontab -e
```
минута | час | день | месяц | день недели | "команда, например `reboot`":
```
9 15 * * * cp /etc/frr/frr.conf /etc/networkbackup
```
```
ls /etc/networkbackup
```
```
frr.conf
```
</details>

<details><summary>подключение по SSH для удалённого конфигурирования устройства</summary>

Настройте подключение по SSH для удалённого конфигурирования устройства HQ-SRV по порту 2222. Учтите, что вам необходимо перенаправить трафик на этот порт по средствам контролирования трафика.

HQ-SRV:
```
apt-get -y install openssh-server
```
```
systemctl enable --now sshd
```
```
nano /etc/openssh/sshd_config
```
```
Port 2222
PermitRootLogin no
PasswordAuthentication yes
```
Подключение
```
ssh student@192.168.0.40 -p 2222
```

</details>

<details><summary>контроль доступа до HQ-SRV по SSH</summary>


Настройте контроль доступа до HQ-SRV по SSH со всех устройств, кроме CLI.

HQ-SRV:
```
nano /etc/openssh/sshd_config
```
Выбор пользователей
```
AllowUsers student@192.168.0.1 student@192.168.0.140 student@192.168.0.129 student@10.10.201.174
```
</details>





