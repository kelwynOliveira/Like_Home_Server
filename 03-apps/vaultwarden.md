# 🛡️ Vaultwarden - Self-Hosted Password Manager

Vaultwarden is a lightweight, self-hosted alternative to Bitwarden — a popular open-source password manager. It allows you to securely store and sync your passwords, notes, and other credentials across devices using official Bitwarden apps and browser extensions.

> This guide covers how to set up Vaultwarden on a local server using Docker and Traefik, including HTTPS and remote access support.

---

## 🚀 Why Use Vaultwarden?

- 🧠 **Private and Secure**: Run your own password manager without relying on cloud providers.
- ⚡ **Lightweight**: Optimized to use fewer resources than the official Bitwarden server.
- 📱 **Compatible**: Works seamlessly with Bitwarden clients (mobile, desktop, browser extensions).
- 🔐 **Encrypted**: All data is end-to-end encrypted using Bitwarden’s official protocols.

---

## 🛠️ Prerequisites

Before you start, make sure you have the following set up:

- Docker and Docker Compose installed on your local server.
- HTTPS reverse proxy (e.g. **Traefik**) configured.
- A domain pointing to your server like `vw.domain.tld`, in my case it is `vaultwarden.lik3.net`.
- TLS certificates (e.g. using Let's Encrypt or Cloudflare).

📄 See [01-Infra-config/https](01-Infra-config/https.md) and [../01-Infra-config/networking.md](../01-Infra-config/networking.md) for HTTPS and DNS setup details.

---

## 📦 Docker Compose Configuration

Here’s a working `docker-compose.yml` example to deploy Vaultwarden:

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      DOMAIN: "https://vaultwarden.lik3.net"
    volumes:
      - /home/like/data/vaultwarden/data:/data/
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.vaultwarden.rule=Host(`vaultwarden.lik3.net`)"
      - "traefik.http.routers.vaultwarden.entrypoints=websecure"
      - "traefik.http.routers.vaultwarden.tls.certresolver=cloudflare"
      - "traefik.http.services.vaultwarden.loadbalancer.server.port=80"
    networks:
      - traefik-net

networks:
  traefik-net:
    external: true
```

---

## 🧭 How to Access Vaultwarden

Once the container is running and your domain is correctly pointed with HTTPS enabled, you can access Vaultwarden at your domain. Mine is:

🔗 `https://vaultwarden.lik3.net`

You can now log in or create an account using:

### ✅ Official Bitwarden Clients:

- **Web**: Visit your custom domain (e.g. `vaultwarden.lik3.net`)

> ⚠️ For the bellow options remember to set Self-Hosted Environment
> ![Valutwarden self-hosted](/assets/vaultwarden-self-hosted.png)

- **Mobile**: Download the [Bitwarden app](https://bitwarden.com/download/)

  - Go to Settings → _Self-Hosted Environment_ → Enter your domain.

- **Browser Extension**: Install the Bitwarden extension for Chrome/Firefox and configure it to use your custom server.
- **Desktop App**: Works the same — just change the server URL in the settings.

---

## 🔒 Notes & Tips

- First user to register becomes the **admin**. You can enable admin panel access with `ADMIN_TOKEN` environment variable (optional).
- Vaultwarden supports:

  - 2FA (TOTP)
  - File attachments
  - Organizations (shared vaults)
  - WebSocket notifications (optional, for real-time sync)

See full documentation: [Vaultwarden GitHub](https://github.com/dani-garcia/vaultwarden)

---

## 📚 References

- 🧩 HTTPS Setup: [01-Infra-config/https](01-Infra-config/https.md)
- 🌐 DNS & Networking: [../01-Infra-config/networking.md](../01-Infra-config/networking.md)
- 📦 Project repo: [Vaultwarden on GitHub](https://github.com/dani-garcia/vaultwarden)
