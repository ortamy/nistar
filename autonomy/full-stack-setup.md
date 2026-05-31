---
title: "Автономная DevOps-платформа"
version: 2.0.0
created: 2025-01-15
updated: 2026-06-01
author: "Or Tam"
file: "autonomy/01_full_stack_setup.md"
path: "nistar/autonomy/01_full_stack_setup.md"
status: "Активен"
changes: "Объединение всех сервисов в один файл. Полная автоматизация развёртывания."
---

# 🚀 Полная автономная DevOps-платформа

**Git · CI · Чат · Уведомления · Мониторинг — всё на одном сервере**

```text
╔════════════════════════════════════════════════════════════════════════════╗
║  Никакого GitHub. Никакого Slack. Никакого Telegram.                      ║
║  Всё своё. Всё под контролем. Всё работает локально.                      ║
╚════════════════════════════════════════════════════════════════════════════╝
```

---

📋 Что входит в платформу

· Gitea — Git-сервер (альтернатива GitHub, GitLab)
· Woodpecker CI — CI/CD пайплайны (альтернатива GitHub Actions)
· Mattermost — командный чат (альтернатива Slack, Discord)
· Ntfy — Push-уведомления на телефон
· Uptime Kuma — мониторинг доступности сервисов

---

🖥️ Требования к серверу

· CPU: 2 ядра
· ОЗУ: 4–8 ГБ
· Диск: 20–30 ГБ
· ОС: Ubuntu 22.04 / Debian 12
· Интернет: только для первоначальной загрузки образов

---

🧰 Быстрая установка (всё сразу)

Шаг 1. Установка Docker

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

```bash
mkdir -p ~/devops-platform && cd ~/devops-platform
nano docker-compose.yml
```

Шаг 3. Полный docker-compose.yml (копируй целиком)

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
  # ============================================================
  # 1. Gitea — Git-сервер
  # ============================================================
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

  # ============================================================
  # 2. Woodpecker CI — CI/CD
  # ============================================================
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

  # ============================================================
  # 3. Mattermost — командный чат
  # ============================================================
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

  # ============================================================
  # 4. Ntfy — push-уведомления
  # ============================================================
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

  # ============================================================
  # 5. Uptime Kuma — мониторинг
  # ============================================================
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

```bash
docker-compose up -d
```

Шаг 5. Настройка Gitea

1. Открой браузер: http://IP_твоего_сервера:3000
2. Пройди регистрацию (первый пользователь → администратор)
3. Перейди в Настройки сайта → Приложения
4. Создай приложение Woodpecker:
   · Имя: Woodpecker
   · Перенаправление URI: http://IP_сервера:3000
5. Скопируй Client ID и Client Secret

Шаг 6. Настройка Woodpecker

1. Останови Woodpecker: docker-compose stop woodpecker-server woodpecker-agent
2. Отредактируй docker-compose.yml — вставь Client ID и Client Secret
3. Запусти снова: docker-compose up -d woodpecker-server woodpecker-agent

Шаг 7. Настройка уведомлений в Mattermost

1. Открой Mattermost: http://IP_сервера:8065
2. Создай команду и канал
3. Перейди в Интеграции → Входящие вебхуки → создай новый вебхук
4. Скопируй URL
5. В Gitea: Настройки сайта → Вебхуки → добавить вебхук
6. Вставь URL, выбери события: push, issues, pull_request

Шаг 8. Настройка ntfy на телефоне

1. Установи приложение ntfy (F‑Droid / Google Play / iOS)
2. Добавь сервер: http://IP_твоего_сервера:2586
3. Подпишись на тему mattermost
4. В Mattermost создай Исходящий вебхук:
   · URL: http://ntfy:80/mattermost
   · Метод: POST
   · Тело:

```json
{
  "topic": "mattermost",
  "title": "{{.ChannelName}}",
  "message": "{{.Text}}",
  "priority": 3
}
```

Шаг 9. Настройка мониторинга Uptime Kuma

1. Открой Uptime Kuma: http://IP_сервера:3001
2. Создай мониторинг для каждого сервиса:
   · http://gitea:3000
   · http://woodpecker-server:9000
   · http://mattermost:8065
   · http://ntfy:80
3. Добавь уведомление в ntfy при падении

---

✅ Проверка работы

Сервис URL Статус
Gitea http://IP:3000 ✅
Woodpecker http://IP:3000 (встроен в Gitea) ✅
Mattermost http://IP:8065 ✅
Ntfy http://IP:2586 ✅
Uptime Kuma http://IP:3001 ✅

---

🧹 Управление

```bash
# Запуск всех сервисов
docker-compose up -d

# Остановка всех сервисов
docker-compose down

# Просмотр логов
docker-compose logs -f

# Перезапуск конкретного сервиса
docker-compose restart gitea

# Обновление всех образов
docker-compose pull
docker-compose up -d

# Резервное копирование данных
tar -czf backup-$(date +%Y%m%d).tar.gz volumes/
```

---

🎯 Что ты получил

· ✅ Свой Git (не GitHub)
· ✅ Свой CI/CD (не GitHub Actions)
· ✅ Свой чат (не Slack)
· ✅ Свои уведомления (не Telegram)
· ✅ Свой мониторинг (не UptimeRobot)

```text
╔════════════════════════════════════════════════════════════════════════════╗
║  🎉 Готово! Твоя автономная DevOps-платформа работает.                     ║
║  🔒 Полностью локально. Никакой зависимости от облачных сервисов.         ║
║  💪 Полный контроль над своей инфраструктурой.                            ║
╚════════════════════════════════════════════════════════════════════════════╝
```