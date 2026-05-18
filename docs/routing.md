## Глобальная архитектура маршрутизации проектов

Документ описывает логику взаимодействия независимых Docker-стеков с главным шлюзом **Traefik**.

## 🧱 Архитектурная концепция
Система построена на принципе **Service Discovery**.  
Traefik слушает системный сокет Docker и автоматически создает маршруты и SSL-сертификаты для контейнеров с корректными метками (labels).

```
[ ИНТЕРНЕТ (Порты 80 / 443 TCP) ]
│
▼
[ РОУТЕР HUAWEI (Проброс на сервер) ]
│
▼
┌─────────────────────────────────────────────┐
│          ГЛАВНЫЙ СТЭК ИНФРАСТРУКТУРЫ        │
│  [Контейнер Traefik] (Слушает сокет Docker) │
└──────────────────────┬──────────────────────┘
│
┌───────────────┴─────────────────────┐
▼ (traefik_bridge_network)
│
├─► [Стек 1: Portainer] ──► portainer.yourdomain.net
├─► [Стек 2: Сайт] ───────► site.yourdomain.net
└─► [Стек 3: Бот] ────────► bot.yourdomain.net
```

## 📑 Инструкция: запуск нового проекта в 3 шага

### Шаг 1. Создай папку проекта

```bash
mkdir -p /projects/my-new-website
cd /projects/my-new-website
nano docker-compose.yml
```
### Шаг 2. Заполни docker-compose.yml по шаблону

```yaml
services:
  my-app-service:
    image: nginx:alpine
    container_name: my-new-website # ⚠️ use as router in lables traefik
    restart: unless-stopped
    networks:
      - traefik_public_network # ⚠️ Name network traefik
    labels:
      - "traefik.enable=true" # enable traefik

      # Base configure router
      # Use exactly one name router: my-new-website
      # Используем строго ОДНО имя роутера: my-new-website
      - "traefik.http.routers.my-new-website.rule=Host(`${DOMAIN:-}`)" # ⚠️ Set access address host
      - "traefik.http.routers.my-new-website.entrypoints=websecure" # Set access port public
      # Use default port app
      # Исправили порт на 80 (родной порт для Nginx)
      - "traefik.http.services.my-new-website.loadbalancer.server.port=80" # ⚠️ Set access port local app

      # ⚠️ Если указано CERT_RESOLVER=myresolver — включится Let's Encrypt.
      # Traefik генерирует self-signed сертификат для этого роутера.
      # Enable TLS and connect to only router my-new-website
      # Включаем TLS и привязываем резолвер К ЭТОМУ ЖЕ роутеру
      - "traefik.http.routers.my-new-website.tls=true" # enable TLS
      - "traefik.http.routers.my-new-website.tls.certresolver=${CERT_RESOLVER:-}"
networks:
  traefik_public_network:
    external: true
```

### Шаг 3. Запусти проект

```bash
docker compose up -d
```

## 🚨 Золотые правила

- Уникальность имен  
    - Имена роутеров и сервисов в labels должны быть уникальными.
        - Пример: my-site-router, telegram-bot-router, crm-router.
- Запрет секции ports  
    - Не использовать ports: в docker-compose.yml.
        - Порты 80/443 заняты Traefik, конфликт приведет к ошибке.
- Один порт внутри  
    - Допустимо указывать одинаковый порт (например, 80) в разных проектах.
        - Traefik маршрутизирует запросы по доменному имени.

## ✅ Итог

Архитектура обеспечивает:
- Чистоту сервера: инфраструктура и проекты разделены.
- Автоматическую маршрутизацию и SSL.
- Простоту запуска новых сервисов без изменения главного стека.
