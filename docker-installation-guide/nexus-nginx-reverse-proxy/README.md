
# 🚀 Nexus Registry with External Nginx Reverse Proxy (SSL)

This guide explains how to deploy **Nexus Repository Manager** as a **private Docker registry** on one server,  
and use **a separate Nginx server** to handle **SSL termination** and **reverse proxying** for both Nexus UI and Docker registry traffic.



## 🏗️ Architecture

```
\[ Client 🖥️ ] ⇆ \[ 🌐 Nginx Server (192.168.1.100) ]
⇅
\[ 📦 Nexus Server (192.168.1.110) ]
```

- **Nginx Server** (192.168.1.100)  
  - Handles HTTPS (port 443) and HTTP → HTTPS redirect  
  - Proxies requests to Nexus UI (port 8081) and Docker registry (port 5000)  
  - Holds SSL certificates  

- **Nexus Server** (192.168.1.110)  
  - Runs `sonatype/nexus3` Docker container  
  - Exposes ports 8081 (UI) & 5000 (Docker Registry)  
  - No Nginx inside Nexus server (to avoid conflicts)  



## 📌 Why This Setup?

- 🔐 **SSL Termination** handled by a single Nginx server for simplicity
- 🧩 **Separation of concerns** — Nexus focuses on artifacts, Nginx focuses on traffic management
- ⚡ Avoids having multiple Nginx instances (less overhead, fewer conflicts)
- 📦 Works for both **UI access** and **Docker push/pull**



## 🖥️ Nexus Server Setup

**IP:** `192.168.1.110`

**Docker Compose file:**

```yaml
version: '3.3'
services:
  nexus:
    image: sonatype/nexus3:latest
    container_name: nexus
    ports:
      - "8081:8081"
      - "5000:5000"
    volumes:
      - nexus-data:/nexus-data
    environment:
      - INSTALL4J_ADD_VM_PARAMS=-Xms1200m -Xmx1200m -XX:MaxDirectMemorySize=2g -Djava.util.prefs.userRoot=/nexus-data/javaprefs
    restart: unless-stopped
    networks:
      - registry-network

volumes:
  nexus-data:
    driver: local

networks:
  registry-network:
    driver: bridge
````

**Deploy Nexus:**

```bash
docker-compose up -d
```



## 🌐 Nginx Server Setup

**IP:** `192.168.1.100`

1️⃣ Install Nginx:

```bash
sudo apt update && sudo apt install nginx -y
```

2️⃣ Place SSL certificate files:

```
/etc/ssl/certs/fullchain.pem
/etc/ssl/private/privkey.pem
```

3️⃣ Create Nginx server block:

**`/etc/nginx/conf.d/registry.example.com.conf`**

```nginx
server {
    listen 443 ssl;
    server_name registry.example.com;
    
    ssl_certificate /etc/ssl/certs/fullchain.pem;
    ssl_certificate_key /etc/ssl/private/privkey.pem;
    
    client_max_body_size 4000m;

    # Nexus Web UI (8081)
    location / {
        proxy_pass http://192.168.1.110:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_buffering off;
    }

    # Docker Registry (5000)
    location /v2/ {
        proxy_pass http://192.168.1.110:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_buffering off;
    }
}

server {
    listen 80;
    server_name registry.example.com;
    return 301 https://$host$request_uri;
}
```

4️⃣ Test and reload:

```bash
sudo nginx -t
sudo systemctl reload nginx
```



## 🔑 Using the Registry

**Login:**

```bash
docker login registry.example.com
```

**Tag & Push:**

```bash
docker tag my-image:latest registry.example.com/my-repo/my-image:latest
docker push registry.example.com/my-repo/my-image:latest
```

**Pull:**

```bash
docker pull registry.example.com/my-repo/my-image:latest
```


## 🧠 Notes & Best Practices

* Ensure **ports 443 & 80** are open on Nginx server
* Use **DNS A record**: `registry.example.com → 192.168.1.100`
* On Nexus:

  * Enable **Docker Hosted** repo
  * Enable **HTTPS** under **HTTP/HTTPS connectors** in Admin UI (optional)
* Keep `client_max_body_size` large enough for your images
* Regularly back up `/nexus-data` volume



## 📚 References

* 📖 [Nexus Repository Manager Documentation](https://help.sonatype.com/repomanager3)
* 📖 [Docker Registry Docs](https://docs.docker.com/registry/)
* 📖 [Nginx Reverse Proxy Guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

> ## 📝 About the Author
> #### Crafted with care and ❤️ by [Ali Rahmati](https://github.com/alirahmti). 👨‍💻
> If this repo saved you time or solved a problem, a ⭐ means everything in the DevOps world. 🧠💾
> Your star ⭐ is like a high five from the terminal — thanks for the support! 🙌🐧
