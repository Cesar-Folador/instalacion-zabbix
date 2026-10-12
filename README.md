# 🚀 Guía de Instalación de Zabbix Server

Documentación paso a paso para la instalación y configuración de Zabbix Server en Ubuntu.

---

## 📋 Requisitos Previos

- **Sistema Operativo:** Ubuntu 22.04 LTS / 24.04 LTS
- **Usuario:** Acceso con privilegios de `sudo`
- **Recursos mínimos:** 2 CPU, 4 GB RAM, 100gb HD

---

## 🛠️ Pasos de Instalación

### 1. Actualizar el sistema e instalar el repositorio de Zabbix

```bash
sudo apt update && sudo apt upgrade -y
apt install mariadb-server
apt install apache2
```
a. Install Zabbix repository


•	wgethttps://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb
•	dpkg -i zabbix-release_latest_7.4+debian13_all.deb
•	apt update

Install Zabbix server, frontend, agent2

•	apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent2

 Install Zabbix agent 2 plugins

•	apt install zabbix-agent2-plugin-mongodb zabbix-agent2-plugin-mssql zabbix-agent2-plugin-postgresql

Create initial database

Make sure you have database server up and running.
Run the following on your database host.

•	mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;

On Zabbix server host import initial schema and data. You will be prompted to enter your newly created password.

•	zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix

Disable log_bin_trust_function_creators option after importing database schema.
•	mysql -uroot -p
password
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;

Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf
DBPassword=password
 


Start Zabbix server and agent processes

Start Zabbix server and agent processes and make it start at system boot.

•	systemctl restart zabbix-server zabbix-agent2 apache2
•	systemctl enable zabbix-server zabbix-agent2 apache2
Open Zabbix UI web page

The default URL for Zabbix UI when using Apache web server is http://localhost/zabbix
