# 🐳 Docker & Docker Compose Commands Guide

## 📋 **Table of Contents**
- [Docker Basic Commands](#docker-basic-commands)
- [Docker Images](#docker-images)
- [Docker Containers](#docker-containers)
- [Docker Networks](#docker-networks)
- [Docker Volumes](#docker-volumes)
- [Docker Compose Commands](#docker-compose-commands)
- [Development Workflow](#development-workflow)
- [Production Deployment](#production-deployment)
- [Troubleshooting](#troubleshooting)
- [Sentinel Examples](#sentinel-examples)

---

## 🐳 **Docker Basic Commands**

### **System Information**
```bash
# Docker version
docker --version

# Docker system info
docker info

# Docker system usage
docker system df

# Clean up unused resources
docker system prune -a --volumes

# Show disk usage
docker system df -v
```

### **Login & Registry**
```bash
# Login to Docker Hub
docker login

# Login to custom registry
docker login registry.example.com

# Logout
docker logout

# Search images
docker search nginx
docker search --filter stars=100 nginx
```

---

## 🖼️ **Docker Images**

### **Image Management**
```bash
# List images
docker images
docker images -a

# Pull image
docker pull nginx:latest
docker pull nginx:alpine

# Build image
docker build -t myapp:latest .
docker build -t myapp:v1.0 -f Dockerfile.prod .

# Tag image
docker tag myapp:latest myapp:v1.0
docker tag myapp:latest registry.example.com/myapp:v1.0

# Push image
docker push myapp:latest
docker push registry.example.com/myapp:v1.0

# Remove image
docker rmi myapp:latest
docker rmi $(docker images -q)  # Remove all images

# Save image to file
docker save myapp:latest > myapp.tar

# Load image from file
docker load < myapp.tar

# Inspect image
docker inspect nginx:latest
```

### **Image History & Layers**
```bash
# Show image history
docker history nginx:latest

# Show image layers
docker inspect nginx:latest | grep -A 10 "RootFS"

# Build with cache
docker build --no-cache -t myapp:latest .

# Build with specific build args
docker build --build-arg NODE_ENV=production -t myapp:latest .
```

---

## 📦 **Docker Containers**

### **Container Lifecycle**
```bash
# Run container
docker run nginx:latest
docker run -d nginx:latest  # Detached mode
docker run --name mynginx nginx:latest  # Named container

# Run with port mapping
docker run -d -p 8080:80 nginx:latest
docker run -d -p 8080:80 -p 8443:443 nginx:latest

# Run with volume mounting
docker run -d -v /host/path:/container/path nginx:latest
docker run -d -v myvolume:/app/data nginx:latest

# Run with environment variables
docker run -d -e NODE_ENV=production myapp:latest
docker run -d --env-file .env myapp:latest

# Run with resource limits
docker run -d --memory=512m --cpus=1.0 nginx:latest

# Run interactive container
docker run -it ubuntu:latest bash
docker run -it --rm ubuntu:latest bash  # Remove after exit
```

### **Container Management**
```bash
# List containers
docker ps                    # Running containers
docker ps -a                 # All containers
docker ps -q                 # Only IDs
docker ps --filter "status=exited"

# Start container
docker start mycontainer
docker start $(docker ps -aq)  # Start all containers

# Stop container
docker stop mycontainer
docker stop $(docker ps -q)   # Stop all running containers
docker stop -t 30 mycontainer # Graceful stop with timeout

# Restart container
docker restart mycontainer
docker restart $(docker ps -q)

# Pause/Unpause container
docker pause mycontainer
docker unpause mycontainer

# Kill container
docker kill mycontainer
docker kill $(docker ps -q)

# Remove container
docker rm mycontainer
docker rm $(docker ps -aq)    # Remove all containers
docker rm -f mycontainer      # Force remove running container
```

### **Container Operations**
```bash
# Execute command in running container
docker exec mycontainer ls -la
docker exec -it mycontainer bash

# Copy files to/from container
docker cp mycontainer:/app/config.json ./config.json
docker cp ./config.json mycontainer:/app/config.json

# Show container logs
docker logs mycontainer
docker logs -f mycontainer                    # Follow logs
docker logs --tail 100 mycontainer           # Last 100 lines
docker logs --since "2024-01-01T00:00:00" mycontainer

# Show container resource usage
docker stats mycontainer
docker stats $(docker ps -q)                 # All containers

# Inspect container
docker inspect mycontainer
docker inspect mycontainer | grep -A 10 "Mounts"
```

### **Container Health Checks**
```bash
# Check container health
docker ps --filter "health=healthy"
docker ps --filter "health=unhealthy"

# Inspect health status
docker inspect mycontainer | grep -A 10 "Health"

# View health check logs
docker logs mycontainer 2>&1 | grep -i health
```

---

## 🌐 **Docker Networks**

### **Network Management**
```bash
# List networks
docker network ls

# Create network
docker network create mynetwork
docker network create --driver bridge mynetwork
docker network create --subnet=192.168.1.0/24 mynetwork

# Inspect network
docker network inspect mynetwork

# Connect container to network
docker network connect mynetwork mycontainer

# Disconnect container from network
docker network disconnect mynetwork mycontainer

# Remove network
docker network rm mynetwork
docker network prune  # Remove unused networks
```

### **Network Types**
```bash
# Bridge network (default)
docker network create --driver bridge mybridge

# Host network
docker run --network host nginx:latest

# None network (isolated)
docker run --network none nginx:latest

# Overlay network (for swarm)
docker network create --driver overlay myoverlay
```

---

## 💾 **Docker Volumes**

### **Volume Management**
```bash
# List volumes
docker volume ls

# Create volume
docker volume create myvolume
docker volume create --driver local myvolume

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume
docker volume prune  # Remove unused volumes
```

### **Volume Operations**
```bash
# Mount named volume
docker run -d -v myvolume:/app/data nginx:latest

# Mount bind mount
docker run -d -v /host/path:/container/path nginx:latest

# Mount with permissions
docker run -d -v /host/path:/container/path:ro nginx:latest  # Read-only
docker run -d -v /host/path:/container/path:Z nginx:latest   # SELinux relabeling

# Copy data to volume
docker run --rm -v myvolume:/data alpine sh -c "echo 'Hello' > /data/hello.txt"

# Backup volume
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine tar czf /backup/backup.tar.gz -C /data .
```

---

## 💾 **Data Persistence & Volumes**

### **Volume Types:**
```yaml
version: '3.8'

services:
  # Named volumes (managed by Docker)
  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro

  # Bind mounts (host directories)
  api:
    volumes:
      - ./packages/@sentinel/services/api:/app
      - /app/node_modules

  # Anonymous volumes (temporary)
  temp-service:
    volumes:
      - /tmp/cache

  # Read-only volumes
  config-service:
    volumes:
      - ./config:/app/config:ro

volumes:
  postgres_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/postgres
```

---

## 🐙 **Docker Compose Commands**

### **Basic Commands**
```bash
# Start services
docker-compose up
docker-compose up -d                    # Detached mode
docker-compose up --build              # Build before starting
docker-compose up --scale api=3        # Scale specific service

# Stop services
docker-compose down
docker-compose down -v                 # Remove volumes
docker-compose down --rmi all          # Remove images

# View status
docker-compose ps
docker-compose top                     # Show running processes
docker-compose ls                      # List compose projects
```

### **Service Management**
```bash
# Start specific service
docker-compose up api
docker-compose up api db

# Stop specific service
docker-compose stop api
docker-compose stop

# Restart services
docker-compose restart
docker-compose restart api

# Pause/Unpause services
docker-compose pause
docker-compose unpause
```

### **Build & Images**
```bash
# Build services
docker-compose build
docker-compose build --no-cache
docker-compose build --parallel

# Pull images
docker-compose pull

# Push images
docker-compose push
```

### **Logs & Monitoring**
```bash
# View logs
docker-compose logs
docker-compose logs -f                 # Follow logs
docker-compose logs --tail=100         # Last 100 lines
docker-compose logs api                # Specific service logs
docker-compose logs --since "1h"       # Logs from last hour

# Execute commands
docker-compose exec api bash
docker-compose exec db psql -U postgres

# Run one-time commands
docker-compose run api npm test
docker-compose run --rm api npm test   # Remove after execution
```

### **Configuration**
```bash
# Validate configuration
docker-compose config

# List services
docker-compose ps --services

# Show environment
docker-compose exec api env

# Port mapping
docker-compose port api 3000
```

---

## 🔄 **Development Workflow**

### **Development Setup**
```bash
# Clone repository
git clone https://github.com/sentinel/sentinel.git
cd sentinel

# Copy environment file
cp .env.example .env

# Start development environment
docker-compose -f docker-compose.dev.yml up -d

# View logs
docker-compose logs -f

# Access services
# API: http://localhost:3000
# Database: localhost:5432
# Redis: localhost:6379
```

### **Development Commands**
```bash
# Run tests
docker-compose exec api npm test
docker-compose exec api npm run test:watch

# Run migrations
docker-compose exec api npx prisma migrate dev

# Generate Prisma client
docker-compose exec api npx prisma generate

# Access database
docker-compose exec db psql -U postgres -d sentinel_dev

# Debug with logs
docker-compose logs -f api
docker-compose logs -f --tail=50 api

# Rebuild and restart
docker-compose up --build api
```

### **Hot Reload Development**
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  api:
    build:
      context: ./packages/@sentinel/services/api
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
      - "9229:9229"  # Debug port
    volumes:
      - ./packages/@sentinel/services/api:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
    command: npm run dev
```

---

## 🚀 **Production Deployment**

### **Production Commands**
```bash
# Build for production
docker-compose -f docker-compose.prod.yml build

# Deploy to production
docker-compose -f docker-compose.prod.yml up -d

# Zero-downtime deployment
docker-compose -f docker-compose.prod.yml up -d --no-deps api

# Check health
docker-compose ps
docker-compose exec api curl -f http://localhost:3000/health

# Scale services
docker-compose up -d --scale api=3
docker-compose up -d --scale worker=5

# Update with new image
docker-compose pull
docker-compose up -d

# Rollback
docker-compose up -d --no-deps api  # If using previous image
```

### **Production docker-compose.prod.yml**
```yaml
version: '3.8'

services:
  api:
    image: sentinel/api:latest
    ports:
      - "80:3000"
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: sentinel
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G

volumes:
  postgres_data:
```

---

## 🔧 **Troubleshooting**

### **Container Issues**
```bash
# Check container status
docker ps -a

# View container logs
docker logs mycontainer
docker logs -f mycontainer

# Inspect container
docker inspect mycontainer

# Check resource usage
docker stats mycontainer

# Restart container
docker restart mycontainer

# Remove and recreate
docker rm -f mycontainer
docker run -d --name mycontainer myimage:latest
```

### **Image Issues**
```bash
# Check image layers
docker history myimage:latest

# Rebuild without cache
docker build --no-cache -t myimage:latest .

# Clean up images
docker image prune -f
docker image prune -a -f

# Check image size
docker images myimage
```

### **Network Issues**
```bash
# Check network connectivity
docker network ls
docker network inspect bridge

# Test container networking
docker exec mycontainer ping google.com
docker exec mycontainer curl http://othercontainer:3000

# Restart network
docker-compose down
docker-compose up -d
```

### **Volume Issues**
```bash
# Check volume usage
docker volume ls
docker volume inspect myvolume

# Check disk space
df -h
docker system df

# Clean up volumes
docker volume prune -f

# Backup and restore
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine tar czf /backup/backup.tar.gz -C /data .
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine tar xzf /backup/backup.tar.gz -C /data
```

### **Performance Issues**
```bash
# Check system resources
docker stats
docker stats $(docker ps -q)

# Check memory usage
docker exec mycontainer free -h
docker exec mycontainer top

# Monitor disk I/O
docker exec mycontainer iotop
docker exec mycontainer df -h

# Check network usage
docker exec mycontainer ifconfig
docker exec mycontainer netstat -tuln
```

---

## 🏗️ **Sentinel Examples**

### **Complete Sentinel Stack**
```yaml
# docker-compose.yml
version: '3.8'

services:
  # API Service
  api:
    build: ./packages/@sentinel/services/api
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:pass@db:5432/sentinel
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://user:pass@rabbitmq:5672
    depends_on:
      - db
      - redis
      - rabbitmq
    volumes:
      - ./packages/@sentinel/services/api:/app
      - /app/node_modules
    command: npm run dev

  # Integrations Service
  integrations:
    build: ./packages/@sentinel/integrations
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:pass@db:5432/sentinel
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://user:pass@rabbitmq:5672
    depends_on:
      - db
      - redis
      - rabbitmq
    volumes:
      - ./packages/@sentinel/integrations:/app
      - /app/node_modules
    command: npm run dev

  # Database
  db:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: sentinel
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # Redis Cache & Queue
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

  # Message Broker
  rabbitmq:
    image: rabbitmq:3.12-management-alpine
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: user
      RABBITMQ_DEFAULT_PASS: pass
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

volumes:
  postgres_data:
  redis_data:
  rabbitmq_data:
```

### **Sentinel Development Commands**
```bash
# Start full stack
docker-compose up -d

# View all services
docker-compose ps

# View logs
docker-compose logs -f

# Run API tests
docker-compose exec api npm test

# Run database migrations
docker-compose exec api npx prisma migrate dev

# Access database
docker-compose exec db psql -U user -d sentinel

# Stop and clean up
docker-compose down -v
```

### **Production Deployment Commands**
```bash
# Build all services
docker-compose build

# Deploy to production
docker-compose -f docker-compose.prod.yml up -d

# Scale API service
docker-compose up -d --scale api=3

# Update API service
docker-compose build api
docker-compose up -d --no-deps api

# Check service health
docker-compose ps
docker-compose exec api curl -f http://localhost:3000/health

# View production logs
docker-compose logs -f --tail=100

# Backup database
docker-compose exec db pg_dump -U user sentinel > backup.sql

# Restore database
docker-compose exec -T db psql -U user sentinel < backup.sql
```

### **Quick Start Commands**
```bash
# One-command setup (development)
git clone https://github.com/sentinel/sentinel.git
cd sentinel
cp .env.example .env
docker-compose up -d

# Check everything is running
docker-compose ps

# Access services
open http://localhost:3000  # API
open http://localhost:15672 # RabbitMQ Management

# Stop everything
docker-compose down

# Clean up (remove volumes)
docker-compose down -v
docker system prune -a --volumes
```

---

## 📝 **Essential Docker Commands Cheat Sheet**

### **Daily Development**
```bash
# Start development environment
docker-compose up -d

# View logs
docker-compose logs -f

# Run tests
docker-compose exec api npm test

# Restart service
docker-compose restart api

# Stop everything
docker-compose down
```

### **Production Operations**
```bash
# Deploy updates
docker-compose pull && docker-compose up -d

# Scale service
docker-compose up -d --scale api=3

# Check health
docker-compose ps && docker stats

# Backup data
docker-compose exec db pg_dump > backup.sql

# Clean up
docker system prune -f
```

### **Debugging**
```bash
# Inspect container
docker inspect mycontainer

# View logs
docker logs mycontainer

# Execute shell
docker exec -it mycontainer bash

# Check resources
docker stats mycontainer

# Network debug
docker network inspect sentinel_default
```

### **Cleanup Commands**
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Everything cleanup
docker system prune -a --volumes
```

---

## 🎯 **Quick Reference**

| Command | Description | Example |
|---------|-------------|---------|
| `docker ps` | List running containers | `docker ps -a` |
| `docker images` | List images | `docker images` |
| `docker run` | Run container | `docker run -d nginx` |
| `docker build` | Build image | `docker build -t myapp .` |
| `docker exec` | Execute in container | `docker exec -it container bash` |
| `docker logs` | View logs | `docker logs -f container` |
| `docker-compose up` | Start services | `docker-compose up -d` |
| `docker-compose down` | Stop services | `docker-compose down` |
| `docker-compose logs` | View service logs | `docker-compose logs -f` |
| `docker-compose exec` | Execute in service | `docker-compose exec api bash` |

This guide covers all essential Docker and Docker Compose commands with practical examples for Sentinel development and production use.
