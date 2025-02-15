---
layout: Prometheus + Grafana 监控：PVE、TrueNAS 和树莓派接入指南
title: Prometheus + Grafana 监控：PVE、TrueNAS 和树莓派接入指南
tags:
  - prometheus
  - grafana
  - docker
  - pve
  - truenas
  - raspberry pi
categories:
  - - 折腾记录
date: 2025-02-15 20:50:17
---

在现代 IT 环境中，监控系统是确保基础设施稳定运行的核心工具。Prometheus 和 Grafana 是目前最受欢迎的开源监控解决方案之一，凭借其强大的功能和灵活性，广泛应用于各种场景。

---

# 引言

随着业务的扩展和技术的进步，服务器、虚拟机和物联网设备的数量不断增加。为了确保这些系统的稳定性和性能，实时监控变得至关重要。Prometheus 是一个强大的开源监控和报警工具，而 Grafana 则是一个功能丰富的数据可视化平台。两者的结合能够为用户提供全面的监控解决方案。

本文的目标是帮助你完成以下任务：

- 安装并配置 Prometheus 和 Grafana（包括使用 Docker 部署）。
    
- 将 PVE、TrueNAS 和树莓派接入到 Prometheus 监控系统中。
    
- 在 Grafana 中创建直观的仪表盘，展示各个设备和系统的性能数据。
    

---

# 使用 Docker 部署 Prometheus 和 Grafana

## 什么是 Docker？

Docker 是一个开源的容器化平台，允许你将应用程序及其依赖打包到一个容器中，从而实现跨平台的快速部署和一致性。

## 安装 Docker

在 Linux 系统上，你可以使用以下命令安装 Docker：

```bash
# 安装 Docker
sudo apt update
sudo apt install -y docker.io

# 验证 Docker 是否安装成功
sudo docker --version
```

## 使用 Docker 部署 Grafana

Grafana 可以通过 Docker 快速部署。以下是部署命令：


```bash
# 创建存储目录
sudo mkdir -p /opt/grafana-storage
sudo mkdir -p /opt/grafana-conf

# 运行 Grafana 容器
sudo docker run -d \
  -p 3000:3000 \
  --name=grafana \
  --restart=always \
  -v /opt/grafana-storage:/var/lib/grafana \
  -v /opt/grafana-conf:/etc/grafana \
  grafana/grafana
```

- `-d`：后台运行容器。
    
- `-p 3000:3000`：将容器的 3000 端口映射到宿主机的 3000 端口。
    
- `--name=grafana`：为容器指定名称。
    
- `--restart=always`：设置容器在重启后自动启动。
    
- `-v /opt/grafana-storage:/var/lib/grafana`：将 Grafana 的数据存储目录挂载到宿主机。
    
- `-v /opt/grafana-conf:/etc/grafana`：将 Grafana 的配置文件目录挂载到宿主机。
    

## 使用 Docker 部署 Prometheus

Prometheus 也可以通过 Docker 快速部署。以下是部署命令：

```bash
# 创建存储目录
sudo mkdir -p /opt/prometheus/data

# 创建 Prometheus 配置文件
sudo nano /opt/prometheus/prometheus.yml
```

在 `prometheus.yml` 中添加以下内容：

```yaml
global:
  scrape_interval: 30s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'pve'
    static_configs:
      - targets: ['your-pve-server:9411']

  - job_name: 'truenas'
    static_configs:
      - targets: ['your-truenas-server:9412']

  - job_name: 'raspberry-pi'
    static_configs:
      - targets: ['your-raspberry-pi-ip:9100']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['your-raspberry-pi-ip:8080']
```

保存并退出。

运行 Prometheus 容器：


```bash
# 运行 Prometheus 容器
sudo docker run -d \
  -p 9090:9090 \
  --name=prometheus \
  --restart=always \
  -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
  -v /opt/prometheus/data:/prometheus \
  prom/prometheus
```

- `-d`：后台运行容器。
    
- `-p 9090:9090`：将容器的 9090 端口映射到宿主机的 9090 端口。
    
- `--name=prometheus`：为容器指定名称。
    
- `--restart=always`：设置容器在重启后自动启动。
    
- `-v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml`：将 Prometheus 的配置文件挂载到容器中。
    
- `-v /opt/prometheus/data:/prometheus`：将 Prometheus 的数据存储目录挂载到宿主机。
    

## 验证 Docker 容器

验证 Prometheus 和 Grafana 容器是否运行正常：

```bash
sudo docker ps
```

访问 Grafana：

- 打开浏览器，访问 `http://your-grafana-ip:3000`，默认用户名和密码为 `admin`。
    

访问 Prometheus：

- 打开浏览器，访问 `http://your-prometheus-ip:9090`。
    

---

# 将 PVE 接入 Prometheus

## 安装 Prometheus PVE Exporter

