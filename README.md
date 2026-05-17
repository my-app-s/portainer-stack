# 📦 Portainer Stack

![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![License](https://img.shields.io/badge/license-GNU%20AGPLv3-red.svg)

> **Infrastructure recipe** for quickly and securely deploying a bundled Edge gateway (Traefik) and Portainer Gateway control panel.

This YAML recipe deploys a full-fledged container management stack: the Portainer-CE panel, protected by an isolated reverse proxy server based on Traefik. The gateway automatically handles dynamic traffic routing and SSL certificate generation (Self-Signed / Let's Encrypt ACME).

## 📖 Documentation(Документация)

To prevent overloading the main file, implementation details are moved to separate modules:

* [🔒 Architecture and Security (DMZ)](docs/architecture.md) — about network isolation and protection against hidden telemetry.
* [⚙️ Complete .env Configuration Reference](docs/configuration.md) — description of environment variables, operating modes (Local/Prod), and flags.

## 🚀 Quick start(Быстрый старт)

**Launch of the entire infrastructure:**

```bash
docker compose up -d
```

**Checking provider status and gateway logs:**

```bash
docker compose logs traefik -f
```

## 🔧 SSL certificate maintenance

If you need to force a reissue of Let's Encrypt certificates (for example, if you change your domain), reset the cache:

```bash
docker compose down
docker volume rm traefik_certificates
docker compose up -d
```

## ⚠️ Disclaimer / Отказ от ответственности

### English Version
This project is an **independent development** provided on an **"AS IS"** basis.

* **Liability:** In no event shall the author be liable for any errors, bugs, or data loss arising from the use of this software.
* **Status:** This is an experimental tool.

> [!CAUTION]
> Any use (operation) of this code is at your own risk.

---

### Русская версия
Данный проект является **независимой разработкой** и предоставляется «как есть».

* **Ответственность:** Автор не несет ответственности за любые ошибки, баги или потерю данных, возникшие в результате использования данного кода.
* **Статус:** Это экспериментальный инструмент.

> [!CAUTION]
> Любое использование (эксплуатация) данного кода осуществляется на ваш страх и риск.

## 📜 License

The project is licensed under the GNU Affero General Public License v3.0 (AGPLv3).
See [LICENSE](./LICENSE) for details.
