# DevOps Observability Project

## Prometheus Installation Guide

This guide provides step-by-step instructions for installing and configuring Prometheus on Linux systems.

### Prometheus Prerequisites

- Linux system with systemd
- sudo privileges  
- Internet connection

### Step 1: Create Prometheus System User

Create a dedicated system user for running Prometheus:

```bash
sudo useradd -r prometheus
```

**What this does:** Creates a system user with no home directory and no login shell for security.

### Step 2: Create Prometheus Directories

Create directories and set ownership for Prometheus:

```bash
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

**Directory purposes:**

- `/etc/prometheus/` - Configuration files
- `/var/lib/prometheus/` - Data storage (owned by prometheus user)

### Step 3: Download Prometheus

Download the latest Prometheus release:

```bash
cd /tmp/
wget https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-arm64.tar.gz
```

> **Note:** Replace `linux-arm64` with `linux-amd64` if you're on an x86_64 system.

### Step 4: Extract Prometheus Archive

```bash
tar -xvf prometheus-3.5.0.linux-arm64.tar.gz
cd prometheus-3.5.0.linux-arm64
```

### Step 5: Install Prometheus Configuration and Binaries

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

### Step 6: Create Prometheus Systemd Service

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

### Step 7: Start and Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl status prometheus
sudo systemctl enable prometheus
```

### Prometheus Verification

Check if Prometheus is running:

```bash
sudo systemctl status prometheus
sudo ss -tlnp | grep :9090
```

