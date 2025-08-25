# Docker
## Основные команды
Образы
Создание образа
```shell
docker build -t <имя-образа> .
```
### Показать список всех образов
```shell
docker images
```

### Удалить образ
```shell
docker rmi <имя-образа>
```

## Контейнеры
### Создание контейнера
```shell
docker run <имя-образа>
```

#### Параметры:

```shell
-d - запуск в фоновом режиме
-p 1234:5432 - прокидывание портов
--name my-container - задание имени контейнера
-v /my/dir:/container/dir - прокидывание вольюмов
--network my-network> - запуск контейнера внутри Docker сети
```
### Показать список всех запущенных контейнеров
```shell
docker ps
```

### Показать список всех контейнеров
```shell
docker ps -a
```

### Удалить контейнер
```shell
docker rm <имя-контейнера>
```

### Удалить запущенный контейнер
```shell
docker rm -f <имя-контейнера>
```

### Docker сети
Создать Docker сеть
```shell
docker network create <имя-сети>
```

## Docker Compose
### Создание образов
```shell
docker compose build
```
### Поднятие всех контейнеров
```shell
docker compose up
```

### Поднятие всех контейнеров + сборка образов
```shell
docker compose up --build
```

### Поднятие всех контейнеров с кастомным названием файла
```shell
docker compose -f docker-compose-custom.yml up
```

## Команды для проекта
Создание Docker сети
```shell
docker network create myNetwork
```

База данных
```shell
docker run --name booking_db \
    -p 6432:5432 \
    -e POSTGRES_USER=abcde \
    -e POSTGRES_PASSWORD=abcde \
    -e POSTGRES_DB=booking \
    --network=myNetwork \
    --volume pg-booking-data:/var/lib/postgresql/data \
    -d postgres:16
```
### Redis
```shell
docker run --name booking_cache \
    -p 7379:6379 \
    --network=myNetwork \
    -d redis:7.4
Nginx
Без SSL (https)

docker run --name booking_nginx \
    --volume ./nginx.conf:/etc/nginx/nginx.conf \
    --network=myNetwork \
    --rm -p 80:80 nginx
```
### С SSL (https)

````shell
docker run --name booking_nginx \
    --volume ./nginx.conf:/etc/nginx/nginx.conf \
    --volume /etc/letsencrypt:/etc/letsencrypt \
    --volume /var/lib/letsencrypt:/var/lib/letsencrypt \
    --network=myNetwork \
    --rm -p 80:80 -p 443:443 -d nginx
````