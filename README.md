
# 🚀 Nexus Registry Deployment Guide

This repository provides deployment examples for **[Nexus Repository Manager](https://www.sonatype.com/products/sonatype-nexus-repository)** as a private container registry and artifact store.

You’ll find:
- 📦 **Docker Compose configurations**
- 🌐 **Reverse proxy setups**
- ☁️ Future **Kubernetes manifests** — all in one place.


## 📌 What is Nexus Registry?

**Nexus Repository Manager** (often just called **Nexus**) is a universal artifact repository by Sonatype that allows you to store, manage, and distribute different kinds of software components in a single place.

When we talk about a **Nexus Registry** in this context, we specifically mean **using Nexus as a private Docker registry** — a secure, self-hosted location where you can push and pull Docker container images.

Think of it like your **own personal Docker Hub**, but:
- Hosted **on your infrastructure**
- Fully under **your control**
- Able to serve **multiple formats** beyond Docker images (Maven, npm, PyPI, Helm charts, etc.)

In DevOps workflows, Nexus can:
- Store your application container images for **deployment to staging/production**
- Cache images from public registries so your builds are **faster and resilient to outages**
- Keep sensitive or proprietary images **private and secure**
- Integrate directly into CI/CD pipelines to automate image storage and retrieval



## 💡 Why Use a Private Nexus Registry?

Running your own Nexus registry gives you:

- 🗄 **Unlimited storage** (depends on your infrastructure)
- 🚫 **No Docker Hub pull limits** or downtime dependency
- 🔒 **Security** — control who can push/pull
- ⚡ **Faster builds** — cached images/artifacts
- 🎯 **Multi-format support** — Docker, Maven, npm, PyPI, and more



## 📂 Repository Structure

```
docker-installation-guide/
├── README.md                  # 📘 Installation steps for Nexus Registry
├── docker-compose.yaml        # 🐳 Docker Compose for Nexus
├── registry.conf              # ⚙️ Example config for Nexus registry
└── nexus-registry-reverse-proxy/
├── README.md               # 📘 Steps for Nexus + Nginx Reverse Proxy
├── docker-compose/docker-compose.yaml
└── nginx-reverse-proxy-config/registryexample.com.conf
````

- **`docker-installation-guide/`** → Standalone Nexus Registry (**HTTP**)
- **`nexus-registry-reverse-proxy/`** → Nexus Registry with **Nginx reverse proxy** (**HTTPS** & custom domain)


## ⚡ Quick Start

### 1️⃣ Basic Nexus Registry

```bash
cd docker-installation-guide
docker-compose up -d
````

**Access:**

* 🌐 Nexus UI → [http://localhost:8081](http://localhost:8081)
* 🐳 Docker Registry → [http://localhost:5000](http://localhost:5000)

**Default admin password:**

```bash
docker exec -it nexus cat /nexus-data/admin.password
```



### 2️⃣ Nexus with Nginx Reverse Proxy

```bash
cd docker-installation-guide/nexus-registry-reverse-proxy
docker-compose up -d
```

✨ **Features:**

* 🔐 HTTPS termination
* 🌍 Custom domain support (`registry.example.com`)
* 🛡 Secure pulls/pushes without `--insecure-registry`


## 🛠 Push & Pull Images

🔑 **Login:**

```bash
docker login registry.example.com
```

📤 **Tag & push:**

```bash
docker tag my-image:latest registry.example.com/my-repo/my-image:latest
docker push registry.example.com/my-repo/my-image:latest
```

📥 **Pull:**

```bash
docker pull registry.example.com/my-repo/my-image:latest
```



## 📅 Roadmap

* ✅ **Basic Docker Compose setup**
* ✅ **Reverse Proxy setup with Nginx**
* ⏳ **Kubernetes manifests** for HA deployments
* ⏳ **Automated backups & restore**
* ⏳ **CI/CD integration examples**


## 📚 References

* 📖 [Nexus Repository Manager Documentation](https://help.sonatype.com/repomanager3)
* 📖 [Docker Registry Docs](https://docs.docker.com/registry/)
* 📖 [Nginx Reverse Proxy Guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)



## ⚖️ License

MIT License — see [LICENSE](LICENSE) for details.


> ## 📝 About the Author
> #### Crafted with care and ❤️ by [Ali Rahmati](https://github.com/alirahmti). 👨‍💻
> If this repo saved you time or solved a problem, a ⭐ means everything in the DevOps world. 🧠💾
> Your star ⭐ is like a high five from the terminal — thanks for the support! 🙌🐧
