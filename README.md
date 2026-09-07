# myapp

A simple **Node.js + Express web application** containerized with Docker and deployed to **AWS EC2** using **Docker Hub, Caddy, DuckDNS, and GitHub Actions CI/CD**.

The deployment was completed in multiple stages, starting with a direct HTTP deployment and later adding Caddy, DuckDNS, HTTPS, and automated CI/CD.

---

## Table of Contents

* [Project Overview](#project-overview)
* [Technology Stack](#technology-stack)
* [Project Structure](#project-structure)
* [Run Locally](#run-locally)
* [Docker Configuration](#docker-configuration)
* [Dockerfile](#dockerfile)
* [Build and Run Docker Locally](#build-and-run-docker-locally)
* [Docker Hub](#docker-hub)
* [AWS EC2 Setup](#aws-ec2-setup)
* [Connect to EC2](#connect-to-ec2)
* [EC2 Docker Installation](#ec2-docker-installation)
* [EC2 Security Group](#ec2-security-group)
* [Deployment Stage 1: Docker + EC2 + HTTP](#deployment-stage-1-docker--ec2--http)
* [Deployment Stage 2: Caddy + DuckDNS + HTTPS](#deployment-stage-2-caddy--duckdns--https)
* [Docker Network](#docker-network)
* [Caddy Configuration](#caddy-configuration)
* [Run the Application Behind Caddy](#run-the-application-behind-caddy)
* [Run Caddy](#run-caddy)
* [Deployment Stage 3: HTTP + HTTPS](#deployment-stage-3-http--https)
* [SSL / TLS](#ssl--tls)
* [GitHub Actions CI/CD](#github-actions-cicd)
* [GitHub Secrets](#github-secrets)
* [Deployment Workflow](#deployment-workflow)
* [Verify Deployment](#verify-deployment)
* [Useful Docker Commands](#useful-docker-commands)
* [Useful Caddy Commands](#useful-caddy-commands)
* [Troubleshooting](#troubleshooting)
* [Final Architecture](#final-architecture)
* [Normal Deployment Process](#normal-deployment-process)

---

# Project Overview

The application is a Node.js + Express web application running on port `3000`.

The application is packaged into a Docker image:

```text
adnanaws/myapp:latest
```

The Docker image is pushed to Docker Hub.

AWS EC2 pulls the latest image and runs the application inside a Docker container.

Caddy acts as a reverse proxy and exposes the application through HTTP and HTTPS.

GitHub Actions automates the build, Docker Hub push, and EC2 deployment whenever changes are pushed to the `main` branch.

---

# Technology Stack

* Node.js 20
* Express.js
* Docker
* Docker Hub
* AWS EC2
* Ubuntu
* Caddy
* DuckDNS
* HTTPS / TLS
* GitHub
* GitHub Actions
* SSH

---

# Project Details

## Docker Hub

```text
adnanaws/myapp:latest
```

## Application Port

```text
3000
```

## AWS EC2 Public IP

```text
13.60.187.67
```

## EC2 Private IP

```text
172.31.43.9
```

## Domain

```text
adnanaws.duckdns.org
```

## SSH Key

```text
~/Downloads/myapp-key.pem
```

---

# Project Structure

The main project structure is:

```text
myapp/
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── Dockerfile
├── package.json
├── package-lock.json
├── server.js
├── README.md
└── other application files
```

GitHub Actions workflow:

```text
.github/workflows/deploy.yml
```

Docker configuration:

```text
Dockerfile
```

Application entry point:

```text
server.js
```

---

# Run Locally

## Install Dependencies

From the project directory:

```bash
npm install
```

## Start the Application

```bash
npm start
```

The application runs on:

```text
http://localhost:3000
```

---

# Docker Configuration

The application is containerized using Docker.

The Docker container exposes port `3000`.

The application starts using:

```text
node server.js
```

---

# Dockerfile

The actual Dockerfile used by this project is:

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install --omit=dev

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

## Dockerfile Explanation

### Base Image

```dockerfile
FROM node:20-alpine
```

Uses Node.js 20 with Alpine Linux as the lightweight base image.

### Working Directory

```dockerfile
WORKDIR /app
```

Sets `/app` as the working directory inside the container.

### Copy Package Files

```dockerfile
COPY package*.json ./
```

Copies `package.json` and `package-lock.json` into the container.

### Install Dependencies

```dockerfile
RUN npm install --omit=dev
```

Installs production dependencies while excluding development dependencies.

### Copy Application

```dockerfile
COPY . .
```

Copies the application source code into the container.

### Application Port

```dockerfile
EXPOSE 3000
```

Documents that the Node.js application listens on port `3000`.

### Start Application

```dockerfile
CMD ["node", "server.js"]
```

Starts the application using `server.js`.

---

# Build and Run Docker Locally

## Build Image

From the project root:

```bash
docker build -t myapp .
```

Check the image:

```bash
docker images
```

## Run Container

```bash
docker run -d --name myapp -p 3000:3000 myapp
```

Open:

```text
http://localhost:3000
```

## Check Container

```bash
docker ps
```

## View Logs

```bash
docker logs myapp
```

## Stop Container

```bash
docker stop myapp
```

## Remove Container

```bash
docker rm myapp
```

---

# Docker Hub

The application image is published to Docker Hub.

Repository:

```text
adnanaws/myapp
```

Image:

```text
adnanaws/myapp:latest
```

## Login

```bash
docker login
```

## Build for Docker Hub

```bash
docker build -t adnanaws/myapp:latest .
```

## Push Image

```bash
docker push adnanaws/myapp:latest
```

## Pull Image

The EC2 server can pull the image using:

```bash
docker pull adnanaws/myapp:latest
```

---

# AWS EC2 Setup

The application is deployed on an Ubuntu AWS EC2 instance.

## EC2 Information

Public IP:

```text
13.60.187.67
```

Private IP:

```text
172.31.43.9
```

EC2 username:

```text
ubuntu
```

---

# Connect to EC2

From Mac Terminal:

```bash
ssh -i ~/Downloads/myapp-key.pem ubuntu@13.60.187.67
```

After connecting, the server prompt is similar to:

```text
ubuntu@ip-172-31-43-9:~$
```

Commands related to Docker, Caddy, and deployment are executed from the EC2 terminal.

---

# EC2 Docker Installation

Update Ubuntu:

```bash
sudo apt update
```

Install Docker:

```bash
sudo apt install -y docker.io
```

Enable Docker:

```bash
sudo systemctl enable docker
```

Start Docker:

```bash
sudo systemctl start docker
```

Check Docker status:

```bash
sudo systemctl status docker
```

Check Docker version:

```bash
docker --version
```

Add the Ubuntu user to the Docker group:

```bash
sudo usermod -aG docker ubuntu
```

If required, disconnect and reconnect to EC2 after changing Docker group permissions.

---

# EC2 Security Group

The EC2 Security Group allows the following inbound traffic:

| Protocol | Port | Source      | Purpose |
| -------- | ---: | ----------- | ------- |
| TCP      |   22 | `0.0.0.0/0` | SSH     |
| TCP      |   80 | `0.0.0.0/0` | HTTP    |
| TCP      |  443 | `0.0.0.0/0` | HTTPS   |

Port `3000` is not publicly exposed in the final deployment.

The Node.js application communicates with Caddy through the Docker network.

---

# Deployment Stage 1: Docker + EC2 + HTTP

The first deployment used Docker directly on EC2.

The application container exposed its internal port `3000` through the EC2 host's port `80`.

```bash
docker run -d \
  --name myapp \
  --restart unless-stopped \
  -p 80:3000 \
  adnanaws/myapp:latest
```

The application was accessed using:

```text
http://13.60.187.67
```

## Stage 1 Architecture

```text
Internet
   |
   v
EC2 :80
   |
   v
Docker myapp :3000
```

This was the initial HTTP-only deployment.

---

# Deployment Stage 2: Caddy + DuckDNS + HTTPS

The deployment was later changed to use Caddy as a reverse proxy.

Caddy handles external HTTP/HTTPS requests and forwards them to the Node.js application.

The domain used was:

```text
adnanaws.duckdns.org
```

---

# DuckDNS Configuration

The DuckDNS domain points to the EC2 public IP:

```text
13.60.187.67
```

The domain should resolve as:

```text
adnanaws.duckdns.org
        |
        v
13.60.187.67
```

## Verify DNS

From the local machine:

```bash
nslookup adnanaws.duckdns.org
```

The result should show:

```text
13.60.187.67
```

---

# Docker Network

Caddy and the application communicate through a dedicated Docker network:

```text
web
```

Create the network:

```bash
docker network create web
```

The network allows containers to communicate by container name.

Caddy can reach the application using:

```text
myapp:3000
```

---

# Caddy Configuration

Create the Caddy directory:

```bash
mkdir -p ~/caddy
```

The Caddy configuration file is:

```text
~/caddy/Caddyfile
```

Edit the file:

```bash
nano ~/caddy/Caddyfile
```

The initial Caddy configuration was:

```caddy
adnanaws.duckdns.org {
    reverse_proxy myapp:3000
}
```

This configuration makes Caddy reverse proxy requests to the Node.js application.

---

# Run the Application Behind Caddy

The original directly exposed application container must be stopped:

```bash
docker stop myapp
```

Remove it:

```bash
docker rm myapp
```

Start the application on the `web` Docker network:

```bash
docker run -d \
  --name myapp \
  --restart unless-stopped \
  --network web \
  adnanaws/myapp:latest
```

Important:

The final application container does **not** use:

```text
-p 80:3000
```

because Caddy owns the public HTTP and HTTPS ports.

The application is available internally as:

```text
myapp:3000
```

---

# Run Caddy

Run Caddy on the same Docker network:

```bash
docker run -d \
  --name caddy \
  --restart unless-stopped \
  --network web \
  -p 80:80 \
  -p 443:443 \
  -v ~/caddy/Caddyfile:/etc/caddy/Caddyfile \
  -v caddy_data:/data \
  caddy:2
```

Check running containers:

```bash
docker ps
```

Expected containers:

```text
myapp
caddy
```

---

# Deployment Stage 3: HTTP + HTTPS

The final setup allows both HTTP and HTTPS access.

HTTP:

```text
http://adnanaws.duckdns.org
```

HTTPS:

```text
https://adnanaws.duckdns.org
```

HTTP is not automatically redirected to HTTPS.

---

# Final Caddyfile

The final configuration is:

```caddy
{
    auto_https disable_redirects
}

http://adnanaws.duckdns.org {
    reverse_proxy myapp:3000
}

https://adnanaws.duckdns.org {
    reverse_proxy myapp:3000
}
```

The configuration means:

```text
HTTP :80
    |
    v
Caddy
    |
    v
myapp:3000
```

and:

```text
HTTPS :443
     |
     v
  Caddy
     |
     v
 myapp:3000
```

---

# Validate Caddy Configuration

After changing the Caddyfile, validate it:

```bash
docker exec caddy caddy validate --config /etc/caddy/Caddyfile
```

A successful configuration returns:

```text
Valid configuration
```

Restart Caddy:

```bash
docker restart caddy
```

Check Caddy logs:

```bash
docker logs caddy
```

---

# SSL / TLS

Caddy handles HTTPS/TLS for the configured domain.

HTTPS endpoint:

```text
https://adnanaws.duckdns.org
```

For HTTPS to work:

1. DuckDNS must point to the EC2 public IP.
2. EC2 port `443` must be open.
3. Caddy must be running.
4. The domain must be configured in the Caddyfile.
5. Caddy must be able to reach `myapp:3000`.

The Node.js application itself does not need to handle HTTPS because Caddy handles TLS at the reverse-proxy layer.

---

# GitHub Actions CI/CD

The project uses GitHub Actions to automate deployment.

Workflow file:

```text
.github/workflows/deploy.yml
```

The workflow runs whenever code is pushed to:

```text
main
```

---

# Actual GitHub Actions Workflow

The workflow used by this project is:

```yaml
name: Build and Deploy to AWS

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/myapp:latest

      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull ${{ secrets.DOCKERHUB_USERNAME }}/myapp:latest
            docker network create web || true
            docker stop myapp || true
            docker rm myapp || true
            docker run -d --name myapp --restart unless-stopped --network web ${{ secrets.DOCKERHUB_USERNAME }}/myapp:latest
            docker image prune -f
```

---

# GitHub Actions Workflow Explanation

## 1. Trigger

```yaml
on:
  push:
    branches: [main]
```

Every push to the `main` branch triggers the deployment workflow.

---

## 2. Checkout Code

```yaml
uses: actions/checkout@v4
```

Downloads the repository source code into the GitHub Actions runner.

---

## 3. Login to Docker Hub

```yaml
uses: docker/login-action@v3
```

Authenticates GitHub Actions with Docker Hub using GitHub Secrets.

---

## 4. Build and Push Docker Image

```yaml
uses: docker/build-push-action@v6
```

Builds the Docker image using the project's `Dockerfile` and pushes it to Docker Hub.

The image tag is:

```text
adnanaws/myapp:latest
```

---

## 5. Connect to EC2

```yaml
uses: appleboy/ssh-action@v1
```

Uses SSH to connect to the EC2 server.

---

## 6. Pull Latest Image

```bash
docker pull ${{ secrets.DOCKERHUB_USERNAME }}/myapp:latest
```

Downloads the latest image from Docker Hub.

---

## 7. Create Docker Network

```bash
docker network create web || true
```

Creates the `web` network if it does not already exist.

The `|| true` prevents the workflow from failing when the network already exists.

---

## 8. Stop Existing Container

```bash
docker stop myapp || true
```

Stops the currently running application container.

---

## 9. Remove Existing Container

```bash
docker rm myapp || true
```

Removes the old application container.

---

## 10. Start New Container

```bash
docker run -d \
  --name myapp \
  --restart unless-stopped \
  --network web \
  ${{ secrets.DOCKERHUB_USERNAME }}/myapp:latest
```

Starts the newly pulled application image.

The container joins the `web` Docker network.

There is intentionally no host port mapping because Caddy handles ports `80` and `443`.

---

## 11. Remove Unused Docker Images

```bash
docker image prune -f
```

Removes unused Docker images from the EC2 server.

---

# GitHub Secrets

The workflow requires these repository secrets:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
EC2_HOST
EC2_USER
EC2_SSH_KEY
```

## DOCKERHUB_USERNAME

Docker Hub username:

```text
adnanaws
```

## DOCKERHUB_TOKEN

Docker Hub access token used by GitHub Actions.

The token must be stored as a GitHub Secret and should not be committed to the repository.

## EC2_HOST

EC2 public IP:

```text
13.60.187.67
```

## EC2_USER

Ubuntu EC2 username:

```text
ubuntu
```

## EC2_SSH_KEY

Private SSH key used to connect to the EC2 server.

Example local key location:

```text
~/Downloads/myapp-key.pem
```

The private key must never be committed to GitHub.

---

# Deployment Workflow

The complete automated deployment process is:

```text
Developer
    |
    | git push origin main
    v
GitHub
    |
    v
GitHub Actions
    |
    v
Checkout Code
    |
    v
Build Docker Image
    |
    v
Push Image to Docker Hub
    |
    v
SSH into EC2
    |
    v
Pull Latest Docker Image
    |
    v
Stop Old myapp Container
    |
    v
Remove Old myapp Container
    |
    v
Start New myapp Container
    |
    v
Docker Network: web
    |
    v
Caddy
    |
    +------------+
    |            |
   HTTP        HTTPS
   :80          :443
    |            |
    +-----+------+
          |
          v
      myapp:3000
```

---

# Git Workflow

After making application changes locally:

Check the status:

```bash
git status
```

Add changes:

```bash
git add .
```

Commit changes:

```bash
git commit -m "Update application"
```

Push to GitHub:

```bash
git push origin main
```

The push automatically starts the GitHub Actions deployment.

---

# Verify Deployment

## Check GitHub Actions

After pushing code:

1. Open the GitHub repository.
2. Open the **Actions** tab.
3. Open the latest workflow.
4. Verify that all steps completed successfully.

---

# Verify EC2 Containers

Connect to EC2:

```bash
ssh -i ~/Downloads/myapp-key.pem ubuntu@13.60.187.67
```

Check running containers:

```bash
docker ps
```

Expected containers:

```text
myapp
caddy
```

---

# Check Application Logs

```bash
docker logs myapp
```

Follow logs:

```bash
docker logs -f myapp
```

---

# Check Caddy Logs

```bash
docker logs caddy
```

Follow Caddy logs:

```bash
docker logs -f caddy
```

---

# Check Docker Network

List networks:

```bash
docker network ls
```

Inspect the `web` network:

```bash
docker network inspect web
```

Both `myapp` and `caddy` should be connected to the `web` network.

---

# Test Application

## HTTP

Open:

```text
http://adnanaws.duckdns.org
```

## HTTPS

Open:

```text
https://adnanaws.duckdns.org
```

Both endpoints should serve the same Node.js application.

---

# Useful Docker Commands

## List Running Containers

```bash
docker ps
```

## List All Containers

```bash
docker ps -a
```

## List Images

```bash
docker images
```

## Pull Latest Image

```bash
docker pull adnanaws/myapp:latest
```

## Restart Application

```bash
docker restart myapp
```

## Restart Caddy

```bash
docker restart caddy
```

## Stop Application

```bash
docker stop myapp
```

## Remove Application Container

```bash
docker rm myapp
```

## Application Logs

```bash
docker logs myapp
```

## Caddy Logs

```bash
docker logs caddy
```

## Follow Application Logs

```bash
docker logs -f myapp
```

## Follow Caddy Logs

```bash
docker logs -f caddy
```

## Remove Unused Images

```bash
docker image prune -f
```

---

# Useful Caddy Commands

## Edit Caddyfile

```bash
nano ~/caddy/Caddyfile
```

## Validate Caddyfile

```bash
docker exec caddy caddy validate --config /etc/caddy/Caddyfile
```

## Restart Caddy

```bash
docker restart caddy
```

## View Caddy Logs

```bash
docker logs caddy
```

## Check Caddy Container

```bash
docker ps --filter name=caddy
```

---

# Troubleshooting

## Application Container Is Not Running

Check all containers:

```bash
docker ps -a
```

Check application logs:

```bash
docker logs myapp
```

Start the application again if required:

```bash
docker start myapp
```

---

## Caddy Is Not Running

Check:

```bash
docker ps -a
```

Check logs:

```bash
docker logs caddy
```

Validate configuration:

```bash
docker exec caddy caddy validate --config /etc/caddy/Caddyfile
```

Restart:

```bash
docker restart caddy
```

---

## Domain Is Not Working

Check DNS:

```bash
nslookup adnanaws.duckdns.org
```

The domain should resolve to:

```text
13.60.187.67
```

Also verify that EC2 ports `80` and `443` are allowed in the Security Group.

---

## HTTPS Is Not Working

Check Caddy:

```bash
docker ps
```

Check Caddy logs:

```bash
docker logs caddy
```

Validate the Caddy configuration:

```bash
docker exec caddy caddy validate --config /etc/caddy/Caddyfile
```

Verify that port `443` is open in the EC2 Security Group.

---

## Caddy Cannot Reach myapp

Check the Docker network:

```bash
docker network inspect web
```

Both containers should be connected to:

```text
web
```

The Caddy reverse proxy target is:

```text
myapp:3000
```

---

# Important Final Port Configuration

The final architecture uses:

| Component   | Port | Public/Internal         |
| ----------- | ---: | ----------------------- |
| Caddy HTTP  |   80 | Public                  |
| Caddy HTTPS |  443 | Public                  |
| Node.js     | 3000 | Internal Docker network |

The final application container runs without:

```text
-p 80:3000
```

Instead:

```bash
docker run -d \
  --name myapp \
  --restart unless-stopped \
  --network web \
  adnanaws/myapp:latest
```

Caddy exposes the application publicly.

---

# Final Architecture

```text
                         Internet
                            |
               +------------+------------+
               |                         |
            HTTP :80                 HTTPS :443
               |                         |
               +------------+------------+
                            |
                            v
                     +-------------+
                     |    Caddy    |
                     | Reverse     |
                     | Proxy       |
                     +-------------+
                            |
                     Docker Network
                          "web"
                            |
                            v
                     +-------------+
                     |    myapp    |
                     | Node.js +   |
                     | Express     |
                     |    :3000    |
                     +-------------+
```

---

# Final URLs

HTTP:

```text
http://adnanaws.duckdns.org
```

HTTPS:

```text
https://adnanaws.duckdns.org
```

---

# Normal Deployment Process

After the initial infrastructure is configured, normal application deployment only requires:

```bash
git add .
git commit -m "Update application"
git push origin main
```

GitHub Actions automatically:

```text
Build
  ↓
Docker Image
  ↓
Docker Hub
  ↓
EC2
  ↓
Pull Latest Image
  ↓
Replace myapp Container
  ↓
Caddy
  ↓
HTTP / HTTPS
```

No manual Docker deployment is required for normal code updates.

---

# Deployment Checklist

* [x] Node.js application created
* [x] Express application configured
* [x] Dockerfile created
* [x] Docker image built
* [x] Docker image pushed to Docker Hub
* [x] AWS EC2 configured
* [x] Docker installed on EC2
* [x] EC2 Security Group configured
* [x] Initial HTTP deployment completed
* [x] DuckDNS domain configured
* [x] Docker `web` network created
* [x] Caddy configured
* [x] Application connected to Caddy
* [x] HTTPS configured
* [x] HTTP access verified
* [x] HTTPS access verified
* [x] HTTP-to-HTTPS automatic redirect disabled
* [x] GitHub Actions CI/CD configured
* [x] Docker Hub authentication configured
* [x] EC2 SSH deployment configured
* [x] Automated deployment tested successfully

---

# Summary

The final application deployment consists of:

```text
Node.js + Express
        ↓
      Docker
        ↓
    Docker Hub
        ↓
     AWS EC2
        ↓
  Docker Network
      "web"
        ↓
      Caddy
     ↙     ↘
 HTTP       HTTPS
 :80        :443
     ↘     ↙
      myapp
       :3000
```

The deployment is automated through GitHub Actions, so every push to the `main` branch builds and publishes the latest Docker image and deploys it automatically to the EC2 server.
