# Nginx

---

## Краткое содержание инструкции

В инструкции описывается настройка и запуск Nginx в Docker для проксирования FastAPI приложения, также запущенного в
Docker. Кратко процесс такой:

### Краткий обзор происходящего

1. Создается Docker-сеть myNetwork для объединения контейнеров.

2. В docker-compose.yml описываются 4 сервиса: FastAPI backend (booking_back_service), два сервиса Celery для фоновых
   задач (worker и beat) и сервис Nginx (booking_nginx_service).

3. Nginx запускается в контейнере с монтированием собственного конфигурационного файла nginx.conf, который настроен как
   обратный прокси на backend по адресу http://booking_back:8000.

4. В конфиге Nginx прописаны заголовки для проксирования, таймауты и базовые настройки.

5. Запуск всех сервисов происходит командой docker-compose up -d --build.

6. Далее проверяется, что все контейнеры работают, подключены к сети, и что Nginx корректно проксирует запросы к
   FastAPI.

`При изменении конфигурации Nginx его можно перезапустить отдельно.`

7. В инструкции также есть раздел с типовыми ошибками и способами их устранения (например, проблемы с именами
   контейнеров, отсутствием секции events в nginx.conf, правами на файл конфигурации).

8. Для остановки системы используется docker-compose down.

### Что происходит в целом

Таким образом, инструкция показывает, как с помощью Docker и Nginx организовать проксирование HTTP-запросов к FastAPI
приложению, обеспечивая удобное управление через docker-compose и изоляцию сервисов в одной сети

### Команды которые используется

| Команда                                                                     | Пояснение                                                                                      |
|-----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| `docker network create myNetwork`                                           | Создает отдельную Docker-сеть для объединения контейнеров приложения и Nginx                   |
| `docker-compose up -d --build`                                              | Собирает образы и запускает все сервисы в фоне (detached mode)                                 |
| `docker-compose up -d --force-recreate booking_nginx_service`               | Перезапускает только контейнер с Nginx с пересозданием, чтобы применить изменения конфигурации |
| `docker ps`                                                                 | Показывает список запущенных контейнеров и их статус                                           |
| `docker logs booking_nginx`                                                 | Выводит логи работы контейнера с Nginx                                                         |
| `docker logs booking_back`                                                  | Выводит логи работы backend-сервиса FastAPI                                                    |
| `docker network inspect myNetwork`                                          | Показывает подробную информацию о Docker-сети и подключенных к ней контейнерах                 |
| `docker run --rm --network=myNetwork busybox nslookup booking_back`         | Проверяет DNS разрешение имени контейнера backend внутри сети myNetwork                        |
| `curl http://localhost`                                                     | Отправляет HTTP-запрос к Nginx для проверки проксирования запросов к backend                   |
| `curl http://localhost:7777`                                                | Отправляет HTTP-запрос напрямую к backend-сервису, минуя Nginx                                 |
| `docker inspect --format='{{.State.Health.Status}}' booking_nginx`          | Проверяет статус healthcheck контейнера Nginx                                                  |
| `docker-compose down`                                                       | Останавливает и удаляет все контейнеры, созданные docker-compose                               |
| `docker exec -it booking_nginx curl -v http://booking_back:8000`            | Выполняет внутри контейнера Nginx запрос к backend для проверки сетевого соединения            |
| `chmod 644 nginx.conf`                                                      | Устанавливает права доступа к файлу конфигурации Nginx                                         |
| `docker run --rm -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf nginx nginx -t` | Проверяет синтаксис файла конфигурации Nginx без запуска контейнера                            |

# Подробная инструкция

---

## Инструкция по настройке Nginx с Docker для FastAPI приложения

#### Предварительные требования

- Установленные Docker и Docker Compose
- FastAPI приложение в Docker-контейнере
- Созданная Docker сеть myNetwork

### Шаг 1: Создание Docker сети

```shell
docker network create myNetwork
```

### Шаг 2: Настройка docker-compose.yml

```text
version: '3.8'

services:
  booking_back_service:
    container_name: booking_back
    build: .
    ports:
      - "7777:8000"  # Для прямого доступа к бэкенду (опционально)
    networks:
      - myNetwork
    env_file:
      - .env

  booking_celery_worker_service:
    container_name: booking_celery_worker
    build: .
    networks:
      - myNetwork
    env_file:
      - .env
    command: "celery --app=src.tasks.celery_app:celery_instance worker -l INFO"

  booking_celery_beat_service:
    container_name: booking_celery_beat
    build: .
    networks:
      - myNetwork
    env_file:
      - .env
    command: "celery --app=src.tasks.celery_app:celery_instance beat -l INFO"

  booking_nginx_service:
    container_name: booking_nginx
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
    networks:
      - myNetwork
    depends_on:
      - booking_back_service
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3

networks:
  myNetwork:
    external: true
```

### Шаг 3: Создание файла nginx.conf

#### Создайте файл nginx.conf в корне проекта:

```text
events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    
    server {
        listen 80;
        server_name localhost;
        
        location / {
            proxy_pass http://booking_back:8000/;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Таймауты для длительных операций
            proxy_connect_timeout 300s;
            proxy_send_timeout 300s;
            proxy_read_timeout 300s;
            send_timeout 300s;
        }
        
        # Раскомментируйте для обслуживания статических файлов
        # location /static/ {
        #   alias /app/static/;
        #   expires 30d;
        # }
    }
}
```

### Шаг 4: Запуск системы

```shell
# Сборка и запуск всех сервисов
docker-compose up -d --build

# Перезапуск только Nginx при изменении конфигурации
docker-compose up -d --force-recreate booking_nginx_service
```

### Шаг 5: Проверка работоспособности

#### Проверка контейнеров

```shell
docker ps
```

`Убедитесь, что все 4 контейнера в статусе "Up"`

#### Проверка логов

```shell
# Логи Nginx
docker logs booking_nginx

# Логи бэкенда
docker logs booking_back
```

#### docker network inspect myNetwork

```shell
docker network inspect myNetwork
```

`Убедитесь, что все контейнеры подключены к сети`

#### Проверка DNS разрешения

```shell
docker run --rm --network=myNetwork busybox nslookup booking_back
```

`Должен вернуть IP-адрес контейнера бэкенда`

#### Тестовые запросы

```shell
# Через Nginx
curl http://localhost

# Прямо к бэкенду
curl http://localhost:7777

# Проверка healthcheck
docker inspect --format='{{.State.Health.Status}}' booking_nginx
```

### Шаг 6: Остановка системы

```shell
docker-compose down
```

---

## Дополнительно

---

### Устранение неполадок

#### "Ошибка: "host not found in upstream"

1. Проверьте имя контейнера бэкенда в nginx.conf
2. Убедитесь, что контейнер бэкенда запущен
3. Проверьте подключение к сети:

```shell
docker exec -it booking_nginx curl -v http://booking_back:8000
```

#### "Ошибка: "no events section in configuration"

- Убедитесь, что в nginx.conf есть секция events:

```text
events {
    worker_connections 1024;
}
```

#### Ошибка монтирования файла

1. Проверьте путь к nginx.conf
2. Убедитесь в правильности прав доступа:

```shell
chmod 644 nginx.conf
```

3. Используйте абсолютный путь в docker-compose.yml при необходимости

#### Проверка синтаксиса Nginx

```shell
docker run --rm -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf nginx nginx -t
```

![BPMN-схема бизнес-процесса](/course_helpers/9%20Docker%20и%20деплой%20проекта/nginx_6.png)

---