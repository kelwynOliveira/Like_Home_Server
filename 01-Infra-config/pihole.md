# 🚫 Pi-hole DNS Ad Blocker Setup

Needed:
[📡 Static IP setup](./static_ip.md)
[🐳 Docker Installation](.README.md#-docker-installation)

Optional (Portainer):
[🐳 Portainer Setup for Docker UI](../02-docker/README.md)

This guide explains how to:

- 🧼 Block ads, trackers, and malicious domains at the network level
- 🔌 Run **Pi-hole** in Docker with persistent data
- 🌐 Use it as your main DNS resolver (locally or on your whole network)

---

## 🧱 Step 1: Prepare the Environment

Before setting up Pi-hole:

- Ensure your server has a [static IP](./static_ip.md) (e.g., `192.168.1.50`)
- Disable `systemd-resolved` if you want Pi-hole to bind to port 53 directly (skip if you're using a custom DNS setup like [CoreDNS](./networking.md#-corefile-dns-config))

Optional but recommended:

- Create a custom Docker network if using reverse proxy (e.g., `traefik-net`) ([Traefik Setup](./networking.md#-step-3-traefik-reverse-proxy))
- Use volumes to persist configuration and logs

---

## 📦 Step 2: Docker Compose

A production-ready setup for Pi-hole:

```yaml
# docker/pihole/docker-compose.yml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    restart: unless-stopped
    environment:
      TZ: "America/Sao_Paulo"
      DNSMASQ_LISTENING: all
      FTLCONF_dns_listeningMode: "all"
      FTLCONF_webserver_api_password: "password"
      FTLCONF_LOCAL_IPV4: 192.168.1.50
    volumes:
      - /mnt/ssd/data/pihole/etc-pihole:/etc/pihole
      - /mnt/ssd/data/pihole/etc-dnsmasq.d:/etc/dnsm
    ports:
      # - "53:53/tcp"
      # - "53:53/udp"
      - "8053:53/tcp"
      - "8053:53/udp"
      # - "80:80/tcp"
      # - "443:443/tcp"
      - "8180:80"

    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.pihole.rule=Host(`pihole.like`)"
      - "traefik.http.routers.pihole.entrypoints=web"
      - "traefik.http.services.pihole.loadbalancer.server.port=80"
      # (Opcional) Redirect para HTTPS, se você usa websecure e TLS:
      # - "traefik.http.routers.pihole.entrypoints=web,websecure"
      # - "traefik.http.routers.pihole.tls=true"
    networks:
      - traefik-net

networks:
  traefik-net:
    external: true
```

📝 Notes:

- `FTLCONF_LOCAL_IPV4` ensures Pi-hole reports the correct host IP instead of its internal Docker IP.
- Ports 53, 80, and 443 are commented out to avoid conflicts with other services like CoreDNS or Traefik.
- Pi-hole Web UI is accessible at `http://192.168.1.50:8180`
- Uses port `8053` for DNS to avoid conflicts with CoreDNS
- Persistent data stored in `/mnt/ssd/data/pihole/`

---

## ▶️ Step 3: Start the Container

```bash
cd docker/pihole
docker compose up -d
```

Access the web interface at:

```url
http://192.168.1.50:8180
```

Use the password defined in `FTLCONF_webserver_api_password`

---

## 🧪 Step 4: Test DNS Resolution

Verify DNS functionality:

```bash
dig @192.168.1.50 google.com
dig @192.168.1.50 doubleclick.net
```

Expected output:

- `google.com`: valid IP
- `doubleclick.net`: blocked result (e.g., `0.0.0.0` or `NXDOMAIN`)

---

## 🌍 Optional: Use Pi-hole as your Default DNS

### Router-wide

Set your router to use `192.168.1.50` as the primary DNS. All devices on the network will follow.

### Per device

Manually configure DNS on your devices (see [Networking Guide](../networking/README.md)).

---

## 🔐 Optional: Use Traefik (Recommended)

To access Pi-hole via a custom domain (e.g., `pihole.like`):

1. Ensure CoreDNS is resolving `.like` domains
2. Use these Traefik labels:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.pihole.rule=Host(`pihole.like`)"
  - "traefik.http.routers.pihole.entrypoints=web"
  - "traefik.http.services.pihole.loadbalancer.server.port=80"
```

Access via:

```url
http://pihole.like
```

---

## 🧹 Useful Commands

- View logs:

```bash
docker logs -f pihole
```

- Restart the container:

```bash
docker compose restart
```

- Change password:

```bash
docker exec -it pihole pihole -a -p
```

---

✅ Your home network now has DNS-level ad-blocking!
Combine it with CoreDNS and Traefik for a complete local infrastructure.

---

Previous: [🌐 Networking Setup](./networking.md)
