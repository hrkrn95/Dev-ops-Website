VPS Setup and Deployment Guide

This guide will walk you, step by step, through setting up a VPS, installing necessary software, deploying a Node/React app, and configuring CI/CD. No prior VPS experience is required.

1. Introduction: What Is a VPS?

VPS (Virtual Private Server): A virtual machine you rent from a hosting provider. It acts like your own server with full root access.

You can install software, host websites, run databases, and more.

2. Provisioning Your VPS on Hostinger

Login to Hostinger: Navigate to your Hostinger dashboard.

Create a New VPS:

Choose a plan (e.g., 1 vCPU, 1 GB RAM).

Select Ubuntu 22.04 as the operating system.

Set the Root Password: During setup, you’ll be prompted to create a root password. Save it securely.

Find Your VPS IP Address: In the Hostinger dashboard, locate your VPS and note its public IP.

3. Connecting to Your VPS via SSH

Open a terminal (macOS/Linux) or PowerShell (Windows).

Connect using SSH:

ssh root@YOUR_VPS_IP

Enter the password you set earlier.

You are now logged in as root.

4. Basic Server Maintenance

Update package lists:

apt update

Upgrade installed packages:

apt upgrade -y

5. Installing Docker

Install prerequisites:

apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

Add Docker’s GPG key and repository:
