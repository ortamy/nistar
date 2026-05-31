# 🚀 Полная автономная DevOps-платформа

**Git · CI · Чат · Уведомления · Мониторинг — всё на одном сервере**

---

**Метаданные файла**

- **Файл:** `autonomy/01_full_stack_setup.md`
- **Путь:** `nistar/autonomy/01_full_stack_setup.md`
- **Версия:** 2.0.0
- **Дата создания:** 2025-01-15
- **Последнее обновление:** 2026-06-01
- **Автор:** Or Tam
- **Статус:** Активен
- **Изменения en этой версии:** Объединение всех сервисов в один файл. Полная автоматизация развёртывания.

---

**Никакого GitHub. Никакого Slack. Никакого Telegram. Всё своё. Всё под контролем. Всё работает локально.**

---

## 📋 Что входит в платформу

- **Gitea** — Git-сервер (альтернатива GitHub, GitLab)
- **Woodpecker CI** — CI/CD пайплайны (альтернатива GitHub Actions)
- **Mattermost** — командный чат (альтернатива Slack, Discord)
- **Ntfy** — push-уведомления на телефон
- **Uptime Kuma** — мониторинг доступности сервисов

---

## 🖥️ Требования к серверу

- **CPU:** 2 ядра
- **ОЗУ:** 4–8 ГБ
- **Диск:** 20–30 ГБ
- **ОС:** Ubuntu 22.04 / Debian 12
- **Интернет:** только для первоначальной загрузки образов

---

## 🧰 Быстрая установка (всё сразу)

### Шаг 1. Установка Docker

Выполните следующие команды для установки Docker и Docker Compose:

```bash
# Обновление системы
sudo apt update && sudo apt upgrade -y

# Установка Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Установка Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Добавление пользователя в группу docker
sudo usermod -aG docker $USER
newgrp docker
```

Шаг 2. Создание рабочей директории и файла конфигурации

Создайте папку для проекта и файл docker-compose.yml:

```bash
mkdir -p ~/devops-platform && cd ~/devops-platform
nano docker-compose.yml
```

Шаг 3. Полный docker-compose.yml

Скопируйте содержимое ниже в файл docker-compose.yml:

```yaml
version: '3.8'

networks:
  devops-net:
    driver: bridge

volumes:
  gitea-data:
  gitea-postgres:
  woodpecker-data:
  woodpecker-postgres:
  mattermost-data:
  mattermost-postgres:
  ntfy-data:
  uptime-kuma-data:

services:

  # ========== Gitea — Git-сервер ==========
  gitea-db:
    image: postgres:16-alpine
    container_name: gitea-db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=gitea
      - POSTGRES_PASSWORD=gitea
      - POSTGRES_DB=gitea
    volumes:
      - gitea-postgres:/var/lib/postgresql/data
    networks:
      - devops-net

  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    restart: unless-stopped
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=gitea-db:5432
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea
      - GITEA__server__DOMAIN=git.local
      - GITEA__server__ROOT_URL=http://git.local:3000
      - GITEA__server__HTTP_PORT=3000
    volumes:
      - gitea-data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "222:22"
    depends_on:
      - gitea-db
    networks:
      - devops-net

  # ========== Woodpecker CI — CI/CD ==========
  woodpecker-db:
    image: postgres:16-alpine
    container_name: woodpecker-db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=woodpecker
      - POSTGRES_PASSWORD=woodpecker
      - POSTGRES_DB=woodpecker
    volumes:
      - woodpecker-postgres:/var/lib/postgresql/data
    networks:
      - devops-net

  woodpecker-server:
    image: woodpeckerci/woodpecker-server:latest
    container_name: woodpecker-server
    restart: unless-stopped
    environment:
      - WOODPECKER_OPEN=true
      - WOODPECKER_ADMIN=your-gitea-username
      - WOODPECKER_HOST=http://ci.local
      - WOODPECKER_GITEA=true
      - WOODPECKER_GITEA_URL=http://gitea:3000
      - WOODPECKER_GITEA_CLIENT_ID=xxx
      - WOODPECKER_GITEA_CLIENT_SECRET=xxx
      - WOODPECKER_DATABASE_DRIVER=postgres
      - WOODPECKER_DATABASE_DATASOURCE=postgres://woodpecker:woodpecker@woodpecker-db:5432/woodpecker?sslmode=disable
    volumes:
      - woodpecker-data:/var/lib/woodpecker
    depends_on:
      - woodpecker-db
      - gitea
    networks:
      - devops-net

  woodpecker-agent:
    image: woodpeckerci/woodpecker-agent:latest
    container_name: woodpecker-agent
    restart: unless-stopped
    environment:
      - WOODPECKER_SERVER=woodpecker-server:9000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - woodpecker-server
    networks:
      - devops-net

  # ========== Mattermost — командный чат ==========
  mattermost-db:
    image: postgres:16-alpine
    container_name: mattermost-db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=mmuser
      - POSTGRES_PASSWORD=mmuser
      - POSTGRES_DB=mattermost
    volumes:
      - mattermost-postgres:/var/lib/postgresql/data
    networks:
      - devops-net

  mattermost:
    image: mattermost/mattermost-team-edition:latest
    container_name: mattermost
    restart: unless-stopped
    environment:
      - MM_USERNAME=mmuser
      - MM_PASSWORD=mmuser
      - MM_DBNAME=mattermost
    volumes:
      - mattermost-data:/mattermost/data
    ports:
      - "8065:8065"
    depends_on:
      - mattermost-db
    networks:
      - devops-net

  # ========== Ntfy — push-уведомления ==========
  ntfy:
    image: binwiederhier/ntfy
    container_name: ntfy
    restart: unless-stopped
    command: serve
    volumes:
      - ntfy-data:/var/lib/ntfy
    ports:
      - "2586:80"
    networks:
      - devops-net

  # ========== Uptime Kuma — мониторинг ==========
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    restart: unless-stopped
    volumes:
      - uptime-kuma-data:/app/data
    ports:
      - "3001:3001"
    networks:
      - devops-net
```

