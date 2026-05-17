# 🔒 Architecture & Network Isolation (DMZ)

This document describes the network topology, isolation model, and security-related design decisions used in this infrastructure template.

---

## 🧩 Overview

The deployment separates components into two Docker networks with different levels of exposure:

- A public network for handling inbound traffic  
- An internal network for isolating application services  

This design reduces direct exposure of internal services and centralizes access through a single entry point.

---

## 🌐 Network Topology

```
[ Internet / External Clients ]
            │
            ▼
┌────────────────────────────────────────────┐
│ Public Network (traefik_bridge_network)    │
│                                            │
│   [ Traefik Reverse Proxy ]                │
│   - Entry points: 80, 443                  │
└────────────────────┬───────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────┐
│ Internal Network (portainer_bridge_network)│
│ (internal: true)                           │
│                                            │
│   [ Portainer Service ]                    │
│   - No direct host port exposure           │
└────────────────────────────────────────────┘
```

---

## 🔐 Network Design

### Public Network (`traefik_bridge_network`)
- Hosts the Traefik reverse proxy  
- Accepts incoming HTTP/HTTPS traffic (ports 80, 443)  
- Provides routing and TLS termination  
- Has outbound connectivity required for ACME (Let's Encrypt)

---

### Internal Network (`portainer_bridge_network`)
- Connects Traefik and Portainer for internal routing  
- Marked as `internal: true`  
- Containers in this network do not have direct external connectivity by default  
- Not directly accessible from the host via published ports  

---

## 🔐 Security Properties

- **No Direct Exposure**  
  Portainer is not published via `ports:` and is not directly accessible from the host network.

- **Centralized Entry Point**  
  All inbound traffic is handled by Traefik, which acts as the only external interface.

- **Network Isolation**  
  Internal services run in a Docker network with restricted external connectivity.

- **TLS Termination**  
  HTTPS is managed at the proxy level (Traefik), including certificate handling (ACME or self-signed).

- **Read-Only Docker Socket**  
  Traefik uses `/var/run/docker.sock` in read-only mode (`:ro`) for service discovery.

---

## ⚠️ Security Notes

- This setup reduces exposure but does not eliminate all attack vectors  
- Security depends on correct configuration of:
  - environment variables  
  - access credentials  
  - firewall rules  
- Additional hardening may be required for production use  

---

## 📌 Design Rationale

The architecture follows common container security practices:

- Separation of concerns (proxy vs application)  
- Minimization of exposed services  
- Use of internal networks for service isolation  
- Controlled access through a single ingress point  

---

## 📎 Limitations

- Does not replace host-level security controls (firewall, kernel hardening)  
- Does not include authentication or authorization policies by default  
- Requires proper domain and DNS configuration for ACME to function  
