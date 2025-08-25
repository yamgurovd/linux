# Инструкция для запуска Docker с БД, Redis и Backend

----

## Краткое содержание инструкции

инструкция описывает процесс запуска и настройки нескольких сервисов (Postgres, Redis, Backend, Celery) в
Docker-контейнерах с использованием пользовательской Docker-сети для их взаимодействия.

### Краткий обзор происходящего

1. Создание Docker-сети
   Команда docker network create myNetwork создаёт отдельную мостовую сеть Docker, позволяющую контейнерам общаться друг
   с другом по именам и IP внутри этой сети.

2. Запуск Postgres
   Запускается контейнер с базой данных Postgres, с пробросом порта 6432 на хост и подключением к созданной сети
   myNetwork. Данные сохраняются в Docker volume pg-booking-data для постоянства данных. Параметры пользователя, пароля
   и имени базы задаются через переменные окружения.

3. Настройка подключения к Postgres
   Пример файла .env содержит параметры подключения к базе данных и Redis, а также настройки JWT для аутентификации в
   backend.

4. Запуск Redis
   Контейнер Redis запускается с пробросом порта 7379 и подключением к той же сети myNetwork. Порт Redis внутри
   контейнера — 6379, но на хосте проброшен как 7379 (возможно, чтобы избежать конфликта).

5. Запуск Backend
   Запускается контейнер с backend-приложением, проброшен порт 7777 на 8000 внутри контейнера, подключён к сети
   myNetwork.

6. Запуск Celery и Celery-beat
   Запускаются контейнеры для фоновых задач (worker) и планировщика задач (beat) Celery, используя тот же образ backend
   и сеть myNetwork.

7. Сборка образа backend
   Команда docker build -t booking_image . создаёт образ backend-приложения из текущей директории.

### Что происходит в целом

В итоге, инструкция описывает создание изолированной сети Docker и запуск в ней базы данных Postgres, кеша Redis,
backend-приложения и фоновых задач Celery, обеспечивая их взаимодействие и сохранность данных.

### Команды которые используется

| Команда                                                                                                                                                                                                           | Пояснение                                                                                                 |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| `docker network create myNetwork`                                                                                                                                                                                 | Создание пользовательской Docker-сети для взаимодействия контейнеров                                      |
| `docker run --name booking_db_1 -p 6432:5432 -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=password -e POSTGRES_DB=booking --network myNetwork --volume pg-booking-data:/var/lib/postgresql/data -d postgres:16` | Запуск контейнера PostgreSQL с пробросом порта, переменными окружения, томом данных и подключением к сети |
| `docker run --name booking_redis -p 7379:6379 --network=myNetwork -d redis:7.4`                                                                                                                                   | Запуск контейнера Redis с пробросом порта и подключением к сети                                           |
| `docker build -t booking_image .`                                                                                                                                                                                 | Сборка Docker-образа для backend-приложения из текущей директории                                         |
| `docker run --name booking_back -p 7777:8000 --network=myNetwork booking_image`                                                                                                                                   | Запуск контейнера с backend-приложением                                                                   |
| `docker run --name booking_celery_worker --network=myNetwork booking_image celery --app=src.tasks.celery_app:celery_instance worker -l INFO`                                                                      | Запуск контейнера с Celery worker для фоновых задач                                                       |
| `docker run --name booking_celery_beat --network=myNetwork booking_image celery --app=src.tasks.celery_app:celery_instance worker -l INFO -B`                                                                     | Запуск контейнера с Celery beat (планировщиком задач)                                                     | 

# Подробная инструкция

---

### Создание сети Docker для взаимодействия Postgres, Redis и Backend

```shell
docker network create myNetwork
```

![Dbeaver](/course_helpers/9%20Docker%20и%20деплой%20проекта/dcoker_network.png)

### Запуск Postgres в Docker-контейнере

```shell
docker run --name booking_db_1 \
    -p 6432:5432 \
    -e POSTGRES_USER=postgres \
    -e POSTGRES_PASSWORD=password \
    -e POSTGRES_DB=booking \
    --network myNetwork \
    --volume pg-booking-data:/var/lib/postgresql/data \
    -d postgres:16
```

#### Настройки подключения к базе данных (пример .env)

```text
MODE=LOCAL

DB_HOST=booking_db_1
DB_PORT=6432       # Здесь изменить при необходимости
DB_USER=postgres   # Здесь изменить при необходимости
DB_PASS=password   # Здесь изменить при необходимости
DB_NAME=booking

REDIS_HOST=booking_redis
REDIS_PORT=7379

JWT_SECRET_KEY=09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

![Dbeaver](/course_helpers/9%20Docker%20и%20деплой%20проекта/docker_DB1.png)

### Запуск Redis в Docker-контейнере

```shell
docker run --name booking_redis \
    -p 7379:6379 \
    --network=myNetwork \
    -d redis:7.4
```

#### Настройка Redis в локальном .env файле

```text
MODE=LOCAL

DB_HOST=booking_db
DB_PORT=6432
DB_USER=postgres
DB_PASS=password 
DB_NAME=booking

REDIS_HOST=booking_redis # Здесь изменить при необходимости 
REDIS_PORT=7379          # Здесь изменить при необходимости

JWT_SECRET_KEY=09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

![Dbeaver](/course_helpers/9%20Docker%20и%20деплой%20проекта/docker_redis.png)

### Запуск Backend в Docker-контейнере

```shell
docker run --name booking_back \
    -p 7777:8000 \
    --network=myNetwork \
    booking_image
```

### Запуск Celery в Docker-контейнере

```shell
docker run --name booking_celery_worker \
    --network=myNetwork \
    booking_image \
    celery --app=src.tasks.celery_app:celery_instance worker -l INFO
```

### Запуск Celery-beat в Docker-контейнере

```shell
docker run --name booking_celery_beat \
    --network=myNetwork \
    booking_image \
    celery --app=src.tasks.celery_app:celery_instance worker -l INFO -B
```

### Сборка образа Backend части приложения

```shell
docker build -t booking_image .
```

![Dbeaver](/course_helpers/9%20Docker%20и%20деплой%20проекта/docker_back.png)


----