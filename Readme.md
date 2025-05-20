# ALT - LINUX    

## Модуль 1 задание 1

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
### Всё тоже самое повторяем на втором роутере.

после настройки проверяем командой `ping`

![image](https://github.com/Danul1545/demo2024/assets/148867600/cf6fddea-e2d4-45c9-8f5c-1be4ebf6c637)

</details>

## Модуль 1 задание 2

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

## Модуль 1 задание 3

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

## Модуль 1 задание 4

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

## Модуль 1 задание 5

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

## Модуль 1 задание 6

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

## Модуль 1 задание 7

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

## Модуль 1 задание 8

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

## Модуль 1 задание 9

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

## Модуль 2

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


<details>
    <summary>Запустите сервис moodle на сервере HQ-SRV</summary>

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
Прожимаем просто enter, т.к. сейчас root без пароля: `Enter`
Прожимаем y для задания пароля: `Y` 
Задаем пароль к нашему root, желательно стандартный: `123qweR%`
Далее нажимаем на всё y, как на скриншоте `Y`

![image](https://github.com/user-attachments/assets/b640b051-00ab-442d-8ea2-9ee6bf4cc4f7)

Теперь заходим в СУБД для создания и настройки базы данных:
mariadb -u root -p
CREATE DATABASE moodledb;
CREATE USER moodle IDENTIFIED BY ‘P@ssw0rd’;
GRANT ALL PRIVILEGES ON moodledb.* TO moodle;
FLUSH PRIVILEGES;
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

Введём нужные данные в следующие строки:
Название базы данных: `moodledb`
Пользователь базы данных: `moodle`
Пароль: `P@ssw0rd`
![image](https://github.com/user-attachments/assets/e8d42f2e-5c9c-4d6a-a034-009fc8556992)

Далее заполняем обязательные поля для создания основного администратора:
Логин: `admin`
Новый пароль: `P@ssw0rd`
Имя: `Администратор` (или другое)
Фамилия: `Пользователь` (или другое)
Адрес электронной почты: `test.test@mail.ru` (или другое)
И нажимаем `Обновить профиль`
![image](https://github.com/user-attachments/assets/802a4878-5b4a-4fde-8c19-ab3d0bd3b66c)

Теперь заполним ещё некоторые строки на следующем шаге:
Полное название сайта: `11` (согласно рабочему месту)
Краткое название сайта: `moodle` (или другое)
Настройки местоположения: `Азия/Барнаул` (согласно региону)
Контакты службы поддержки: `test.test@mail.ru` (или другое)
И жмём `Сохранить изменения` в конце страницы:
![image](https://github.com/user-attachments/assets/d8075191-8449-4105-a929-60d1ecd4397d)

И после всего нас встречает рабочий сайт moodle, смотрим, что все наши указанные параметры отображаются:
![image](https://github.com/user-attachments/assets/ee034847-324c-41df-b3bc-9a7857c9a5ed)




</details>
