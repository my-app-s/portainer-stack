# ⚙️ Configuration Reference

This document describes environment variables and Docker Compose parameters used in this deployment template.

---

## 🧩 Environment Variables (`.env`)

All configuration is controlled via the `.env` file.

Default values are defined using Docker Compose fallbacks (`${VAR:-default}`).
Uncomment and adjust variables as needed.

---

### 📦 Container Versions

| Variable | Default | Description |
| :--- | :--- | :--- |
| `VERSION_T` | `latest` | Traefik image tag |
| `VERSION_P` | `latest` | Portainer CE image tag |

---

### 🌐 Network & Ports

Used when default ports (80, 443) are already occupied.

| Variable | Default | Description |
| :--- | :--- | :--- |
| `FORWARD_PORT_HTTP` | `80` | External HTTP port |
| `FORWARD_PORT_HTTPS` | `443` | External HTTPS port |

---

### 🌍 Deployment Settings

| Variable | Values | Description |
| :--- | :--- | :--- |
| `DOMAIN` | IP, `localhost`, or domain | Target address for accessing the service |
| `CERT_RESOLVER` | empty or `myresolver` | TLS mode selector |
| `ACME_EMAIL` | email | Used by Let's Encrypt for certificate notifications |

---

## 🚀 Deployment Modes

### Local / Testing (Self-Signed TLS)

```env
DOMAIN=192.168.100.180
CERT_RESOLVER=
```

- Uses self-signed certificates  
- Suitable for local networks and testing  

---

### Production (Domain + Let's Encrypt)

```env
DOMAIN=portainer.example.com
CERT_RESOLVER=myresolver
ACME_EMAIL=admin@example.com
```

- Requires a valid domain name  
- Requires ports 80/443 to be publicly accessible  
- Uses Let's Encrypt (ACME) for TLS certificates  

---

## 🐳 Docker Compose Parameters

### Service: `traefik`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `ports: ${FORWARD_PORT_HTTP:-80}:80` | Infra | HTTP entry point |
| `ports: ${FORWARD_PORT_HTTPS:-443}:443` | Infra | HTTPS entry point |
| `/var/run/docker.sock:ro` | Volume | Read-only Docker API access |
| `traefik_certificates` | Volume | Stores TLS certificates |
| `--providers.docker.exposedbydefault=false` | Security | Only explicitly enabled containers are exposed |
| `--global.checknewversion=false` | Config | Disables automatic version checks |

---

### Service: `portainer`

| Label | Value | Description |
| :--- | :--- | :--- |
| `traefik.enable` | `true` | Enables routing |
| `traefik.docker.network` | `portainer_bridge_network` | Network used for routing |
| `traefik.http.services.portainer.loadbalancer.server.port` | `9000` | Internal service port |
| `traefik.http.routers.portainer.rule` | `Host(...)` | Routing rule |
| `traefik.http.routers.portainer.tls` | `true` | Enables TLS |
| `traefik.http.routers.portainer.tls.certresolver` | `${CERT_RESOLVER}` | TLS resolver |

---

## ⚠️ Notes

- Let's Encrypt requires a valid domain name (not IP or localhost)  
- `CERT_RESOLVER` must match the resolver defined in Traefik configuration  
- Security depends on correct configuration of credentials and environment variables  

---
