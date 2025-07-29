# 🐳 Docker & Portainer Setup

This guide documents the setup of **Docker** and **Portainer** on an Ubuntu Server. Portainer provides a convenient UI for managing Docker containers, volumes, networks, and images.

---

## 📦 Docker Installation

Install Docker on your Ubuntu server:

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

Check if Docker is running:

```bash
sudo docker ps
```

---

## 🧭 Portainer Setup

We'll deploy **Portainer CE** (Community Edition) using Docker Compose.

> 💡 Portainer will allow you to manage containers via a web UI at `http://<your-ip>:9000` or using a custom domain like `portainer.like` if you're using a local DNS and reverse proxy like [Traefik](https://doc.traefik.io/traefik/).

> ✅ If you're not using Traefik ([step of Networking Setup](../01-Infra-config/networking.md)), you can safely remove the `labels` and `networks` sections.

### 📄 `docker-compose.yml`

```yaml
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    ports:
      - "9000:9000" # HTTP access
      - "9443:9443" # HTTPS access
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/ssd/data/portainer:/data
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.portainer.rule=Host(`portainer.like`)"
      - "traefik.http.routers.portainer.entrypoints=web"
      - "traefik.http.services.portainer.loadbalancer.server.port=9000"
    networks:
      - traefik-net

networks:
  traefik-net:
    external: true
```

---

## 📂 Volumes & Storage

Make sure the Portainer data folder is persistent and located on a mounted volume:

[💾 Mounting External SSD (Persistent)](../01-Infra-config/README.md#-mounting-external-ssd-persistent)

```bash
sudo mkdir -p /mnt/ssd/data/portainer
sudo chown -R 1000:1000 /mnt/ssd/data/portainer
```

---

## 🔐 Accessing Portainer

- Default access via IP:

  - http\://`<your-server-ip>`:9000
  - https\://`<your-server-ip>`:9443

- If you're using a local DNS and Traefik:

  - http\://`portainer.like`

On first access, you'll be prompted to set up an admin password and select whether to manage a local or remote Docker environment.

---

## ✅ Optional: Enable Docker Compose

If you don't have Docker Compose installed:

```bash
sudo apt install docker-compose
```

Or, for the most recent version:

```bash
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o $DOCKER_CONFIG/cli-plugins/docker-compose
sudo chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
docker-compose --version
```
