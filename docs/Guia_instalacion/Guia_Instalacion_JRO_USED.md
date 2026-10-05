# Guía de Instalación y Configuración: Ubuntu Server (JRO-USLimpia)

---

## CONFIGURACIÓN DE CLASE (IES Los Sauces)

### FASE 1: Preparación de la Máquina Virtual (VirtualBox)

1. Abre **VirtualBox** y haz clic en **Nueva**.
2. **Nombre y Sistema Operativo:**
   - Nombre: `JRO-USED`
   - Carpeta de máquina: (Tu preferencia)
   - Tipo: `Linux`
   - Versión: `Ubuntu (64-bit)`
3. **Hardware:**
   - Memoria RAM: `2048 MB` (2 GB)
   - Procesadores (CPU): `2`
4. **Disco Duro:**
   - Selecciona "Crear un disco duro virtual ahora".
   - Formato: `VDI (VirtualBox Disk Image)`.
   - Tamaño: 150GB (`/`), 350GB (`/var`) y 4GB de `swap`.
5. **Configuración de Red (Antes de iniciar):**
   - Ve a las **Preferencias** de la máquina creada > **Red**.
   - **IMPORTANTE (Dominio de la Junta):** Debes **deshabilitar la tarjeta de red** (desmarcando la casilla "Habilitar adaptador de red") antes de iniciar la máquina. Esto evitará que el instalador intente conectarse a repositorios de internet, ya que algunos están bloqueados por el DNS de la Junta y provocarían fallos en la instalación.

---

### FASE 2: Instalación del Sistema Operativo (Ubuntu Server)

Inicia la máquina virtual seleccionando la ISO de **Ubuntu Server** (versión recomendada 26.04 LTS según Heraclio).

1. **Idioma y Teclado:** Selecciona español.
2. **Conexión de Red (Clase):** 
   - Como hemos deshabilitado la tarjeta en el paso anterior, puedes omitir la configuración de red durante la instalación. Lo configuraremos manualmente luego.
3. **Configuración de Almacenamiento (Particionamiento Manual):**
   - Selecciona **Custom storage layout** (Personalizado).
   - Crea las siguientes particiones en tu disco libre:
     - **Partición 1 (Sistema):** Tamaño `150G`, Formato `ext4`, Punto de montaje `/`.
     - **Partición 2 (Swap):** Tamaño `4G` (RAM * 2), Formato `swap`.
     - **Partición 3 (Datos):** Tamaño `350G`, Formato `ext4`, Punto de montaje `/var`.
4. **Configuración del Perfil (Usuario principal):**
   - Nombre: `Juan`
   - Nombre del servidor: `JRO-USED`
   - Nombre de usuario: `miadmin`
   - Contraseña: `paso`
5. **SSH Setup:** Marca la casilla **"Install OpenSSH server"**.
6. Finaliza la instalación, **apaga la máquina virtual** y retira la ISO.

---

### FASE 3: Configuración Post-Instalación y Red Definitiva

**¡ATENCIÓN!** Antes de volver a encender la máquina, ve a la configuración de VirtualBox > **Red** y **vuelve a habilitar el adaptador de red** (como Adaptador Puente) para poder tener conexión.

Inicia sesión con tu usuario `miadmin` y contraseña `paso`.

#### 1. Configurar y Aplicar Red Definitiva mediante Netplan
```bash
# Acceder al directorio
cd /etc/netplan

# Visualizar el directorio
ls 00-installer-config.yaml

# Hacer copia de seguridad de la configuración
sudo cp 00-installer-config.yaml 00-installer-config.yaml.backup

# Editar la configuración (Añadir IP 10.199.8.97/22, Gateway 10.199.8.1 y DNS 10.151.123.21, 10.151.126.21)
sudo nano 00-installer-config.yaml

# Aplicar configuracion de red
sudo netplan apply
```

![Foto del netplan](netplan.png)

#### 2. Actualizar el Sistema Operativo
```bash
sudo apt update && sudo apt upgrade -y
```

#### 3. Instalar SSH, Cortafuegos, Cuentas Administradoras y Antivirus
```bash
# Instalar ssh (si no se instaló en el paso previo)
sudo apt install openssh-server -y

# Comprobar SSH
sudo systemctl status ssh

# Activar el ssh en el ufw
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status

# Crear miadmin2
sudo adduser miadmin2
# (te pedirá introducir la contraseña, escribe: paso)

# Darle permisos de administrador a miadmin2
sudo usermod -aG sudo miadmin2

# Comprobar Fecha y Hora
timedatectl status
sudo timedatectl set-timezone Europe/Madrid

# Instalar Antivirus
sudo apt update
sudo apt install clamav clamav-daemon -y

# Parar Antivirus
sudo systemctl stop clamav-freshclam

# Actualizar manualmente
sudo freshclam
sudo systemctl start clamav-freshclam
sudo systemctl enable clamav-freshclam

```

