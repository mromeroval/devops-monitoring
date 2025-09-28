# DevOps Observability Project


## Prometheus Installation Guide

This guide provides step-by-step instructions for installing and configuring Prometheus on Linux systems.

## Prerequisites
- Linux system with systemd
- sudo privileges  
- Internet connection

## Step 1: Create System User

Create a dedicated system user for running Prometheus:

```bash
sudo useradd -r prometheus
```

**What this does:** Creates a system user with no home directory and no login shell for security.

## Step 2: Create Directories

Create directories and set ownership for Prometheus:

```bash
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

**Directory purposes:**
- `/etc/prometheus/` - Configuration files
- `/var/lib/prometheus/` - Data storage (owned by prometheus user)

## Step 3: Download Prometheus

Download the latest Prometheus release:

```bash
cd /tmp/
wget https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-arm64.tar.gz
```

> **Note:** Replace `linux-arm64` with `linux-amd64` if you're on an x86_64 system.

## Step 4: Extract Archive

```bash 
tar -xvf prometheus-3.5.0.linux-arm64.tar.gz
cd prometheus-3.5.0.linux-arm64
```

## Step 5: Install Configuration and Binaries

Move configuration file and set ownership:

```bash
sudo mv prometheus.yml /etc/prometheus/
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

Move binaries to system PATH:

```bash 
sudo mv prometheus promtool /usr/local/bin/
sudo chmod 755 /usr/local/bin/prometheus /usr/local/bin/promtool
```

## Step 6: Create Systemd Service

Create the service file:

```bash 
sudo vim /etc/systemd/system/prometheus.service
```

Add this configuration:

```ini
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090

[Install]
WantedBy=multi-user.target
```

## Step 7: Start and Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl status prometheus
sudo systemctl enable prometheus
```

## Verification

Check if Prometheus is running:

```bash
sudo systemctl status prometheus
sudo ss -tlnp | grep :9090
```

Access web interface: http://localhost:9090

## Troubleshooting

Check logs if issues occur:
```bash
sudo journalctl -u prometheus -f
```
