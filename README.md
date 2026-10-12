# Guía de Instalación de Zabbix Server 7.4 en Linux Debian 13 trixie

Después de casi 18 años sin tocar Linux, me puse como desafío personal armar un laboratorio para implementar un sistema de monitoreo en la empresa.

Documentación paso a paso para la instalación y configuración de Zabbix Server en Ubuntu.

---

## 📋 Requisitos Previos

- **Sistema Operativo:** Debian Trixie
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

a. Instalar el repositorio de Zabbix

```bash
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb
dpkg -i zabbix-release_latest_7.4+debian13_all.deb
apt update
```

b. Install Zabbix server, frontend, agent2

```bash
 apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent2
```

c. Install Zabbix agent 2 plugins
You may want to install Zabbix agent 2 plugins.

```bash
 apt install zabbix-agent2-plugin-mongodb zabbix-agent2-plugin-mssql zabbix-agent2-plugin-postgresql
```

c.1. Instalar mariaDB

```bash
sudo apt update
sudo apt install mariadb-server  -y
```

d. Crear base de datos inicial
    Make sure you have database server up and running.
    Ejecuta lo siguiente en el host de base de datos.

```bash
# mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;
```

En el servidor Zabbix, importe el esquema y los datos iniciales. Se le pedirá que ingrese la contraseña recién creada.

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

Disable log_bin_trust_function_creators option after importing database schema.

```bash
# mysql -uroot -p
password
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;
```

e. Configurar la base de datos para el servidor Zabbix
Editar archivo /etc/zabbix/zabbix_server.conf

```bash
DBPassword=password
```

f. Inicia los procesos del agente y del servidor Zabbix
Inicia los procesos del agente y del servidor Zabbix y configúralos para que se inicien con el sistema.

```bash
systemctl restart zabbix-server zabbix-agent2 apache2
systemctl enable zabbix-server zabbix-agent2 apache2
```

g. Open Zabbix UI web page
The default URL for Zabbix UI when using Apache web server is http://localhost/zabbix




Para configurar una ip fija

```bash
sudo nano /etc/network/interfaces
```

```bash
auto ens192
iface ens192 inet static
    address 192.168.100.***
    netmask 255.255.255.0
    gateway 192.168.***.***
```

```bash
sudo nano /etc/resolv.conf
```

```bash
search.domino.test
nameserver 192.168.***.***
nameserver 192.168.***.***
nameserver 1.1.1.1
```

```bash
sudo systemctl restart networking
```

--------------------------------------------------------------------------------------------------------------

Para asegurar el correcto funcionamiento de Zabbix y proteger tu servidor Linux (Debian) usando el firewall nativo UFW (Uncomplicated Firewall), debes habilitar los puertos del servidor Zabbix, la interfaz Web y SSH.

Puertos esenciales de Zabbix

10051/TCP: Puerto del Zabbix Server (escucha los datos que envían los agentes o proxies).
10050/TCP: Puerto del Zabbix Agent (solo si el servidor se monitorea a sí mismo).
80/TCP y 443/TCP: Puertos para acceder a la interfaz Web de Zabbix.
22/TCP: Puerto SSH para administración del servidor.

1.- Permitir el acceso SSH (Evita quedar bloqueado):
Asegúrate de permitir primero la conexión de administración por SSH antes de activar el firewall:

```bash
sudo ufw allow 22/tcp
```

Verificación: Ejecuta sudo ufw status para verificar las reglas añadidas.2.

2.- Abrir puertos de la Web y Zabbix Server:
     Abre el acceso al panel Web y a la comunicación con los agentes de tu red:

# Permitir acceso Web (HTTP/HTTPS)
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

# Permitir puerto de Zabbix Server (recepción de métricas)
```bash
sudo ufw allow 10051/tcp
```

# Permitir puerto del Agente local (opcional)
```bash
sudo ufw allow 10050/tcp
```

3.- Habilitar el firewall:
    Activa el firewall para aplicar las reglas:
```bash
sudo ufw enable
```
Verificación: Presiona y cuando solicite confirmación y presiona Enter.

4.- Confirmar el estado final del firewall:
    Muestra la lista de puertos abiertos y la política de seguridad ejecutando:

```bash
  sudo ufw status verbose
```
Verificación: Debes ver las reglas en estado ALLOW para los puertos 22, 80, 443, 10050 y 10051.


