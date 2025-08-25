# Docker Compose

---

## Краткое содержание инструкции

В кратце, инструкция описывает процесс запуска и управления проектом с помощью Docker Compose:

### Краткий обзор происходящего

- Сначала проверяется наличие внешней сети myNetwork в Docker с помощью команды docker network ls, так как эта сеть
  используется в docker-compose.yml.

- Если сети нет, её создают командой docker network create myNetwork, чтобы контейнеры могли использовать эту сеть.

- Запуск проекта выполняется командой docker compose up, которая создаёт и запускает все сервисы, описанные в файле
  docker-compose.yml.

- Для остановки работы проекта в терминале достаточно нажать CTRL + C.

- Чтобы полностью удалить контейнеры, созданные Compose, используется команда docker compose down.

- Для удаления всех контейнеров и образов Docker применяются команды остановки и удаления контейнеров (docker stop и
  docker rm), а затем удаление всех образов через docker rmi -f.

- Для полной очистки Docker от неиспользуемых образов, томов, контейнеров и сетей можно применить команду docker system
  prune -a --volumes -f.

### Что происходит в целом

Таким образом, инструкция помогает подготовить окружение (создать сеть), запустить проект, управлять его жизненным
циклом и при необходимости очистить Docker от всех ресурсов.

### Таблица с командами

| Команда                                | Описание                                                                            | Флаги и параметры                                                              |
|----------------------------------------|-------------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| `docker network ls`                    | Просмотр списка всех сетей Docker                                                   | —                                                                              |
| `docker network create myNetwork`      | Создание внешней сети с именем `myNetwork` для использования в `docker-compose.yml` | —                                                                              |
| `docker compose up`                    | Запуск всех сервисов, описанных в `docker-compose.yml`                              | —                                                                              |
| `CTRL + C`                             | Остановка работы проекта в терминале (прерывание `docker compose up`)               | —                                                                              |
| `docker compose down`                  | Остановка и удаление контейнеров, сетей, томов, созданных `docker compose up`       | —                                                                              |
| `docker stop $(docker ps -a -q)`       | Остановка всех запущенных контейнеров                                               | `ps -a -q` — список всех контейнеров (только ID)                               |
| `docker rm $(docker ps -a -q)`         | Удаление всех контейнеров (включая остановленные)                                   | `ps -a -q` — список всех контейнеров (только ID)                               |
| `docker rmi -f $(docker images -a -q)` | Удаление всех Docker-образов, принудительно удаляя даже занятые образы              | `-f` — принудительное удаление; `images -a -q` — все образы (только ID)        |
| `docker system prune -a --volumes -f`  | Полная очистка Docker: удаляет все неиспользуемые образы, контейнеры, тома и сети   | `-a` — все неиспользуемые образы; `--volumes` — тома; `-f` — без подтверждения |



# Подробная инструкция

---

## Запуск проекта для compose



### Проверить, существует ли сеть с именем `myNetwork` в вашем Docker. Сделать это можно командой:

```shell
docker network ls
```

`Примечание:` Название сети нужно смотреть в модуле `docker-compose.yml`

#### Выглядит так пример:

```text
  booking_celery_worker_service:
    container_name: 'booking_celery_worker'
    build:
      context: .
    networks:   
      - myNetwork    <--- ВОТ ЗДЕСЬ
    env_file:
      - .env
    command: "celery --app=src.tasks.celery_app:celery_instance worker -l INFO"
```

### Создайте сеть командой:

```shell
docker network create myNetwork
```

![Dbeaver](/course_helpers/9%20Docker%20и%20деплой%20проекта/docker-network.png)

### Запускать проект командой:

```shell
docker compose up
```

![Dbeaver](/course_helpers/9%20Docker%20и%20деплой%20проекта/compose_up.png) ![Dbeaver](/course_helpers/9%20Docker%20и%20деплой%20проекта/compose_up2.png)

### Остановить работу проекта внутри Docker

```shell
CTRL + C
```

### Удалить контейнеры

```shell
docker compose down
```

![Dbeaver](/course_helpers/9%20Docker%20и%20деплой%20проекта/compose_delete.png) ![Dbeaver](/course_helpers/9%20Docker%20и%20деплой%20проекта/compose_delete_2.png)

### Удалить все контейнеры и образы

```shell
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
docker rmi -f $(docker images -a -q)
```

- docker images -a -q — выводит ID всех образов.
- docker rmi -f — принудительно удаляет образы, даже если они используются (после удаления контейнеров это безопасно).

### удалить все неиспользуемые образы, тома, контейнеры и сети сразу, можно использовать команду:

```shell
docker system prune -a --volumes -f
```

- -a — удаляет все неиспользуемые образы, не только висячие.
- --volumes — удаляет неиспользуемые тома.
- -f — пропускает подтверждение.

![Dbeaver](/course_helpers/9%20Docker%20и%20деплой%20проекта/docker_delete_3.png)  ![Dbeaver](/course_helpers/9%20Docker%20и%20деплой%20проекта/compose_delete_3.png)

---







































