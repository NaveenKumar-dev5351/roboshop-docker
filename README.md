# 🐳 Roboshop — Docker Containerization

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![NodeJS](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=java&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=for-the-badge&logo=mongodb&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-005C84?style=for-the-badge&logo=mysql&logoColor=white)

## 🎯 Project Overview
Roboshop is a full-stack e-commerce microservices 
application fully containerized using Docker and 
Docker Compose. Each microservice runs in its own 
container with proper networking, security hardening,
volume management, and environment configuration.
Implements Docker best practices including multi-stage
builds, non-root users, and Alpine base images.

## 🏗️ Architecture
┌─────────────┐
                │   Browser   │
                └──────┬──────┘
                       │ :80
                ┌──────▼──────┐
                │   Frontend  │
                │(Nginx:Alpine)│
                └──────┬──────┘
                       │
     ┌─────────────────┼──────────────────┐
     │                 │                  │
     ┌─────────────┐
                │   Browser   │
                └──────┬──────┘
                       │ :80
                ┌──────▼──────┐
                │   Frontend  │
                │(Nginx:Alpine)│
                └──────┬──────┘
                       │
     ┌─────────────────┼──────────────────┐
     │                 │                  │
     ┌──────▼──────┐  ┌──────▼──────┐  ┌───────▼──────┐
│  Catalogue  │  │    User     │  │     Cart     │
│(Node:Alpine)│  │(Node:Alpine)│  │ (Node:Alpine)│
└──────┬──────┘  └──────┬──────┘  └───────┬──────┘
│                │                  │
┌──────▼──────┐  ┌──────▼──────┐  ┌───────▼──────┐
│   MongoDB   │  │    MySQL    │  │    Redis     │
└─────────────┘  └─────────────┘  └──────────────┘
│
┌─────────────┼──────────────┐
│             │              │
┌──────▼─────┐ ┌────▼──────┐ ┌────▼──────┐
│  Shipping  │ │  Payment  │ │ RabbitMQ  │
│   (Java)   │ │  (Python) │ │           │
└────────────┘ └───────────┘ └───────────┘
## 🛠️ Tech Stack

| Component | Technology | Base Image | Port |
|---|---|---|---|
| Frontend | Nginx | nginx:alpine | 80 |
| Catalogue | Node.js | node:20-alpine3.21 | 8080 |
| User | Node.js | node:20-alpine3.21 | 8080 |
| Cart | Node.js | node:20-alpine3.21 | 8080 |
| Payment | Python | python:alpine | 8080 |
| Shipping | Java | maven:alpine | 8080 |
| MongoDB | Database | mongo | 27017 |
| MySQL | Database | mysql | 3306 |
| Redis | Cache | redis:alpine | 6379 |
| RabbitMQ | Messaging | rabbitmq | 5672 |

## 🐳 Docker Best Practices Implemented

### 1. Multi-Stage Builds
```dockerfile
# Stage 1 — Builder
FROM node:20-alpine3.21 AS builder
WORKDIR /opt/server
COPY package.json .
RUN npm install
COPY *.js .

# Stage 2 — Production
FROM node:20-alpine3.21
COPY --from=builder /opt/server /opt/server
```
Reduces final image size by ~60% — 
build tools and npm cache excluded from production image.

### 2. Non-Root User Security
```dockerfile
RUN addgroup -S roboshop && \
    adduser -S roboshop -G roboshop
USER roboshop
```
All services run as dedicated non-root user — 
eliminates container escape risks.

### 3. Alpine Minimal Base Images
- `node:20-alpine3.21` — ~180MB vs 900MB+ full image
- `python:alpine` — minimal Python runtime
- `nginx:alpine` — minimal web server

### 4. Layer Caching Optimization
```dockerfile
COPY package.json .    # Copy dependencies first
RUN npm install        # Cache this layer
COPY *.js .            # Source code last
```
Dependencies cached unless package.json changes —
speeds up subsequent builds significantly.

### 5. Environment Variable Configuration
```dockerfile
ENV MONGO="true" \
    MONGO_URL="mongodb://mongodb:27017/catalogue"
```
All service configuration via environment variables —
12-factor app methodology.

