# 🧠 Pi-hole + Unbound on Raspberry Pi Zero (Docker Compose)

This project sets up a **privacy-first recursive DNS server** using [Pi-hole](https://pi-hole.net) and [Unbound](https://nlnetlabs.nl/projects/unbound/about/) on a **Raspberry Pi Zero (ARMv6/32-bit)**. It uses Docker Compose for easy deployment.

---

## 📦 Features

- 🔐 **Recursive DNS via Unbound**
- 🧹 **Network-wide ad blocking with Pi-hole**
- 🐳 **Lightweight Dockerized setup**
- 💡 **Optimized for Raspberry Pi Zero (32-bit)**

---

## ⚙️ Setup Instructions

### 1. Create project directories and configuration files

```bash
mkdir -p ~/pihole-unbound/unbound
cd ~/pihole-unbound/unbound
nano unbound.conf  # Paste config from below

cd ~/pihole-unbound
nano docker-compose.yaml  # Paste config from below
```

### 2. Paste the following into ~/pihole-unbound/unbound/unbound.conf:

```bash
server:
  verbosity: 1
  interface: 127.0.0.1
  port: 5335
  do-ip4: yes
  do-udp: yes
  do-tcp: yes
  access-control: 127.0.0.0/8 allow
  root-hints: "/etc/unbound/root.hints"
  harden-glue: yes
  harden-dnssec-stripped: yes
  cache-min-ttl: 3600
  cache-max-ttl: 86400

```

### 3. Paste the following into ~/pihole-unbound/docker-compose.yaml:

```bash
version: "3"

services:
  unbound:
    image: mvance/unbound:arm32v7
    container_name: unbound
    volumes:
      - './unbound:/etc/unbound'
    restart: unless-stopped
    network_mode: host

  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    depends_on:
      - unbound
    environment:
      TZ: 'America/New_York'
      WEBPASSWORD: 'changeme'  # Choose a strong password
      DNS1: 127.0.0.1#5335
      DNS2: 127.0.0.1#5335
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
    network_mode: host

```

### 4. Download the root hints file:

```bash
curl -o ./unbound/root.hints https://www.internic.net/domain/named.root
```

### 🚀 Start the Containers:

```bash
docker compose up -d
```

### 🔧 Access the Pi-hole Admin Panel:

 - Visit: <pre>http://<your-pi-zero-ip>/admin</pre>
 - Login with your <pre>WEBPASSWORD</pre>

### ✅ Final Configuration Steps:

 - Go to Settings → DNS in the Pi-hole admin panel
 - Verify that only <pre>127.0.0.1#5335</pre> is set as the upstream DNS
 - On your client device (PC, phone, router), set your Pi Zero’s IP as the DNS server
 - Test with:

```bash
dig google.com @<your-pi-zero-ip>
```

You should get a response — confirming your recursive DNS is working! 🎉

### 🔁 Optional: Next Steps:

 - Add a second Pi-hole server for redundancy
 - Block telemetry and tracking domains more aggressively
 - Set up HTTPS access using a reverse proxy
