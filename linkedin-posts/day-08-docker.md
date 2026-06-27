# Day 8 — Docker 🐳

## LinkedIn Post:

```
🐳 [Day 8/15] DevOps Cheat Sheet: Docker

"It works in Docker!" — until your image is 2GB
and takes 10 minutes to deploy.

Here's how I run containers in production:

━━━━━━━━━━━━━━━━━

# Run with restart policy (survives reboots)
docker run -d --name myapp -p 8080:80 --restart=always nginx:latest

# Logs — your first debugging step
docker logs -f --tail 100 <container>

# Get inside a running container
docker exec -it <container> /bin/bash

# Quick health check
docker inspect --format='{{.State.Health.Status}}' <container>

# Nuclear cleanup
docker stop $(docker ps -q)             # Stop all
docker rm $(docker ps -aq)              # Remove all
docker image prune -a                   # Remove unused images

━━━━━━━━━━━━━━━━━

# Multi-stage build (shrink images by 80%):

FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:20-alpine
RUN adduser -S appuser
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER appuser
EXPOSE 3000
CMD ["node", "dist/index.js"]

━━━━━━━━━━━━━━━━━

💡 Pro tips:
→ Always use specific tags (nginx:1.25) not :latest
→ Run as non-root user (USER appuser)
→ Use .dockerignore (like .gitignore for Docker)
→ One process per container

♻️ Repost for your Docker-learning network
#Docker #DevOps #Containers #CloudEngineering #SRE
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ Includes docker-compose examples too!

Tomorrow: Kubernetes — kubectl commands I use 10x daily ☸️
```
