# <p align="center">ALT - LINUX</p>

# Модуль 1

### <p align="center">модуль 1 задание 1</p>

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

Топология сети

![Снимок экрана 2023-12-06 143215](https://github.com/Danul1545/demo2024/assets/148867600/24a47534-3ea4-4205-83db-53da5d0edea6)

Таблица сети (разбитая на подсети)

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



Выполнение задания  
Метод, если интерфейсы были добалены до полной установки системы
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
Добовление интерфейсов.  
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

Всё тоже самое повторил на других интерфейсах

</details>

<details>
<summary>Настройка тунеля на роутерах</summary> 

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
Всё тоже самое повторяем на втором роутере.

после настройки проверяем командой `ping`

![image](https://github.com/Danul1545/demo2024/assets/148867600/cf6fddea-e2d4-45c9-8f5c-1be4ebf6c637)

</details>

### <p align="center">модуль 1 задание 2</p>

<details>
    <summary>NAT с помощью iptables</summary>

Включить ip-адресацию `/etc/net/sysctl.conf`
```
net.ipv4.ip_forward = 1
```

Приминить изменения
```
sudo sysctl -p
```

Интерфейсы:
- `eth0` - внешний интерфейс
- `eth1` - внутрений интерфейс

Интерфейс с раздачей интернета:
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Разрешения на передачу адресации:
внутри
```
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
```
снаружи
```
iptables -A FORWARD -i eth0 -o eth1 -m state --state ESTABLISHED,RELATED -j ACCEPT
```

Сохранить настройку:
```
iptables-save
```

</details>

<details>
    <summary>NAT с помощью firewalld</summary>
    
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

</details>

### <p align="center">модуль 1 задание 3</p>

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

### <p align="center">модуль 1 задание 4</p>

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

### <p align="center">модуль 1 задание 5</p>

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

### <p align="center">модуль 1 задание 6</p>

<details><summary>Измерьте пропускную способность сети между двумя узлами</summary>


Измерьте пропускную способность сети между двумя узлами HQ-R-ISP по средствам утилиты iperf 3. Предоставьте описание пропускной способности канала со скриншотами.

```
apt-get -y install iperf3
```
ISP как сервер:
если надо открыть порт
```
iptables -A INPUT -p tcp --dport 5201 -j ACCEPT
```
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

### <p align="center">модуль 1 задание 7</p>

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

### <p align="center">модуль 1 задание 8</p>

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

### <p align="center">модуль 1 задание 9</p>

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

# Модуль 2


### <p align="center">модуль 2 задание 1</p>

<details><summary>Настройка доменного контроллера Samba на машине BR-SRV</summary>

Перед настройкой самого контроллера домена удалим службу bind с нашего сервера:
```
apt-get remove bind
```

Переходим к настройке BR-SRV:
Проверьте, что /etc/resolv.conf хранит запись `nameserver 127.0.0.1` и применяем её:
```
resolvconf -u
```

Теперь ставим долгожданную службу samba на BR-SRV:
```
apt-get update
apt-get install task-samba-dc admc -y
```

Проверяем, что установлено полное доменное имя у BR-SRV: `hostname -f`

![image](https://github.com/user-attachments/assets/27b50faf-9c5c-4234-8186-89c58916625d)


Если запись не соответствует рисунку выше, то нужно его настроить:
```
hostnamectl set-hostname br-srv.au-team.irpo; exec bash
```

Настроим hosts, добавив новую запись в конец файла: `mcedit /etc/hosts` (Ставим свой Ip)

![image](https://github.com/user-attachments/assets/bdffc8c5-0d6f-4a41-8baf-edccff048313)

Теперь в конфигурацию нашего DNS-сервера на HQ-SRV добавим следующую строку: `server=/au-team.irpo/192.168.4.2`


![image](https://github.com/user-attachments/assets/f614d707-d84a-4fc0-b11f-e8ee9877b86c)

Перезапускаем dnsmasq как службу:
```
systemctl restart dnsmasq
```

А теперь запускаем автонастройку доменного контроллера на BR-SRV. Если предложенные значения верны, те, что находятся в [], то нажимаем Enter, если нет, то нужно проверять предыдущие настройки.
- samba-tool domain provision
- AU-TEAM.IRPO
- AU-TEAM
- dc
- SAMBA_INTERNAL
- 192.168.1.2 (Здесь вводим Ip вручную)
- P@ssw0rd

![image](https://github.com/user-attachments/assets/2a083e6d-a486-4e45-9f51-b60316769ea6)

После настройки должно появится такое в терминале, это значит, что всё настроено верно:

![image](https://github.com/user-attachments/assets/5ca2d482-87c0-4893-9f14-b6c0315dacb2)

Перемещаем сгенерированный конфиг krb5.conf и включаем службу samba:
```
mv -f /var/lib/samba/private/krb5.conf /etc/krb5.conf
systemctl enable samba
```

Из-за того, что на Alt Linux могут пропадать IP-адреса после перезагрузки системы, добавим запись о перезапуске службы network и samba в crontab (именно в таком порядке), пишем в консоль:
```
export EDITOR=mcedit
сrontab -e
```

И вносим в конец файла следующие строки:
```
@reboot /bin/systemctl restart network
@reboot /bin/systemctl restart samba
```

![image](https://github.com/user-attachments/assets/92a32f4c-6758-4a7e-b7dd-a62f3fc5440c)

Теперь ПЕРЕЗАПУСКАЕМ машину BR-SRV: `reboot`

Проверяем работу домена:
```
samba-tool domain info 127.0.0.1
```

![image](https://github.com/user-attachments/assets/eff03722-21e1-4d9a-a2cc-545578f834f7)

Домен работает, у вас должно всё соответствовать картинке выше.
Теперь создадим 5 пользователей:
```
samba-tool user add user1.hq P@ssw0rd
samba-tool user add user2.hq P@ssw0rd
samba-tool user add user3.hq P@ssw0rd
samba-tool user add user4.hq P@ssw0rd
samba-tool user add user5.hq P@ssw0rd
```

Теперь создадим группу и поместим туда созданных пользователей:
```
samba-tool group add hq
samba-tool group addmembers hq user1.hq,user2.hq,user3.hq,user4.hq,user5.hq
```

Теперь введём HQ-CLI `Центр управления системой -> Айтентийфикация` заполняем по скриншоту:

![image](https://github.com/user-attachments/assets/b9ece031-d083-4ac4-b75c-50fa16d30e9f)

Вводим пароль, который вводили при настройке домена через `samba-tool`.

После ввода в домен должно появиться следующее сообщение на экране: 

![image](https://github.com/user-attachments/assets/ca4cb9a5-990a-4595-a356-ef1f94e15ae0)

Перезагружаем машину HQ-CLI.

Чтобы настроить права созданных нами пользователей, нужно установить ещё один пакет на BR-SRV, но перед этим нужно подключить нужный репозиторий следующей командой:
```
apt-repo add rpm http://altrepo.ru/local-p10 noarch local-p10
```

Теперь обновляем список пакетов и можем устанавливать нужный нам пакет:
```
apt-get update
apt-get install sudo-samba-schema
```

Далее добавляем новую схему следующей командой:
```
sudo-schema-apply
```

В открывшимся окне, нажимаем yes: 

![image](https://github.com/user-attachments/assets/9f13e018-cb6a-4ae1-b5f4-11b1944ccb11)

Затем прописываем пароль от доменного администратора.
После этого должно появиться такое окно:

![image](https://github.com/user-attachments/assets/64c0cbba-d4ac-4714-a3f1-4432c709d460)

Далее мы создаём новое правило следующей командой (которую он сам предлагает в этом окне): `create-sudo-rule`
И вносим следующие изменения (имя правила можно любое):
- Имя правила      : `prava_hq`
- sudoCommand      : `/bin/cat`
- sudoUser         : `%hq`

При успешном добавлении выведет следующие строки:

![image](https://github.com/user-attachments/assets/1a6fef2c-1dc1-47f7-ba71-f320a2cfaae6)

На HQ-CLI ставим пакет:
```
apt-get install admc
```

Затем создаём тикет доменного администратора, чтобы получить права на редактирование правил на сервере:
```
kinit administrator
P@ssw0rd
```

И запускаем admc:

![image](https://github.com/user-attachments/assets/694282fb-df82-48b3-81a0-2e021f8c6899)

Включим дополнительные возможности через настройки:

![image](https://github.com/user-attachments/assets/29271ece-816b-46b8-a5a6-72ca769970e6)


Поменяем опцию `sudoOption` в созданном нами ранее правиле `prava_hq` (правило всегда будет находиться в OU с названием sudoers):

![image](https://github.com/user-attachments/assets/f6d61af1-9b63-4230-b9bf-e4f7c033fbbc)

Новое значение будет: `!authenticate`

И добавим ещё две команды в опцию sudoCommand: `grep и id`

![image](https://github.com/user-attachments/assets/46fb15a1-0963-44b0-bba4-faa6370a1419)

Теперь, чтобы работали все созданные нами правила, нужно зайти на HQ-CLI и установить дополнительные пакеты:
```
apt-get update
apt-get install sudo libsss_sudo
```

Разрешаем использование sudo:
```
control sudo public
```

Настроим конфиг `sssd.conf`:
```
mcedit /etc/sssd/sssd.conf
services = nss, pam, sudo
sudo_provider = ad
```

![image](https://github.com/user-attachments/assets/1875680f-a944-42dd-ae00-f6aabd842f2d)

Теперь отредактируем `nsswitch.conf`:

```
mcedit /etc/nsswitch.conf
sudoers: files sss
```

![image](https://github.com/user-attachments/assets/fac63c5f-28ce-4bd5-9d0c-8fe62809f098)

Теперь перезагрузим нашу клиентскую машину HQ-CLI `reboot`

На данном этапе мы можем проверить настроенные нами права и правильность настроек конфигурационных файлов. Сделать мы это можем под локальной учётной записью, у которого есть права администратора, в нашем случае это просто root. А ещё мы можем открыть вторую сессию нажав сочетание клавиш: `Ctrl+Alt+F2`
В дальнейшем мы можем переключаться между ними, т.к. нажатием тех же клавиш, но теперь уже с F1 мы вернемся на первую нашу сессию с графической оболочкой: `Ctrl+Alt+F1`
После того как зашли на вторую сессию, логинимся под `root`.

На всякий случай, нужно очистить кэш и удалить остаточные файлы, чтобы всё перезаписалось и применилось, для этого пишем следующие команды:
```
rm -rf /var/lib/sss/db/*
sss_cache -E
```

И перезагружаем службу sssd:
```
systemctl restart sssd
```

Теперь проверим, какие правила для sudoers получил наш доменный пользователь:
```
sudo -l -U user1.hq
```

![image](https://github.com/user-attachments/assets/8be98128-378b-45a3-9add-8683c69c3b06)

Вернёмся в первую сессию и залогинимся под нашем доменным пользователем user1.hq и проверить настроенные права наглядно: `Ctrl+Alt+F1`
```
sudo cat /etc/passwd | sudo grep root && sudo id root
```

![image](https://github.com/user-attachments/assets/5a112569-d820-4ab8-88d9-5c6b297b1ea2)

Приступаем к следующему этапу – импортируем пользователей из таблицы `Users.csv`.

Для этого нам нужно скачать файл `Users.csv`, но на ДЭ он уже будет скачан и лежать в каталоге `/opt`. Мы же, для обучения, скачиваем сейчас его сами и перемещаем его в `/opt`. Для этого пишем следующие команды:
```
curl -L https://bit.ly/3C1nEYz > /root/users.zip
unzip /root/users.zip
mv /root/Users.csv /opt/Users.csv
```

![image](https://github.com/user-attachments/assets/09a6bcaf-9691-4893-954d-9b92db6a64f2)

Создаём файл import и пишем туда следующий код:
```
#!/bin/bash
csv_file=”/opt/Users.csv”
while IFS=”;” read -r firstName lastName role phone ou street zip city country password; do
if [ “$firstName” == “First Name” ]; then
                continue
fi
username=”${firstName,,}.${lastName,,}”
sudo samba-tool user add “$username” 123qweR%
done < “$csv_file”
```

![image](https://github.com/user-attachments/assets/4434e219-28bb-464f-a6b6-abe665e29db5)

Сохраняем этот файл и выдаём ему право на выполнение и запускаем его:
```
chmod +x /root/import
bash /root/import
```

![image](https://github.com/user-attachments/assets/38b021ad-4df9-4f6b-a07c-159ecc792216)

</details>

### <p align="center">модуль 2 задание 2</p>

<details><summary>Конфигурация файлового хранилища на HQ-SRV</summary>

Качаем утилиту:
```
apt-get install -y mdadm
```

Создаём 3 диска по 1 гб и смотрим созданные диски: `lsblk`

![image](https://github.com/user-attachments/assets/f08c11d5-6eea-44f3-afec-d034ebc87bb4)

Стираем данные суперблоков:
```
/sbin/mdadm --zero-superblock --force /dev/sd{b,с,d}
```

Если мы получили такой ответ то значит, что диски не использовались ранее для RAID:

![image](https://github.com/user-attachments/assets/3d578b36-5b02-4f0b-9e1b-f2c315b55cd8)

После удалим старые метаданные и подпись на дисках:
```
/sbin/wipefs --all --force /dev/sd{b,c}
```

Создание RAID:
```
/sbin/mdadm --create --verbose /dev/md0 -l 5 -n 3 /dev/sd{b,c,d}
```

![image](https://github.com/user-attachments/assets/8897224b-3e89-4edb-84cc-73815f4b66e6)

где:

- /dev/md0 — устройство RAID, которое появится после сборки;
- -l 5 — уровень RAID;
- -n 3 — количество дисков, из которых собирается массив;
- /dev/sd{b,c,d} — Диски для установки.

Проверяем: `lsblk`

![image](https://github.com/user-attachments/assets/d3e61522-fb95-4542-8ae9-deb5e215edd0)

Создание файла mdadm.conf:
```
mkdir /etc/mdadm
echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
/sbin/mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
```

Содержимое `mdadm.conf`:

![image](https://github.com/user-attachments/assets/9278689b-93fa-404c-ac1c-b3376a5731ef)

Создание файловой системы для массива:
```
/sbin/mkfs.ext4 /dev/md0
```

Автозагрузка раздела с помощью fstab. Смотрим идентификатор раздела:
```
/sbin/blkid | grep /dev/md0
```

![image](https://github.com/user-attachments/assets/4660f0a6-8105-40b5-81dc-1d01803b3421)

Зписываем этот идентификатор, открываем fstab и добавляем строку:
```
nano /etc/fstab
```

![image](https://github.com/user-attachments/assets/bba354e6-18d5-4121-9a25-a2aa0cfaa731)


Выполняем монтирование `mount -a` и проверяем `df -h`

![image](https://github.com/user-attachments/assets/67ee4764-f7b7-44e8-ad44-af015d82a7dd)

</details>


<details><summary>Настройка NFS-сервера</summary>

Установка пакетов для NFS сервера:
```
apt-get install -y nfs-server
apt-get install -y rpcbind
apt-get install -y nfs-clients
apt-get install -y nfs-utils
```

Автозагрузка:
```
systemctl enable --now nfs
```

Создание директории общего доступа:
```
mkdir /mnt/nfs_share
chmod 777 /mnt/nfs_share
```

Редактируем `exports`:
```
nano /etc/exports
```

![image](https://github.com/user-attachments/assets/0a9a8734-6883-4f5c-aeb5-a406c064822c)

где:

- /mnt/nfs_share - общий ресурс
- 192.168.0.0/25 - клиентская сеть, которой разрешено монтирования общего ресурса
- rw — разрешены чтение и запись
- no_root_squash — отключение ограничения прав root

Экспорт файловой системы:
```
/usr/sbin/exportfs -arv
```

![image](https://github.com/user-attachments/assets/a6c66b79-3a0f-49ab-a2cc-68910feb6165)

exportfs с флагом -a, означающим экспортировать или отменить экспорт всех каталогов, -r означает повторный экспорт всех каталогов, синхронизируя /var/lib/nfs/etab с /etc/exports и файлами в /etc/exports.d, а флаг -v включает подробный вывод.

Запускаем и добавляем в автозагрузку NFS-сервер:
```
systemctl enable --now nfs-server
```

</details>


<details><summary>Настройка NFS-клиента</summary>

Установка пакетов для NFS-клиента:
```
apt-get update && apt-get install -y nfs-{utils,clients}
```

Создадим директорию для монтирования общего ресурса:
```
mkdir /opt/share
chmod 777 /opt/share
```

Настраиваем автомонтирование общего ресурса через fstab:
```
nano /etc/fstab
```

- где: 192.168.0.40 - адрес файлового сервера

Монтируем: `mount -a` и проверяем `df -h`

![image](https://github.com/user-attachments/assets/d97bba9e-3ee7-412b-b1c1-597d7c4ae7fd)

![image](https://github.com/user-attachments/assets/c8e6fd8f-5fe2-4d4d-a284-d508ef4e49aa)

![image](https://github.com/user-attachments/assets/0dc9b132-587e-43f1-bd40-b2ac758199dd)

</details>

<details><summary>Сервер HQ-SRV</summary>

Создание общих папок на сервере:
```
mkdir /mnt/network -p
mkdir /mnt/admin_files -p
mkdir /mnt/branch_files -p
```

Заносим в exports:
```
nano /etc/exports
/mnt/network 192.168.0.0/25 192.168.100.0/27(rw,sync,no_root_squash)
/mnt/branch_files 192.168.100.0/27(rw,sync,no_root_squash)
/mnt/admin_files 192.168.0.0/25 4.4.4.0/30(rw,sync,no_root_squash)
```

Экспортируем:
```
/usr/sbin/exportfs -arv
```

</details>

<details><summary>Клиент HQ-SRV</summary>

Создаём папку:
```
mkdir /opt/admin
```

Задаём права:
```
chmod 777 /opt/admin/
```

Автозагрузка в fstab:
```
192.168.0.40:/mnt/admin_files /opt/admin nfs defaults 0 0
```

Монтаж: `mount -a`

![image](https://github.com/user-attachments/assets/9c40318b-e296-4263-b670-a11ed0e40bc5)

</details>

### <p align="center">модуль 2 задание 5</p>

<details><summary>Запустите сервис MediaWiki используя docker на сервере HQ-SRV</summary>

Установка Docker и Docker-compose:

```
apt-get update && apt-get install -y docker-engine
apt-get install -y docker-compose
```

Автозагрузка Docker:

```
systemctl enable --now docker
```
Загружаем образы следующей командой:

```
docker pull mediawiki
docker pull mariadb
```

Создаем в домашней директории пользователя файл, в качестве пользователя, которого мы создавали при установке ОС, у нас – user, а его домашний каталог – /home/user, файл называется – wiki.yml, для приложения MediaWiki:

```
mcedit /home/user/wiki.yml
```

И заполняем его следующими строками
```
services:
  mariadb:
    image: mariadb
    container_name: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: P@ssw0rd
      MYSQL_DATABASE: mediawiki
      MYSQL_USER: wiki
      MYSQL_PASSWORD: WikiP@ssw0rd
    volumes: [ mariadb_data:/var/lib/mysql ]
  wiki:
    image: mediawiki
    container_name: wiki
    restart: always
    environment:
      MEDIAWIKI_DB_HOST: mariadb
      MEDIAWIKI_DB_USER: wiki
      MEDIAWIKI_DB_PASSWORD: WikiP@ssw0rd
      MEDIAWIKI_DB_NAME: mediawiki
    ports:
      - "8080:80"
  #volumes: [ /home/user/mediawiki/LocalSettings.php:/var/www/html/LocalSettings.php ]
volumes:
  mariadb_data:
```
![image](https://github.com/user-attachments/assets/42f34c05-326b-4efc-9c7a-43476d81d9a7)

После всех настроек строку volumes.. мы обратно раскомментируем, убрав символ #!

Обычная версия:
```
docker-compose -f /home/user/wiki.yml up -d
```

Вторая версия `docker compose -f /home/user/wiki.yml up -d`

Заходим с клиента HQ-CLI на сайт после запуска контейнера:
![image](https://github.com/user-attachments/assets/b563650a-e7c6-468a-82be-1cb082ebbba9)

Видим, что файл LocalSettings.php не найден, и нажимаем на complete the installation или set up the wiki.

Выбираем удобный язык и нажиаем далее.
Заполняем строки состояния:
- Хост базы данных: `mariadb`
- Имя базы данных (без дефисов): `mediawiki`
- Имя пользователя базы данных: `wiki`
- Пароль базы данных: `WikiP@ssw0rd`    

Далее автоматически скачивается файл LocalSettings.php, который нужно переместить теперь на сервер с mediawiki, а именно на BR-SRV c HQ-CLI:
scp -P 2024 /home/user/Загрузки/LocalSettings.php sshuser@192.168.4.2:/home/sshuser/
![image](https://github.com/user-attachments/assets/ad61519c-4531-4a4f-9fc0-a73fa413212a)

Теперь заходим на сервер BR-SRV и перемещаем скачанный файл в /root, но перед этим удаляем то, что создалось в /root:
rm -rf /home/user/LocalSettings.php
mkdir /home/user/mediawiki
mv /home/sshuser/LocalSettings.php /home/user/mediawiki/
ls /home/user/mediawiki/


убираем # со строки volume.

Теперь перезапускаем контейнеры путём запуска контейнера ещё раз:
```
docker compose -f wiki.yml up -d
```

Проверим работу сайта, зайдем вновь через клиента HQ-CLI и увидим домашнюю страницу сайта:
![image](https://github.com/user-attachments/assets/4f00042a-eb35-4457-85ef-d00111a71d25)

</details>

### <p align="center">модуль 2 задание 6</p>

<details><summary>статическая трансляция портов</summary>

Пробросим порт 80 в порт 8080 и порт 2024 в порт 2024 на BR-SRV на маршрутизаторе BR-RTR, для обеспечения работы сервиса mediawiki и ssh, правила прописываем через консоль: (Ip адреса свои)
```
iptables -t nat -A PREROUTING -p tcp -d 192.168.4.1 --dport 80 -j DNAT --to-destination 192.168.4.2:8080
iptables -t nat -A PREROUTING -p tcp -d 192.168.4.1 --dport 2024 -j DNAT --to-destination 192.168.4.2:2024
```

Сохраняем правила, не забывайте, что у вас уже есть правила, которые мы писали ещё в первом модуле, проверьте, чтобы в этом файле сохранялись и прошлые, и новые (которые мы сейчас ввели):
```
iptables-save > /root/rules
```

Пробросим порт 2024 в порт 2024 на HQ-SRV на маршрутизаторе HQ-RTR, для обеспечения работы сервиса ssh, правило прописываем через консоль:
```
iptables -t nat -A PREROUTING -p tcp -d 192.168.1.1 --dport 2024 -j DNAT --to-destination 192.168.1.2:2024
```

Сохраняем правила:
```
iptables-save > /root/rules
```

Теперь перезагружаем ОБА роутера и проверим правила путём подключения с клиента HQ-CLI по ssh к серверу BR-SRV через IP-адрес роутера BR-RTR:
```
ssh -p 2024 sshuser@192.168.4.1
```

![image](https://github.com/user-attachments/assets/be405383-71bc-4d6e-8c2a-73bfd9cf14e1)


</details>

### <p align="center">модуль 2 задание 7</p>

<details><summary>Запустите сервис moodle на сервере HQ-SRV</summary>

Устанавливаем для ряд пакетов, которые будут нам нужны для работы:
```
apt-get update

apt-get install apache2 php8.2 apache2-mod_php8.2 mariadb-server php8.2-opcache php8.2-curl php8.2-gd php8.2-intl php8.2-mysqli php8.2-xml php8.2-xmlrpc php8.2-ldap php8.2-zip php8.2-soap php8.2-mbstring php8.2-json php8.2-xmlreader php8.2-fileinfo php8.2-sodium
```

Включаем службы httpd2 и mysqld для дальнейшей работы с ними следующей командой:

```
systemctl enable –now httpd2 mysqld
```

Теперь настроим безопасный доступ к нашей будущей базе данных с помощью команды: `mysql_secure_installation`
- Прожимаем просто enter, т.к. сейчас root без пароля: `Enter`
- Прожимаем y для задания пароля: `Y` 
- Задаем пароль к нашему root, желательно стандартный: `123qweR%`
- Далее нажимаем на всё y, как на скриншоте `Y`

![image](https://github.com/user-attachments/assets/b640b051-00ab-442d-8ea2-9ee6bf4cc4f7)

Теперь заходим в СУБД для создания и настройки базы данных:
- mariadb -u root -p
- CREATE DATABASE moodledb;
- CREATE USER moodle IDENTIFIED BY ‘P@ssw0rd’;
- GRANT ALL PRIVILEGES ON moodledb.* TO moodle;
- FLUSH PRIVILEGES;

![image](https://github.com/user-attachments/assets/0d058122-9020-49a3-a895-101060238d54)


Теперь скачаем сам мудл стабильной версии:
```
curl -L https://github.com/moodle/moodle/archive/refs/tags/v4.5.0.zip > /root/moodle.zip
```

![image](https://github.com/user-attachments/assets/2e7f0c19-cc57-4bb4-a85e-75d9d8bd26e3)

Разархивируем его в /var/www/html/ для дальнейшей настройки:
```
unzip /root/moodle.zip -d /var/www/html
mv /var/www/html/moodle-4.5.0/* /var/www/html/
ls /var/www/html
```

Создадим новый каталог moodledata, там будут храниться данные и изменим владельца на каталогах html и moodledata:
```
mkdir /var/www/moodledata
chown apache2:apache2 /var/www/html
chown apache2:apache2 /var/www/moodledata
```

Поменяем значение параметра max_input_vars в файле php.ini:
```
mcedit /etc/php/8.2/apache2-mod_php/php.ini
```
Жмём F7 для поиска нужной нам строки и пишем туда:
```
max_input_vars
```

![image](https://github.com/user-attachments/assets/c35e3e86-e661-48fc-b8bb-348b2571e61d)

Раскомментируем и пишем новое значение: `max_input_vars = 5000`

Удаляем стандартную страницу apache:
```
cd /var/www/html
ls
rm index.html
```
Перезапускаем службу httpd2:
```
systemctl restart httpd2
```
Теперь подключаемся с клиента HQ-CLI и начинаем настройку:
```
http://192.168.1.2/install.php
```
![image](https://github.com/user-attachments/assets/a775d4e9-cb07-4021-bf9a-33cd531bcb79)

Выбираем MariaDB в качестве драйвера базы данных:
![image](https://github.com/user-attachments/assets/1198a998-79b3-4879-afaf-8598b10b9e72)

- Введём нужные данные в следующие строки:
- Название базы данных: `moodledb`
- Пользователь базы данных: `moodle`
- Пароль: `P@ssw0rd`

![image](https://github.com/user-attachments/assets/e8d42f2e-5c9c-4d6a-a034-009fc8556992)

Далее заполняем обязательные поля для создания основного администратора:
- Логин: `admin`
- Новый пароль: `P@ssw0rd`
- Имя: `Администратор` (или другое)
- Фамилия: `Пользователь` (или другое)
- Адрес электронной почты: `test.test@mail.ru` (или другое)
- И нажимаем `Обновить профиль`

![image](https://github.com/user-attachments/assets/802a4878-5b4a-4fde-8c19-ab3d0bd3b66c)

Теперь заполним ещё некоторые строки на следующем шаге:
- Полное название сайта: `11` (согласно рабочему месту)
- Краткое название сайта: `moodle` (или другое)
- Настройки местоположения: `Азия/Барнаул` (согласно региону)
- Контакты службы поддержки: `test.test@mail.ru` (или другое)
- И жмём `Сохранить изменения` в конце страницы:

![image](https://github.com/user-attachments/assets/d8075191-8449-4105-a929-60d1ecd4397d)

И после всего нас встречает рабочий сайт moodle, смотрим, что все наши указанные параметры отображаются:

![image](https://github.com/user-attachments/assets/ee034847-324c-41df-b3bc-9a7857c9a5ed)

</details>

### <p align="center">модуль 2 задание 8</p>

<details><summary>веб-сервер nginx как обратный прокси-сервер на HQ-RTR</summary>

Поменяем значение wwwroot в конфигурации moodle на HQ-SRV: `$CFG->wwwroot        = ‘http://moodle.au-team.irpo’;`

```
mcedit /var/www/html/config.php
```

![image](https://github.com/user-attachments/assets/05b68b37-2500-4fe3-95aa-fd6841587213)

Устанавливаем пакет nginx на HQ-RTR для дальнейшей настройки:
```
apt install nginx
```

Создаём новый конфигурационный файл proxy:
```
mcedit /etc/nginx/sites-available/proxy
```

И заполняем его следующими строками:
```
server {
listen 80;
server_name moodle.au-team.irpo;
location / {
proxy_pass http://192.168.1.2:80;
proxy_set_header Host $host;
                     proxy_set_header X-Real-IP  $remote_addr;
                     proxy_set_header X-Forwarded-For $remote_addr;
             }
}
server {
listen 80;
server_name wiki.au-team.irpo;
location / {
proxy_pass http://192.168.4.2:8080;
proxy_set_header Host $host;
                     proxy_set_header X-Real-IP  $remote_addr;
                     proxy_set_header X-Forwarded-For $remote_addr;
             }
}
```

![image](https://github.com/user-attachments/assets/f5fc89b3-768d-4b34-b756-fa03578e10cb)

Удаляем конфигурацию `default`, которую создал nginx, потом включаем созданную нами ранее `proxy`, путём создания символической ссылки,  а затем перезапускаем службу nginx:
```
rm -rf /etc/nginx/sites-available/default
rm -rf /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/proxy /etc/nginx/sites-enabled
ls -la /etc/nginx/sites-enabled
systemctl restart nginx
```

Проверим работу нашего обратного прокси и зайдем на наши поднятые ранее сайты moodle и wiki с клиента HQ-CLI.
![image](https://github.com/user-attachments/assets/3b69676f-3cb3-40e1-974f-a56c1fa76dac)
![image](https://github.com/user-attachments/assets/1a24d3cf-0352-4b24-bbe5-cd402feebdc2)


</details>

### <p align="center">модуль 2 задание 9</p>

<details><summary>установить Яндекс Браузере на HQ-CLI</summary>

Установим Яндекс Браузер на HQ-CLI через терминал командами:
```
apt-get update
apt-get install yandex-browser-stable
```

Должен быть по пути: `Пуск → Интернет → Yandex Browser`

![image](https://github.com/user-attachments/assets/67a6f00e-a76b-4370-9a1e-90998362e51e)

</details>
