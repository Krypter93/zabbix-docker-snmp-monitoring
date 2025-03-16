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

## 💿 Pulling docker images:

```bash
docker pull mysql:8.0.41
docker pull zabbix/zabbix-server-mysql
docker pull zabbix/zabbix-web-nginx-mysql
docker pull zabbix/zabbix-java-gateway
```

## 📦 Deploying Zabbix with Docker
Instead of using `docker-compose`, we will manually start the containers using `docker run` commands.

### 🌐 Create perzonalized network:
```bash
docker network create zabbix-net
```
This allows all containers to communicate using their host names.

Or you can create a custom network with an specific IP range and subnet mask.

### 1️⃣ Start MySQL Database Container:
```bash
docker run --name mysql-server --network zabbix-net -t \
  -e MYSQL_DATABASE="zabbix" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="zabbix" \
  -e MYSQL_ROOT_PASSWORD="zabbix" \
  -d mysql:8.0.41 --character-set-server=utf8 --collation-server=utf8_bin
```

### 2️⃣ Start Zabbix Java Gateway Container:
This container will be in charge of managing Java communication for Zabbix.
```bash
docker run --name zabbix-java-gateway --network zabbix-net -t \
  --restart unless-stopped -d zabbix/zabbix-java-gateway
```

### 3️⃣ Start Zabbix Server Container:
Configure the server to use the MySQL container and communicate with the Java Gateway.
```bash
docker run --name zabbix-server-mysql --network zabbix-net -t \
  -e DB_SERVER_HOST="mysql-server" \
  -e MYSQL_DATABASE="zabbix" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="zabbix" \
  -e MYSQL_ROOT_PASSWORD="zabbix" \
  -e ZBX_JAVAGATEWAY="zabbix-java-gateway" \
  -p 10051:10051 --restart unless-stopped -d zabbix/zabbix-server-mysql
```

### 4️⃣ Start Zabbix Web Container:
It is important that the container points to the Zabbix server instead of "localhost". To do this, define the environment variable ZBX_SERVER_HOST with the name of the server container.
```bash
docker run --name zabbix-web-nginx-mysql --network zabbix-net -t \
  -e DB_SERVER_HOST="mysql-server" \
  -e MYSQL_DATABASE="zabbix" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="zabbix" \
  -e MYSQL_ROOT_PASSWORD="zabbix" \
  -e ZBX_SERVER_HOST="zabbix-server-mysql" \
  -p 80:8080 --restart unless-stopped -d zabbix/zabbix-web-nginx-mysql
```

### Additional verification steps:
Verify that all containers are running:
```bash
docker ps
```
Verify containers logs:
```bash
docker logs mysql-server
docker logs zabbix-server-mysql
docker logs zabbix-web-nginx-mysql
```

## 🚀 Accesing Zabbix Web Interface:
Once the containers are running, access the Zabbix web interface at `http://<SERVER_IP>:8080/`

If you are testing on the same machine where Docker runs, open in the browser: `http://localhost:8080/`

### 🌐 Configuring SNMP on Fortinet:
To monitor the Fortinet router using SNMP:

1. **Enable SNMP** on the Fortinet router:
    - Access the router's admin interface.
    - Go to *System* > *SNMP* and enable SNMP v1, v2 or v3 if you want more security.
    - Configure the **SNMP community** (e.g., `public`).
    - Add the IP of the server running Zabbix as an allowed host.
    - Save the changes.

2. **Add the router to Zabbix:**
    - Log in to the Zabbix web interface.
    - Go to *Data Collection* > *Hosts* and click *Create Host*.
    - Enter the router's name and IP address.
    - In *Templates*, add the `Template Net Network Generic Device SNMP` or select an official template for your Fortinet router (recommended option).
    - In *Interfaces*, select **SNMP** and use the community configured on Fortinet.
    - Save and verify that Zabbix starts collecting data.


## 📊 Verification and Monitoring:
- Go to *Monitoring* > *Hosts* and check the router's data.
- Configure graphs and triggers as needed.


## 📌 Conclusion
This method allows monitoring a Fortinet router with **Zabbix in Docker** using **SNMP**. It is a flexible and scalable solution for real-time network monitoring.

If you have any questions or suggestions, feel free to open an issue in this repository! 🚀