Access web interface: [http://localhost:9090](http://localhost:9090)

### Prometheus Troubleshooting

Check logs if issues occur:

```bash
sudo journalctl -u prometheus -f
```

## Alertmanager Installation Guide

This guide provides step-by-step instructions for installing and configuring Alertmanager on Linux systems.

### Alertmanager Prerequisites

- Prometheus already installed (see above)
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

## Grafana Installation Guide

This guide provides step-by-step instructions for installing and configuring Grafana visualization platform on Linux systems.

### Grafana Prerequisites

- Prometheus and Alertmanager already installed (see above)
- sudo privileges
- Internet connection

### Step 1: Install Dependencies

Install required system dependencies for Grafana:

```bash
sudo apt-get update
sudo apt-get install -y adduser libfontconfig1 musl
```

**What these dependencies do:**

- `adduser` - User management utilities
- `libfontconfig1` - Font configuration library for proper text rendering
- `musl` - C standard library implementation

### Step 2: Download Grafana

Download the latest Grafana Enterprise edition:

```bash
cd /tmp/
wget https://dl.grafana.com/grafana-enterprise/release/12.2.0/grafana-enterprise_12.2.0_17949786146_linux_arm64.deb
```

> **Note:** Replace `linux_arm64` with `linux_amd64` if you're on an x86_64 system.

### Step 3: Install Grafana Package

Install Grafana using the downloaded .deb package:

```bash
sudo dpkg -i grafana-enterprise_12.2.0_17949786146_linux_arm64.deb
```

**What this does:**

- Installs Grafana binaries
- Creates grafana user and group
- Sets up default configuration files
- Creates systemd service file

### Step 4: Configure and Start Service

Enable and start the Grafana service:

```bash
# Reload systemd daemon to recognize Grafana service
sudo systemctl daemon-reload

# Enable Grafana to start at boot
sudo systemctl enable grafana-server

# Start Grafana service
sudo systemctl start grafana-server

# Check service status
sudo systemctl status grafana-server
```

### Step 5: Initial Configuration

**Default Access Information:**

- **URL:** [http://localhost:3000](http://localhost:3000)
- **Default Username:** admin
- **Default Password:** admin

**First Login Steps:**

1. Open web browser and navigate to [http://localhost:3000](http://localhost:3000)
2. Login with admin/admin credentials
3. Change the default password when prompted

### Step 6: Add Prometheus as Data Source

**Via Web Interface:**

1. Go to **Configuration** → **Data Sources**
2. Click **Add data source**
3. Select **Prometheus**
4. Configure the following settings:
   - **Name:** Prometheus
   - **URL:** [http://localhost:9090](http://localhost:9090)
   - **Access:** Server (default)
5. Click **Save & Test**

### Grafana Verification

**1. Check Service Status:**

```bash
sudo systemctl status grafana-server
```

**2. Check Network Connectivity:**

```bash
sudo ss -tlnp | grep :3000
```

**3. Access Web Interface:**
Open your browser and navigate to: [http://localhost:3000](http://localhost:3000)

### Grafana Troubleshooting

Check logs if issues occur:

```bash
sudo journalctl -u grafana-server -f
```

## Node Exporter Installation Guide

This guide provides step-by-step instructions for installing and configuring Node Exporter to collect system metrics on Linux systems.

### Prerequisites

- Prometheus already installed (see above)
- sudo privileges
- Internet connection

### Step 1: Create System User

Create a dedicated system user for running Node Exporter:

```bash
sudo useradd -r node_exporter
```

**What this does:** Creates a system user with no home directory and no login shell for security.

### Step 2: Download Node Exporter

Download the latest Node Exporter release:

```bash
cd /tmp/
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-arm64.tar.gz
```

> **Note:** Replace `linux-arm64` with `linux-amd64` if you're on an x86_64 system.

### Step 3: Extract Archive

Extract the downloaded archive:

```bash
tar -xvf node_exporter-1.9.1.linux-arm64.tar.gz
cd node_exporter-1.9.1.linux-arm64/
```

### Step 4: Install Binary

Move the binary to system PATH and set proper permissions:

```bash
sudo mv node_exporter /usr/local/bin/
sudo chmod 755 /usr/local/bin/node_exporter
```

**What this installs:**

- `node_exporter` - Collects hardware and OS metrics from Linux systems

### Step 5: Create Systemd Service

Create the systemd service file:

```bash
sudo vim /etc/systemd/system/node_exporter.service
```

Add this configuration:

```ini
[Unit]
Description=Node Exporter
Documentation=https://prometheus.io/docs/guides/node-exporter/
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter \
    --web.listen-address=0.0.0.0:9100

[Install]
WantedBy=multi-user.target
```

**Service configuration explained:**

- Runs as `node_exporter` user for security
- Automatically restarts on failure
- Listens on port 9100 (default Node Exporter port)
- Collects system metrics automatically

### Step 6: Start and Enable Service

```bash
# Reload systemd daemon to recognize new service
sudo systemctl daemon-reload

# Start Node Exporter service
sudo systemctl start node_exporter

# Check service status
sudo systemctl status node_exporter

# Enable service to start at boot
sudo systemctl enable node_exporter
```

Restart Prometheus to apply the configuration:

```bash
sudo systemctl restart prometheus
```

### Verification

**1. Check Service Status:**

```bash
sudo systemctl status node_exporter
```

**2. Check Network Connectivity:**

```bash
sudo ss -tlnp | grep :9100
```

**3. Test Metrics Endpoint:**

```bash
curl http://localhost:9100/metrics
```

This should return a large amount of system metrics.

### Node Explorer Troubleshooting

Check logs if issues occur:

```bash
sudo journalctl -u node_exporter -f
```

## Prometheus Targets Configuration

This section explains how to configure Prometheus to scrape metrics from all the services in your monitoring stack.

### Overview

After installing all components (Prometheus, Alertmanager, Grafana, Node Exporter), you need to configure Prometheus to collect metrics from each service. This creates a comprehensive monitoring setup.

### Step 1: Edit Prometheus Configuration

Open the Prometheus configuration file:

```bash
sudo systemctl stop prometheus
sudo vim /etc/prometheus/prometheus.yml
```

### Step 2: Add All Monitoring Targets

Add the following jobs to the `scrape_configs` section of your `prometheus.yml`:

```yaml
# Global configuration
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - localhost:9093

# Rule files (for alerting rules)
rule_files:
  # - "alert_rules.yml"

# Scrape configuration
scrape_configs:
  # Prometheus itself
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  # Alertmanager monitoring
  - job_name: "alertmanager"
    static_configs:
      - targets: ["localhost:9093"]
    scrape_interval: 30s

  # Grafana monitoring  
  - job_name: "grafana"
    static_configs:
      - targets: ["localhost:3000"]
    scrape_interval: 30s
    metrics_path: '/metrics'

  # Node Exporter (system metrics)
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
    scrape_interval: 15s
```

### Step 3: Validate Configuration

Before restarting, validate the configuration syntax:

```bash
promtool check config /etc/prometheus/prometheus.yml
```

Expected output should show:

```bash
Checking /etc/prometheus/prometheus.yml
SUCCESS: /etc/prometheus/prometheus.yml is valid prometheus config file syntax
```

### Step 4: Restart Services

Restart Prometheus to apply the new configuration:

```bash
sudo systemctl restart prometheus
sudo systemctl status prometheus
```

### Target Explanation

**Service Monitoring Targets:**

| Service | Port | Purpose | Metrics Available |
|---------|------|---------|-------------------|
| **Prometheus** | 9090 | Self-monitoring | Query performance, storage, rule evaluation |
| **Alertmanager** | 9093 | Alert handling metrics | Alert processing, notification success/failure |
| **Grafana** | 3000 | Dashboard metrics | User sessions, dashboard usage, query performance |
| **Node Exporter** | 9100 | System metrics | CPU, memory, disk, network, hardware sensors |

**Scrape Intervals Explained:**

- **15s (default):** For critical system metrics (Prometheus, Node Exporter)
- **30s:** For application metrics (Alertmanager, Grafana) - less critical

### Step 5: Verification

**1. Check Targets Status:**
Navigate to the Prometheus targets page: [http://localhost:9090/targets](http://localhost:9090/targets)

All targets should show status **UP** in green.

**2. Verify Each Target:**

Test each target endpoint manually:

```bash
# Prometheus metrics
curl -s http://localhost:9090/metrics | head -5

# Alertmanager metrics  
curl -s http://localhost:9093/metrics | head -5

# Grafana metrics
curl -s http://localhost:3000/metrics | head -5

# Node Exporter metrics
curl -s http://localhost:9100/metrics | head -5
```

**3. Query Sample Metrics:**

In the Prometheus web UI ([http://localhost:9090](http://localhost:9090)), try these queries:

```promql
# Check all services are up
up

# Prometheus query rate
rate(prometheus_http_requests_total[5m])

# System CPU usage
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Grafana active users
grafana_stat_active_users

# Alertmanager alerts
alertmanager_alerts_active
```

### Troubleshooting

**Common Issues:**

1. **Target DOWN status:**

   ```bash
   # Check if service is running
   sudo systemctl status <service-name>
   
   # Check if port is listening
   sudo ss -tlnp | grep :<port>
   ```

2. **Configuration errors:**

   ```bash
   # Check Prometheus logs
   sudo journalctl -u prometheus -f
   
   # Validate config again
   promtool check config /etc/prometheus/prometheus.yml
   ```

3. **Network connectivity:**

   ```bash
   # Test connectivity to each target
   telnet localhost 9090
   telnet localhost 9093
   telnet localhost 3000
   telnet localhost 9100
   ```

## Test with Stress tool

You can use the `stress` tool to generate CPU and memory load on your system, which can help you test the monitoring setup. Here's how to install and use it:

### Step 1: Install Stress Tool

```bash
sudo apt-get update
sudo apt-get install -y stress
```

### Step 2: Run Stress Test

To generate CPU and memory load, run the following command:

```bash
stress --cpu 4 --vm 2 --vm-bytes 256M --timeout 300s
```

**Parameters explained:**

- `--cpu 4`: Spawns 4 CPU stressors to load the CPU.
- `--vm 2`: Spawns 2 memory stressors to allocate memory.
- `--vm-bytes 256M`: Each memory stressor allocates 256MB
- `--timeout 300s`: Runs the stress test for 300 seconds (5 minutes).

### Step 3: Monitor Metrics

While the stress test is running, you can monitor the metrics in Prometheus and Grafana to see how your system handles the load. Look for CPU usage, memory usage, and other relevant metrics.
