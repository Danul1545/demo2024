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

<p align="center">
  <img src="./pic1.png">
</p>

Жмакаем на плюсик и добавляем свою виртуалку в список SSH подключений

<p align="center">
  <img src="./pic2.png">
</p>

<p align="center">
  <img src="./pic3.png">
</p>

Обнавляем список подключений

<p align="center">
  <img src="./pic4.png">
</p>

Добавляем папку которая находится на удаленном устройстве

<p align="center">
  <img src="./pic5.png">
</p>

<p align="center">
  <img src="./pic6.png">
</p>

Ждем, когда на виртуалку установится VSCode

Открываем папку `ansible`, если папки не существует, то создаем ее

<p align="center">
  <img src="./pic7.png">
</p>
