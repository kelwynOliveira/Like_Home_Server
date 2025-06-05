# 🔄 n8n Installation (Multi-User Setup)

This setup runs multiple isolated instances of [n8n](https://n8n.io/) on a single server — ideal when different users (e.g. you and your wife) want separate workspaces.

---

## 📦 Docker Compose File

Here's the `docker-compose.yml` configuration to run 3 n8n instances, each with its own persistent volume and domain (optional).

```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - TZ=America/Sao_Paulo
      - N8N_SECURE_COOKIE=false
    volumes:
      - /home/like-server/docker/apps/n8n/data:/home/node/.n8n
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.n8n.rule=Host(`n8n.like`)"
      - "traefik.http.routers.n8n.entrypoints=web"
      - "traefik.http.services.n8n.loadbalancer.server.port=5678"
    networks:
      - traefik-net
    dns:
      - 8.8.8.8
      - 1.1.1.1

  n8n-kelwyn:
    image: n8nio/n8n:latest
    container_name: n8n-kelwyn
    restart: unless-stopped
    ports:
      - "5679:5678"
    environment:
      - TZ=America/Sao_Paulo
      - N8N_SECURE_COOKIE=false
    volumes:
      - /home/like-server/docker/apps/n8n/data-kelwyn:/home/node/.n8n
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.n8n1.rule=Host(`n8n1.like`)"
      - "traefik.http.routers.n8n1.entrypoints=web"
      - "traefik.http.services.n8n1.loadbalancer.server.port=5678"
    networks:
      - traefik-net
    dns:
      - 8.8.8.8
      - 1.1.1.1

  n8n-lidia:
    image: n8nio/n8n:latest
    container_name: n8n-lidia
    restart: unless-stopped
    ports:
      - "5680:5678"
    environment:
      - TZ=America/Sao_Paulo
      - N8N_SECURE_COOKIE=false
    volumes:
      - /home/like-server/docker/apps/n8n/data-lidia:/home/node/.n8n
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.n8n2.rule=Host(`n8n2.like`)"
      - "traefik.http.routers.n8n2.entrypoints=web"
      - "traefik.http.services.n8n2.loadbalancer.server.port=5678"
    networks:
      - traefik-net
    dns:
      - 8.8.8.8
      - 1.1.1.1

networks:
  traefik-net:
    external: true
```

---

## 🧭 Accessing Each Instance

You can access the instances in two ways:

### 🔹 Without Traefik or DNS

Use your server’s IP and the mapped ports:

- `http://<server-ip>:5678` → Main instance
- `http://<server-ip>:5679` → Kelwyn's instance
- `http://<server-ip>:5680` → Lidia's instance

### 🔹 With Traefik + Local DNS

If you're using [Traefik](https://doc.traefik.io/traefik/) with a local DNS like CoreDNS or Pi-hole, you can access them via custom hostnames:

- `http://n8n.like`
- `http://n8n1.like`
- `http://n8n2.like`

> 📌 See [../01-Infra-config/networking.md](../01-Infra-config/networking.md) for DNS and Traefik setup.

---

## 📁 Persistent Volumes

Each instance stores its workflows and credentials in its own mounted folder. Make sure those folders exist and have proper permissions:

```bash
sudo mkdir -p \
  /home/like-server/docker/apps/n8n/data \
  /home/like-server/docker/apps/n8n/data-kelwyn \
  /home/like-server/docker/apps/n8n/data-lidia

sudo chown -R 1000:1000 /home/like-server/docker/apps/n8n/
```

---

## 🛠️ Tips & Extras

- Add more users by duplicating the service block with a new name, port, volume, and domain.
- You can later integrate each instance with tools like PostgreSQL, Redis, S3, or external APIs.
- [n8n Desktop](https://docs.n8n.io/hosting/desktop/) is an alternative if you want personal use only — but Docker is more scalable.
