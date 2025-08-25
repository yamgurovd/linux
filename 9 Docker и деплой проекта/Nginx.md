# Nginx

## Общая информация

### Зачем нужен Nginx

- Proxy сервер (посредник между клиентом и сайтом)
- Балансировка нагрузки
- Блокировка нежелательного трафика
- Защита сервисов паролем
- Быстро отдает статический контент (картинки, видео)
- Кэширование данных
- Ограничения скорости отдачи ответов
- Защита от DDOS атак

#### Пример схемы

![BPMN-схема бизнес-процесса](/course_helpers/9%20Docker%20и%20деплой%20проекта/Nginx2.png)

### Работа с выделенным сервером или VPS

- Купить хостинг
- Купить домен (опционально, но рекомендуется)
- Создать ssh ключи для доступа к удаленному репозиторию (если репозиторий приватный)
- Установить Docker и Git
- Склонировать репозиторий
- Добавить конфигурационный файл. В нашем проекте это .env Не забудьте сменить пароли по умолчанию на сложные пароли.
- Установить Nginx (в докере или на виртуальную машину)
- Сконфигурировать Nginx
- Поднять сервисы через Docker Compose

#### Пример схемы

![BPMN-схема бизнес-процесса](/course_helpers/9%20Docker%20и%20деплой%20проекта/Nginx.png)

## Запуск Nginx

```shell
docker run --name booking_nginx \
  --volume ./nginx.conf: /etc/nginx/nginx.conf \
  --network=myNetwork \
  --rm -p 80:80 nginx
```

### Настроить nginx.conf

- Создать файл  `nginx.conf`
- Настроить минимальную конфигурацию

```text
events {}

http {
    server / {
        proxy_pass http://booking_back:8000/;
    }
}
```

#### Примечание: обратите внимание `booking_back` взят из

`docker-compose.yml` и должен совпадать названием контейнера в nginx.conf

```text
services:
  booking_back_service:
    container_name: 'booking_back'   <--- Смотреть сюда 
    build:
      context: .
    ports:
      - "7777:8000"
    networks:
      - myNetwork
    env_file:
      - .env
```

![Nginx](/course_helpers/9%20Docker%20и%20деплой%20проекта/nginx_5.png)


### Проверьте существующие сеть с названием myNetwork
```shell
docker network ls
```

#### Если сети нет - создайте её:
```shell
docker network create myNetwork
```

```shell
docker run --name booking_nginx \
  --volume ./nginx.conf:/etc/nginx/nginx.conf \
  --network=myNetwork \
  --rm -p 80:80 nginx
```