Prometheus PVE Exporter 是一个用于监控 Proxmox VE（PVE）的 Prometheus Exporter，可以通过 `pip3` 安装。

### 安装步骤

1. **更新软件包并安装 Python 3 环境**
    
    ```bash
    apt update && apt upgrade -y
    apt install python3 python3-pip -y
    ```
    
2. **安装 prometheus-pve-exporter**
    
    ```bash
    pip3 install prometheus-pve-exporter
    ```
    
    安装成功后，文件会存储在 `/usr/local/bin/pve_exporter`。
    
3. **创建 PVE 监控用户**
    
    - 在 PVE 的 Web 界面中，创建一个用户专用于 Prometheus 监控，用户名为 `monitoring`，领域选择 `Proxmox VE authentication server`，并设置密码。
        
    - 为用户设置权限：数据中心 -> 权限 -> 添加 -> 用户权限，路径选择 `/`，用户选择刚才创建的，权限选择 `PVEAuditor`。
        
4. **创建配置文件**
    
    使用您熟悉的编辑器创建配置文件 `/etc/prometheus-pve-exporter/pve.yml`：
    
    ```yaml
    default:
      user: monitoring@pve
      password: 你刚才设置的密码
      verify_ssl: false
    ```
    
5. **创建 systemd 服务**
    
    创建 systemd 服务文件 `/etc/systemd/system/prometheus-pve-exporter.service`：

    
```ini
    [Unit]
    Description=Proxmox Exporter
    After=network.target
    
    [Service]
    ExecStart=/usr/local/bin/pve_exporter --config.file /etc/prometheus-pve-exporter/pve.yml --web.listen-address=:9221
    Restart=always
    
    [Install]
    WantedBy=multi-user.target
```
    
    保存并关闭文件，然后执行以下命令启动服务：
    
```bash
systemctl daemon-reload
systemctl start prometheus-pve-exporter
systemctl enable prometheus-pve-exporter
```
    
6. **验证 Exporter 是否运行**
    
    通过以下命令检查服务状态：
    
    ```bash
    systemctl status prometheus-pve-exporter
    ```
    
    如果正常运行，您可以通过浏览器访问 `http://<PVE_SERVER_IP>:9221/pve` 查看数据。

    
## 在 Prometheus 中配置 PVE 监控

1. **编辑 Prometheus 配置文件**： 编辑 Prometheus 的配置文件 `/etc/prometheus/prometheus.yml`：
    
    ```bash
    sudo nano /etc/prometheus/prometheus.yml
    ```
    
2. **添加 PVE 监控配置**： 在文件末尾添加以下内容（替换 `your-pve-ip` 为实际的 PVE 服务器 IP 地址）：
    
    ```yaml
    - job_name: 'pve'
      metrics_path: /pve
      static_configs:
        - targets: ['your-pve-ip:9221']
    ```
    
3. **重启 Prometheus**：
    
    bash复制
    
    ```bash
    sudo systemctl restart prometheus
    ```
    

## 访问 Prometheus PVE Exporter

访问 Prometheus PVE Exporter 的 Web 界面：

- 打开浏览器，访问 `http://your-pve-ip:9221`。
    
- 在页面中，你可以通过 `/pve?target=your-pve-ip` 查看监控数据。

---

# 将 TrueNAS 接入 Prometheus

