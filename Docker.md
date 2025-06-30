> [!NOTE]
> Generated with claude

# Complete Docker Guide: Beginner to Advanced

## Table of Contents
1. [Docker Fundamentals](#docker-fundamentals)
2. [Installation and Setup](#installation-and-setup)
3. [Basic Commands and Operations](#basic-commands-and-operations)
4. [Working with Images](#working-with-images)
5. [Containers Deep Dive](#containers-deep-dive)
6. [Dockerfile Mastery](#dockerfile-mastery)
7. [Data Management](#data-management)
8. [Networking](#networking)
9. [Multi-Container Applications](#multi-container-applications)
10. [Docker Compose](#docker-compose)
11. [Registry and Image Distribution](#registry-and-image-distribution)
12. [Security Best Practices](#security-best-practices)
13. [Production Deployment](#production-deployment)
14. [Monitoring and Logging](#monitoring-and-logging)
15. [Advanced Topics](#advanced-topics)

---

## Docker Fundamentals

### What is Docker?
Docker is a containerization platform that packages applications and their dependencies into lightweight, portable containers. Unlike virtual machines, containers share the host OS kernel, making them more efficient.

### Key Concepts

**Container**: A runtime instance of an image - essentially a lightweight, standalone package that includes everything needed to run an application.

**Image**: A read-only template used to create containers. Images are built from a Dockerfile and can be stored in registries.

**Dockerfile**: A text file containing instructions to build a Docker image.

**Registry**: A storage and distribution system for Docker images (Docker Hub, AWS ECR, etc.).

**Docker Engine**: The runtime that manages containers, images, networks, and volumes.

### Architecture Overview
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Application   │    │   Application   │    │   Application   │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│   Container     │    │   Container     │    │   Container     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
┌─────────────────────────────────────────────────────────────────┐
│                    Docker Engine                                │
├─────────────────────────────────────────────────────────────────┤
│                    Host Operating System                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## Installation and Setup

### Linux Installation
```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# CentOS/RHEL
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
```

### Post-Installation Setup
```bash
# Verify installation
docker --version
docker run hello-world

# Configure Docker daemon
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl reload docker
```

---

## Basic Commands and Operations

### Essential Commands
```bash
# Image operations
docker images                    # List images
docker pull nginx               # Download image
docker rmi nginx               # Remove image
docker image prune             # Remove unused images

# Container operations
docker ps                      # List running containers
docker ps -a                   # List all containers
docker run nginx              # Run container
docker start <container_id>    # Start stopped container
docker stop <container_id>     # Stop container
docker rm <container_id>       # Remove container
docker exec -it <container_id> bash  # Execute command in container

# System information
docker info                    # System-wide information
docker version                # Version information
docker system df              # Disk usage
docker system prune           # Clean up unused resources
```

### Container Lifecycle
```bash
# Run container with options
docker run -d \
  --name web-server \
  -p 8080:80 \
  -v /host/path:/container/path \
  -e ENV_VAR=value \
  nginx

# Container inspection
docker inspect <container_id>  # Detailed container info
docker logs <container_id>     # View container logs
docker stats <container_id>    # Real-time resource usage
```

---

## Working with Images

### Image Basics
```bash
# Search for images
docker search python

# Pull specific version
docker pull python:3.9-alpine

# Image history
docker history python:3.9

# Export/Import images
docker save nginx > nginx.tar
docker load < nginx.tar

# Tag images
docker tag nginx:latest myregistry/nginx:v1.0
```

### Building Images
```bash
# Build from Dockerfile
docker build -t myapp:latest .

# Build with context
docker build -t myapp:latest -f Dockerfile.prod .

# Build with build args
docker build --build-arg VERSION=1.0 -t myapp:latest .

# Multi-stage build example
docker build --target production -t myapp:prod .
```

---

## Containers Deep Dive

### Advanced Container Operations
```bash
# Run with resource limits
docker run -d \
  --name resource-limited \
  --memory=512m \
  --cpus=1.5 \
  --memory-swap=1g \
  nginx

# Run with security options
docker run -d \
  --name secure-container \
  --user 1000:1000 \
  --read-only \
  --tmpfs /tmp \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  nginx

# Container networking
docker run -d \
  --name networked \
  --network custom-network \
  --ip 172.20.0.100 \
  nginx
```

### Container Debugging
```bash
# Debug running container
docker exec -it <container_id> /bin/bash
docker cp file.txt <container_id>:/path/to/destination

# Attach to container
docker attach <container_id>

# View processes
docker top <container_id>

# Container events
docker events --filter container=<container_id>
```

---

## Dockerfile Mastery

### Basic Dockerfile Structure
```dockerfile
# Use official runtime as parent image
FROM python:3.9-slim

# Set metadata
LABEL maintainer="your-email@example.com"
LABEL version="1.0"
LABEL description="Python web application"

# Set working directory
WORKDIR /app

# Copy requirements first (for better caching)
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
RUN chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# Set environment variables
ENV PYTHONPATH=/app
ENV FLASK_ENV=production

# Define volume mount point
VOLUME ["/app/data"]

# Default command
CMD ["python", "app.py"]
```

### Advanced Dockerfile Techniques

#### Multi-Stage Build
```dockerfile
# Build stage
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:16-alpine AS production
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

#### Build Arguments and Environment Variables
```dockerfile
ARG NODE_VERSION=16
FROM node:${NODE_VERSION}-alpine

ARG BUILD_DATE
ARG VCS_REF
LABEL org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.vcs-ref=$VCS_REF

ENV NODE_ENV=production
ENV PORT=3000

# Use build args in RUN commands
ARG USERNAME=appuser
RUN adduser -D $USERNAME
```

### Best Practices for Dockerfiles
```dockerfile
# Use specific tags, not 'latest'
FROM node:16.14-alpine

# Combine RUN statements to reduce layers
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Use .dockerignore
# Create .dockerignore file:
# node_modules
# npm-debug.log
# .git
# .gitignore
# README.md
# .env
# coverage/
# .nyc_output

# Order layers by likelihood of change
COPY package*.json ./
RUN npm ci --only=production
COPY . .
```

---

## Data Management

### Volumes
```bash
# Create named volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Use volume in container
docker run -d -v mydata:/data nginx

# Remove volume
docker volume rm mydata

# Prune unused volumes
docker volume prune
```

### Bind Mounts
```bash
# Bind mount host directory
docker run -d -v /host/path:/container/path nginx

# Read-only bind mount
docker run -d -v /host/path:/container/path:ro nginx

# Mount with specific options
docker run -d --mount type=bind,source=/host/path,target=/container/path,readonly nginx
```

### tmpfs Mounts
```bash
# Create tmpfs mount
docker run -d --tmpfs /tmp nginx

# tmpfs with options
docker run -d --mount type=tmpfs,destination=/tmp,tmpfs-size=100m nginx
```

### Advanced Storage Management
```bash
# Create volume with driver options
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/path/to/dir \
  nfs-volume

# Backup volume
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/backup.tar.gz -C /data .

# Restore volume
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/backup.tar.gz -C /data
```

---

## Networking

### Network Types
```bash
# List networks
docker network ls

# Inspect network
docker network inspect bridge

# Create custom networks
docker network create mynetwork
docker network create --driver bridge --subnet=172.20.0.0/16 custom-bridge
docker network create --driver overlay --attachable swarm-network
```

### Bridge Networks
```bash
# Create bridge network with options
docker network create \
  --driver bridge \
  --subnet=172.20.0.0/16 \
  --ip-range=172.20.240.0/20 \
  --gateway=172.20.0.1 \
  mybridge

# Connect container to network
docker network connect mybridge container1

# Disconnect from network
docker network disconnect mybridge container1
```

### Host and None Networks
```bash
# Use host networking (container shares host network stack)
docker run -d --network host nginx

# No networking
docker run -d --network none alpine
```

### Service Discovery
```bash
# Create network
docker network create app-network

# Run containers on same network
docker run -d --name db --network app-network postgres
docker run -d --name web --network app-network nginx

# Containers can communicate using container names as hostnames
# web container can reach db container at hostname 'db'
```

### Port Mapping
```bash
# Map specific port
docker run -d -p 8080:80 nginx

# Map to random port
docker run -d -P nginx

# Map specific interface
docker run -d -p 127.0.0.1:8080:80 nginx

# Map UDP port
docker run -d -p 8080:80/udp nginx

# Multiple port mappings
docker run -d -p 8080:80 -p 8443:443 nginx
```

---

## Multi-Container Applications

### Container Communication
```bash
# Create shared network
docker network create app-tier

# Database container
docker run -d \
  --name postgres-db \
  --network app-tier \
  -e POSTGRES_DB=myapp \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=password \
  postgres:13

# Application container
docker run -d \
  --name webapp \
  --network app-tier \
  -p 8000:8000 \
  -e DATABASE_URL=postgresql://user:password@postgres-db:5432/myapp \
  myapp:latest
```

### Container Dependencies
```bash
# Wait for dependency
docker run -d --name db postgres:13
docker run -d --name app --link db:database myapp:latest

# Better approach with health checks
docker run -d \
  --name db \
  --health-cmd="pg_isready -U postgres" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  postgres:13

# Use wait scripts or tools like dockerize
```

---

## Docker Compose

### Basic docker-compose.yml
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - .:/code
    environment:
      - DEBUG=1
    depends_on:
      - db
      - redis

  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Advanced Compose Features
```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.prod
      args:
        - BUILD_ENV=production
    image: myapp:latest
    container_name: myapp-web
    restart: unless-stopped
    ports:
      - "80:8000"
    volumes:
      - ./static:/app/static:ro
      - logs:/app/logs
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      db:
        condition: service_healthy
    networks:
      - frontend
      - backend
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: postgres:13
    container_name: myapp-db
    restart: unless-stopped
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 30s
      timeout: 10s
      retries: 5

  nginx:
    image: nginx:alpine
    container_name: myapp-nginx
    restart: unless-stopped
    ports:
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - web
    networks:
      - frontend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true

volumes:
  postgres_data:
    driver: local
  logs:
    driver: local

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Compose Commands
```bash
# Start services
docker-compose up
docker-compose up -d  # detached mode
docker-compose up --build  # rebuild images

# Scale services
docker-compose up --scale web=3

# Stop services
docker-compose stop
docker-compose down  # stop and remove containers
docker-compose down -v  # also remove volumes

# View logs
docker-compose logs
docker-compose logs -f web  # follow logs for specific service

# Execute commands
docker-compose exec web bash
docker-compose run web python manage.py migrate

# Configuration
docker-compose config  # validate and view configuration
docker-compose ps  # list containers
```

### Environment Files
```bash
# .env file
POSTGRES_DB=myapp
POSTGRES_USER=user
POSTGRES_PASSWORD=secret
DEBUG=true
```

```yaml
# Use in docker-compose.yml
services:
  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
```

### Override Files
```yaml
# docker-compose.override.yml (automatically loaded)
version: '3.8'

services:
  web:
    volumes:
      - .:/code
    environment:
      - DEBUG=1

# docker-compose.prod.yml (explicit)
version: '3.8'

services:
  web:
    restart: unless-stopped
    environment:
      - DEBUG=0
```

```bash
# Use specific override file
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

## Registry and Image Distribution

### Docker Hub
```bash
# Login
docker login

# Tag for Docker Hub
docker tag myapp:latest username/myapp:latest
docker tag myapp:latest username/myapp:v1.0

# Push to Docker Hub
docker push username/myapp:latest
docker push username/myapp:v1.0

# Pull from Docker Hub
docker pull username/myapp:latest
```

### Private Registry
```bash
# Run local registry
docker run -d -p 5000:5000 --restart=always --name registry registry:2

# Tag for private registry
docker tag myapp:latest localhost:5000/myapp:latest

# Push to private registry
docker push localhost:5000/myapp:latest

# Pull from private registry
docker pull localhost:5000/myapp:latest
```

### Secure Registry with Authentication
```bash
# Create htpasswd file
mkdir auth
docker run --entrypoint htpasswd registry:2 -Bbn testuser testpassword > auth/htpasswd

# Run registry with authentication
docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v $(pwd)/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  registry:2

# Login to private registry
docker login localhost:5000
```

### Registry with TLS
```bash
# Generate certificates
mkdir certs
openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt

# Run secure registry
docker run -d \
  --restart=always \
  --name registry \
  -v $(pwd)/certs:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -p 443:443 \
  registry:2
```

---

## Security Best Practices

### Image Security
```dockerfile
# Use official base images
FROM node:16-alpine

# Don't run as root
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs

# Use specific versions
FROM node:16.14.0-alpine3.15

# Remove unnecessary packages
RUN apk add --no-cache git \
    && apk del git

# Use multi-stage builds to reduce attack surface
FROM node:16-alpine AS builder
# ... build steps ...

FROM node:16-alpine AS runner
COPY --from=builder /app/dist ./dist
```

### Container Security
```bash
# Run with limited privileges
docker run -d \
  --user 1000:1000 \
  --read-only \
  --tmpfs /tmp \
  --tmpfs /var/run \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  --security-opt no-new-privileges:true \
  nginx

# Use AppArmor/SELinux
docker run -d --security-opt apparmor:docker-default nginx

# Limit resources
docker run -d \
  --memory=512m \
  --cpus=1.0 \
  --pids-limit=100 \
  nginx
```

### Secrets Management
```bash
# Using Docker secrets (Swarm mode)
echo "mysecretpassword" | docker secret create db_password -

# Using environment files
docker run -d --env-file secrets.env myapp

# Using external secret management
docker run -d \
  -v /var/run/secrets:/var/run/secrets \
  myapp
```

### Security Scanning
```bash
# Scan images for vulnerabilities
docker scout cves myapp:latest

# Use security scanning tools
# Clair, Trivy, Snyk, etc.
trivy image myapp:latest
```

### Network Security
```bash
# Use custom networks instead of default bridge
docker network create --driver bridge secure-network

# Internal networks for backend services
docker network create --internal backend-network

# Encrypt swarm networks
docker network create --driver overlay --opt encrypted secure-overlay
```

---

## Production Deployment

### Docker Swarm
```bash
# Initialize swarm
docker swarm init --advertise-addr 192.168.1.100

# Join swarm
docker swarm join --token <token> 192.168.1.100:2377

# Deploy stack
docker stack deploy -c docker-compose.yml myapp

# Scale services
docker service scale myapp_web=5

# Update service
docker service update --image myapp:v2 myapp_web

# Remove stack
docker stack rm myapp
```

### Docker Stack File
```yaml
version: '3.8'

services:
  web:
    image: myapp:latest
    ports:
      - "80:8000"
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      placement:
        constraints:
          - node.role == worker
    networks:
      - webnet
    secrets:
      - source: db_password
        target: /run/secrets/db_password

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet

networks:
  webnet:
    driver: overlay

secrets:
  db_password:
    external: true
```

### Health Checks and Rolling Updates
```yaml
services:
  web:
    image: myapp:latest
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        monitor: 60s
        max_failure_ratio: 0.3
      rollback_config:
        parallelism: 1
        delay: 5s
        failure_action: pause
        monitor: 60s
```

### Configuration Management
```yaml
version: '3.8'

services:
  web:
    image: myapp:latest
    configs:
      - source: nginx_config
        target: /etc/nginx/nginx.conf
      - source: app_config
        target: /app/config.json
    secrets:
      - db_password
      - api_key

configs:
  nginx_config:
    file: ./nginx.conf
  app_config:
    external: true

secrets:
  db_password:
    external: true
  api_key:
    external: true
```

---

## Monitoring and Logging

### Container Logging
```bash
# View logs
docker logs <container_id>
docker logs -f <container_id>  # follow logs
docker logs --since="2023-01-01" <container_id>
docker logs --tail=50 <container_id>

# Configure logging driver
docker run -d \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nginx

# Syslog driver
docker run -d \
  --log-driver=syslog \
  --log-opt syslog-address=tcp://192.168.1.42:514 \
  nginx
```

### Centralized Logging with ELK Stack
```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - esdata:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch

  app:
    image: myapp:latest
    logging:
      driver: "gelf"
      options:
        gelf-address: "udp://localhost:12201"
    depends_on:
      - logstash

volumes:
  esdata:
```

### Monitoring with Prometheus and Grafana
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

volumes:
  prometheus_data:
  grafana_data:
```

### Resource Monitoring
```bash
# Monitor resource usage
docker stats
docker stats --no-stream
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# System events
docker events
docker events --filter container=myapp
docker events --filter type=network

# Container inspection
docker inspect <container_id>
docker top <container_id>
```

---

## Advanced Topics

### Docker BuildKit
```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Build with BuildKit features
docker build --platform linux/amd64,linux/arm64 -t myapp:latest .

# Cache mounts
FROM golang:1.19-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN --mount=type=cache,target=/go/pkg/mod \
    go mod download

# Secret mounts
RUN --mount=type=secret,id=github_token \
    git config --global url."https://$(cat /run/secrets/github_token)@github.com/".insteadOf "https://github.com/"
```

### Multi-Architecture Builds
```bash
# Create builder instance
docker buildx create --name mybuilder --use

# Build for multiple architectures
docker buildx build \
  --platform linux/amd64,linux/arm64,linux/arm/v7 \
  -t myapp:latest \
  --push .

# Inspect builder
docker buildx inspect mybuilder
```

### Container Runtime Security
```bash
# Use different runtime
docker run -d --runtime=runsc nginx  # gVisor
docker run -d --runtime=kata-runtime nginx  # Kata Containers

# Rootless Docker
dockerd-rootless-setuptool.sh install
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
```

### Docker Context
```bash
# Create context for remote Docker
docker context create remote --docker "host=tcp://192.168.1.100:2376"

# Use context
docker context use remote
docker ps  # This runs on remote host

# List contexts
docker context ls
```

### Custom Networks with Plugins
```bash
# Install network plugin
docker plugin install store/weaveworks/net-plugin:latest

# Create network with plugin
docker network create -d weave mynetwork

# Use macvlan for direct host network access
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  macvlan-net
```

### Docker API and Automation
```bash
# Access Docker API
curl --unix-socket /var/run/docker.sock http://localhost/containers/json

# Docker SDK for Python
pip install docker

# Python automation example
import docker
client = docker.from_env()

# List containers
containers = client.containers.list()

# Run container
container = client.containers.run(
    "nginx:latest",
    ports={'80/tcp': 8080},
    detach=True
)

# Build image
image = client.images.build(
    path=".",
    dockerfile="Dockerfile",
    tag="myapp:latest"
)
```

### Docker Content Trust
```bash
# Enable content trust
export DOCKER_CONTENT_TRUST=1

# Generate delegation keys
docker trust key generate mykey

# Sign and push image
docker trust sign myapp:latest

# Verify signed image
docker trust inspect myapp:latest
```

### Performance Optimization

#### Image Optimization
```dockerfile
# Use multi-stage builds
FROM node:16-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:16-alpine AS production
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY . .
CMD ["node", "app.js"]

# Optimize layer caching
COPY package*.json ./
RUN npm install
COPY . .

# Use .dockerignore
# .dockerignore content:
# node_modules
# .git
# .gitignore
# README.md
# Dockerfile
# .dockerignore
# npm-debug.log
```

#### Container Performance
```bash
# Optimize container limits
docker run -d \
  --name optimized-app \
  --memory=1g \
  --memory-swap=1g \
  --cpus=2.0 \
  --cpu-shares=1024 \
  --oom-kill-disable=false \
  myapp:latest

# Use init system for proper signal handling
docker run -d --init myapp:latest

# Optimize networking
docker run -d --network=host myapp:latest  # Use host networking for performance
```

### Container Storage Optimization
```bash
# Use volume drivers for performance
docker volume create \
  --driver local \
  --opt type=tmpfs \
  --opt device=tmpfs \
  --opt o=size=1g \
  temp-volume

# Optimize overlay2 storage driver
# /etc/docker/daemon.json
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true",
    "overlay2.size=20G"
  ]
}
```

### Docker in CI/CD

#### GitLab CI Example
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  DOCKER_DRIVER: overlay2

services:
  - docker:dind

before_script:
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

build:
  stage: build
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE

test:
  stage: test
  script:
    - docker run --rm $DOCKER_IMAGE npm test

deploy:
  stage: deploy
  script:
    - docker pull $DOCKER_IMAGE
    - docker stop myapp || true
    - docker rm myapp || true
    - docker run -d --name myapp -p 80:3000 $DOCKER_IMAGE
  only:
    - main
```

#### GitHub Actions Example
```yaml
# .github/workflows/docker.yml
name: Docker Build and Deploy

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1
    
    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_HUB_USERNAME }}
        password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v2
      with:
        context: .
        platforms: linux/amd64,linux/arm64
        push: true
        tags: |
          myapp:latest
          myapp:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

### Advanced Debugging and Troubleshooting

#### Container Debugging
```bash
# Debug container that won't start
docker run -it --entrypoint=/bin/sh myapp:latest

# Debug running container
docker exec -it <container_id> /bin/bash

# Debug container filesystem
docker diff <container_id>
docker export <container_id> | tar -tv

# Debug container networking
docker exec -it <container_id> netstat -tulpn
docker exec -it <container_id> ip addr show

# Debug container processes
docker exec -it <container_id> ps aux
docker exec -it <container_id> top

# Debug with tools container
docker run -it --rm \
  --pid container:<container_id> \
  --net container:<container_id> \
  --cap-add SYS_PTRACE \
  nicolaka/netshoot
```

#### Resource Investigation
```bash
# Investigate resource usage
docker stats --no-stream
docker system df
docker system events

# Check container resource limits
docker inspect <container_id> | jq '.[0].HostConfig.Memory'
docker inspect <container_id> | jq '.[0].HostConfig.CpuShares'

# Debug storage issues
docker exec -it <container_id> df -h
docker exec -it <container_id> du -sh /*
```

#### Log Analysis
```bash
# Analyze container logs
docker logs --timestamps <container_id>
docker logs --since="1h" --until="30m" <container_id>

# Export logs
docker logs <container_id> > container.log 2>&1

# Parse JSON logs
docker logs <container_id> | jq '.message'
```

### Kubernetes Integration

#### Docker to Kubernetes Migration
```yaml
# Convert Docker Compose to Kubernetes
# Original docker-compose.yml
version: '3.8'
services:
  web:
    image: myapp:latest
    ports:
      - "80:8000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb

# Kubernetes Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: myapp:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          value: "postgres://user:pass@db:5432/mydb"
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
```

#### Docker Registry for Kubernetes
```yaml
# Private registry deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: docker-registry
  template:
    metadata:
      labels:
        app: docker-registry
    spec:
      containers:
      - name: registry
        image: registry:2
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: registry-storage
          mountPath: /var/lib/registry
      volumes:
      - name: registry-storage
        persistentVolumeClaim:
          claimName: registry-pvc
```

### Enterprise Docker Features

#### Docker Enterprise Security
```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1
export DOCKER_CONTENT_TRUST_SERVER=https://notary.example.com

# Image signing
docker trust sign myapp:latest

# Role-based access control
docker trust signer add --key cert.pem john myapp

# Vulnerability scanning
docker scout cves myapp:latest
```

#### Docker Secrets Management
```bash
# Create secret
echo "mysecretpassword" | docker secret create db_password -

# Use secret in service
docker service create \
  --name myapp \
  --secret db_password \
  --env POSTGRES_PASSWORD_FILE=/run/secrets/db_password \
  myapp:latest

# Rotate secret
echo "newsecretpassword" | docker secret create db_password_v2 -
docker service update \
  --secret-rm db_password \
  --secret-add db_password_v2 \
  myapp
```

### Docker Best Practices Summary

#### Development Best Practices
```dockerfile
# Use multi-stage builds
FROM node:16-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

FROM node:16-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

#### Production Best Practices
```bash
# Use specific image tags
docker pull nginx:1.21.6-alpine

# Implement health checks
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Use read-only containers
docker run -d --read-only --tmpfs /tmp nginx

# Implement proper logging
docker run -d \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nginx

# Use resource limits
docker run -d \
  --memory=512m \
  --cpus=1.0 \
  --restart=unless-stopped \
  nginx
```

#### Security Best Practices
```dockerfile
# Run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001
USER nextjs

# Use distroless images
FROM gcr.io/distroless/nodejs:16

# Scan for vulnerabilities regularly
# Use tools like Trivy, Clair, or Snyk

# Keep base images updated
FROM node:16.19.0-alpine3.17

# Remove unnecessary packages
RUN apk add --no-cache git && \
    # ... use git ... && \
    apk del git
```

### Docker Troubleshooting Guide

#### Common Issues and Solutions

**Container Won't Start**
```bash
# Check logs
docker logs <container_id>

# Run with interactive mode
docker run -it --entrypoint=/bin/sh myapp:latest

# Check image
docker inspect myapp:latest
```

**Permission Issues**
```bash
# Fix file permissions
docker exec -it <container_id> chown -R user:group /path/to/files

# Run as different user
docker run -d --user 1000:1000 myapp:latest
```

**Network Connectivity Issues**
```bash
# Check container networking
docker exec -it <container_id> netstat -tulpn
docker exec -it <container_id> ping google.com

# Inspect network
docker network inspect bridge
```

**Storage Issues**
```bash
# Check disk usage
docker system df
docker system prune

# Inspect volumes
docker volume inspect myvolume
```

**Performance Issues**
```bash
# Monitor resources
docker stats
docker exec -it <container_id> top

# Check container limits
docker inspect <container_id> | grep -i memory
```

### Conclusion

This comprehensive guide covers Docker from basic concepts to advanced enterprise usage. Key takeaways:

1. **Start with fundamentals**: Understand containers, images, and the Docker architecture
2. **Master Dockerfile optimization**: Use multi-stage builds, proper layering, and security best practices
3. **Implement proper data management**: Use volumes, bind mounts, and tmpfs appropriately
4. **Design for production**: Implement health checks, resource limits, and monitoring
5. **Focus on security**: Use non-root users, scan images, and follow security best practices
6. **Automate everything**: Use CI/CD pipelines and infrastructure as code
7. **Monitor and debug**: Implement comprehensive logging and monitoring solutions

Remember to always test in development environments before deploying to production, and keep your Docker installation and images updated for security and performance improvements. 