## 📁 Project Structure
roboshop-docker/
├── docker-compose.yaml     # Multi-container orchestration
├── catalogue/
│   ├── Dockerfile          # Multi-stage Node.js build
│   ├── package.json        # Dependencies
│   └── server.js           # Application code
├── cart/
│   ├── Dockerfile          # Multi-stage Node.js build
│   ├── package.json
│   └── server.js
├── user/
│   ├── Dockerfile          # Multi-stage Node.js build
│   ├── package.json
│   └── server.js
├── payment/
│   ├── Dockerfile          # Python service
│   ├── payment.py          # Payment logic
│   ├── payment.ini         # Configuration
│   └── rabbitmq.py         # Message queue integration
├── shipping/
│   └── Dockerfile          # Java Maven build
├── Mongodb/
│   ├── Dockerfile          # MongoDB with seed data
│   └── master-data.js      # Initial data
├── mysql/
│   └── Dockerfile          # MySQL with schema
└── frontend/
└── Dockerfile          # Nginx configuration
## 🚀 How to Run

### Prerequisites
```bash
# Install Docker
curl -fsSL https://get.docker.com | sh

# Install Docker Compose
sudo apt install docker-compose-plugin

# Verify installation
docker --version
docker compose version
```

### Quick Start
## 🔗 Service Dependencies
mongodb ←── catalogue
mongodb ←── user ←── redis
redis   ←── cart ←── catalogue
mysql   ←── shipping ←── cart
rabbitmq ←── payment ←── user, cart
frontend ←── all services

## 🌐 Docker Network
All services communicate through custom bridge 
network named `roboshop`. Only frontend exposes 
port 80 externally — all other services are 
internal only for security.

## 💾 Persistent Volumes
| Volume | Service | Data |
|---|---|---|
| mongodb | MongoDB | Product catalogue data |
| mysql | MySQL | User and order data |
| redis | Redis | Session cache |
| rabbitmq | RabbitMQ | Message queue data |

```bash
# Clone repository
git clone https://github.com/NaveenKumar-dev5351/roboshop-docker.git
cd roboshop-docker

# Start all services
docker compose up -d

# Check all containers running
docker compose ps

# View logs
docker compose logs -f

# Access application
open http://localhost
```

### Useful Commands
```bash
# Build specific service
docker compose build catalogue

# Restart specific service
docker compose restart payment

# View service logs
docker compose logs -f shipping

# Check resource usage
docker stats

# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v
```

## 🔍 Verify Services Running
```bash
# Check all containers
docker compose ps

# Expected output:
# catalogue    running   0.0.0.0:8080->8080/tcp
# user         running   0.0.0.0:8081->8080/tcp
# cart         running   0.0.0.0:8082->8080/tcp
# payment      running   0.0.0.0:8083->8080/tcp
# shipping     running   0.0.0.0:8084->8080/tcp
# frontend     running   0.0.0.0:80->80/tcp
# mongodb      running   27017/tcp
# mysql        running   3306/tcp
```

## 🔧 Common Issues & Solutions

| Issue | Cause | Solution |
|---|---|---|
| Port already in use | Another service using port | `docker compose down` then restart |
| Container not starting | Check logs | `docker compose logs service-name` |
| DB connection failed | DB not ready | Add `depends_on` with health check |
| Out of memory | Docker memory limit | Increase Docker memory to 4GB+ |
| Image pull failed | Network issue | `docker pull` individually |

## 📈 Image Size Comparison

| Service | Single Stage | Multi-Stage | Reduction |
|---|---|---|---|
| Catalogue | ~950MB | ~180MB | ~81% |
| User | ~950MB | ~180MB | ~81% |
| Cart | ~950MB | ~180MB | ~81% |
| Payment | ~920MB | ~50MB | ~95% |

## 🔗 Related Projects
- [Roboshop AWS Infrastructure](https://github.com/NaveenKumar-dev5351/roboshop-aws-infrastructure)
- [Roboshop Ansible](https://github.com/NaveenKumar-dev5351/ansible-roboshop)
- [Roboshop Shell](https://github.com/NaveenKumar-dev5351/shell-roboshop)

## 👨‍💻 Author
**Naveen Kumar Lingampelly**
DevOps Engineer | [LinkedIn](https://linkedin.com/in/naveenlingampelli) | 
[GitHub](https://github.com/NaveenKumar-dev5351)