Шаг 4. Запуск всех сервисов

Запустите все сервисы командой:

```bash
docker-compose up -d
```

Шаг 5. Настройка Gitea

1. Откройте браузер и перейдите по адресу http://IP_вашего_сервера:3000
2. Пройдите регистрацию — первый пользователь станет администратором
3. Перейдите в Настройки сайта → Приложения
4. Создайте новое приложение Woodpecker:
   · Имя: Woodpecker
   · Перенаправление URI: http://IP_вашего_сервера:3000
5. Скопируйте Client ID и Client Secret

Шаг 6. Настройка Woodpecker

1. Остановите Woodpecker: docker-compose stop woodpecker-server woodpecker-agent
2. Отредактируйте docker-compose.yml — вставьте Client ID и Client Secret
3. Запустите снова: docker-compose up -d woodpecker-server woodpecker-agent

Шаг 7. Настройка уведомлений в Mattermost

В Mattermost:

1. Откройте http://IP_вашего_сервера:8065
2. Создайте команду и канал
3. Перейдите в Интеграции → Входящие вебхуки
4. Создайте новый вебхук и скопируйте URL

В Gitea:

1. Перейдите в Настройки сайта → Вебхуки
2. Добавьте вебхук → Gitea
3. Вставьте URL из Mattermost
4. Выберите события: push, issues, pull_request
5. Сохраните

Шаг 8. Настройка ntfy на телефоне

1. Установите приложение ntfy (F‑Droid / Google Play / iOS)
2. Добавьте сервер: http://IP_вашего_сервера:2586
3. Подпишитесь на тему mattermost

В Mattermost:

1. Создайте Исходящий вебхук
2. Название: Ntfy Notifications
3. URL: http://ntfy:80/mattermost
4. Метод: POST
5. Тело запроса:

```json
{
  "topic": "mattermost",
  "title": "{{.ChannelName}}",
  "message": "{{.Text}}",
  "priority": 3
}
```

Шаг 9. Настройка мониторинга Uptime Kuma

1. Откройте http://IP_вашего_сервера:3001
2. Создайте мониторинг для каждого сервиса:
   · http://gitea:3000
   · http://woodpecker-server:9000
   · http://mattermost:8065
   · http://ntfy:80
3. Добавьте уведомление в ntfy при падении любого сервиса

---

✅ Проверка работы

После завершения настройки сервисы будут доступны по следующим адресам:

· Gitea — http://IP_вашего_сервера:3000
· Woodpecker — встроен в Gitea, тот же адрес http://IP_вашего_сервера:3000
· Mattermost — http://IP_вашего_сервера:8065
· Ntfy — http://IP_вашего_сервера:2586
· Uptime Kuma — http://IP_вашего_сервера:3001

---

🧹 Управление

Основные команды для управления платформой:

```bash
# Запуск всех сервисов
docker-compose up -d

# Остановка всех сервисов
docker-compose down

# Просмотр логов всех сервисов
docker-compose logs -f

# Просмотр логов конкретного сервиса
docker-compose logs -f gitea

# Перезапуск конкретного сервиса
docker-compose restart gitea

# Обновление всех образов
docker-compose pull
docker-compose up -d

# Резервное копирование данных
tar -czf backup-$(date +%Y%m%d).tar.gz volumes/
```

---

🎯 Что вы получили

· Свой Git-сервер (не GitHub, не GitLab)
· Свой CI/CD (не GitHub Actions)
· Свой командный чат (не Slack, не Discord)
· Свои push-уведомления (не Telegram)
· Свой мониторинг (не UptimeRobot)

Всё на одном сервере. Всё в Docker. Всё под вашим контролем. Полностью локально. Никакой зависимости от облачных сервисов.

---

📝 История изменений

· Версия 2.0.0 (2026-06-01) — Объединение всех сервисов в один файл. Полная автоматизация развёртывания.
· Версия 1.0.0 (2025-01-15) — Первоначальный выпуск. Базовая конфигурация.

```