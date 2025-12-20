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
### apache2
ไว้โหลดไฟล์ gpg
#### server

สร้างโฟเดอร์  /opt/gpg
```bash
mkdir -p /opt/gpg

```
docker compose
```bash
services:
  apache:
    image: httpd:2.4
    container_name: apache2
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - /opt/gpg:/usr/local/apache2/htdocs
```
dowload gpg ของ docker ไว้ที่ /opr/gpg

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /opt/gpg/docker.gpg
```


#### client
dowload gpg
```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl http://192.168.100.180/docker.gpg \
 | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

```
---

### วิธีการติดตั้ง docker
Uninstall เวอร์ชั่นเก่าออก:
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

```
อัปเดต Package Index ติดตั้ง Package ที่จำเป็น และ เพิ่ม Docker’s official GPG key:
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
sudo install -m 0755 -d /etc/apt/keyrings
curl http://192.168.100.180/docker.gpg \
 | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

```
ตั้งค่า Repository:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```
ติดตั้ง Docker Engine:
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

```
ตรวจสอบการทำงาน Service Docker และให้ทำงานอัตโนมัติทุกครั้งที่ Reboot


```bash
sudo systemctl status docker
sudo systemctl enable docker

```
---


### ติดตั้ง verdaccio
Uninstall เวอร์ชั่นเก่าออก:
```bash
mkdir -p /opt/verdaccio_conf
```
docker compose
```bash
version: "3.8"

services:
  verdaccio:
    image: verdaccio/verdaccio:5
    container_name: npm-cache
    restart: unless-stopped
    ports:
      - "4873:4873"
    volumes:
      - verdaccio_data:/verdaccio/storage
      - /opt/verdaccio_conf:/verdaccio/conf
    environment:
      - VERDACCIO_PUBLIC_URL=http://192.168.100.180:4873

volumes:
  verdaccio_data:

```
config
```bash
nano /opt/verdaccio_conf/config.yaml
```
ใส่:
```bash
storage: /verdaccio/storage

uplinks:
  npmjs:
    url: https://registry.npmjs.org/

packages:
  "**":
    access: $all
    publish: $all
    proxy: npmjs

logs:
  - {type: stdout, format: pretty, level: http}

listen: 0.0.0.0:4873

```
---
