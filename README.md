# 🚀 Production Observability Stack

A production-pattern monitoring and alerting stack built with **Prometheus, Grafana, Loki, Promtail, Alertmanager, Node Exporter, and Blackbox Exporter** — fully containerized with Docker and integrated with **Rocket.Chat** for real-time alerting.

![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge\&logo=prometheus\&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge\&logo=grafana\&logoColor=white)
![Loki](https://img.shields.io/badge/Loki-F0A500?style=for-the-badge\&logo=grafana\&logoColor=white)
![Promtail](https://img.shields.io/badge/Promtail-3F51B5?style=for-the-badge\&logo=grafana\&logoColor=white)
![Alertmanager](https://img.shields.io/badge/Alertmanager-E6522C?style=for-the-badge\&logo=prometheus\&logoColor=white)
![Node Exporter](https://img.shields.io/badge/Node%20Exporter-0A7EA4?style=for-the-badge\&logo=prometheus\&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge\&logo=docker\&logoColor=white)
![RocketChat](https://img.shields.io/badge/Rocket.Chat-F5455C?style=for-the-badge\&logo=rocket.chat\&logoColor=white)

---

# 📐 Architecture

```text
┌─────────────────────────────────────────────────────────────────┐
│                        Docker Network: monitoring               │
│                                                                 │
│  ┌──────────────┐     scrape     ┌─────────────────────────┐   │
│  │ node_exporter│◄───────────────│                         │   │
│  │   :9100      │                │      Prometheus         │   │
│  ├──────────────┤                │        :9090            │   │
│  │   Blackbox   │◄───────────────│                         │   │
│  │   :9115      │                │  - metrics collection   │   │
│  ├──────────────┤                │  - rule evaluation      │   │
│  │   Jenkins    │◄───────────────│  - alert firing         │   │
│  │   :8080      │                └────────────┬────────────┘   │
│  └──────────────┘                             │ alert          │
│                                               ▼                │
│  ┌──────────────┐                ┌─────────────────────────┐   │
│  │    Promtail  │──── push ─────►│      Alertmanager       │   │
│  │    :9080     │                │        :9093            │   │
│  └──────┬───────┘                │  - routing              │   │
│         │ push logs              │  - deduplication        │   │
│         ▼                        │  - silencing            │   │
│  ┌──────────────┐                └────────────┬────────────┘   │
│  │     Loki     │                             │ webhook        │
│  │    :3100     │                             ▼                │
│  └──────┬───────┘                ┌─────────────────────────┐   │
│         │                        │      Rocket.Chat        │   │
│         ▼                        │        :4000            │   │
│  ┌──────────────┐                │  🔥 FIRING alerts       │   │
│  │   Grafana    │◄───────────────│  ✅ RESOLVED alerts     │   │
│  │    :3000     │  datasources   └─────────────────────────┘   │
│  └──────────────┘                                               │
└─────────────────────────────────────────────────────────────────┘
```

---

# ✨ Features

* 📊 Infrastructure monitoring with Prometheus
* 📈 Grafana dashboards for metrics and logs
* 📜 Centralized log aggregation using Loki
* 🚚 Log shipping with Promtail
* 🌐 HTTP/HTTPS endpoint monitoring via Blackbox Exporter
* 🚨 Real-time alerting with Alertmanager
* 💬 Rocket.Chat webhook notifications
* 🔍 Systemd service monitoring
* 🔐 SSL certificate expiry monitoring
* 🐳 Fully containerized with Docker Compose
* 📡 Host monitoring with Node Exporter
* ⚡ Real-time FIRING → RESOLVED alert lifecycle

---

# 🧱 Stack Components

| Component         | Role                                       | Port |
| ----------------- | ------------------------------------------ | ---- |
| Prometheus        | Metrics collection & alert rule evaluation | 9090 |
| Grafana           | Metrics & logs visualization               | 3000 |
| Loki              | Log aggregation                            | 3100 |
| Promtail          | Log collection agent                       | 9080 |
| Alertmanager      | Alert routing & notifications              | 9093 |
| Node Exporter     | Host metrics collection                    | 9100 |
| Blackbox Exporter | HTTP/HTTPS probing                         | 9115 |
| Rocket.Chat       | Alert notification receiver                | 4000 |

---

# 🚀 Quick Start

## 📋 Prerequisites

* Docker
* Docker Compose
* Linux Host (Ubuntu 22.04 recommended)
* Rocket.Chat instance

---

## ⚙️ Recommended Host Resources

* 2 CPU cores minimum
* 4 GB RAM minimum
* 10 GB free disk space

---

## 1️⃣ Create Docker Network

```bash
docker network create monitoring
```

---

## 2️⃣ Clone Repository

```bash
git clone https://github.com/YOUR_USERNAME/observability-stack.git
cd observability-stack
```

---

## 3️⃣ Configure Rocket.Chat Webhook

Edit:

```bash
alertmanager/alertmanager.yml
```

Replace webhook URL with your own Rocket.Chat webhook URL.

---

## 4️⃣ Start the Stack

```bash
docker compose -f prometheus/prometheus-compose.yml up -d

docker compose -f node-exporter/node-exporter.yml up -d

docker compose -f alertmanager/alertmanager-container.yml up -d

docker compose -f blackbox/blackbox-compose.yml up -d

docker compose -f loki/loki-compose.yml up -d

docker compose -f promtail/promtail-compose.yml up -d

docker compose -f grafana/grafana-compose.yml up -d
```

---

## 5️⃣ Access the Services

| Service           | URL                   |
| ----------------- | --------------------- |
| Prometheus        | http://localhost:9090 |
| Grafana           | http://localhost:3000 |
| Alertmanager      | http://localhost:9093 |
| Loki              | http://localhost:3100 |
| Promtail          | http://localhost:9080 |
| Node Exporter     | http://localhost:9100 |
| Blackbox Exporter | http://localhost:9115 |
| Rocket.Chat       | http://localhost:4000 |

---

## 6️⃣ Verify Targets

Open:

```text
http://localhost:9090/targets
```

All targets should show:

```text
UP
```

---

# 🔑 Default Credentials

| Service | Username | Password |
| ------- | -------- | -------- |
| Grafana | admin    | admin    |

> Change default credentials immediately in production.

---

# 📊 What Gets Monitored

## System Metrics

Collected using Node Exporter:

* CPU usage
* Memory utilization
* Disk usage and I/O
* Network throughput
* System load average

---

## Systemd Services

Monitored services:

* apache2.service
* nginx.service
* jenkins.service
* ssh.service
* docker.service

---

## HTTP Endpoints

Monitored using Blackbox Exporter:

* Website uptime
* Response latency
* SSL certificate expiry

---

## Logs

Collected using Promtail → Loki:

* `/var/log/syslog`
* `/var/log/auth.log`
* Docker container logs
* Application logs

---

# 🚨 Alert Rules

## System Alerts

| Alert        | Condition               | Severity |
| ------------ | ----------------------- | -------- |
| HighCPU      | CPU > 80% for 2m        | warning  |
| HighMemory   | Memory > 85% for 2m     | warning  |
| DiskSpaceLow | Disk > 80% for 5m       | warning  |
| HighLoad     | Load average > 5 for 2m | warning  |

---

## Service Alerts

| Alert         | Condition                       | Severity |
| ------------- | ------------------------------- | -------- |
| ServiceDown   | systemd service inactive for 1m | critical |
| ServiceFailed | systemd failed state detected   | critical |

---

## HTTP Alerts

| Alert               | Condition            | Severity |
| ------------------- | -------------------- | -------- |
| WebsiteDown         | probe_success == 0   | critical |
| WebsiteSlowResponse | Response time > 3s   | warning  |
| SSLCertExpiringSoon | SSL expiry < 30 days | warning  |
| JenkinsDown         | Jenkins unreachable  | critical |

---

# 🔔 Rocket.Chat Alert Notifications

Example alert message:

```text
🔥 FIRING — ServiceDown
• Severity: critical
• Instance: node_exporter:9100
• Summary: Service DOWN: apache2.service

✅ RESOLVED — ServiceDown
• Severity: critical
• Instance: node_exporter:9100
• Summary: Service DOWN: apache2.service
```

---

# 🔗 Rocket.Chat Webhook Integration

## Step 1 — Create Incoming Webhook

Go to:

```text
Administration → Integrations
```

Create:

```text
New Integration → Incoming WebHook
```

Configuration:

| Setting         | Value                   |
| --------------- | ----------------------- |
| Enabled         | ON                      |
| Name            | prometheus-alerts       |
| Post to Channel | #alerts                 |
| Username        | prometheus-boat         |
| Alias           | Prometheus Alertmanager |

---

## ⚠️ Important

Enable:

```text
Script Enabled → ON
```

Otherwise Rocket.Chat receives alerts without formatted content.

---

## Step 2 — Add Webhook Script

<details>
<summary>Rocket.Chat Webhook Script</summary>

```javascript
class Script {
  process_incoming_request({ request }) {
    try {
      const body = request.content;
      const alerts = body.alerts;

      if (!alerts || alerts.length === 0) {
        return { content: { text: "⚠️ Alertmanager sent empty payload" } };
      }

      let text = "";

      alerts.forEach(alert => {
        const status = (alert.status || "").toLowerCase();

        let stateText = "";
        let emoji = "";

        if (status === "firing") {
          stateText = "DOWN";
          emoji = "🔴";
        } else if (status === "resolved") {
          stateText = "UP";
          emoji = "🟢";
        } else {
          stateText = status.toUpperCase();
          emoji = "⚠️";
        }

        const name = alert.labels.alertname || "Unknown";
        const severity = alert.labels.severity || "unknown";
        const summary = alert.annotations.summary || "No summary";
        const instance = alert.labels.instance || "unknown";

        text += `${emoji} *${stateText}* — ${name}\n`;
        text += `• Severity: ${severity}\n`;
        text += `• Instance: ${instance}\n`;
        text += `• Summary: ${summary}\n`;
        text += `---\n`;
      });

      return { content: { text } };

    } catch (e) {
      return { content: { text: "Script error: " + e.toString() } };
    }
  }
}
```

</details>

---

# 📥 Recommended Grafana Dashboards

| Dashboard           | ID    |
| ------------------- | ----- |
| Node Exporter Full  | 1860  |
| Blackbox Exporter   | 7587  |
| Loki Logs Dashboard | 13639 |

Import dashboards from:

```text
Grafana → Dashboards → Import
```

---

# 📁 Repository Structure

```text
observability-stack/
├── prometheus/
│   ├── prometheus.yml
│   ├── alert.rules.yml
│   └── prometheus-compose.yml
│
├── alertmanager/
│   ├── alertmanager.yml
│   └── alertmanager-container.yml
│
├── grafana/
│   └── grafana-compose.yml
│
├── loki/
│   ├── loki-local-config.yaml
│   └── loki-compose.yml
│
├── promtail/
│   ├── promtail.yml
│   └── promtail-compose.yml
│
├── node-exporter/
│   └── node-exporter.yml
│
├── blackbox/
│   └── blackbox-compose.yml
│
├── rocketchat/
│   └── webhook-script.js
│
└── docs/
    └── screenshots/
```

---

# 📸 Screenshots

## Grafana — Node Exporter Dashboard

Real-time CPU, memory, disk, and network metrics.

```text
docs/screenshots/grafana-node-exporter.png
```

---

## Blackbox Exporter — HTTP Monitoring

Live HTTP probe status and uptime monitoring.

```text
docs/screenshots/blackbox-alerts.png
```

---

## Rocket.Chat — FIRING Alerts

Critical alerts routed from Alertmanager.

```text
docs/screenshots/service-down-alerts.png
```

---

## Rocket.Chat — RESOLVED Alerts

Automatic recovery notifications.

```text
docs/screenshots/service-resolved-alerts.png
```

---

## Loki + Grafana Logs

Centralized real-time log analytics.

```text
docs/screenshots/loki-logs.png
```

---

# 🔧 Real-World Troubleshooting

## Rocket.Chat + Grafana Port Conflict

### Problem

Both services default to:

```text
Port 3000
```

Solution:

```text
Grafana     → 3000
Rocket.Chat → 4000
```

---

## Important Docker Networking Rule

Inside Docker networks:

```text
Containers communicate using INTERNAL container ports
```

Example:

```yaml
❌ WRONG
url: "http://rocketchat:4000/hooks/..."

✅ CORRECT
url: "http://rocketchat:3000/hooks/..."
```

---

## Correct Alertmanager Config

```yaml
receivers:
  - name: rocketchat
    webhook_configs:
      - url: "http://rocketchat:3000/hooks/<WEBHOOK_ID>/<TOKEN>"
        send_resolved: true
```

---

## Rocket.Chat ROOT_URL Fix

```yaml
environment:
  - ROOT_URL=http://localhost:4000
```

Without this:

* broken image previews
* invalid redirects
* wrong generated URLs

---

# 🧠 Key Learnings

## Docker Networking

* Containers communicate using container names
* Port mapping only applies outside Docker
* Internal Docker communication uses container ports

---

## Prometheus + Alertmanager

* Prometheus evaluates rules
* Alertmanager handles routing and deduplication
* Incorrect YAML indentation silently breaks discovery

---

## Node Exporter systemd Collector

Required mounts:

```yaml
/run/systemd:/run/systemd:ro
/var/run/dbus/system_bus_socket:/var/run/dbus/system_bus_socket:ro
```

Also required:

```yaml
privileged: true
pid: host
```

---

## Blackbox Exporter

Important rule:

```text
localhost inside container ≠ host machine
```

Use:

* container names
* actual IP addresses
* real domain names

---

## Loki + Promtail

* Promtail ships logs to Loki
* Loki stores logs as labeled streams
* Grafana queries logs using LogQL

Example query:

```logql
{job="docker"} |= "error"
```

---

# 🚀 Future Improvements

* Kubernetes deployment using Helm
* Grafana provisioning as code
* Persistent storage volumes
* Multi-node monitoring
* TLS/HTTPS support
* Authentication hardening
* CI/CD pipeline integration
* Alert silencing examples
* Infrastructure as Code with Terraform
* GitHub Actions automation

---

# 👤 Author

## Sachin

DevOps Engineer transitioning into SRE

### Skills

* Linux
* Docker
* Prometheus
* Grafana
* Loki
* Alertmanager
* Jenkins
* Ansible
* Terraform
* Kubernetes
* ArgoCD
* GitHub Actions
* Shell Scripting

---

# ⭐ Support

If you found this project useful:

* Star the repository
* Fork the project
* Share feedback
* Open issues or improvements

---

