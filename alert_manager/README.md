# Alertmanager Installation Guide

This guide provides step-by-step instructions for installing and configuring Alertmanager on Linux systems.

## Prerequisites

- A Linux server (Ubuntu, Debian, CentOS, etc.)
- sudo privileges
- Internet connection

### Step 1: Create Alertmanager System User

Create a dedicated system user for running Alertmanager:

```bash
sudo useradd -r alertmanager
```

**What this does:** Creates a system user with no home directory and no login shell for security.

### Step 2: Create Alertmanager Directories

Create directories for Alertmanager configuration and data:

```bash
sudo mkdir -p /etc/alertmanager /var/lib/alertmanager
sudo chown alertmanager:alertmanager /var/lib/alertmanager
```

**Directory purposes:**

- `/etc/alertmanager/` - Configuration files
- `/var/lib/alertmanager/` - Data storage (owned by alertmanager user)

### Step 3: Download Alertmanager

Download the latest Alertmanager release:

```bash
cd /tmp/
wget https://github.com/prometheus/alertmanager/releases/download/v0.28.1/alertmanager-0.28.1.linux-arm64.tar.gz
```

> **Note:** Replace `linux-arm64` with `linux-amd64` if you're on an x86_64 system.

### Step 4: Extract Alertmanager Archive

```bash
tar -xvf alertmanager-0.28.1.linux-arm64.tar.gz
cd alertmanager-0.28.1.linux-arm64/
```

### Step 5: Install Alertmanager Configuration and Binaries

Move configuration file and set ownership:

```bash
sudo mv alertmanager.yml /etc/alertmanager/
sudo chown alertmanager:alertmanager /etc/alertmanager/alertmanager.yml
```

Move binaries to system PATH and set proper permissions:

```bash
sudo mv alertmanager amtool /usr/local/bin/
sudo chmod 755 /usr/local/bin/alertmanager /usr/local/bin/amtool
```

**What this installs:**

- `alertmanager` - Main alertmanager binary
- `amtool` - Command-line tool for interacting with Alertmanager

### Step 6: Create Alertmanager Systemd Service

Create the systemd service file:

```bash
sudo vim /etc/systemd/system/alertmanager.service
```

Add this configuration:

```ini
[Unit]
Description=Alertmanager
Documentation=https://prometheus.io/docs/alerting/alertmanager/
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/alertmanager \
    --config.file=/etc/alertmanager/alertmanager.yml \
    --storage.path=/var/lib/alertmanager/ \
    --web.listen-address=0.0.0.0:9093

[Install]
WantedBy=multi-user.target
```

### Step 7: Update Prometheus Configuration

Update Prometheus configuration to include Alertmanager:

```bash
sudo systemctl stop prometheus
sudo vim /etc/prometheus/prometheus.yml
```

Update the Alertmanager configuration section:

```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - localhost:9093
```

### Step 8: Start and Enable Services

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl start alertmanager
sudo systemctl status prometheus
sudo systemctl status alertmanager
sudo systemctl enable alertmanager
```

### Step 9: Configure amtool

Create amtool configuration directory and file:

```bash
sudo mkdir /etc/amtool
sudo vim /etc/amtool/config.yml
```

Add this configuration:

```yaml
alertmanager.url: http://localhost:9093
```

Check `amtool` configuration by displaying the current running configuration of the Prometheus Alertmanager server:

```bash
amtool config show
```

### Alertmanager Verification

Check if Alertmanager is running:

```bash
sudo systemctl status alertmanager
sudo ss -tlnp | grep :9093
```

Access web interface: [http://localhost:9093](http://localhost:9093)

### Alertmanager Troubleshooting

Check logs if issues occur:

```bash
sudo journalctl -u alertmanager -f
```

## High Availability Setup

To set up Alertmanager in a high-availability (HA) configuration, you need to run multiple instances of Alertmanager and configure them to form a cluster.

### Prerequisites for HA
- Multiple servers (minimum 2, recommended 3 for proper quorum)
- Network connectivity between all Alertmanager instances
- Same Alertmanager configuration file on all nodes

### Step 1: Install Alertmanager on Additional Servers

Follow steps 1-6 from the main installation guide on each additional server.

### Step 2: Configure Clustering

Modify the systemd service file on each server to include cluster configuration:

**Primary Node (Server 1):**
```ini
[Unit]
Description=Alertmanager
Documentation=https://prometheus.io/docs/alerting/alertmanager/
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/alertmanager \
    --config.file=/etc/alertmanager/alertmanager.yml \
    --storage.path=/var/lib/alertmanager/ \
    --web.listen-address=0.0.0.0:9093 \
    --cluster.listen-address=0.0.0.0:9094

[Install]
WantedBy=multi-user.target
```

**Secondary Nodes (Server 2, 3, etc.):**
```ini
[Unit]
Description=Alertmanager
Documentation=https://prometheus.io/docs/alerting/alertmanager/
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/alertmanager \
    --config.file=/etc/alertmanager/alertmanager.yml \
    --storage.path=/var/lib/alertmanager/ \
    --web.listen-address=0.0.0.0:9093 \
    --cluster.listen-address=0.0.0.0:9094 \
    --cluster.peer=<primary-node-ip>:9094

[Install]
WantedBy=multi-user.target
```

### Step 3: Update Prometheus Configuration

Update Prometheus to use all Alertmanager instances:

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - <server1-ip>:9093
          - <server2-ip>:9093
          - <server3-ip>:9093
```

### Step 4: Start Services

```bash
# On all servers
sudo systemctl daemon-reload
sudo systemctl restart alertmanager
sudo systemctl status alertmanager
```

### HA Verification

Check cluster status on any node:
```bash
curl http://localhost:9093/api/v1/status
```

Verify cluster members:
```bash
amtool cluster show
```