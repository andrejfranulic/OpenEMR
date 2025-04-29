# 🩺 OpenEMR – Guía Profesional de Restauración desde Backup

Esta guía permite restaurar una instalación funcional de OpenEMR 7.0.2 en un servidor limpio con Ubuntu 22.04, utilizando:

- ✅ Base de datos (`openemr_db_backup.sql`)
      Descargala desdde este repositorio.
- ✅ Archivos del sistema (`openemr_files_backup.tar.gz`)
  - 📦 Disponible en Google Drive:
    [Descargar .tar.gz](https://drive.google.com/file/d/1FE_8bLo5EdrwU17I5dC23uNpAUQT5Qax/view?usp=sharing)

---

## ⚙️ Requisitos

- Ubuntu 22.04 LTS
- Acceso a Internet
- Usuario con permisos `sudo` o `root`
- Archivos: `.sql` y `.tar.gz`

---

## 🧱 Paso 1 – Instalar dependencias necesarias

```bash

sudo apt update

sudo apt install openssh-server
sudo systemctl start ssh

sudo apt install apache2 mariadb-server php php-mysql libapache2-mod-php \
php-xml php-mbstring php-zip php-soap php-gd php-curl unzip wget -y




```

---

## 🧰 Paso 2 – Activar servicios

```bash
sudo systemctl enable apache2 mariadb
sudo systemctl start apache2 mariadb
```

---

## 📥 Paso 3 – Subir archivos al servidor

1. Descargar `openemr_files_backup.tar.gz` desde Google Drive y subirlo al servidor:

   Desde tu PC:

   ```bash
   scp openemr_files_backup.tar.gz usuario@IP:/home/usuario/ Ejemplo: scp C:\Users\Tu_usuario\Downloads\openemr_files_backup.tar.gz david@111.22.44.66:/home/david/

   scp openemr_db_backup.sql usuario@IP:/home/usuario/       Ejemplo: scp C:\Users\Tu_usuario\Downloads\openemr_db_backup.sql david@1111.22.44.66:/home/david/

   ```

---

## 📂 Paso 4 – Restaurar archivos de OpenEMR

```bash
sudo tar -xzvf openemr_files_backup.tar.gz -C /var/www/html/
sudo chown -R www-data:www-data /var/www/html/openemr
sudo chmod -R 755 /var/www/html/openemr
```

---

## 🗃️ Paso 5 – Restaurar base de datos en MariaDB

### 1. Crear la base de datos:

```bash
sudo mariadb -u root -p
```

Dentro del prompt de MariaDB:

```sql
CREATE DATABASE openemr CHARACTER SET utf8 COLLATE utf8_general_ci;
EXIT;
```

### 2. Importar el `.sql`:

```bash
sudo mariadb -u root -p openemr < openemr_db_backup.sql
```

---

## 🔧 Paso 6 – Revisar conexión en `sqlconf.php`

```bash
sudo nano /var/www/html/openemr/sites/default/sqlconf.php
```

Valores típicos:

```php
$host   = 'localhost';
$port   = '3306';
$login  = 'openemr'; --usuario del servidor
$pass   = 'tu_contraseña_segura'; --contraseña usuario (root)
$dbase  = 'openemr';
$db_encoding    = 'utf8mb4';;


Mis datos por ejemplo son :

$host   = 'localhost';
$port   = '3306';
$login  = 'david';
$pass   = '1234';
$dbase  = 'openemr';
$db_encoding    = 'utf8mb4';

Hay que darle permiso al usuario en la base de datos  para poder acceder 

sudo mariadb -u root -p

GRANT ALL PRIVILEGES ON openemr.* TO 'david'@'localhost' IDENTIFIED BY '1234';
FLUSH PRIVILEGES;
EXIT;

sudo systemctl restart apache2

```

---

## 🌐 Paso 7 – Configurar Apache (URLs amigables)

1. Activar `mod_rewrite`:

```bash
sudo a2enmod rewrite
sudo apt install php-intl -y    #para que funcione el calendario

sudo systemctl restart apache2
```

2. Editar archivo de configuración:

```bash
sudo nano /etc/apache2/apache2.conf
```

Modificar:

```apache
<Directory /var/www/>
    AllowOverride All
</Directory>
```

3. Reiniciar Apache:

```bash
sudo systemctl restart apache2
```

---

## 🛠️ Paso 8 – Corregir errores comunes

### ❗ Error: `Site ID is missing from session data!`

```bash
sudo chown -R www-data:www-data /var/www/html/openemr/sites
sudo chown -R www-data:www-data /var/lib/php/sessions
sudo systemctl restart apache2
```

### ❗ Error: `404 Not Found` en módulos

1. Asegúrate de haber hecho los pasos de `mod_rewrite` y `AllowOverride`.
2. Borra caché navegador (`Ctrl + Shift + R` o `Ctrl + F5`).

---

## ✅ Paso 9 – Acceder a OpenEMR desde navegador

```bash
http://IP_DEL_SERVIDOR/openemr
```
admin creado por defecto :

**Usuario:** `Prueba_admin1234`  
**Contraseña:** `123456789123`

---

## 🧪 Verificación

Para comprobar que todo va bien:

```bash
sudo tail -f /var/log/apache2/error.log
```

---

## 📌 Notas

- Esta instalación contiene datos de ejemplo, no reales.
- Los usuarios y configuraciones ya están preconfigurados.

---

© Preparado por David — Restauración completa de OpenEMR.