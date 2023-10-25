# demo2024

# №1
# Описание задания.
Выполните базовую настройку всех устройств:

1) Соберите топологию согласно рисунку. Все устройства работают на OC Linux - Debian
2) Присвоить имена в соответствии с топологией
3) Рассчитайте IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактируйте таблицу.
4) Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.
5) Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.
  
# Топология №1.

![Снимок экрана (1)](https://github.com/Danul1545/demo2024/assets/148867600/eb66723d-fa8d-4208-a537-32cbba510b32)

# Таблица №1 (разбита на подсети).

| Имя устройства | Итерфейс |  IPv4/IPv6   | Маска/Префикс |       Шлюз       |
| -------------- | -------- | ------------ | ------------- |    ----------    |
|                |  Gi1     | 10.12.25.10  | /24           | 10.12.25.254     |
| ISP            |  Gi2     | 192.168.0.1  | /27           |                  |
|                |  Gi3     | 192.168.0.35 | /27           |                  |
| HQ-R           |  Gi1     | 192.168.0.81 | /25           |                  |
|                |  Gi2     | 192.168.0.2  | /27           | 192.168.0.1      |
| BR-R           |  Gi1     | 192.168.0.65 | /26           |                  |
|                |  Gi2     | 192.168.0.34 | /27           | 192.168.0.35     |
| HQ-SRV         |  Gi1     | 192.168.0.82 | /25           | 192.168.0.81     |
| BR-SRV         |  Gi1     | 192.168.0.66 | /26           | 192.168.0.65     |

# 1. Настройка интерфесов.

Просмотр существующих интерфейсов.
```
ip a
```
Заходим на файл  конфигурации интерфейсов.
```
nano /etc/network/interfaces
```
Назначаем IP адреса в соотвествии с таблицей.
```
auto ens192
iface ens192 inet static
address 10.12.25.10
netmask 255.255.255.0
gateway 10.12.25.254

auto ens224
iface ens224 inet static
address 192.168.0.1
netmask 255.255.255.224

auto ens256
iface ens256 inet static
address 192.168.0.35
netmask 255.255.255.224
```
Сохраняем комбинацией клавишь.
```
Ctrl+S
```
Выходим с комбинацией клавишь.
```
Ctrl+X
```
Перезагружаем сеть.
```
systemctl restar networking
```
Делаем тоже самое на других виртуальных машинах.
# №1.2
