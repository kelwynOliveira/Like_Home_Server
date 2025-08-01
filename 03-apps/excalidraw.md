# 🎨 Excalidraw — Virtual Hand-Drawn Whiteboard

[Excalidraw](https://github.com/excalidraw/excalidraw) is an open-source, virtual whiteboard that mimics a hand-drawn style.

- ✏️ **Draw and brainstorm visually** in a freeform canvas
- 🤝 **Collaborative** — share your board with others in real-time
- 🔐 **End-to-End Encrypted** sessions for privacy

---

## 📦 Docker Compose Deployment

Below is an example `docker-compose.yml` for running Excalidraw locally or behind Traefik:

```yaml
services:
  excalidraw:
    image: excalidraw/excalidraw:latest
    container_name: excalidraw
    restart: unless-stopped
    ports:
      - "5000:80" # Expose on http://<server-ip>:5000
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.excalidraw.rule=Host(`excalidraw.like`)"
      - "traefik.http.routers.excalidraw.entrypoints=web"
      - "traefik.http.services.excalidraw.loadbalancer.server.port=80"
    networks:
      - traefik-net

networks:
  traefik-net:
    external: true
```

---

## 🧭 Accessing Excalidraw

### 🔹 Without Traefik or DNS

Access via the server IP and exposed port:

```
http://<server-ip>:5000
```

---

### 🔹 With Traefik + Local DNS

If you’re using [Traefik](https://doc.traefik.io/traefik/) and a local DNS service (like **CoreDNS** or **Pi-hole**), you can map a custom hostname:

```
http://excalidraw.like
```

> 📌 For DNS and Traefik setup, see [../01-Infra-config/networking.md](../01-Infra-config/networking.md).

---

### ✅ Tips

- Use HTTPS with Traefik for secure collaboration if exposing it outside your LAN.
- Excalidraw supports exporting boards to **.png**, **.svg**, and **.excalidraw** formats.
- For collaborative mode, make sure your instance is reachable from all participants’ devices.
