# Monitoring Stack: Prometheus + Grafana + Node Exporter

This repository provides a complete monitoring stack using Docker Compose, including:

- 📊 **Grafana** – powerful visualization and dashboard tool
- 📡 **Prometheus** – metrics collection and alerting system
- 🖥 **Node Exporter** – exposes system metrics (CPU, memory, disk, etc.)

---

## 📁 Project Structure

```md
monitoring
├── docker-compose.yml
├── grafana
│   └── provisioning
│       ├── dashboards
│       │   ├── dashboards.yml
│       │   └── node_exporter_dashboard.json
│       └── datasources
│           └── prometheus.yml
└── prometheus
    └── prometheus.yml
```
---

## 🚀 Getting Started

### 1. Clone the repository

```
bash
git clone https://github.com/YOUR_USERNAME/monitoring-stack.git
cd monitoring-stack
```
### 2. Download the Node Exporter dashboard for Grafana
```
wget https://grafana.com/api/dashboards/1860/revisions/31/download -O grafana/provisioning/dashboards/node_exporter_dashboard.json
```
### 🛠 Configuration

Prometheus (prometheus/prometheus.yml)
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']
```
Grafana Datasource (grafana/provisioning/datasources/prometheus.yml)
```
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```
Grafana Dashboard Loader (grafana/provisioning/dashboards/dashboards.yml)
```
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    options:
      path: /etc/grafana/provisioning/dashboards
      updateIntervalSeconds: 10
      allowUiUpdates: true
```
### 🧱 Running the Stack

Start the stack from the project root:
```
docker-compose up -d --build
```
Stop the stack:
```
docker-compose down
```

### 🌐 Access the Services

| Service       | URL                                                            | Notes                              |
| ------------- | -------------------------------------------------------------- | ---------------------------------- |
| Grafana       | [http://localhost:3000](http://localhost:3000)                 | Login: `admin` / `admin` (default) |
| Prometheus    | [http://localhost:9090](http://localhost:9090)                 | Query metrics directly             |
| Node Exporter | [http://localhost:9100/metrics](http://localhost:9100/metrics) | Raw system metrics                 |

### 🧼 Cleanup (Optional)

To remove persistent volumes (⚠️ this deletes all stored data):
```
docker volume rm monitoring_prometheus_data monitoring_grafana_data
```
### 📌 Notes

- Designed for local or dev use — not hardened for production.

- You can add more exporters by editing prometheus.yml and the Compose file.

- For production:

    - Secure Grafana with users and roles.

    - Set retention rules in Prometheus.

    - Add alerting and notification integrations.
 
### 📎 Resources

- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana Dashboards](https://grafana.com/grafana/dashboards/)
- [Node Exporter Dashboard (ID: 1860)](https://grafana.com/grafana/dashboards/1860-node-exporter-full/)

### ✅ TODO (Optional Enhancements)

- Add Alertmanager for notifications

- Add SSL with reverse proxy (e.g., Nginx, Traefik)

- Add container metrics with cAdvisor

- Create custom dashboards

---

## 🪟 Monitoring Windows Hosts with Windows Exporter

This stack supports monitoring Windows machines on your network using the [windows_exporter](https://github.com/prometheus-community/windows_exporter).

### ✅ 1. Install `windows_exporter` on Windows Hosts

1. Download the latest `.msi` installer from the [releases page](https://github.com/prometheus-community/windows_exporter/releases).
2. Run the installer with default settings (or enable specific collectors).
3. The exporter will start as a Windows service and expose metrics at: http://<windows-host>:9182/metrics

⚠️ Make sure port **9182** is allowed through the Windows firewall.

### ⚙️ 2. Add Windows Hosts to `prometheus.yml`

Edit the Prometheus configuration file:

```
  - job_name: 'win-exporter'
    static_configs:
      - targets: ['windows-host:9182']
```
Replace the IPs with your actual Windows host IP addresses.

Then restart Prometheus:
```
docker-compose restart prometheus
```
To verify the target is working, visit:
```
http://localhost:9090
```
And run this PromQL query:
```
up{job="windows"}
```
You should see a 1 for each working Windows exporter.

### 📊 3. Import Dashboard 14499 Automatically in Grafana

Grafana will provision a prebuilt dashboard for the Windows Exporter using its provisioning system.

a) Download the dashboard JSON
Run this in your project root:
```
wget https://grafana.com/api/dashboards/14694/revisions/2/download \
 -O grafana/provisioning/dashboards/windows_node_dashboard.json
```
c) Fix dashboard datasource variable (optional)
If you see ${DS_PROMETHEUS} in panel errors:
1. Open the JSON file windows_node_dashboard.json
2. Replace ${DS_PROMETHEUS} with Prometheus
You can do this using:
```
sed -i 's/\${DS_PROMETHEUS}/Prometheus/g' grafana/provisioning/dashboards/windows_node_dashboard.json
```
Then restart Grafana:
```
docker-compose restart grafana
```
