# Zabbix + SNMP in Docker on Ubuntu Server

## 📌 Introduction
This repository contains a detailed guide on how to set up a monitoring system using **Zabbix** in **Docker** on **Ubuntu**, and how to configure SNMP to monitor a **Fortinet router**.

## 📋 Prerequisites
Before starting make sure you meet the following requirements:

- A server with **Ubuntu** isntalled.
- **Docker** installed on the server.
- Access to the **Fortinet router** to configure SNMP.
- Network connection to the **Fortinet router**.

## 🛠 Installing Docker
Run the following commands on your Ubuntu server to install Docker:

```bash

# Install required packages
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common -y

This command installs the packages required for apt to use an HTTPS repository, including apt-transport-https, ca-certificates, curl, and software-properties-common.

# Add the official Docker GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg.

This command retrieves and adds the official GPG key for Docker to verify the integrity of the packages.

# Add the Docker repository:
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

This command adds the Docker repository to the system's list of software sources.

# Install Docker:
sudo apt-get update
sudo apt-get install docker-ce -y

This command updates the package database and installs Docker Community Edition.

# Verify Docker installation:
sudo docker --version
```

## 📦 Deploying Zabbix with Docker
Instead of using `docker-compose`, we will manually start the containers using `docker run` commands.