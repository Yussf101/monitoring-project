# 📊 Grafana Monitoring Stack with Prometheus and Node Exporter (Docker-Based)

This repository contains a complete, production-ready **monitoring stack** for tracking multiple Linux systems' metrics(CPU, RAM, Storage...) using:

- [Grafana](https://grafana.com/): for dashboards and visualization
- [Prometheus](https://prometheus.io/): for scraping and storing metrics
- [Node Exporter](https://prometheus.io/docs/guides/node-exporter/): for exposing system metrics

Everything runs in **Docker containers** via `docker-compose`.

## 📆 Project Structure

```
monitoring-project/
├── docker-compose.yml
├── generate-prometheus-config.sh
├── prometheus/
│   ├── prometheus.template.yml
│   └── prometheus.yml
├── grafana/
│   ├── grafana.ini
│   └── provisioning/
│       ├── dashboards/
│       │   └── node-dashboard.json
│       ├── datasources/
│       │   └── prometheus-datasource.yml
│       └── variables/
│           └── variables.yml
├── vm_list.txt
├── README.md
```
---

## 📁 Contents Overview

| File/Folder                                                  | Description                                                           |
| ------------------------------------------------------------ | --------------------------------------------------------------------- |
| `docker-compose.yml`                                         | Defines and starts Grafana and Prometheus services using Docker       |
| `generate-prometheus-config.sh`                              | Script to generate `prometheus.yml` dynamically from `vm_list.txt`    |
| `prometheus/`                                                | Contains Prometheus configuration files                               |
| `prometheus/prometheus.template.yml`                         | Template with placeholder `${VM_LIST}` used to generate actual config |
| `prometheus/prometheus.yml`                                  | Generated config file with real targets for Prometheus                |
| `grafana/`                                                   | Contains Grafana configuration and provisioning folders               |
| `grafana/grafana.ini`                                        | Main Grafana configuration (port, login, etc.)                        |
| `grafana/provisioning/dashboards/node-dashboard.json`        | Dashboard with pre-defined system metrics panels                      |
| `grafana/provisioning/datasources/prometheus-datasource.yml` | Prometheus data source auto-provisioning                              |
| `grafana/provisioning/variables/variables.yml`               | (Optional) Allows defining custom variables in dashboards             |
| `vm_list.txt`                                                | List of IP addresses or hostnames of the VMs to monitor               |
| `README.md`                                                  | Full documentation of the project                                     |

---


---

## 🛠️ Prerequisites

- A Linux machine to run the monitoring stack (e.g., Ubuntu Server)
- Docker & Docker Compose installed


## 🚀 Running the Monitoring Stack on VM1 (Monitoring Server)

### Step 1: Clone the Repository

```bash
cd ~
git clone https://github.com/Yussf101/monitoring-project.git
cd monitoring-project
```

### Step 2: Add VM IPs to `vm_list.txt`

Edit the file:

```bash
nano vm_list.txt
```

Add one IP per line: (for example for 2 VMs)

```
192.168.28.129
192.168.28.130
```

### Step 3: Generate Prometheus Configuration

```bash
chmod +x generate-prometheus-config.sh
./generate-prometheus-config.sh
```

### Step 4: Start the Monitoring Stack

```bash
docker-compose up -d
```

Grafana: http\://:3000  \
Prometheus: http\://:9090  \
Login: `admin / admin`

---

## 📅 Setting Up a VM to Be Monitored (Client VM)

On each client VM (VM2, VM3, ...):

### Step 1: Install Docker (if not already installed)

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker --now
```

### Step 2: Run Node Exporter

```bash
docker run -d \
  --name node_exporter \
  -p 9100:9100 \
  --restart unless-stopped \
  -v /:/host:ro,rslave \
  prom/node-exporter \
  --path.rootfs=/host
```
*if it doesn't work, try this instead:*

```bash
docker run -d \
  --name node_exporter \
  -p 9100:9100 \
  --restart unless-stopped \
  prom/node-exporter
```

---

## ➕ Adding a New VM Later

1. On VM client: run Node Exporter (see above)
2. On VM1(monitoring server):
   - Edit `vm_list.txt` to add the new VM's IP in a new line
   - Regenerate config: `./generate-prometheus-config.sh`
   - Restart Prometheus:
     ```bash
     docker-compose restart prometheus
     ```
3. The new VM will appear in Grafana's instance dropdown automatically.

---

## 📊 Dashboards

The pre-provisioned dashboard `node-dashboard.json` includes:

- CPU usage (total and per core).
- RAM usage (bytes and %).
- Disk space (free and usage %).
- Network I/O.

Use the **instance** dropdown at the top of Grafana to filter by VM.

---

## ✨ Tips

- To monitor many VMs, use the multi-select `$instance` variable.
- Customize the dashboard to match your needs or make your own.
- You can add alerts and email integrations via Grafana Alerts.

---


