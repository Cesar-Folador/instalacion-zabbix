> ⚠️ **ATENCIÓN / AVISO:** Este documento se encuentra actualmente **en proceso de revisión** y constante actualización. Las instrucciones o configuraciones descritas pueden estar sujetas a cambios.

# Guía de Instalación y Configuración de Zabbix Server 7.4 en Debian 13 (Trixie)

> Documentación paso a paso para desplegar un entorno de monitoreo profesional con Zabbix Server, MariaDB y Apache sobre Debian Linux.

---

## 💻 Requisitos Previos
- **Sistema Operativo:** Debian 13 (Trixie)
- **Usuario:** Acceso con privilegios de `sudo` o `root`
- **Recursos mínimos recomendados:** 2 CPU, 4 GB RAM, 100 GB HDD

---

## 🛠️ Pasos de Instalación

### 1. Actualizar el sistema e instalar dependencias iniciales
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install mariadb-server apache2 ufw wget -y
```

2. Configurar el repositorio oficial de Zabbix

```bash
wget [https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb](https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb)
sudo dpkg -i zabbix-release_latest_7.4+debian13_all.deb
sudo apt update
```
3. Instalar Zabbix Server, Frontend, Agente y Plugins

# Servidor, Interfaz Web, Agente 2 y Scripts de SQL
```bash
sudo apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent2
```

# Plugins para Zabbix Agent 2
```bash
sudo apt install -y zabbix-agent2-plugin-mongodb zabbix-agent2-plugin-mssql zabbix-agent2-plugin-postgresql
```

4. Crear y configurar la base de datos MariaDB
a. Crear base de datos y usuario
Ingresa a la consola de MariaDB:

```bash
sudo mysql -u root -p
```

Ejecuta las siguientes sentencias SQL:

```bash
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'tu_contraseña_segura';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
FLUSH PRIVILEGES;
EXIT;
```

b. Importar el esquema e historial inicial de Zabbix
```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

c. Desactivar la opción de funciones globales
```bash
sudo mysql -u root -p -e "SET GLOBAL log_bin_trust_function_creators = 0;"
```

d. Optimización de memoria para MariaDB
Edita el archivo de configuración para evitar cuellos de botella en disco:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Agrega al final de la sección [mariadb]:

```bash
innodb_buffer_pool_size = 2G
innodb_flush_log_at_trx_commit = 2
```

Reinicia MariaDB:

```bash
sudo systemctl restart mariadb
```

5. Configurar el Servidor Zabbix
Edita el archivo principal de configuración de Zabbix:

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```
Ajusta los siguientes parámetros:
Ini, TOML

```bash
# Contraseña asignada en MariaDB
DBPassword=tu_contraseña_segura

# Ajuste para evitar la alerta "Value cache working in low-memory mode"
ValueCacheSize=64M
```

6. Iniciar los servicios de Zabbix y Apache
   
```bash
sudo systemctl restart zabbix-server zabbix-agent2 apache2
sudo systemctl enable zabbix-server zabbix-agent2 apache2
```

7. Configuración de Red Estática (Opcional pero Recomendado)

```bash
sudo nano /etc/network/interfaces
```

Configura tu adaptador de red (ejemplo ens192):

```bash
auto ens192
iface ens192 inet static
    address 192.168.100.173
    netmask 255.255.255.0
    gateway 192.168.100.1
    dns-nameservers 8.8.8.8 1.1.1.1
```
Aplica los cambios reiniciando la red:

```bash
sudo systemctl restart networking
```

8. Configuración de Seguridad con UFW Firewall
Habilita las reglas esenciales para administrar y recibir métricas de monitoreo:

```bash
8. Configuración de Seguridad con UFW Firewall
Habilita las reglas esenciales para administrar y recibir métricas de monitoreo:

# 1. Permitir administración por SSH (Evita bloqueo)
sudo ufw allow 22/tcp

# 2. Permitir tráfico Web (Panel de Zabbix)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 3. Permitir comunicación con Zabbix Server y Agentes
sudo ufw allow 10051/tcp
sudo ufw allow 10050/tcp

# 4. Habilitar y verificar el Firewall
sudo ufw enable
sudo ufw status verbose
```

9. Acceso a la Interfaz Web

http://192.168.100.173/zabbix (o http://localhost/zabbix)

Sigue el asistente de instalación Web para completar la conexión con la base de datos.



--------------------------
Utilizacion de la memoria.

<img width="261" height="249" alt="image" src="https://github.com/user-attachments/assets/426b0247-c7b1-4bd0-9a4a-82ecfad8934a" />


Si hay mucho consumo de memoria en el servidor Zabbix, y no se cuenta con muchos recursos.

El consumo del 64.82% que marca el reloj es un comportamiento normal en Linux, ya que el sistema utiliza la memoria libre para acelerar lecturas en disco (búf/caché). Sin embargo, como tu servidor solo monitorea unas 20 máquinas virtuales, este porcentaje se puede bajar considerablemente liberando los recursos que no se están aprovechando.  
Para reducir el consumo de memoria en tu servidor, aplica estos tres ajustes en orden:

Reducir el Buffer Pool de MariaDB a 1 GB:

Para 20 máquinas virtuales, tener el buffer en 2 GB es más de lo necesario. Redúcelo a 1 GB para liberar 1 GB de RAM al sistema:

1.- Abre la configuración de MariaDB:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

2.- Cambia la línea de 2G a 1G:

```bash
innodb_buffer_pool_size = 1G
```

3.- Guarda los cambios (Ctrl + O, Enter, Ctrl + X) y reinicia MariaDB:

```bash
sudo systemctl restart mariadb
```

Verificación: Ejecuta sudo mysql -e "SHOW VARIABLES LIKE 'innodb_buffer_pool_size';" y confirma que indique aproximadamente 1073741824 bytes.


<img width="427" height="115" alt="image" src="https://github.com/user-attachments/assets/66e6a1dd-2b98-4336-b0f9-0b820784e0f8" />


<img width="280" height="278" alt="image" src="https://github.com/user-attachments/assets/cdb6bd60-eb84-4281-a15d-17b8b62afb37" />


