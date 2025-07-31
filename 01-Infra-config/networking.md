# 🌐 Networking Setup

Needed:
[📡 Static IP setup](./static_ip.md)
[🐳 Docker Installation](.README.md#-docker-installation)

Optional (Portainer)
[🐳 Portainer Setup for Docker UI](../02-docker/README.md)

This guide explains how to:

- 🧭 Use **CoreDNS** to resolve internal domains (e.g. `home.like`).
  > The .like TLD is a personalized touch, combining my wife Lídia's name with mine, Kelwyn.
- 🚦 Configure **Traefik** as a reverse proxy
- 🔐 Optionally add **HTTPS** with internal certificates

---

## 📡 Step 1: Set Static IP

Ensure your server uses a fixed IP so all DNS and routing are stable. [📡 Static IP setup](./static_ip.md)

---

## 🧭 Step 2: CoreDNS (Local DNS Resolver)

This allows mapping custom domains like `home.like` to your server's static IP.

### disable systemd-resolved

1. stop the service systemd-resolved:

```bash
sudo systemctl stop systemd-resolved
```

2. disable the service so it does not auto start on restart

```bash
sudo systemctl disable systemd-resolved
```

3. Remove simbol link `resolv.conf`

```bash
cat /etc/resolv.conf
sudo rm /etc/resolv.conf
```

4. Create a new `resolv.conf`

```bash
sudo nano /etc/resolv.conf
```

5. Add DNSs:

```bash
nameserver 192.168.1.50
nameserver 192.168.1.51
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
```

6. Restart Docker service

```bash
sudo systemctl restart docker
```

### 📝 Corefile (DNS Config)

```hcl
like:53 {
  log
  errors
  reload

  template IN A {
    match ^(.+\.)like
    answer "{{ .Name }} 60 IN A 192.168.1.100"
  }

}

.:53 {
  errors
  reload

  forward . 192.168.1.100:8053
}
```

> - The 192.168.1.100:8053 address directs queries to Pi-hole ([🚫 Pi-hole DNS Ad Blocker Setup](./pihole.md)). You can easily swap this with any other DNS server (e.g., 8.8.8.8, 1.1.1.1, or your router's IP) if you prefer not to use Pi-hole.
> - This means any query not for '.like' will be sent to your Pi-hole (or similar)

### 📦 Docker compose (Basic)

```yaml
# docker/coredns/docker-compose.yml
services:
  coredns:
    image: coredns/coredns
    container_name: coredns
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
    volumes:
      - ./Corefile:/Corefile
```

### ▶️ Start

```bash
cd docker/coredns
docker-compose up -d
```

### 🧪 Test on client machine

On another device:

> ⚠️ To make it the default resolver for the client (dev machine), set DNS manually in your OS or router (point to `192.168.1.50` or `192.168.1.51`) [Set DNS on your personal device (dev machine)](./networking.md#-set-dns-on-your-personal-device-dev-machine).

```bash
dig @192.168.1.51 home.like
dig @192.168.1.50 home.like
dig @192.168.1.51 google.com
dig @192.168.1.50 google.com
```

You should get `192.168.1.50` or `192.168.1.51` as a response.

---

### 🧩 Set DNS on your personal device (dev machine)

To access domains like `http://home.like` from your personal computer, follow these steps:

1. **Set the DNS manually to your server's IP (e.g., `192.168.1.50`):**

   - macOS:

     - System Settings → Network → Wi-Fi (or Ethernet) → Details → DNS
     - Add `192.168.1.50` as the first DNS, and optionally `8.8.8.8` as fallback.

   - Windows:

     - Settings → Network & Internet → Change Adapter Options
     - Right-click on your active connection → Properties → Select `Internet Protocol Version 4 (TCP/IPv4)` → Properties
     - Use custom DNS: Primary `192.168.1.50`, Secondary `8.8.8.8`.

   - Linux (NetworkManager):

     ```bash
     nmcli connection modify <connection-name> ipv4.dns "192.168.1.50 8.8.8.8"
     nmcli connection up <connection-name>
     ```

2. **Reconnect to the network:**
   Disconnect and reconnect your Wi-Fi or restart the computer to apply DNS changes.

3. **Flush DNS cache (optional but recommended):**

   - macOS:

     ```bash
     sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
     ```

   - Windows:

     ```cmd
     ipconfig /flushdns
     ```

   - Linux (systemd):

     ```bash
     sudo systemd-resolve --flush-caches
     ```

4. **Test it:**
   Open your browser and access:

   ```url
   http://home.like
   ```

   Or test via terminal:

   ```bash
   dig home.like
   ```

---

## 🚦 Step 3: Traefik Reverse Proxy

Allows routing requests like `http://home.like` to the correct container.

[traefik releases](https://doc.traefik.io/traefik/deprecation/releases/)

### 📦 Docker compose

```yaml
# docker/traefik/docker-compose.yml
services:
  traefik:
    image: traefik:v2.11
    container_name: traefik
    command:
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --api.dashboard=true
      - --log.level=DEBUG
      - --api.insecure=true
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`traefik.like`)"
      - "traefik.http.routers.traefik.entrypoints=web"
      - "traefik.http.services.traefik.loadbalancer.server.port=8080"
      - "traefik.http.routers.traefik.service=api@internal"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080" # Traefik dashboard
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/ssd/data/traefik:/data
    networks:
      - traefik-net

networks:
  traefik-net:
    external: true
```

### 🏗️ Example App with Labels

```yaml
# docker/whoami/docker-compose.yml
services:
  whoami:
    image: traefik/whoami
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`home.like`)"
      - "traefik.http.routers.whoami.entrypoints=web"
    networks:
      - traefik-net

networks:
  traefik-net:
    external: true
```

Start traefik network:

```bash
docker network create traefik-net
```

Start both:

```bash
docker-compose -f docker/traefik/docker-compose.yml up -d
docker-compose -f docker/whoami/docker-compose.yml up -d
```

Now open `http://home.like` in your browser.

---

Next: [🚫 Pi-hole DNS Ad Blocker Setup](./pihole.md)

Previous: [📡 Static IP setup](./static_ip.md)
