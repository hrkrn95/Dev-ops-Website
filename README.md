VPS Setup and Deployment Guide

This guide will walk you, step by step, through setting up a VPS, installing necessary software, deploying a Node/React app, and configuring CI/CD. No prior VPS experience is required.

1. Introduction: What Is a VPS?

VPS (Virtual Private Server): A virtual machine you rent from a hosting provider. It acts like your own server with full root access.

You can install software, host websites, run databases, and more.

2. Provisioning Your VPS on Hostinger

Login to Hostinger: Navigate to your Hostinger dashboard.

Create a New VPS:

Choose a plan (e.g., 1 vCPU, 1 GB RAM).

Select Ubuntu 22.04 as the operating system.

Set the Root Password:

During setup, you’ll be prompted to create a root password. Save it securely.

Find Your VPS IP Address:

In the Hostinger dashboard, locate your VPS and note its public IP.

3. Connecting to Your VPS via SSH

Open a Terminal (macOS/Linux) or PowerShell (Windows).

SSH Command:

ssh root@YOUR_VPS_IP

Enter the Password you set earlier.

You Are Now Logged In as root.

4. Basic Server Maintenance

Update package lists:

apt update

Upgrade installed packages:

apt upgrade -y

5. Installing Docker

Install prerequisites:

apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

Add Docker’s GPG key and repo:

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | tee /etc/apt/sources.list.d/docker.list > /dev/null

Install Docker Engine:

apt update
apt install -y docker-ce docker-ce-cli containerd.io

Start and enable Docker:

systemctl start docker
systemctl enable docker

6. Installing Nginx Proxy Manager (NPM)

Create a directory:

mkdir ~/nginx-proxy-manager && cd ~/nginx-proxy-manager

Create docker-compose.yml:

version: '3'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt

Deploy NPM:

docker-compose up -d

Access the UI at http://YOUR_VPS_IP:81 (default admin/admin).

7. Installing Node.js, npm, MySQL, and MongoDB

Node.js & npm (via NodeSource):

curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt install -y nodejs

MySQL:

apt install -y mysql-server
mysql_secure_installation

MongoDB:

apt install -y mongodb
systemctl start mongodb
systemctl enable mongodb

8. Installing PM2 (Process Manager)

npm install -g pm2
pm2 startup systemd

This ensures PM2 restarts your apps on server boot.

9. Uploading Your Project Files

Using SCP (command line):

scp -r /path/to/your/project root@YOUR_VPS_IP:/root/projects

Using FileZilla:

Host: sftp://YOUR_VPS_IP

Username: root

Password: (your root password)

Upload into /root/projects.

10. Installing Node Modules & Starting App

cd /root/projects/your-app
npm install
pm run build      # For React apps
pm start          # Or use pm2 start
pm2 start npm --name "your-app" -- start
pm2 save

11. Setting Up Jenkins for CI/CD

Install Java & Jenkins:

apt install -y openjdk-11-jdk
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | apt-key add -
sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
apt update
apt install -y jenkins
systemctl start jenkins
systemctl enable jenkins

Access Jenkins at http://YOUR_VPS_IP:8080, unlock with the initial admin password.

Create a Pipeline Job:

Source: Connect to your GitHub repo.

Stages:

Pull Code

Install Dependencies: npm install

Build (if React): npm run build

Restart App: sudo pm2 restart your-app

12. Configuring Nginx Proxy Host & SSL

In Nginx Proxy Manager UI:

Add Proxy Host:

Domain Names: your-domain.com

Forward Hostname/IP: 127.0.0.1

Forward Port: 3000 (or your app port)

Enable SSL:

Request a Let's Encrypt certificate

Advanced Location Blocks (for static assets):

location /images/ {
  root /root/projects/your-app/public;
}

13. Debugging Tips

Check Logs:

Docker: docker logs <container>

PM2: pm2 logs your-app

Nginx: /var/log/nginx/error.log

Server Health: top, htop, df -h

Firewall: Ensure ports 22, 80, 443, 3000, 8080 are allowed (use ufw).

Congratulations! By following these steps, you will have a fully functional VPS with your Node/React application deployed, secured, and on a CI/CD pipeline. Feel free to customize as needed and explore further enhancements.

