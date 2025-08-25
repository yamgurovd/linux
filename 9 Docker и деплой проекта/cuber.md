# Установка и базовая настройка Kubernetes с помощью MicroK8s на Ubuntu

---

## Краткое содержание инструкции

В данной инструкции описывается процесс установки и первоначальной настройки локального кластера Kubernetes на Ubuntu с
помощью MicroK8s. Такой кластер отлично подходит для обучения, тестирования и разработки.

---

## Краткое содержание происходящего

1. **Установка MicroK8s**  
   С помощью snap-пакета устанавливается минималистичная версия Kubernetes — MicroK8s.

2. **Добавление пользователя в группу microk8s**  
   Для работы с MicroK8s без sudo текущий пользователь добавляется в соответствующую группу.

3. **Перезапуск сессии для применения изменений**  
   Необходимо перезайти в сессию или использовать команду для обновления групповых прав.

4. **Проверка статуса кластера**  
   Проверяется, что кластер успешно запущен и готов к работе.

5. **Работа с кластером через kubectl**  
   Используется встроенная команда `microk8s kubectl` для взаимодействия с кластером.

6. **Включение полезных аддонов**  
   По необходимости активируются компоненты: DNS, Dashboard, Storage и др.

---

## Что происходит в целом

Инструкция позволяет быстро развернуть локальный Kubernetes-кластер на Ubuntu без сложных настроек и виртуализации.
Такой подход идеален для самостоятельного обучения, экспериментов и разработки приложений под Kubernetes.

---

## Таблица команд для установки и настройки MicroK8s

| Команда                                                  | Описание                                                                                    |
|----------------------------------------------------------|---------------------------------------------------------------------------------------------|
| `sudo snap install microk8s --classic`                   | Устанавливает MicroK8s через snap-пакет.                                                    |
| `sudo usermod -a -G microk8s $USER`                      | Добавляет текущего пользователя в группу microk8s для работы без sudo.                      |
| `sudo chown -f -R $USER ~/.kube`                         | Меняет владельца директории конфигурации kubectl на текущего пользователя.                  |
| `newgrp microk8s`                                        | Перезапускает сессию для применения групповых изменений (или просто выйти/войти в систему). |
| `microk8s status --wait-ready`                           | Проверяет статус и готовность кластера.                                                     |
| `microk8s kubectl get nodes`                             | Показывает список нод в кластере.                                                           |
| `microk8s kubectl get services`                          | Показывает список сервисов в кластере.                                                      |
| `microk8s enable dashboard dns hostpath-storage`         | Включает Dashboard, DNS и встроенное хранилище.                                             |
| `microk8s dashboard-proxy`                               | Запускает прокси для доступа к Kubernetes Dashboard из браузера.                            |
| `microk8s kubectl create deployment nginx --image=nginx` | Разворачивает тестовое приложение nginx в кластере.                                         |
| `microk8s stop` / `microk8s start`                       | Останавливает/запускает кластер MicroK8s.                                                   |

---

## Пояснения

- **MicroK8s** — минималистичная, быстрая и простая в установке реализация Kubernetes для локального использования.
- **Добавление в группу** — позволяет работать с MicroK8s без постоянного использования sudo.
- **`microk8s kubectl`** — встроенный аналог kubectl, используется для управления кластером.
- **Аддоны** — дополнительные компоненты (Dashboard, DNS, Storage), которые можно включить при необходимости.
- **Dashboard** — веб-интерфейс для управления кластером.

---

## Пример вывода команд

### Проверка статуса кластера

1. `microk8s status --wait-ready`

![ssh](/course_helpers/9%20Docker%20и%20деплой%20проекта/cub1.png)

2. `microk8s is running
high-availability: no
addons: enabled 0 of 29 `

![ssh](/course_helpers/9%20Docker%20и%20деплой%20проекта/cub2.png)

### Список нод

`microk8s kubectl get nodes`
![ssh](/course_helpers/9%20Docker%20и%20деплой%20проекта/cub3.png)

### Вход в дашборд

1. `microk8s dashboard-proxy`
   ![ssh](/course_helpers/9%20Docker%20и%20деплой%20проекта/cub4.png)
2. `https://127.0.0.1:10443/#/login`
   ![ssh](/course_helpers/9%20Docker%20и%20деплой%20проекта/cub5.png)
   ![ssh](/course_helpers/9%20Docker%20и%20деплой%20проекта/cub6.png)

Токен который был использован 
```text
eyJhbGciOiJSUzI1NiIsImtpZCI6IkhNRXFiT2drNVFPM1ZzeDRVcDM5U0dpeUFwNzIyZzVnbjFVZDhoU1RwSVUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJtaWNyb2s4cy1kYXNoYm9hcmQtdG9rZW4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBmMGI4NjM0LTJhMzAtNGRjNC1iOThhLTg2N2YwNTA1ZDE2MCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpkZWZhdWx0In0.wZRs0S-Mp3nqCDwkm3DQ-48IujyisDhsS8yBXDI1G2jCQCnPzxPlZjAQlIQvCgFJ2tYNsIoI-pAO1_0Ipm6n5IhbmL-cEgtDlZUOhM3BW5Aydov2e8yst87dLQBMCwM5ckmchdxwd7aKTAcsU2etFMLMSdXE1P2Xv-eT3gaHl_cbqATeK3ZH8e3WkNuir9LI8Lub2oH2C7NqqGHHwcoF4l68pk4xqkzVzof-1k_IyKC_LwD6lmIWvUAgLSe5oqQreYmRCaQY43ETVDSekwypwX14OKRbaYwlqpvDPk2v1q1-In3gUvgoqKiF7GTy6qh0PmsrgZkPmrQci01JvcqigA
```
---

# Подробная инструкция

---

### 1. Установка MicroK8s

Выполните в терминале:

```shell
sudo snap install microk8s --classic
```

---

### 2. Добавление пользователя в группу microk8s

```shell
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
```

---

### 3. Перезапуск сессии

Чтобы изменения вступили в силу, выполните:

```shell
newgrp microk8s
```

Или просто выйдите из системы и войдите снова.

---

### 4. Проверка статуса кластера

```shell
microk8s status --wait-ready
```

---

### 5. Проверка работы кластера

```shell
microk8s kubectl get nodes
microk8s kubectl get services
```

---

### 6. Включение аддонов (по необходимости)

```shell
microk8s enable dashboard dns hostpath-storage
```

---

### 7. Запуск Dashboard

```shell
microk8s dashboard-proxy
```

В терминале появится ссылка для доступа к Dashboard через браузер.

---

### 8. Разворачивание тестового приложения (опционально)

```shell
microk8s kubectl create deployment nginx --image=nginx
```

---

### 9. Остановка и запуск MicroK8s (при необходимости)

```shell
microk8s stop
microk8s start
```

---

## Что было проделано

- Установлен MicroK8s — локальный Kubernetes-кластер.
- Настроены права пользователя для работы без sudo.
- Проверена готовность и работоспособность кластера.
- Включены необходимые аддоны.
- Развернуто тестовое приложение nginx (опционально).
- Описаны команды для управления кластером.

---

**Теперь вы готовы изучать и экспериментировать с Kubernetes на своём Ubuntu-компьютере!**