*(Opcional: Conectarse desde el SSH remoto a la IP asignada: `ssh miadmin@10.199.8.97`)*

---

### FASE 4: Configuración de DESARROLLO WEB

#### 1. Crear Cuentas de Desarrolladores
```bash
# Crear operadorweb
sudo adduser operadorweb
# (Contraseña: paso)


#### 2. Instalar Servicios Web (Apache, MySQL, XDebug, intl)
sudo apt install apache2 mysql-server php-mysql php-xdebug php-intl -y
```

#### 3. Configurar Apache HTTP y HTTPS Server
```bash
# Activar el módulo SSL
sudo a2enmod ssl

# Activar el sitio por defecto con HTTPS
sudo a2ensite default-ssl

# Reiniciar Apache para aplicar los cambios
sudo systemctl reload apache2

# Permitir tráfico HTTP y HTTPS en el cortafuegos
sudo ufw allow 'Apache Full'
```

#### 4. Comprobar el Estado de los Servicios
```bash
# Comprobar SSH
sudo systemctl status ssh

# Comprobar Firewall
sudo systemctl status ufw

# Comprobar Apache (HTTP/HTTPS)
sudo systemctl status apache2

# Comprobar MySQL
sudo systemctl status mysql
```


#### 5.Instalacion y Configuracion de PHP-FPM

Al estar trabajando con **Ubuntu Server 26** tenemos que instalar la versión disponible de **PHP-FPM**, que es la 8.5.

```bash
# Instalamos PHP-FPM
sudo apt install php8.5-fpm -y

# Comprobamos PHP-FPM
sudo systemctl status php8.5-fpm

# Configuramos Apache para usar PHP-FPM
sudo a2enmod proxy_fcgi setenvif
sudo a2dismod php8.5-fpm
sudo a2enconf php8.5-fpm

# Reiniciamos los servicios FPM y Apache
sudo systemctl restart apache2.service
sudo systemctl restart php8.5-fpm.service
```

#### 6.Configuración de la cuenta 'operadorweb' y permisos en el servidor

Para trabajar en una aplicación web, tenemos que configurar una cuenta que pueda subir y mantener la aplicación, pero no tenga permisos en el servidor.

```bash
# Vamos a cambiar el directorio personal de la cuenta para que solo acceda al servidor
sudo usermod -d /var/www/html -s /bin/bash operadorweb

# Cambiamos los permisos del grupo www-data (apache) y del directorio

sudo chown -R operadorweb:www-data /var/www/html
sudo chmod -R 775 /var/www/html
# Comprobamos con ls -ld /var/ww/html
```
![Foto de la comprobación](apachepermisos.PNG)

```bash
# Añadimos a operadorweb al grupo www-data
sudo usermod -aG www-data operadorweb
```


### FASE 6: Mantenimiento del SERVIDOR WEB

Estos son los comandos de mantenimiento de un servidor web y sus servicios:

#### Comandos de servidor
```bash
# Actualizamos la lista de paquetes.
sudo apt update -y

# Instalamos las actualizaciones de los paquetes instalados.
sudo apt upgrade -y

# Eliminamos paquetes y dependencias que no se necesitan.
sudo apt autoremove -y

# Limpia la cache de paquetes descargados por APT.
sudo apt autoclean
```

#### Comandos de servicio
```bash
# Comprobamos si un servicio esta activo
sudo systemctl status [nombre_servicio]

# Reiniciamos un servicio
sudo systemctl restart [nombre_servicio]
```

#### Comandos de memoria y disco
```bash
# Muestra el espacio disponible en las particiones del disco
df -h

# Consulta cuanto estapcio ocupa una carpeta especifica
du -sh [directorio]

# Revisa el consumo actual de la memoria RAM y del espacio de SWAP
free -h
```


## CONFIGURACIÓN DE CASA

Para configurar la máquina en casa, puedes repetir el proceso de instalación o utilizar la misma máquina de clase adaptando la red.

### FASE 1: Configuración de Red (Antes de iniciar)
1. Ve a las **Preferencias** de la máquina creada > **Red**.
2. Adaptador 1: Asegúrate de que la casilla "Habilitar adaptador de red" está marcada y cambia la configuración a **Adaptador Puente (Bridged)** para que tome IP de la red de tu casa.

### FASE 2: Adaptar configuración IP
Si vas a realizar una instalación desde cero en casa o necesitas cambiar la IP manualmente usando `Netplan`:

1. **Conexión de Red (Casa):** 
   - **Subnet:** `192.168.1.0/24`
   - **Address (IP):** `192.168.1.100`
   - **Gateway (Puerta de enlace):** `192.168.1.1`
   - **Name servers (DNS):** `8.8.8.8, 8.8.4.4`

*Las configuraciones de particionado, creación de usuarios, UFW, instalación de paquetes (Apache, MySQL, etc.) son exactamente las mismas que las descritas en la sección de clase.*
