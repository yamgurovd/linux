# Работа с Docker

---

![Docker Logo](https://upload.wikimedia.org/wikipedia/commons/4/4e/Docker_%28container_engine%29_logo.svg)

## Содержание

1. [Установка и настройка Docker на локальном компьютере (Linux)](/course_helpers/9%20Docker%20и%20деплой%20проекта/Docker1.md)
2. [Запуск Redis в Docker](/course_helpers/9%20Docker%20и%20деплой%20проекта/Docker_redis.md)
3. [Запуск проекта с Dockerfile](/course_helpers/9%20Docker%20и%20деплой%20проекта/Dockerfile.md)
4. [Инструкция для запуска Docker с БД, Redis и Backend](/course_helpers/9%20Docker%20и%20деплой%20проекта/Docker5.md)
5. [Docker Compose](/course_helpers/9%20Docker%20и%20деплой%20проекта/Compose.md)
5. [Nginx](/course_helpers/9%20Docker%20и%20деплой%20проекта/NGINX.md)
---

### Docker: Основные команды

#### Установка и настройка
| Команда | Описание |
|---------------------------------------------------|----------------------------------------------------------------------------------------------|
| `sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc` | Загрузка GPG-ключа Docker |
| `sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin` | Установка Docker Engine и компонентов |
| `sudo docker run hello-world` | Проверка установки (тестовый контейнер) |
| `sudo usermod -aG docker $USER` | Добавление пользователя в группу docker (для работы без sudo) |
| `rm -r ~/.docker` | Удаление локальных настроек Docker |

#### Работа с образами
| Команда | Описание |
|--------------------------|------------------------------------------------------------------------|
| `docker build -t <имя_образа> .` | Сборка образа из Dockerfile в текущей директории |
| `docker pull <образ>:<тег>` | Загрузка образа из реестра (напр. `redis:7.4`) |
| `docker images` | Просмотр локальных образов |
| `docker rmi <ID_образа>` | Удаление образа |
| `docker rmi -f $(docker images -a -q)` | Принудительное удаление всех образов |

#### Управление контейнерами
| Команда | Описание |
|----------------------------------------|-------------------------------------------------------------------------------------|
| `docker run -it <образ> bash` | Запуск в интерактивном режиме с bash |
| `docker run -d --name <имя> <образ>` | Запуск в фоне с именем |
| `docker run -p <хост>:<контейнер> <образ>` | Проброс портов (напр. `-p 8888:8000`) |
| `docker ps` | Просмотр запущенных контейнеров |
| `docker stop <ID>` | Остановка контейнера |
| `docker rm <ID>` | Удаление контейнера |
| `docker rm $(docker ps -a -q)` | Удаление всех контейнеров |

#### Сети и тома
| Команда | Описание |
|----------------------------------|--------------------------------------------------------|
| `docker network create <сеть>` | Создание сети (напр. `myNetwork`) |
| `docker network ls` | Просмотр сетей |
| `docker run --network <сеть>` | Запуск контейнера в сети |
| `docker run --volume <том>:<путь>` | Подключение тома (напр. `--volume pg-data:/var/lib/postgresql/data`) |

#### Docker Compose
| Команда | Описание |
|---------------------|------------------------------------------|
| `docker compose up` | Запуск сервисов из docker-compose.yml |
| `docker compose down` | Остановка и удаление сервисов |

#### Системные операции
| Команда | Описание |
|---------------------------------|------------------------------------------------------------|
| `docker system prune -a --volumes -f` | Полная очистка неиспользуемых ресурсов |
| `reboot` | Перезагрузка (для применения изменений группы) |

---

### Примеры запуска сервисов

#### База данных PostgreSQL
```bash
docker run --name db \
  -p 6432:5432 \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=booking \
  --network myNetwork \
  --volume pg-data:/var/lib/postgresql/data \
  -d postgres:16
```

#### Redis
```shell
docker run --name redis \
  -p 7379:6379 \
  --network=myNetwork \
  -d redis:7.4
```

#### Backend-приложение
```shell
docker run --name backend \
  -p 8000:8000 \
  --network=myNetwork \
  <ваш_образ>
```

#### Celery Worker
```shell
docker run --name celery_worker \
  --network=myNetwork \
  <ваш_образ> \
  celery --app=src.tasks.celery_app:celery_instance worker -l INFO
```

#### Celery Beat
```shell
docker run --name celery_beat \
  --network=myNetwork \
  <ваш_образ> \
  celery --app=src.tasks.celery_app:celery_instance beat -l INFO
```
#### Примеры запуска сервисов
```shell
# PostgreSQL
docker run --name db -p 6432:5432 -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=password -e POSTGRES_DB=booking --network myNetwork --volume pg-data:/var/lib/postgresql/data -d postgres:16

# Redis
docker run --name redis -p 7379:6379 --network=myNetwork -d redis:7.4

# Backend
docker run --name backend -p 8000:8000 --network=myNetwork backend_image
```

### Подробное пояснение флагов Docker

#### Основные флаги командной строки
| Флаг | Полная форма | Описание | Пример использования |
|------|--------------|----------|----------------------|
| `-i` | `--interactive` | Поддержка интерактивного режима | `docker run -it ubuntu bash` |
| `-t` | `--tty` | Создание псевдо-TTY (терминала) | `docker run -it ubuntu bash` |
| `-d` | `--detach` | Запуск контейнера в фоновом режиме | `docker run -d nginx` |
| `-p` | `--publish` | Проброс портов (хост:контейнер) | `docker run -p 8080:80 nginx` |
| `-e` | `--env` | Установка переменных окружения | `docker run -e "VAR=value" app` |
| `-v` | `--volume` | Подключение томов | `docker run -v /data:/app/data app` |
| `--name` | - | Назначение имени контейнеру | `docker run --name myapp app` |
| `--rm` | - | Автоматическое удаление после остановки | `docker run --rm temp-app` |

#### Флаги для docker build
| Флаг | Описание | Пример |
|------|----------|--------|
| `-t` | Тегирование образа (name:tag) | `docker build -t app:1.0 .` |
| `--no-cache` | Сборка без использования кеша | `docker build --no-cache -t app .` |
| `--pull` | Принудительное обновление базового образа | `docker build --pull -t app .` |

#### Флаги для docker ps
| Флаг | Описание | Пример |
|------|----------|--------|
| `-a` | Показать все контейнеры (включая остановленные) | `docker ps -a` |
| `-q` | Только ID контейнеров (quiet) | `docker ps -aq` |
| `--filter` | Фильтрация вывода | `docker ps --filter "status=running"` |

#### Флаги для docker rm/docker rmi
| Флаг | Описание | Пример |
|------|----------|--------|
| `-f` | Принудительное удаление (force) | `docker rm -f container_id` |
| `-v` | Удаление связанных томов | `docker rm -v container_id` |

#### Флаги для docker system
| Флаг | Описание | Пример |
|------|----------|--------|
| `--volumes` | Удаление томов при очистке | `docker system prune --volumes` |
| `-a` | Удаление всех неиспользуемых объектов | `docker system prune -a` |
| `-f` | Без подтверждения (force) | `docker system prune -f` |

#### Комбинированные флаги
1. **`-it`** - стандартная комбинация для интерактивных контейнеров:
   ```bash
   docker run -it python:3.9 bash
   

### Особые флаги для работы с сетями Docker

#### Основные сетевые флаги
| Флаг | Полная форма | Описание | Пример использования |
|------|--------------|----------|----------------------|
| `--network` | - | Подключение контейнера к указанной сети | `docker run --network my_bridge_network nginx` |
| `--network host` | - | Использование сетевого стека хоста (без изоляции) | `docker run --network host nginx` |
| `--network none` | - | Полная изоляция от сети | `docker run --network none alpine` |
| `--dns` | - | Настройка DNS-серверов для контейнера | `docker run --dns 8.8.8.8 alpine` |
| `--hostname` | `-h` | Установка hostname в сети Docker | `docker run --hostname myapp app_image` |
| `--add-host` | - | Добавление записи в /etc/hosts | `docker run --add-host "db:192.168.1.100" app` |

#### Продвинутые сетевые настройки
| Флаг | Описание | Пример |
|------|----------|--------|
| `--ip` | Назначение статического IPv4-адреса | `docker run --network my_net --ip 172.20.0.10 app` |
| `--ip6` | Назначение статического IPv6-адреса | `docker run --network my_net --ip6 2001:db8::1 app` |
| `--mac-address` | Установка MAC-адреса | `docker run --mac-address 02:42:ac:11:00:02 app` |
| `--link` | (Устаревшее) Связывание с другим контейнером | `docker run --link db:mysql app` |

#### Флаги для создания сетей
```bash
# Создание bridge-сети с пользовательскими настройками
docker network create \
  --driver=bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  my_custom_network