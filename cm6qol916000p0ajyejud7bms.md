---
title: "[Config] Konfigurasi VPS Linux"
datePublished: Tue Feb 04 2025 16:15:26 GMT+0000 (Coordinated Universal Time)
cuid: cm6qol916000p0ajyejud7bms
slug: config-konfigurasi-vps-linux
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/40XgDxBfYXM/upload/4abd040c2310d933545447e36e0aa765.jpeg
tags: hosting, vps

---

Spertinya ini sudah ke-sekian kalinya setup VPS. Note basic setup simple yang bisa dilakukan

### Membuat User

```bash
ssh root@your-server-ip
adduser newuser
usermod -aG sudo newuser
su - newuser #Test user
```

### Setup SSH

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
ssh-copy-id -i ~/.ssh/id_ed25519.pub newuser@your-server-ip
ssh newuser@your-server-ip
```

### Harden SSH (Dikonfigruasi pada VPS)

```bash
sudo vim /etc/ssh/sshd_config

# ubah line berikut
PermitRootLogin no # Disable root login
PasswordAuthentication no  # Disable password based auth

# Restart SSH dan Test SSH dari local
sudo systemctl restart ssh
ssh newuser@your-server-ip
```

### Install Uncomplicated Firewall (UFW)

```bash
sudo apt install ufw

sudo ufw allow OpenSSH    # SSH
sudo ufw allow 80/tcp     # HTTP
sudo ufw allow 443/tcp    # HTTPS

# enable ufw dan cek status
sudo ufw enable
sudo ufw status
```

### Setup Fail2ban

```bash
# Install Fail2Ban
sudo apt install fail2ban

# Copy default ke local untuk dirubah nantinya
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vim /etc/fail2ban/jail.local

# Pastikan configurasi ssh untuk akeses dikonfigurasi sebagai berikut 
# [sshd]
# enabled = true
# port = 22 # sesuaikan dengan port ssh di vps
# maxretry = 5
# bantime = 3600

# Restart Fail2Ban service
sudo systemctl restart fail2ban

# Check Fail2Ban status
sudo fail2ban-client status
sudo fail2ban-client status sshd
```