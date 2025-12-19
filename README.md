# cacher-server

### apt-cacher-ng
#### server

สร้างโฟเดอร์  /opt/apt-cache
```bash
mkdir -p /opt/apt-cache

```
docker compose
```bash
services:
  apt-cacher-ng:
    image: sameersbn/apt-cacher-ng
    container_name: apt-cacher-ng
    ports:
      - "3142:3142"
    volumes:
      - /opt/apt-cache:/var/cache/apt-cacher-ng
    restart: unless-stopped

```
#### client

เข้าไปใน /etc/apt/apt.conf.d/02proxy
```bash
sudo nano /etc/apt/apt.conf.d/02proxy


```
ใส่:
```bash
Acquire::http::Proxy "http://<IP-Server>:3142";
```
ทดสอบ update:
```bash
sudo apt update

```
---
