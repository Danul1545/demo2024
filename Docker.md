# Создание и настройка Docker.
## Задание 1

скачиваем докер
``` 
apt-get install docker-engine -y
```

Запускаем Docker:
```
systemctl enable --now docker
```

Скачиваем контейнер Hello-World:
```
docker pull hello-world
```

Смотрим список контейнеров:
```
docker images
```

Запускаем контейнер Hello-World:
```
docker run hello-world
```

Останавливаем контейнер:
```
docker rm -v *ID контейнера* 
```

Удаляем контейнер:
```
docker image rm -f *имя* или *ID контейнера*
```
## Задание 2

Для добовления docker с nginx нужно:

Скачать сам nginx
```
apt-get update
apt-get install nginx -y
```

Запустить его
```
systemctl enabled --now nginx
```
Зваустить его в docker
```
docker pull nginx
```

Вписать эту команду
```
docker run --rm -d --name nginx -v /data/app:/var/www/html -p 0.0.0.0:80:80 nginx
```

И проверить работу
```
docker ps
```


Сетка должна выглядить так

![image](https://github.com/Danul1545/demo2024/assets/148867600/ccab2d57-f9aa-434e-80d1-048e8187e822)



Вписываем в строку браузера ip выходдящий в интернет, если работает то должно выйти такое окно

![image](https://github.com/Danul1545/demo2024/assets/148867600/f5726f6e-d0f1-485c-99c3-fed742e16aeb)

## Задание 3

Для того чтобы изметить имя на сервере nginx нужно:

Запустьть интерактивную оболочку в контейнере Docker
```
docker exec -it nginx /bin/bash
```

Скачать редактор файлов `nano` или `vim`
```
apt-get update
apt-get install -y nano
```

Открываем файл 
```
nano /usr/share/nginx/html/index.html
```

В файле меняем строчку `<h1>Hi Nginx</h1>` вписывая `<h1>своё имя</h1>`

Сохраняем файл и проверям по его ip.

## Задание 4

Для начала обратимся к Python для создания нового окружения, которое мы назовём `docker`
```
pyton3 -m venv docker
```
Открываем его
```
source docker/bin/activate
```
Далее мы создаём новую директорию, называем её по своему усмотрению и заходим в неё
```
mkdir /fastapi
cd /fastapi
```
В нём открываем файл `requirements.txt`, куда мы запишем наши зависимости
```
nano requirements.txt
```
Заполняем его следующим содержимым
```
fastapi>=0.68.0,<0.69.0
pydantic>=1.8.0,<2.0.0
uvicorn>=0.15.0,<0.16.0
```
И устанавливаем зависимости при помощи `pip`
```
pip install -r requirements.txt
```
В ней создаём директорию `app`, переходим в неё и создаём 2 файла: `__init__.py` и `main.py`. Файл `__init__.py` оставляем пустым
```
mkdir /fastapi/app
cd /fastapi/app
nano /fastapi/app/__init__.py (сохраняем и выходим сочетаниями клавиш ctrl+s, ctrl+x)
nano /fastapi/app/main.py
```
Заполняем файл `main.py` следующим образом
```
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Union[str, None] = None):
    return {"item_id": item_id, "q": q}
```
Далее в директории `/fastapi` создаём файл Dockerfile и заполняем его `nano /fastapi/Dockerfile`
```
#
FROM python:3.9
#
WORKDIR /code
#
COPY ./requirements.txt /code/requirements.txt
#
RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt 
#
COPY ./app /code/app 
#
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "80"]
```
Не выходя из директории `/fastapi`, создаём в ней образ приложения FastAPI
```
docker build -t fastapi /fastapi
```
После прошедшей загрузки запускаем сам контейнер
```
docker run -d --name fastapi -p 80:80 fastapi
```
Для проверки прописать в строке браузера `свойip/docks`

## Задание 5

Запускаем контейнер PostgreSQL со всеми нужными нам настройками
```
docker run --name college-pg -p 5433:5433 -e POSTGRES_USER=Student -e POSTGRES_PASSWORD=StudentPass postgres:16.3
```

Где:
```
POSTGRES_USER - имя нашего пользователя
POSTGRES_PASSWORD - пароль нашего пользователя
POSTGRES_DB - имя нашей базы данных
```
Заходим в __окружение docker__
```
source docker/bin/activate
```
Создаём в нём директорию __compose-postgres__ и переходим в неё
```
mkdir compose-postgres
cd compose-postgres
```
Создаём в ней файл __docker-compose.yml__ (или compose.yml)
```
nano docker-compose.yml
```
И заполняем его следующим:
```
version: '3.9'

services:
  postgres:
    container_name: postgres_container
    image: postgres:16.3
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-postgresdb}
      POSTGRES_USER: ${POSTGRES_USER:-Student}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-StudentPass}
      PGDATA: /data/postgres
    volumes:
       - postgres:/data/postgres
    ports:
      - "5432:5432"
    networks:
      - postgres
    restart: unless-stopped
  
  pgadmin:
    container_name: pgadmin_container
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-pgadmin4@pgadmin.org}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD:-admin}
      PGADMIN_CONFIG_SERVER_MODE: 'False'
    volumes:
       - pgadmin:/var/lib/pgadmin

    ports:
      - "${PGADMIN_PORT:-5050}:80"
    networks:
      - postgres
    restart: unless-stopped

networks:
  postgres:
    driver: bridge

volumes:
    postgres:
    pgadmin:
```
Устанавливаем программу __pgAdmin__ и проверяем доступность нашей базы данных


