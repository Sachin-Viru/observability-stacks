# 🔭 Observability Stack

A production-pattern monitoring and alerting stack built with Prometheus, Grafana, Loki, and Alertmanager — fully containerized with Docker, alerting to Rocket.Chat in real time.

![Stack](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge\&logo=prometheus\&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge\&logo=grafana\&logoColor=white)
![Loki](https://img.shields.io/badge/Loki-F0A500?style=for-the-badge\&logo=grafana\&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge\&logo=docker\&logoColor=white)
![RocketChat](https://img.shields.io/badge/Rocket.Chat-F5455C?style=for-the-badge\&logo=rocket.chat\&logoColor=white)

---

## 📐 Architecture

```
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

## 🧱 Stack Components

| Component             | Role                                             | Port |
| --------------------- | ------------------------------------------------ | ---- |
| **Prometheus**        | Metrics collection & alert rule evaluation       | 9090 |
| **Node Exporter**     | Host metrics (CPU, RAM, Disk) + systemd services | 9100 |
| **Blackbox Exporter** | HTTP/HTTPS endpoint probing                      | 9115 |
| **Alertmanager**      | Alert routing, deduplication, notifications      | 9093 |
| **Grafana**           | Visualization (metrics + logs)                   | 3000 |
| **Loki**              | Log aggregation                                  | 3100 |
| **Promtail**          | Log collection agent                             | 9080 |
| **Rocket.Chat**       | Alert notification receiver                      | 4000 |

---

## 📊 What Gets Monitored

### System Metrics (via Node Exporter)

* CPU usage per core
* Memory utilization
* Disk space and I/O
* Network throughput
* System load average

### Systemd Services (via Node Exporter systemd collector)

* `apache2.service`
* `nginx.service`
* `jenkins.service`
* `ssh.service`
* `docker.service`

### HTTP Endpoints (via Blackbox Exporter)

* External URLs — up/down status
* Response time
* SSL certificate expiry (alerts at <30 days)

### Logs (via Promtail → Loki)

* System logs (`/var/log/syslog`)
* Auth logs (`/var/log/auth.log`)
* Application logs

---

## 🚨 Alert Rules

### System Alerts

| Alert          | Condition           | Severity |
| -------------- | ------------------- | -------- |
| `HighCPU`      | CPU > 80% for 2m    | warning  |
| `HighMemory`   | Memory > 85% for 2m | warning  |
| `DiskSpaceLow` | Disk > 80% for 5m   | warning  |
| `HighLoad`     | Load avg > 5 for 2m | warning  |

### Service Alerts

| Alert           | Condition                         | Severity |
| --------------- | --------------------------------- | -------- |
| `ServiceDown`   | systemd service not active for 1m | critical |
| `ServiceFailed` | systemd service in failed state   | critical |

### HTTP Alerts

| Alert                 | Condition                  | Severity |
| --------------------- | -------------------------- | -------- |
| `WebsiteDown`         | probe_success == 0 for 1m  | critical |
| `WebsiteSlowResponse` | response > 3s for 2m       | warning  |
| `SSLCertExpiringSoon` | cert expiry < 30 days      | warning  |
| `JenkinsDown`         | Jenkins unreachable for 1m | critical |

---

## 🔔 Alert Notifications (Rocket.Chat)

Alerts fire to a Rocket.Chat channel via incoming webhook with a custom script:

```
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

## 🔗 Creating Webhook for Alertmanager (Rocket.Chat)

### Step 1: Create Incoming Webhook
* First Create User **prometheus-boat** 
* Then Go to **Administration → Integrations**
* Click **New Integration → Incoming WebHook**

Fill:

* Enabled → ON
* Name → `prometheus-alerts`
* Post to Channel → `#alerts`
* Username → `prometheus-boat`
* Alias → `Prometheus Alertmanager`
* Avatar URL →

  ```
  https://prometheus.io/assets/favicons/android-chrome-192x192.png
  ```

---

### ⚠️ Important

Enable **Script Enabled → ON**

Otherwise, Rocket.Chat will only show notification without message content.

---

### Step 2: Add Script

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

        // Map states clearly
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

---

### Step 3: Add Webhook to Alertmanager

```yaml
receivers:
  - name: rocketchat
    webhook_configs:
      - url: "http://rocketchat:3000/hooks/<WEBHOOK_ID>/<TOKEN>"
        send_resolved: true
```

---

## 🚀 Quick Start

### Prerequisites

* Docker + Docker Compose
* Linux host (Ubuntu 22.04 recommended)
* Rocket.Chat instance

### 1. Create Docker network

```bash
docker network create monitoring
```

### 2. Clone and configure

```bash
git clone https://github.com/YOUR_USERNAME/observability-stack.git
cd observability-stack
```

### 3. Start the stack

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

## 👤 Author

**Sachin** — DevOps Engineer transitioning to SRE
Hands-on with: Docker · Prometheus · Grafana · Loki · Ansible · Jenkins · ArgoCD · Kubernetes (learning)

