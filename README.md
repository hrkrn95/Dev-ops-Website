# VPS Setup & Deployment Guide

> A step-by-step walkthrough for absolute beginners—provisioning an Ubuntu VPS on Hostinger, installing Docker, Node.js, databases, PM2, Jenkins CI/CD, setting up Nginx proxy hosts & SSL, and debugging.

---

## Table of Contents

1. [Provision Your VPS](#1-provision-your-vps)  
2. [SSH & Basic Updates](#2-ssh--basic-updates)  
3. [Install Docker](#3-install-docker)  
4. [Install Nginx Proxy Manager (NPM)](#4-install-nginx-proxy-manager-npm)  
5. [Install Node.js, npm, MySQL & MongoDB](#5-install-nodejs-npm-mysql--mongodb)  
6. [Install PM2](#6-install-pm2)  
7. [Create Project Folder & Transfer Files](#7-create-project-folder--transfer-files)  
8. [Install & Configure Jenkins CI/CD](#8-install--configure-jenkins-cicd)  
9. [Configure Proxy Hosts & SSL](#9-configure-proxy-hosts--ssl)  
10. [Debugging Tips](#10-debugging-tips)  

---

## 1. Provision Your VPS

**What is a VPS?**  
A Virtual Private Server (VPS) is a virtual machine rented from a hosting provider. It behaves like your own Linux server, but runs on shared hardware.

**Why do we need it?**  
We need a VPS to deploy our applications, run services, and make them accessible over the internet—independently of your personal computer.

**How to provision on Hostinger:**  
1. Log in to your Hostinger dashboard.  
2. Click **VPS** → **Create VPS**.  
3. Select **Ubuntu 22.04 LTS** (or the latest LTS).  
4. Set a secure **root** password you’ll remember.  
5. Save the **public IP** address shown—this is how you’ll connect via SSH.

---

## 2. SSH & Basic Updates

**What is SSH?**  
SSH (Secure Shell) lets you securely access your VPS’s command line over the internet.

**Why update packages?**  
`apt update` and `apt upgrade` fetch and install the latest security fixes and software updates.

**How to connect & update:**

```bash
ssh root@YOUR_SERVER_IP
```

> Enter your root password when prompted.

```bash
# Refresh package lists
apt update -y

# Upgrade all installed packages
apt upgrade -y
```
if you are enable to connect check if firewal is enabled with ufw enable and then ufw allow ssh this will allow ssh through firewall
---

## 3. Install Docker

**What is Docker?**  
Docker is a containerization platform that packages apps and their dependencies into portable “containers.”

**Why use Docker?**  
Containers isolate your application, making deployments consistent across environments (development, staging, production).

### 3.1 Install prerequisites

```bash
apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

### 3.2 Add Docker’s GPG key & repository

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 3.3 Install Docker Engine

```bash
apt update
apt install -y docker-ce docker-ce-cli containerd.io
```

### 3.4 Start & enable Docker

```bash
systemctl start docker
systemctl enable docker
```

---

## 4. Install Nginx Proxy Manager (NPM)

**What is Nginx Proxy Manager?**  
A Dockerized web interface that makes configuring Nginx proxy hosts, SSL, redirects, and more super easy.

**Why use it?**  
It gives you a friendly UI to manage domain routing and SSL certificates without writing Nginx configs by hand.

### 4.1 Create a working directory

```bash
mkdir ~/nginx-proxy-manager && cd ~/nginx-proxy-manager
```

### 4.2 Create a `docker-compose.yml`

```yaml
version: '3'
services:
  app:
    image: jc21/nginx-proxy-manager:latest
    restart: always
    ports:
      - "80:80"    # HTTP
      - "81:81"    # Admin UI
      - "443:443"  # HTTPS
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

### 4.3 Launch NPM

```bash
docker-compose up -d
```

> Access the UI at `http://YOUR_SERVER_IP:81`  
> Default login: `admin@example.com` / `changeme`

---

## 5. Install Node.js, npm, MySQL & MongoDB

### 5.1 Node.js & npm

**What is Node.js?**  
A JavaScript runtime that allows you to run JS on the server.

**Why install via Nodesource?**  
Provides up-to-date Node releases for Ubuntu.

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt install -y nodejs
```

Verify:

```bash
node -v
npm -v
```

### 5.2 MySQL

**What is MySQL?**  
A relational database to store structured data.

**Why secure installation?**  
`mysql_secure_installation` guides you through removing defaults and setting strong passwords.

```bash
apt install -y mysql-server
mysql_secure_installation
```

### 5.3 MongoDB

**What is MongoDB?**  
A NoSQL document database—good for flexible, JSON-like data.

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu \
  $(lsb_release -cs)/mongodb-org/6.0 multiverse" \
  | tee /etc/apt/sources.list.d/mongodb-org-6.0.list

apt update
apt install -y mongodb-org
systemctl start mongod
systemctl enable mongod
```

---

## 6. Install PM2

**What is PM2?**  
A process manager for Node.js apps—keeps them alive, lets you restart on code changes, and manages logs.

```bash
npm install -g pm2
pm2 startup systemd
```

> **Important:** Copy the generated `pm2 startup …` command shown in your terminal and paste it back to complete setup.

---

## 7. Create Project Folder & Transfer Files

**What are we doing?**  
Creating a directory on the server for our app and moving local code up to it.

### 7.1 On the server

```bash
mkdir -p ~/projects/my-app
```

### 7.2 From your local machine

#### a) Using `scp`

```bash
scp -r ./my-app root@YOUR_SERVER_IP:~/projects/my-app
```

#### b) Using FileZilla  
- **Host:** `YOUR_SERVER_IP`  
- **Protocol:** SFTP  
- **User:** `root`  
- **Password:** *your root password*  
- **Port:** `22`  

Upload your local `my-app` folder into `~/projects/my-app`.

### 7.3 Install dependencies on the server

```bash
cd ~/projects/my-app
npm install
```

---

## 8. Install & Configure Jenkins CI/CD

**What is Jenkins?**  
An automation server for building, testing, and deploying code.

**Why use CI/CD?**  
Automatically deploy your app on every Git push—no manual steps.

### 8.1 Install Jenkins

```bash
wget -qO - https://pkg.jenkins.io/debian-stable/jenkins.io.key | apt-key add -
echo "deb https://pkg.jenkins.io/debian-stable binary/" \
  | tee /etc/apt/sources.list.d/jenkins.list

apt update
apt install -y openjdk-11-jdk jenkins
systemctl start jenkins
systemctl enable jenkins
```

> After install, visit `http://YOUR_SERVER_IP:8080` and run:
> ```bash
> cat /var/lib/jenkins/secrets/initialAdminPassword
> ```

### 8.2 Create a Pipeline Job

1. Install plugins: **Git**, **NodeJS**, **Docker**, **PM2**.  
2. New Item → **Pipeline**.  
3. Select **Pipeline script from SCM** → GitHub repo URL.  
4. Add this `Jenkinsfile` at your repo root:

    ```groovy
    pipeline {
      agent any
      environment {
        APP_DIR = "/root/projects/my-app"
      }
      stages {
        stage('Checkout') {
          steps {
            git branch: 'main', url: 'https://github.com/YourUser/YourRepo.git'
          }
        }
        stage('Install') {
          steps { sh 'cd $APP_DIR && npm install' }
        }
        stage('Build') {
          steps { sh 'cd $APP_DIR && npm run build' }
        }
        stage('Deploy') {
          steps {
            sh """
              pm2 stop my-app || true
              pm2 start npm --name "my-app" -- start
              pm2 save
            """
          }
        }
      }
    }
    ```

5. Save & **Build Now**. Your app auto-deploys on each push!

---

## 9. Configure Proxy Hosts & SSL

**What is a proxy host?**  
It routes incoming domain traffic (e.g. `example.com`) to your app on a specific port.

### 9.1 Via Nginx Proxy Manager UI

1. **Proxy Hosts → Add Proxy Host**  
2. Domain Names: `example.com`, `www.example.com`  
3. Scheme: `http` → Forward to `127.0.0.1:3000`  
4. Enable **Block Common Exploits**  
5. Under **SSL**, request a new Let’s Encrypt certificate  

### 9.2 Manual Nginx Configuration

#### 9.2.1 Create a site config

```nginx
# /etc/nginx/sites-available/my-app.conf
server {
  listen 80;
  server_name example.com www.example.com;

  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }

  location /static/ {
    alias /root/projects/my-app/build/static/;
  }
}
```

#### 9.2.2 Enable & reload

```bash
ln -s /etc/nginx/sites-available/my-app.conf /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

#### 9.2.3 Issue SSL with Certbot

```bash
apt install -y certbot python3-certbot-nginx
certbot --nginx -d example.com -d www.example.com
```

---

## 10. Debugging Tips

**Why debug?**  
Deployments can fail—logs and checks help you pinpoint issues.

- **Docker logs:**  
  ```bash
  docker-compose logs -f
  ```
- **PM2 logs:**  
  ```bash
  pm2 logs my-app
  ```
- **Jenkins console:**  
  In Jenkins → *your job* → *Console Output*  
- **Nginx syntax & errors:**  
  ```bash
  nginx -t
  tail -f /var/log/nginx/error.log
  ```
- **System health:**  
  ```bash
  htop
  df -h
  ```

---

*Congratulations!*  