注：参考项目 [truenas-graphite-to-prometheus](https://github.com/Supporterino/truenas-graphite-to-prometheus)
## 在 TrueNAS 中运行 Exporter

TrueNAS 没有官方的 Prometheus Exporter，但可以使用自定义应用来实现。以下是具体步骤：

1. **创建自定义应用**
    
    - 在 TrueNAS 中，打开 `Apps`（应用）选项卡。
        
    - 点击 `Discover Apps`（发现应用），然后选择 `Custom App`（自定义应用）。
        
    - 按照提示填写应用名称，使用默认版本。
        
    - 在镜像设置中，使用以下镜像：
        
        - 镜像名称：`ghcr.io/supporterino/truenas-graphite-to-prometheus`
            
        - 镜像标签：`latest`
            
    - 镜像策略可以保持默认设置。
        
2. **配置端口转发**
    
    - 需要转发两个端口，一个用于 Graphite，另一个用于 Prometheus。
        
    - Graphite 的容器端口为 `9109`，节点端口可以使用相同端口或指定其他可用端口，协议为 `TCP`。
        
    - Prometheus 的容器端口为 `9108`，同样使用 `TCP` 协议。
        
    - 配置示例如下：
        
        - Graphite：`9109`（TCP）
            
        - Prometheus：`9108`（TCP）
            
3. **设置资源限制**
    
    - Exporter 非常轻量级，因此通常不需要设置资源限制。

## 在 Prometheus 中配置 TrueNAS 监控

1. **编辑 Prometheus 配置文件**
    
    ```bash
    sudo nano /etc/prometheus/prometheus.yml
    ```
    
2. **添加 TrueNAS 监控配置**
    
    在文件末尾添加以下内容（替换 `your-truenas-server` 为实际的 TrueNAS 服务器地址）：
    
    ```yaml
    - job_name: 'truenas'
      static_configs:
        - targets: ['your-truenas-server:9108']
    ```
    
3. **重启 Prometheus**
    
    ```bash
    sudo systemctl restart prometheus
    ```
        
---
# 将 Raspberry Pi 接入 Prometheus

## 使用 Docker 部署 `node_exporter` 和 `cadvisor`

### 树莓派安装 Docker

在树莓派上安装 Docker 可以通过以下步骤完成：

1. **更新软件包**
    
    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```
    
2. **安装 Docker**
    
    ```bash
    sudo apt install -y docker.io
    ```
    
3. **验证 Docker 安装**
    
    ```bash
    sudo docker --version
    ```
    
4. **将当前用户添加到 docker 组**
    
    ```bash
    sudo usermod -aG docker pi
    ```
    
    使更改生效：
    
    ```bash
    newgrp docker
    ```

### 使用 Docker 部署 `node_exporter`

运行 `node_exporter` 容器：

```bash
# 运行 node_exporter 容器
sudo docker run -d \
  -p 9100:9100 \
  --name=node_exporter \
  --restart=always \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /:/rootfs:ro \
  prom/node-exporter \
  --path.procfs /host/proc \
  --path.sysfs /host/sys \
  --collector.filesystem.ignored-mount-points \
  "^/host/(sys|proc|dev|etc|var/lib/docker/containers|var/lib/docker/)"
```

- `-d`：后台运行容器。
    
- `-p 9100:9100`：将容器的 9100 端口映射到宿主机的 9100 端口。
    
- `--name=node_exporter`：为容器指定名称。
    
- `--restart=always`：设置容器在重启后自动启动。
    
- `-v /proc:/host/proc:ro`：将宿主机的 `/proc` 挂载到容器中。
    
- `-v /sys:/host/sys:ro`：将宿主机的 `/sys` 挂载到容器中。
    
- `-v /:/rootfs:ro`：将宿主机的根文件系统挂载到容器中。
    

### 使用 Docker 部署 `cadvisor`

运行 `cadvisor` 容器：

```bash
# 运行 cadvisor 容器
sudo docker run --restart=always \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:v0.49.2 \
  --housekeeping_interval=15s \
  --docker_only=true \
  --store_container_labels=false
```

- `--restart=always`：设置容器在重启后自动启动。
    
- `--volume=/:/rootfs:ro`：将宿主机的根文件系统挂载到容器中。
    
- `--volume=/var/run:/var/run:ro`：将宿主机的 `/var/run` 挂载到容器中。
    
- `--volume=/sys:/sys:ro`：将宿主机的 `/sys` 挂载到容器中。
    
- `--volume=/var/lib/docker/:/var/lib/docker:ro`：将宿主机的 Docker 数据目录挂载到容器中。
    
- `--volume=/dev/disk/:/dev/disk:ro`：将宿主机的磁盘设备挂载到容器中。
    
- `--publish=8080:8080`：将容器的 8080 端口映射到宿主机的 8080 端口。
    
- `--detach=true`：后台运行容器。
    
- `--name=cadvisor`：为容器指定名称。
    
- `--privileged`：赋予容器特权模式。
    
- `--device=/dev/kmsg`：将宿主机的 `/dev/kmsg` 设备挂载到容器中。
    

### 在 Prometheus 中配置树莓派监控

在 `prometheus.yml` 中添加以下内容：

```yaml
  - job_name: 'raspberry-pi-node-exporter'
    static_configs:
      - targets: ['your-raspberry-pi-ip:9100']

  - job_name: 'raspberry-pi-cadvisor'
    static_configs:
      - targets: ['your-raspberry-pi-ip:8080']
```

重启 Prometheus：

```bash
sudo systemctl restart prometheus
```

---

# 在 Grafana 中创建 Dashboard

## 添加 Prometheus 数据源

在 Grafana 中，导航到 `Configuration` > `Data Sources`，添加一个 Prometheus 数据源。

1. 点击 `Add data source`。
    
2. 选择 `Prometheus`。
    
3. 在 `HTTP` 选项卡中，输入 Prometheus 的地址（如 `http://localhost:9090`）。
    
4. 点击 `Save & Test`，确保连接成功。
    
## 创建监控图表

进入 `Dashboard`，点击 `Add Panel`，选择 `import`

- raspberry仪表盘： [raspberry-pi-docker-monitoring](https://grafana.com/grafana/dashboards/15120-raspberry-pi-docker-monitoring/)
- pve仪表盘：[proxmox-via-prometheus](https://grafana.com/grafana/dashboards/10347-proxmox-via-prometheus/)
- truenas仪表盘：在上面提到的项目`truenas-graphite-to-prometheus`的Dashboard目录中
