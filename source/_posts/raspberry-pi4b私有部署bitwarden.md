---
title: Raspberry Pi4b私有部署Bitwarden
tags:
  - Bitwarden
  - Docker
  - Raspberry Pi4b
  - 树莓派
  - 树莓派4b
id: '403'
categories:
  - - 嵌入式开发
comments: false
date: 2022-04-09 17:10:15
---

## 准备

*   烧写好系统镜像的Raspberry Pi4b
*   良好的网络环境

## 开始

*   本文使用 [bitwordken\_rs](https://github.com/dani-garcia/vaultwarden "bitwordken_rs")
*   使用 Docker（树莓派上使用 docker 参考 [Raspberry Pi4b安装Docker](https://www.taoidle.com/raspberry-pi4b%e5%ae%89%e8%a3%85docker.html "Raspberry Pi4b安装Docker")）

* * *

### 下载运行docker镜像

**1.使用 Docker 拉取最新版本的 Bitwarden RS**

```bash
docker pull vaultwarden/server:latest
```

**2.运行Bitwarden镜像**

```bash
sudo docker run -d --name bitwarden  --restart=always \
    -v /bw-data/:/data/ -p 127.0.0.1:8080:80 \
    -p 127.0.0.1:3012:3012 vaultwarden/server:latest
```

这里将网络服务器暴露在端口 8080 上，端口 3012，这是 Bitwarden 的 Web 套接字用来进行通信的端口。

* * *

### 配置NGINX

**1.下载nginx**

```bash
sudo apt install nginx
```

**2.停止运行nginx**

```bash
sudo systemctl stop nginx
```

**3.为nginx生成一个 SSL 证书, 在 Raspberry Pi 上生成证书有两种不同的方式**

*   自签证书
*   Lets Encrypt等

**4\. 使用以下命令生成自签名证书**

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout /etc/ssl/private/nginx-bitwarden.key -out /etc/ssl/certs/nginx-bitwarden.crt
```

**5.创建一个 Diffie-Hellman**

```bash
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

**6.查看 NGINX 的默认配置文件**

```bash
cat /etc/nginx/nginx.conf
```

![file](https://cdn.site.taoidle.com/wp-content/uploads/2022/04/1649492956-image-1649492954071.png)

**7.为 NGINX 创建一个新的配置文件**

```bash
sudo vim /etc/nginx/conf.d/bitwarden.conf
```

**8.编写配置**

```
server {
    listen 80;
    listen [::]:80;
    server_name localhost;
    return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;
  server_name localhost;

  ssl_certificate      /etc/ssl/certs/nginx-bitwarden.crt;
  ssl_certificate_key  /etc/ssl/private/nginx-bitwarden.key;

  ssl_dhparam /etc/ssl/certs/dhparam.pem;

  client_max_body_size 128M;

  location / {
    proxy_pass http://0.0.0.0:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  location /notifications/hub {
    proxy_pass http://0.0.0.0:3012;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }

  location /notifications/hub/negotiate {
    proxy_pass http://0.0.0.0:8080;
  }
}
```

**9.保存退出重启NGINX**

```bash
sudo systemctl restart nginx
```

* * *

### 创建 Bitwarden 账户

```
https://你的树莓派ip
```

![file](https://cdn.site.taoidle.com/wp-content/uploads/2022/04/1649494073-image-1649494070786.png)

### 启用 Bitwarden 管理面板

**1.生成管理员令牌**

```bash
openssl rand -base64 48
```

**2.停止当前正在运行的容器**

```bash
sudo docker stop bitwarden
```

**3.删除现有的 Bitwarden 容器**

```bash
sudo docker rm bitwarden
```

**4.需要重新运行 docker 命令**

```bash
sudo docker run -d --name bitwarden  -e ADMIN_TOKEN=你的管理员令牌 \
    --restart=always -v /bw-data/:/data/ \
    -p 127.0.0.1:8080:80 -p 127.0.0.1:3012:3012 \
    vaultwarden/server:latest
```

### 访问 Bitwarden 管理面板

**1.访问管理面板**

```
https://你的树莓派ip/admin
```

![file](https://cdn.site.taoidle.com/wp-content/uploads/2022/04/1649495073-image-1649495070774.png) 输入你的管理员令牌

**2.管理您的用户以及配置 Bitwarden 的设置**

![file](https://cdn.site.taoidle.com/wp-content/uploads/2022/04/1649495182-image-1649495179498.png)

### 禁用 Bitwarden 新用户创建

![file](https://cdn.site.taoidle.com/wp-content/uploads/2022/04/1649495329-image-1649495327106.png) ![file](https://cdn.site.taoidle.com/wp-content/uploads/2022/04/1649495363-image-1649495361101.png) ![file](https://cdn.site.taoidle.com/wp-content/uploads/2022/04/1649495397-image-1649495394882.png)