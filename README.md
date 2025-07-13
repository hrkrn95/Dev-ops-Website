# VPS Setup & Deployment Guide

> A step-by-step walkthrough for absolute beginners—provisioning an Ubuntu VPS on Hostinger, installing Docker, Node.js, databases, PM2, Jenkins CI/CD, setting up Nginx proxy hosts & SSL, and troubleshooting.

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

1. Log in to your Hostinger account.  
2. Create a new **VPS** instance.  
3. Choose **Ubuntu 22.04 LTS** (or latest LTS).  
4. Set a **root** password you’ll remember.  
5. Note your server’s **public IP** (appears in the dashboard).

---

## 2. SSH & Basic Updates

Open a terminal on your local machine and run:

```bash
ssh root@YOUR_SERVER_IP
```

> Enter the **root** password you set on Hostinger.

Once logged in, update packages:

```bash
apt update -y
apt upgrade -y
```

---

## 3. Install Docker

### 3.1 Install prerequisites

```bash
apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

### 3.2 Add Docker’s GPG key & repo

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

### 4.1 Create a directory

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
      - "80:80"
      - "81:81"
      - "443:443"
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

```bash
apt install -y mysql-server
mysql_secure_installation
```

### 5.3 MongoDB

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

```bash
npm install -g pm2
pm2 startup systemd
```

> **Copy & paste** the generated `pm2 startup` command back into your shell to complete the setup.

---

## 7. Create Project Folder & Transfer Files

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

- **Host**: `YOUR_SERVER_IP`  
- **Protocol**: SFTP  
- **User**: `root`  
- **Password**: *your root password*  
- **Port**: `22`  
- Upload your local `my-app` folder into `~/projects/my-app`

### 7.3 Install dependencies on the server

```bash
cd ~/projects/my-app
npm install
```

---

## 8. Install & Configure Jenkins CI/CD

### 8.1 Install Jenkins

```bash
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | apt-key add -
echo "deb https://pkg.jenkins.io/debian-stable binary/" \
  | tee /etc/apt/sources.list.d/jenkins.list

apt update
apt install -y openjdk-11-jdk jenkins
systemctl start jenkins
systemctl enable jenkins
```

> Visit `http://YOUR_SERVER_IP:8080` and copy the initial admin password from:
> ```bash
> cat /var/lib/jenkins/secrets/initialAdminPassword
> ```

### 8.2 Create a Pipeline Job

1. **Install plugins**: Git, NodeJS, Docker, PM2.  
2. **Create New Item → Pipeline**.  
3. **Pipeline Script from SCM** pointing to your GitHub repo.  
4. Add a `Jenkinsfile` at your repo root:

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
      steps {
        sh 'cd $APP_DIR && npm install'
      }
    }
    stage('Build') {
      steps {
        sh 'cd $APP_DIR && npm run build'
      }
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

5. **Save** and **Build Now**—your code will deploy automatically on each push.

---

## 9. Configure Proxy Hosts & SSL

### 9.1 Via Nginx Proxy Manager UI

- **Proxy Hosts → Add Proxy Host**  
  - Domain Names: `example.com`, `www.example.com`  
  - Scheme: `http`  
  - Forward Hostname/IP: `127.0.0.1`  
  - Forward Port: `3000`  
  - Check **Block Common Exploits**  
  - Under SSL → **Request a new SSL Certificate** (Let’s Encrypt)

### 9.2 Manual Nginx (alternative)

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

- **Docker logs**  
  ```bash
  docker-compose logs -f
  ```
- **PM2 logs**  
  ```bash
  pm2 logs my-app
  ```
- **Jenkins console** → click the build **Console Output**.  
- **Nginx syntax**  
  ```bash
  nginx -t
  tail -f /var/log/nginx/error.log
  ```
- **System health**  
  ```bash
  htop
  df -h
  ```

---

*That’s it!* Your students can now follow **every** step in order, copy any shell snippet with one click, and have a production-ready Node.js app running behind Docker, Jenkins, and Nginx with SSL.
