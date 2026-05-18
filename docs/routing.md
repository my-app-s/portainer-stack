# Глобальная архитектура маршрутизации проектов (Traefik + Docker)

## Архитектура

Принцип: **Service Discovery через Docker socket**

Traefik:
- читает Docker socket
- автоматически создает:
  - routers
  - services
  - TLS (Let's Encrypt при наличии resolver)

Схема:

```

Internet (80/443)
↓
Router (NAT)
↓
Traefik (Docker provider)
↓
traefik_public_network
↓
[containers with labels]

```

---

## Запуск нового проекта

### 1. Создание директории

```bash
mkdir -p /projects/<project>
cd /projects/<project>
```

---

### 2. docker-compose.yml

```yaml
services:
  app:
    image: nginx:alpine
    container_name: <project>

    restart: unless-stopped

    networks:
      - traefik_public_network

    labels:
      - "traefik.enable=true"

      # router
      - "traefik.http.routers.<project>.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.<project>.entrypoints=websecure"

      # service
      - "traefik.http.services.<project>.loadbalancer.server.port=80"

      # TLS
      - "traefik.http.routers.<project>.tls=true"
      - "traefik.http.routers.<project>.tls.certresolver=${CERT_RESOLVER}"

networks:
  traefik_public_network:
    external: true
```

---

### 3. Запуск

```bash
docker compose up -d
```

---

## Правила

### Имена

* router = уникальный
* service = уникальный
* container_name = совпадает с router

---

### Ports

* **НЕ использовать `ports:`**
* 80/443 принадлежат Traefik

---

### Сеть

* всегда:

  ```yaml
  networks:
    - traefik_public_network
  ```

---

### Порт приложения

* указывается только внутренний:

  ```yaml
  loadbalancer.server.port=<port>
  ```

---

## Traefik (инфраструктура)

```yaml
services:
  traefik:
    image: traefik:${VERSION_T:-latest}
    container_name: traefik

    restart: unless-stopped

    ports:
      - "${FORWARD_PORT_HTTP:-80}:80"
      - "${FORWARD_PORT_HTTPS:-443}:443"

    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - traefik_certificates:/letsencrypt

    command:
      - "--log.level=INFO"
      - "--log.format=common"

      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--providers.docker.network=traefik_public_network"

      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"

      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"

      # ACME
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=${ACME_EMAIL}"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"

    networks:
      - traefik_local_network
      - traefik_public_network
```

---

## Portainer (пример сервиса)

```yaml
  portainer:
    image: portainer/portainer-ce:${VERSION_P:-latest}
    container_name: portainer

    restart: unless-stopped

    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data_volume:/data

    labels:
      - "traefik.enable=true"

      - "traefik.http.routers.portainer.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.portainer.entrypoints=websecure"

      - "traefik.http.services.portainer.loadbalancer.server.port=9000"

      - "traefik.http.routers.portainer.tls=true"
      - "traefik.http.routers.portainer.tls.certresolver=${CERT_RESOLVER}"

    networks:
      - traefik_local_network
```

---

## Сети

```yaml
networks:
  traefik_local_network:
    internal: true

  traefik_public_network:
    external: true
```

---

## Volumes

```yaml
volumes:
  traefik_certificates:
  portainer_data_volume:
```

---

## Переменные окружения

| Переменная    | Назначение             |
| ------------- | ---------------------- |
| DOMAIN        | домен сервиса          |
| CERT_RESOLVER | включает Let's Encrypt |
| ACME_EMAIL    | email для ACME         |
| VERSION_T     | версия Traefik         |
| VERSION_P     | версия Portainer       |

---

## Поведение TLS

* `tls=true` → HTTPS включен
* `certresolver` задан → используется Let's Encrypt
* `certresolver` пуст → self-signed сертификат Traefik

---

## Итог

* маршрутизация по Host()
* один публичный вход: Traefik
* изоляция через сети
* масштабирование без изменения инфраструктуры
