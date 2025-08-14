# 🚀 Deploying Nexus Repository with SSL (2025 Updated Guide)

This guide will walk you through deploying **Sonatype Nexus Repository Manager 3** using Docker Compose, secured with a **self-signed SSL certificate**. This documentation has been updated to align with the latest best practices and image versions in **2025**.

## ✅ Prerequisites

Before you begin, ensure you have the following:

* 🐳 Docker & Docker Compose (v2.20+ recommended)
* 🔐 OpenSSL installed
* 🧠 Basic understanding of Docker volumes and container networking

## 🔧 Step 1: Generate Self-Signed SSL Certificates

### 📥 Install OpenSSL (if not already installed)

* **Ubuntu/Debian**:

```bash
sudo apt update && sudo apt install openssl
```

* **CentOS/RHEL**:

```bash
sudo yum install openssl
```

* **Windows**:
  Download the installer from [OpenSSL Website](https://slproweb.com/products/Win32OpenSSL.html).

### 🔐 Generate the Private Key

```bash
openssl genrsa -out private.key 2048
```

### 📝 Create a Certificate Signing Request (CSR)

```bash
openssl req -new -key private.key -out certificate.csr
```

🔍 Ensure **Common Name (CN)** matches your domain (e.g., `registry.example.com`).

### 🧾 Create the Self-Signed Certificate

```bash
openssl x509 -req -days 365 -in certificate.csr -signkey private.key -out certificate.crt
```

### 📁 Organize Certificates

```bash
mkdir certs
mv certificate.crt private.key certs/
```

## 🐳 Step 2: Deploy Nexus with Docker Compose

### 🧾 docker-compose.yaml (Updated 2025)

```yaml
version: '3.9'

services:
  nexus:
    image: sonatype/nexus3:latest
    container_name: nexus
    restart: unless-stopped
    ports:
      - "8081:8081"
      - "5000:5000"
    volumes:
      - nexus-data:/nexus-data
    environment:
      - INSTALL4J_ADD_VM_PARAMS=-Xms1200m -Xmx1200m -XX:MaxDirectMemorySize=2g -Djava.util.prefs.userRoot=/nexus-data/javaprefs
    networks:
      - nexus-network

  nginx:
    image: nginx:stable-alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./registry.conf:/etc/nginx/conf.d/default.conf:ro
      - ./certs:/etc/ssl:ro
    depends_on:
      - nexus
    networks:
      - nexus-network

volumes:
  nexus-data:
    driver: local

networks:
  nexus-network:
    driver: bridge
```

### ⚙️ Nginx Config - `registry.conf`

```nginx
server {
    listen 80;
    server_name registry.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name registry.example.com;

    ssl_certificate /etc/ssl/certificate.crt;
    ssl_certificate_key /etc/ssl/private.key;

    client_max_body_size 1G;

    location / {
        proxy_pass http://nexus:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> 🛠 **Note**: Make sure your domain (e.g., `registry.example.com`) points to the server's public IP using proper DNS records.

## ▶️ Step 3: Launch Services

```bash
docker-compose up -d
```

## 🔍 Step 4: Access and Validate

* Access Nexus UI: [http://localhost:8081](http://localhost:8081)

  * Default username: `admin`
  * Password: Located in `/nexus-data/admin.password`

* Test Registry via Nginx:

  * Go to: [https://registry.example.com](https://registry.example.com)
  * Accept the self-signed certificate warning if prompted

## 🧪 Troubleshooting

To debug issues:

```bash
docker-compose logs nexus
```

```bash
docker-compose logs nginx
```

Check certificate validity:

```bash
openssl x509 -in certs/certificate.crt -text -noout
```

## 📝 Additional Notes

* 🔁 **Restart Policies**: Added `restart: unless-stopped` to ensure services auto-restart.
* 🧊 **Alpine Image**: Nginx uses `stable-alpine` for smaller size and better stability.
* 🔒 **Security Tip**: For production, consider using a certificate from Let's Encrypt instead of self-signed.
* 🌐 **Firewall Rules**: Ensure ports 80 and 443 are open on your server.

## ✅ Final Thoughts

You now have a secure, private Nexus registry running with HTTPS support! 🧰💡 This setup is ideal for development teams needing artifact management with Docker image hosting.

🔗 For more details:

* Nexus Docs: [https://help.sonatype.com/repomanager3](https://help.sonatype.com/repomanager3)
* Docker Hub: [https://hub.docker.com/r/sonatype/nexus3](https://hub.docker.com/r/sonatype/nexus3)
* Nginx Docs: [https://nginx.org/en/docs/](https://nginx.org/en/docs/)

Happy DevOps-ing! 🎉

## 🧑‍💻 **nexus-nginx-reverse-proxy Folder**

The **`nexus-nginx-reverse-proxy`** folder contains the configuration for setting up Nginx as a reverse proxy to securely handle traffic to your Nexus services. The Nginx configuration file (`registry.conf`) defines how to forward traffic from HTTPS to the Nexus repository, ensuring a secure and efficient setup.

### **To deploy this:**

* Navigate to the **nexus-nginx-reverse-proxy** folder.
* Ensure you have the required SSL certificates and update the Nginx configuration as needed.
* Follow the steps to deploy the services as outlined above.


> ## 📝 About the Author
> #### Crafted with care and ❤️ by [Ali Rahmati](https://github.com/alirahmti). 👨‍💻
> If this repo saved you time or solved a problem, a ⭐ means everything in the DevOps world. 🧠💾
> Your star ⭐ is like a high five from the terminal — thanks for the support! 🙌🐧
