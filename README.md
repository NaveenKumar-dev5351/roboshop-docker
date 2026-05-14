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
