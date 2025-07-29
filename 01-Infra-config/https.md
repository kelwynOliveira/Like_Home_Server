# 🔒 HTTPS for Local Domains

You can enable HTTPS using self-signed certificates for local development (`mkcert`), or automate valid public certificates with **Let's Encrypt**.

- Option 1 - [Local TLS with `mkcert` (Development Only)](#-option-1--local-tls-with-mkcert-development-only)
- Option 2 - [Automatic HTTPS with Let's Encrypt (Production/Public Domains)](#-option-2--automatic-https-with-lets-encrypt-productionpublic-domains)

---

### ✅ Option 1 — Local TLS with `mkcert` (Development Only)

Use this if you're accessing services like `https://app.like.home` in your local network and want HTTPS without browser warnings.

---

#### 👤 On Your Local Machine (Client)

Install and trust the local certificate authority (CA) so your browser accepts self-signed certificates.

**macOS:**

```bash
brew install mkcert
brew install nss # optional (for Firefox)
mkcert -install
```

**Ubuntu:**

```bash
sudo apt install libnss3-tools
curl -JLO "https://dl.filippo.io/mkcert/latest?for=linux/amd64"
chmod +x mkcert-v*-linux-amd64
sudo mv mkcert-v*-linux-amd64 /usr/local/bin/mkcert
mkcert -install
```

1. Generate certificates for your local domain:

   ```bash
   mkcert like "*.like"
   mv like+1.pem cert.pem
   mv like+1-key.pem key.pem
   ```

---

#### 🖥️ On Your Server (Traefik Host)

1. Generate certificates for your local domain:

   ```bash
   mkcert like "*.like"
   mv _wildcard.like.pem cert.pem
   mv _wildcard.like-key.pem key.pem
   ```

2. Move them to a shared volume (e.g. `/mnt/ssd/data/certs`):

   ```bash
   mkdir -p /mnt/ssd/data/certs
   mv cert.pem key.pem /mnt/ssd/data/certs/
   ```

3. In your `docker-compose.yml`, mount the certs expose port `443` and configure Traefik:

   ```yaml
   services:
     traefik:
       image: traefik:v2.11
       container_name: traefik
       command:
         - --entrypoints.web.address=:80
         - --providers.docker=true
         - --api.dashboard=true
         - --log.level=DEBUG
         - --api.insecure=true
         - --entrypoints.websecure.address=:443
        labels:
         - "traefik.tls.certificates[0].certFile=/certs/cert.pem"
         - "traefik.tls.certificates[0].keyFile=/certs/key.pem"
       ports:
         - "80:80"
         - "443:443"
         - "8080:8080" # Traefik dashboard
       volumes:
         - /var/run/docker.sock:/var/run/docker.sock
         - /mnt/ssd/data/traefik:/data
         - /mnt/ssd/data/certs:/certs
       networks:
         - traefik-net

   networks:
     traefik-net:
       external: true
   ```

4. Add TLS labels to your services:

   ```yaml
   services:
     whoami:
       image: traefik/whoami
       labels:
         - "traefik.enable=true"
         - "traefik.http.routers.whoami.rule=Host(`home.like`)"
         # - "traefik.http.routers.whoami.entrypoints=web"
         - "traefik.http.routers.whoami.entrypoints=websecure"
         - "traefik.http.routers.whoami.tls=true"
       networks:
         - traefik-net

   networks:
     traefik-net:
       external: true
   ```

5. Restart services:

   ```bash
   docker-compose down
   docker-compose up -d
   ```

6. Visit: [https://home.like](https://home.like) — no warnings should appear if your local CA is trusted.

---

### 🌍 Option 2 — Automatic HTTPS with Let's Encrypt (Production/Public Domains)

If you are using a public domain and want valid HTTPS certificates, use Let's Encrypt via Traefik's built-in resolver.

I bought the lik3.net domain and I am using Cloudflare (DNS API) to use DNS-01 with automatic Let’s Encrypt.

#### 1. Create Cloudflare API Token for Traefik:

Go to [https://dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens) and click on _Create Token_.

Select the template _Edit Zone DNS_

Then set _Zone Resouces_: include your domain and save the token on a .env in the same folder where the docker compose file is.

```.env
CF_DNS_API_TOKEN=cloudflare_token
```

#### 2. Disable proxy (Proxied) for domains using Traefik:

Go to DNS -> Records in Cloudflare for your domain then create a new A or CNAME registers (Add Record) with _Proxy status: DNS only_ for your apps. Content can be any value like `192.168.1.100`, since DNS-01 only uses the domain, it does not test IP.

![cloudflare DNS](../assets/cloudflare-dns.png)

#### 3. Configure Traefik to use Let's Encrypt:

```yaml
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
      - --certificatesresolvers.cloudflare.acme.dnschallenge=true
      - --certificatesresolvers.cloudflare.acme.dnschallenge.provider=cloudflare
      - --certificatesresolvers.cloudflare.acme.email=your@email.com
      - --certificatesresolvers.cloudflare.acme.storage=/data/acme.json
    env_file:
      - .env
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080" # Traefik dashboard
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/ssd/data/traefik:/data
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`traefik.lik3.net`)"
      - "traefik.http.services.traefik.loadbalancer.server.port=8080"
      - "traefik.http.routers.traefik.entrypoints=websecure"
      - "traefik.http.routers.traefik.tls.certresolver=cloudflare"
      - "traefik.http.routers.traefik.service=api@internal"
    networks:
      - traefik-net

networks:
  traefik-net:
    external: true
```

#### 4. Give permission to `acme.json`:

```
sudo touch /apps/traefik/data/acme.json
sudo chmod 600 /apps/traefik/data/acme.json
```

#### 5. Add labels to your service/router:

Example:

```yaml
services:
  whoami:
    image: traefik/whoami
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`home.like`)"
      - "traefik.http.routers.whoami.entrypoints=web"
      - "traefik.http.routers.whoami.entrypoints=websecure"
      - "traefik.http.routers.whoami.tls=true"
      - "traefik.http.routers.myapp.tls.certresolver=letsencrypt"
    networks:
      - traefik-net

networks:
  traefik-net:
    external: true
```
