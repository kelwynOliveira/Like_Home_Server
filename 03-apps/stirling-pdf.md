# 🗂️ Stirling-PDF – Self-Hosted PDF Manipulation Tool

[Stirling-PDF](https://www.stirlingpdf.com) is a robust, locally hosted, web-based PDF manipulation tool running via Docker.
It allows you to perform a wide range of operations on PDF files, including:

- Splitting and merging
- Rotating and reordering pages
- Adding or removing images
- Converting between PDF and other formats
- Compressing and repairing files
- Extracting text or images
- Applying OCR, watermarks, and more

This tool is **self-hosted**, meaning all processing happens locally on your server:

- Files only exist in memory during execution or temporarily on disk.
- Files are deleted once they are downloaded by the user.
- No external cloud services are required.

Homepage: [https://stirlingpdf.com](https://stirlingpdf.com)
Documentation: [https://docs.stirlingpdf.com](https://docs.stirlingpdf.com)

---

## 🛠️ Prerequisites

Before you start, make sure you have:

- Docker and Docker Compose installed on your server.
- (Optional) HTTPS reverse proxy like **Traefik** for secure access.
- (Optional) A domain pointing to your server with TLS certificates (Let's Encrypt or Cloudflare).

📄 For reverse proxy and networking setup, see:

- [01-Infra-config/https](01-Infra-config/https.md)
- [../01-Infra-config/networking.md](../01-Infra-config/networking.md)

---

## 📦 Docker Compose Example

Here’s a working `docker-compose.yml` example to deploy **Stirling-PDF**:

```yaml
services:
  stirling-pdf:
    image: docker.stirlingpdf.com/stirlingtools/stirling-pdf:latest
    container_name: stirling-pdf
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - /home/like/data/StirlingPDF/trainingData:/usr/share/tessdata # OCR languages
      - /home/like/data/StirlingPDF/extraConfigs:/configs
      - /home/like/data/StirlingPDF/customFiles:/customFiles/
      - /home/like/data/StirlingPDF/logs:/logs/
      - /home/like/data/StirlingPDF/pipeline:/pipeline/
    environment:
      - DISABLE_ADDITIONAL_FEATURES=false
      - LANGS=pt_BR
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.stirling-pdf.rule=Host(`stirling-pdf.like`)"
      - "traefik.http.routers.stirling-pdf.entrypoints=web"
      - "traefik.http.services.stirling-pdf.loadbalancer.server.port=8080"
    networks:
      - traefik-net

networks:
  traefik-net:
    external: true
```

---

## 🧭 How to Access Stirling-PDF

Once the container is running:

### 🔹 Without Traefik or DNS

Access via your server IP and mapped port:

```
http://<server-ip>:8080
```

### 🔹 With Traefik + Local

If using a reverse proxy and domain (like `stirling-pdf.like`):

```
http://stirling-pdf.like
```

---

## 🔒 Notes & Tips

- Supports **50+ PDF operations** including merge, split, convert, compress, OCR, and more.
- Supports **custom pipelines** for automated multi-step processing.
- Optional **user authentication** and **enterprise features** like SSO are available.
- Data privacy: files are processed locally and automatically deleted after download.

---

## 📚 References

- 🏠 Homepage: [stirlingpdf.com](https://www.stirlingpdf.com)
- 📖 Documentation: [docs.stirlingpdf.com](https://docs.stirlingpdf.com)
- 📦 Project repo: [Stirling-PDF on GitHub](https://github.com/Stirling-Tools/Stirling-PDF)
