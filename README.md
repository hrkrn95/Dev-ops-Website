# VPS Setup & Deployment Guide

> A step-by-step walkthrough for absolute beginners—provisioning a Ubuntu VPS on Hostinger, installing Docker, Node.js, databases, PM2, Jenkins CI/CD, setting up Nginx proxy hosts & SSL, and troubleshooting.

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
