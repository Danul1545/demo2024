# Установка Python 3.12 на ALT Linux

Для начала установите отдельную виртуальную машину на свой стенд и подключите ее к интернету

Теперь проверим установлена ли утилита для скачивания файлов из интернета

```
wget
```

Если выдало вот такое:

```
-bash: wget: команда не найдена
```

Значит нужно установить wget

```
apt-get install wget
```

После установки wget переходим в папку `/tmp/`
И скачиваем файл с сайта https://www.python.org/

```
cd /tmp/
wget https://www.python.org/ftp/python/3.12.1/Python-3.12.1.tgz
```
![pic1](https://github.com/Danul1545/demo2024/assets/148867600/f576f598-ba1b-44a6-9cac-4bd94c2e01db)

Разархивируем полученный файл

```
tar zxvf Python-3.12.1.tgz
```

Скопируем разархивированную папку

```
cp -r Python-3.12.1 /usr/local/bin
cd /usr/local/bin/Python-3.12.1/
```

Установим пакеты для компиляции

```
apt-get install zlib-devel libssl-devel libsqlite3-devel libffi-devel gcc
apt-get install pip
```

Добавим немного параметров для компиляции

```
./configure --prefix=/usr/local --with-ensurepip=install
```

Начинаем компилировать

```
make
make install
```

Теперь почистим за собой весь мусор

```
make clean
cd /
rm -rf /usr/local/bin/Python-3.12.1
```

Проверяем, что Python3.12 установился

```
python3.12
```
![pic2](https://github.com/Danul1545/demo2024/assets/148867600/27d38833-f148-4435-ba4b-7e71f08d72b8)

# Создание виртуального окружения Python

В домашнем каталоге пользователя создаем папку `ansible`
У меня пользователь `student`

```
mkdir /home/student/ansible
cd /home/student/ansible
```

Создаем виртуальное окружение

```
python3.12 -m venv .env
```

Пробуем активироваль его

```
source .env/bin/activate
```

Должно получиться так
![pic1](https://github.com/Danul1545/demo2024/assets/148867600/7f7019d5-23a8-4129-a1d7-b0a128777937)

# Подключение VSCode по SSH к серверу

Проверяем что в `VSCode` установленно расширение `Remote explorer` 

Если нет расширения, то устанавливаем

![pic1](https://github.com/Danul1545/demo2024/assets/148867600/9051751a-3ace-431b-bcdb-1d5bc9649e9d)

Жмакаем на плюсик и добавляем свою виртуалку в список SSH подключений

![pic2](https://github.com/Danul1545/demo2024/assets/148867600/44255c6a-c990-43b7-a6c4-fd617dbd4843)

![pic3](https://github.com/Danul1545/demo2024/assets/148867600/5f6a4b80-d9b2-4396-9934-67a53826492c)

Обнавляем список подключений

![pic4](https://github.com/Danul1545/demo2024/assets/148867600/7cee86d9-4f5a-44d7-8f39-4c9026324cc6)

Добавляем папку которая находится на удаленном устройстве

![pic5](https://github.com/Danul1545/demo2024/assets/148867600/c631b073-6341-460b-87c4-5c9a650c60ce)

![pic6](https://github.com/Danul1545/demo2024/assets/148867600/ae985544-14f4-4046-b504-21c230543b6f)

Ждем, когда на виртуалку установится VSCode

Открываем папку `ansible`, если папки не существует, то создаем ее

![pic7](https://github.com/Danul1545/demo2024/assets/148867600/1a51d8e8-451b-421b-88be-cb9d171ca44e)

# Установка ansible

В `VSCode` откройте терминал.
Проверьте что вы находитесь в папке `ansible` и у вас активированно виртуальное оркужение

![pic1](https://github.com/Danul1545/demo2024/assets/148867600/c1622b12-4efc-4983-9198-35c74ad9ae44)

`Ansible` это пакет `python`. Пакет можно установить при помощи утилиты командной строки `pip`

```
pip install ansible
```

### Начальная настройка ansible

В папке с витуальным окружением создайте файл `ansible.cfg` со следующим содержимом

```
[defaults]

inventory = ./hosts.yaml
ask_pass = False
host_key_checking = False
```

В тойже папке создайте файл `hosts.yaml` и добавьте в него свой ESXi сервер и виртуальные машины

```
VMs:
  hosts:
    ISP:
      ansible_ssh_host: 10.15.15.2
      vars:
        interfaces: [
          { ifname: 'ens161', ifaddr: '2.2.2.1', mask: '/30'},
          { ifname: 'ens192', ifaddr: '10.12.66.2', mask: '/24'},
          { ifname: 'ens224', ifaddr: '1.1.1.1', mask: '/30'},
          { ifname: 'ens256', ifaddr: '94.41.92.1', mask: '/24'}
        ]
    HQ-R:
      ansible_ssh_host: 10.15.15.3
      vars:
        interfaces: [
          { ifname: 'ens192', ifaddr: '192.168.0.1', mask: '/25'},
          { ifname: 'ens224', ifaddr: '1.1.1.2', mask: '/30', gw: '1.1.1.1'}
        ]
    BR-R:
      ansible_ssh_host: 10.15.15.4
      vars:
        interfaces: [
          { ifname: 'ens192', ifaddr: '2.2.2.2', mask: '/30', gw: '2.2.2.1'},
          { ifname: 'ens224', ifaddr: '192.168.0.129', mask: '/27'}
        ]
    HQ-SRV:
      ansible_ssh_host: 10.15.15.5
      vars:
        interfaces: [
          { ifname: 'ens192', ifaddr: '192.168.0.2', mask: '/25', gw: '192.168.0.1'}
        ]
        
    BR-SRV:
      ansible_ssh_host: 10.15.15.6
      vars:
        interfaces: [
          { ifname: 'ens192', ifaddr: '192.168.0.130', mask: '/27', gw: '192.168.0.129'}
        ] 
  vars:
    ansible_connection: ssh 
    ansible_user: root
    ansible_password: P@ssw0rd

```

### Подготовка виртуальных машин к управлению через SSH

На всех виртуальных машинах нужно установить `open-vm-tools-desktop` и `sshpass`

```
apt-get install open-vm-tools-desktop
apt-get install sshpass
```

На вашем `ESXi` хосте создайте дополнительну `portgroup` с названием `ansible`

![pic1](https://github.com/Danul1545/demo2024/assets/148867600/02b5eb34-2ba3-47af-a92a-d5159febbb7d)

На виртуальных машинах добавьте еще один интерфейс и подключите его в созданой `portgroup`

![pic2](https://github.com/Danul1545/demo2024/assets/148867600/d931cffe-731b-4ca3-8667-d6ba3ae03331)

Задайте IP созданным интерфейсам из не существующей подсети. Например из `10.15.15.0/24`

Настройте возможность подключения по `ssh` к виртуальным машинам под пользователем `root`

Для этого нужно отредактировать файл `/etc/openssh/sshd_config`

Заменяем `#PermitRootLogin without_password` на `PermitRootLogin yes`

```
sed -i -e 's/#PermitRootLogin without_password/PermitRootLogin yes/g' /etc/openssh/sshd_config
```

Перезагружаем ssh

```
/etc/init.d/sshd restart
```

Теперь проверяем что с виртуальной машины `ansible` можем подключиться ко всем другим вируальным машинам
`ssh root@IP виртуальной машины`

например:
```
ssh root@10.15.15.2
```

### Изменение hostname на виртуальной машине через ESXi

Для того, чтобы данный способ работал нужно на виртуалку установить `open-vm-tools-desktop`

В папке с виртуальным окружение создайте `playbook` с название `changeHostname.yaml`

```
---

- name: show version
  hosts: esxi
  connection: local
  tasks:
    - name: Change hostname of guest machine
      community.vmware.vmware_vm_shell:
        validate_certs: no
        hostname: "10.12.66.1"  # IP вашего ESXi
        username: "root" # Имя пользователя для ESXi
        password: "P@ssw0rd" # Пароль пользователя для ESXi
        vm_id: "DEMO-ISP - 5001" # Имя виртуальной машины
        vm_username: root # Имя пользователя для виртуальной машины
        vm_password: P@ssw0rd # Пароль пользователя для виртуальной машины
        vm_shell: "/usr/bin/hostnamectl" # Полный путь до команды
        vm_shell_args: "set-hostname ISP-22" # Аргументы команды
      delegate_to: localhost
```

Запускаем скрипт

```
ansible-playbook /home/ansible/playbook/changeHostname.yaml 
```

### Изменение hostname через SSH

Создаем `play-book` с названием `changeHostnameViaSSH.yaml`

```
---

- name: Init settings
  hosts: VMs
  tags:
    - skip_ansible_lint
  tasks:
    - name: Set hostname
      ansible.builtin.hostname:
        name: "{{ inventory_hostname }}"
```

### Назначаем IP на интерфейсы

```
---

    - name: Create folder for interfaces
      ansible.builtin.file:
        path: "/etc/net/ifaces/{{ item['ifname'] }}"
        state: directory
        mode: '0777'
      loop: "{{ vars.vars.interfaces }}"
    - name: Creating a file with IP address
      ansible.builtin.copy:
        dest: "/etc/net/ifaces/{{ item['ifname'] }}/ipv4address"
        content: "{{ item['ifaddr']+item['mask'] }}"
        mode: '0777'
      loop: "{{ vars.vars.interfaces }}"
    - name: Create file options fot interfaces
      ansible.builtin.copy:
        dest: "/etc/net/ifaces/{{ item['ifname'] }}/options"
        content: "TYPE=eth
        DISABLE=no
        NM_CONTROLLED=no
        BOOTPROTO=static
        CONFIG_IPV4=yes"
        mode: '0777'
      loop: "{{ vars.vars.interfaces }}"
    - name: Enable routing
      shell:
        cmd: "sed -i -e 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/g' /etc/net/sysctl.conf"
    - name: Setting default gateway on Servers
      when: item['gw'] is defined
      ansible.builtin.copy:
        dest: "/etc/net/ifaces/{{ item['ifname'] }}/ipv4route"
        content: "default via {{ item['gw'] }}"
        mode: '0777'
      loop: "{{ vars.vars.interfaces }}"
    - name: Restart network
      ansible.builtin.systemd:
        name: network
        state: restarted

```

### Создание модулей проекта

Сейчас все наш сценарий хранится в одном файле. Со временем в файл вы будите добавлять новые задачи. Что затруднит ориентацию в этотм сценарии. Поэтому разделим весь сценарий на части

Для этого в корневой папке вашего проекта создадим папку `tasks`

В этой папке создадим следующие файлы:

```
setHostname.yaml
enableRouting.yaml
setIpAddress.yaml
setDefaultGateway.yaml
restartNetwork.yaml
```

Теперь наполним созданные файлы следующим содержимом

`setHostname.yaml`

```
---

- name: Set hostname
  ansible.builtin.hostname:
    name: "{{ inventory_hostname }}"
```

`enableRouting.yaml`

```
---

- name: Enable routing
  shell:
    cmd: "sed -i -e 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/g' /etc/net/sysctl.conf"

```

`setIpAddress.yaml`

```
---

- name: Creating a folder for interfaces
  ansible.builtin.file:
    path: "/etc/net/ifaces/{{ item['ifname'] }}"
    state: directory
    mode: '0777'
  loop: "{{ vars.vars.interfaces }}"
- name: Creating a file with IP address
  ansible.builtin.copy:
    dest: "/etc/net/ifaces/{{ item['ifname'] }}/ipv4address"
    content: "{{ item['ifaddr']+item['mask'] }}"
    mode: '0777'
  loop: "{{ vars.vars.interfaces }}"
- name: Creating a file options for interfaces
  ansible.builtin.copy:
    dest: "/etc/net/ifaces/{{ item['ifname'] }}/options"
    content: "TYPE=eth
    DISABLE=no
    NM_CONTROLLED=no
    BOOTPROTO=static
    CONFIG_IPV4=yes"
    mode: '0777'
  loop: "{{ vars.vars.interfaces }}"
```

`setDefaultGateway.yaml`

```
---

- name: Setting default gateway on Servers
  when: item['gw'] is defined
  ansible.builtin.copy:
    dest: "/etc/net/ifaces/{{ item['ifname'] }}/ipv4route"
    content: "default via {{ item['gw'] }}"
    mode: '0777'
  loop: "{{ vars.vars.interfaces }}"
```

`restartNetwork.yaml`

```
---

- name: Restart network
  ansible.builtin.systemd:
    name: network
    state: restarted
```

## Теперь объединим все эти задачи в один `playbook`

в папке `/playbook` создадим файл `initSettings.yaml`

```
---

- name: Init settings
  hosts: VMs
  tasks:
    - include_tasks: tasks/setHostname.yaml
    - include_tasks: tasks/enableRouting.yaml
    - include_tasks: tasks/setIpAddress.yaml
    - include_tasks: tasks/setDefaultGateway.yaml
    - include_tasks: tasks/restartNetwork.yaml
```

тем самым мы сделали такой же сценарий, но теперь проще его воспринимать и масштабировать

### Создание туннельных интерфейсов.

Перед написанием сценария нужно добавить тунельные интерфейсы в инвентарный файл.

Но в дальнешем мы сталкнемся с проблемой как программой поймет что это туннельный интерфейс ведь настройка обычного интерфейса и туннельного отличается.

Поэтому предварительно у `каждого` интерфейса в инвентарном файле укажем его тип:

```
{ ifname: 'ens192', type: 'eth', ifaddr: '2.2.2.2', mask: '/30', gw: '2.2.2.1'}
```

После этого добавим на `HQ-R` и `BR-R` информацию о туннельных интерфейсах. Будте внимательный с названием интерфеса к которому привязываем туннель.

```
HQ-R:
    ansible_ssh_host: 10.15.15.3
    vars:
    interfaces: [
        { ifname: 'ens192', type: 'eth', ifaddr: '192.168.0.1', mask: '/25'},
        { ifname: 'ens224', type: 'eth', ifaddr: '1.1.1.2', mask: '/30', gw: '1.1.1.1'},
        { ifname: 'tun1', type: 'iptun', ifaddr: '172.16.0.1', mask: '/30', tunlocal: '1.1.1.2', tunremote: '2.2.2.2', interface: 'ens224'}
    ]
BR-R:
    ansible_ssh_host: 10.15.15.4
    vars:
    interfaces: [
        { ifname: 'ens192', type: 'eth', ifaddr: '2.2.2.2', mask: '/30', gw: '2.2.2.1'},
        { ifname: 'ens224', type: 'eth', ifaddr: '192.168.0.129', mask: '/27'},
        { ifname: 'tun1', type: 'iptun', ifaddr: '172.16.0.2', mask: '/30', tunlocal: '2.2.2.2', tunremote: '1.1.1.2', interface: 'ens192'}
    ]
```

## Написание сценария

Внесем изменения в файл `setIpAddress.yaml` таким образом чтобы задача по созданию файла `options` выполнялась по условию

```
- name: Creating a file options for  ether interfaces
  when: item['type'] == 'eth' # Выполнится только для интерфейсов с типом eth
  ansible.builtin.copy:
    dest: "/etc/net/ifaces/{{ item['ifname'] }}/options"
    content: "TYPE=eth
    DISABLE=no
    NM_CONTROLLED=no
    BOOTPROTO=static
    CONFIG_IPV4=yes"
    mode: '0777'
  loop: "{{ vars.vars.interfaces }}"
- name: Creating a file options for tunnel interfaces
  when: item['type'] == 'iptun' # Выполнится для интерфесов с типом iptun
  ansible.builtin.copy:
    dest: "/etc/net/ifaces/{{ item['ifname'] }}/options"
    content: "TYPE=iptun
      TUNTYPE=gre
      TUNLOCAL={{ item['tunlocal'] }}
      TUNREMOTE={{ item['tunremote'] }}
      TUNOPTIONS='ttl 64'
      HOST={{ item['interface'] }}"
    mode: '0777'
  loop: "{{ vars.vars.interfaces }}"
```


## Настройка пользователей.
#### Файл `Admin_CLI.yaml`
```
- name: create users 
  hosts: CLI
  tasks: 
    - name: addmin
      shell: adduser admin

    - name: root priv
      shell: usermod -aG root admin

    - name: passi
      shell: passwd admin
      shell: echo "P@ssw0rd" | passwd --stdin admin   
```
#### Файл `Admin_HQ.yaml`
```
---

- name: Create local users
  hosts: HQ-SRV, HQ-R
  tasks: 
    - name: addmin
      shell: useradd admin

    - name: root priv
      shell: usermod -aG root admin

    - name: Pass
      shell: passwd admin
      shell: echo "P@ssw0rd" | passwd --stdin admin   
```
#### Файл `Branchadmin.yaml`
```
---

- name: Create local users
  hosts: BR-R, BR-SRV
  tasks: 
    - name: addmin
      shell: useradd branchadmin

    - name: root priv
      shell: usermod -aG root branchadmin

    - name: Pass
      shell: passwd branchadmin
      shell: echo "P@ssw0rd" | passwd --stdin abranchadmin
```
#### Файл `Networkadmin.yaml`
```
---

- name: Create local users
  hosts: BR-R, BR-SRV, HQ-R
  tasks: 
    - name: addmin
      shell: useradd networkadmin

    - name: root priv
      shell: usermod -aG root networkadmin

    - name: Pass
      shell: passwd networkadmin
      shell: echo "P@ssw0rd" | passwd --stdin networkadmin
```

#### Объеденение фалов.

```
---

- name: Settings
  hosts: VMs
  tasks:
    - include_tasks: Addusers/Admin_HQ.yaml
    - include_tasks: Addusers/Branchadmin.yaml
    - include_tasks: Addusers/Networkadmin.yaml
    - include_tasks: Addusers/Admin_CLI
    
```
