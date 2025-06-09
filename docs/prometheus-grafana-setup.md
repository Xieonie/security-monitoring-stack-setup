# Prometheus & Grafana Setup Guide

This guide outlines the steps to set up Prometheus for metrics collection and Grafana for visualization, typically using Docker Compose for ease of deployment.

## Prerequisites

* Docker and Docker Compose installed on your server.
* Basic understanding of metrics monitoring.

## 1. Directory Structure

It's good practice to organize your configuration files. Create a directory structure like this for your Prometheus/Grafana setup (can be part of the main `security-monitoring-stack-setup` repo):

```
	prometheus-grafana/
	├── docker-compose.yml
	├── prometheus/
	│   ├── prometheus.yml
	│   └── rules/
	│       └── security-alerts.rules.yml # Or other rule files
	└── alertmanager/
	└──	alertmanager.yml
	└── grafana_data/ # Docker will create this for persistent Grafana data
	└── prometheus_data/ # Docker will create this for persistent Prometheus data
```

*Place the `docker-compose.yml` from `docker-compose/prometheus-grafana-compose.yml` in this `prometheus-grafana/` directory.*
*Place `prometheus.yml.example` as `prometheus/prometheus.yml`.*
*Place `alertmanager.yml.example` as `alertmanager/alertmanager.yml`.*
*Place `security-alerts.rules.yml` in `prometheus/rules/`.*

## 2. Docker Compose Configuration

Use the `docker-compose/prometheus-grafana-compose.yml` file from this repository. Place it in your `prometheus-grafana/` directory. Key aspects:

* **Prometheus:**
    * Mounts `prometheus.yml` for configuration.
    * Mounts a directory for alert rules.
    * Mounts a volume for persistent data (`prometheus_data`).
* **Alertmanager:**
    * Mounts `alertmanager.yml` for configuration.
    * Mounts a volume for persistent data (`alertmanager_data`).
* **Grafana:**
    * Mounts a volume for persistent data (`grafana_data`) which stores dashboards, data sources, etc.
    * Sets admin user/password via environment variables (change these!).
* **Node Exporter (Optional in Compose):**
    * You might run `node_exporter` directly on the hosts you want to monitor or as a Docker container on each host (not typically in the central monitoring stack's compose file unless monitoring the Docker host itself this way).

## 3. Prometheus Configuration (`prometheus/prometheus.yml`)

This file defines global settings, scrape configurations for targets, and rule file locations.
Refer to `prometheus/prometheus.yml.example` in this repository.

**Key sections:**
* `global`: `scrape_interval`, `evaluation_interval`.
* `rule_files`: Points to your alert rule files (e.g., `rules/*.rules.yml`).
* `scrape_configs`: Defines jobs to scrape metrics from targets.
    * Example for Prometheus itself:
      ```yaml
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      ```
    * Example for Node Exporter (if running on host `192.168.1.10`):
      ```yaml
      - job_name: 'node_exporter_server1'
        static_configs:
          - targets: ['192.168.1.10:9100']
      ```
* `alerting`: Configures Alertmanager instances.

## 4. Alertmanager Configuration (`alertmanager/alertmanager.yml`)

This file defines how alerts are routed, grouped, and what receivers (email, Slack, etc.) are used.
Refer to `prometheus/alertmanager.yml.example` in this repository.

**Key sections:**
* `global`: SMTP settings, Slack API URL, etc.
* `route`: Defines the default routing tree for alerts.
    * `receiver`: Default receiver.
    * `group_by`: How to group alerts.
    * `routes`: More specific routing rules based on labels.
* `receivers`: Defines notification channels.
    * `name`: Name of the receiver.
    * `email_configs`, `slack_configs`, `webhook_configs`, etc.

## 5. Prometheus Alerting Rules (`prometheus/rules/security-alerts.rules.yml`)

Define your alerting conditions here.
Refer to `prometheus/rules/security-alerts.rules.yml` in this repository for examples.

**Example rule:**
```yaml
groups:
- name: HostAlerts
  rules:
  - alert: HostHighCpuLoad
    expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host high CPU load on {{ $labels.instance }}"
      description: "{{ $labels.instance }} CPU load is {{ $value }}% for the last 5 minutes."
```


## 6. Start Services

	Navigate to your prometheus-grafana/ directory and run:
```
docker-compose up -d
```

## 7. Access Services

Prometheus: ```http://<your_server_ip>:9090```
		Check ```Status``` -> ```Targets``` to see if Prometheus is scraping your configured targets.
		Check ```Alerts``` to see active alerts.

Alertmanager: ```http://<your_server_ip>:9093```

Grafana: ```http://<your_server_ip>:3000```
		Log in with the admin credentials defined in ```docker-compose.yml``` (default: ```admin/changethisgrafanapassword``` - ```CHANGE THIS!```).



## 8. Configure Grafana

  1.  Add Data Source:

      Go to ```Configuration (gear icon)``` -> ```Data Sources```.

      Click ```Add data source```.

      Select ```Prometheus```.

      HTTP URL: ```http://prometheus:9090``` (Grafana can reach Prometheus via its service name because they are in the same Docker Compose network).

      Click ```Save & Test```.



  2. Import/Create Dashboards:

      You can import dashboards from Grafana.com (e.g., Node Exporter Full dashboard ID: ```1860```).

      You can create your own dashboards by adding panels and querying your Prometheus data source.

      An example basic dashboard JSON structure is provided in ```grafana-dashboards/system-overview-dashboard.json```. You can import this via ```Create (plus icon)``` -> ```Import```


## Node Exporter Setup (on monitored hosts)

Prometheus needs exporters to gather metrics from your target hosts. ```node_exporter``` is common for system-level metrics.

### Download and Run Node Exporter:
    Go to the Prometheus downloads page and get the latest release for your target OS.

    Extract and run it (typically listens on port ```9100```).

```
# Example for Linux

wget [https://github.com/prometheus/node_exporter/releases/download/vX.Y.Z/node_exporter-X.Y.Z.linux-amd64.tar.gz](https://github.com/prometheus/node_exporter/releases/download/vX.Y.Z/node_exporter-X.Y.Z.linux-amd64.tar.gz)
tar xvfz node_exporter-X.Y.Z.linux-amd64.tar.gz
cd node_exporter-X.Y.Z.linux-amd64
./node_exporter
```

   Consider running ```node_exporter``` as a systemd service for persistence.


### Add to Prometheus Config: Add the host and port (e.g., ```your_target_host_ip:9100```) to your ```prometheus.yml``` under ```scrape_configs```.

### Firewall: Ensure your Prometheus server can reach port ```9100``` (or the configured port) on the target hosts.