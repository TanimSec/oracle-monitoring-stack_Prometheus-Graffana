# Oracle Cloud Monitoring Stack (Dokploy / Coolify Edition)

A production-grade, lightweight, and fully integrated observability stack optimized for **Oracle Cloud Always Free Ampere A1 (2 OCPU / 12 GB RAM / ARM64)** and deployed seamlessly via **Dokploy**.

---

## 🚀 What's Included

| Service | Role | Ports / UI | Default Memory Limit |
|---|---|---|---|
| **Prometheus** | Metrics aggregation & time-series database | Internal (9090) | 768 MB |
| **Node Exporter** | Host OS & hardware metrics (CPU, RAM, Disk, Net) | Host (9100) | 128 MB |
| **cAdvisor** | Per-container metrics (CPU, RAM, Network, I/O) | Internal (8080) | 256 MB |
| **Loki** | Centralized Docker container & host log storage | Internal (3100) | 512 MB |
| **Promtail** | Automatically ships Docker & syslog to Loki | Internal (9080) | 128 MB |
| **Alertmanager** | Alert routing (Discord, Telegram, Slack, Webhook) | Internal (9093) | 128 MB |
| **Grafana** | Unified observability dashboard (Prometheus + Loki) | `3002` (Web UI) | 512 MB |
| **Uptime Kuma** | Real-time website, ping, and SSL expiration monitoring | `3001` (Web UI) | 384 MB |

> **Resource Budgeting**: Under peak load, this entire stack consumes less than **2.8 GB RAM** (typically ~1 GB at idle), leaving over **9 GB RAM** and plenty of CPU available on your 12 GB VM for Dokploy, your databases (PostgreSQL, Redis), and client applications.

---

## 📁 Repository Structure

```text
oracle-monitoring-stack/
├── docker-compose.yml
├── .env.example
├── README.md
├── prometheus/
│   ├── prometheus.yml              # Scrapes all services + host
│   └── rules/
│       └── alerts.yml              # Host & container threshold alert rules
├── alertmanager/
│   └── alertmanager.yml            # Alert routing & notification channels
├── loki/
│   └── loki-config.yml             # TSDB schema + 7-day retention
├── promtail/
│   └── promtail-config.yml         # Scrapes /var/log and Docker containers
└── grafana/
    └── provisioning/
        ├── datasources/
        │   └── datasources.yml     # Auto-provisions Prometheus & Loki!
        └── dashboards/
            └── dashboards.yml      # Dashboard provider configuration
```

---

## 🛠️ Step-by-Step Deployment on Dokploy

### Step 1: Push This Repository to GitHub
If you haven't pushed yet:
```bash
git add .
git commit -m "feat: complete production monitoring stack with loki, cadvisor, alertmanager, and uptime kuma"
git push origin main
```

### Step 2: Create Application in Dokploy
1. Open your Dokploy dashboard.
2. Navigate to your **Project** → Click **Create Service**.
3. Choose **Compose**.
4. Set Name: `monitoring-stack`.
5. Select **Provider**: **GitHub**.
6. Select your repository: `oracle-monitoring-stack_Prometheus-Graffana`.
7. Branch: `main`.
8. Compose Path: `docker-compose.yml`.

### Step 3: Configure Environment Variables in Dokploy
In the Dokploy application settings, go to the **Environment** tab and add:
```env
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=SetYourVeryStrongPasswordHere!
GRAFANA_PORT=3002 # Host port (avoid 3000 to prevent conflict with Dokploy dashboard)
UPTIME_KUMA_PORT=3001
PROMETHEUS_RETENTION_TIME=15d
LOKI_RETENTION_TIME=168h
```

### Step 4: Deploy
Click **Deploy** in Dokploy. Wait 1–2 minutes while Dokploy pulls the ARM64 container images and starts the stack.

---

## 🌐 Configuring Domains & SSL in Dokploy

In Dokploy, you can route domain names with automatic free Let's Encrypt SSL certificates:

1. **Grafana Dashboard** (`monitor.yourdomain.com`):
   - Go to the **Domains** tab in your Dokploy Compose service.
   - Click **Add Domain**.
   - Host: `monitor.yourdomain.com`
   - Service: `grafana`
   - Port: `3000`
   - HTTPS: Enabled (Let's Encrypt)

2. **Uptime Kuma Status Page** (`status.yourdomain.com`):
   - Click **Add Domain**.
   - Host: `status.yourdomain.com`
   - Service: `uptime-kuma`
   - Port: `3001`
   - HTTPS: Enabled (Let's Encrypt)

---

## 🔒 Oracle Cloud & UFW Security Hardening

> [!CAUTION]
> **DO NOT** expose internal monitoring ports (`9100`, `9090`, `8080`, `3100`, `9093`) to the public internet in your Oracle Cloud Security Lists.

Only expose ports **80** and **443** (handled by Dokploy / Traefik) and your SSH port.

If your host has UFW enabled, protect Node Exporter:
```bash
sudo ufw deny 9100/tcp
```
Prometheus communicates with Node Exporter internally via Docker's host gateway (`host.docker.internal:9100`).

---

## 📊 Grafana Setup & Recommended Dashboards

When you open Grafana (`https://monitor.yourdomain.com` or `http://<SERVER_IP>:3002`):

1. **Datasources are Pre-Configured**:
   - **Prometheus** (Default metrics source) is already registered and ready.
   - **Loki** (Centralized logs source) is already registered and ready.

2. **Import Ready-Made Dashboards**:
   Navigate to **Dashboards** → **New** → **Import** and enter the following dashboard IDs:

   - **`1860` — Node Exporter Full**:
     Comprehensive host overview: CPU cores, RAM usage, swap, disk I/O, network traffic, system load.
   - **`14282` — Docker Container Monitoring (cAdvisor)**:
     Live per-container CPU usage, memory consumption, network throughput, and restarts.
   - **`13639` — Docker Loki Container Logs**:
     Search, filter, and stream live logs from all Docker containers on your server.

---

## 🔔 Enabling Notifications in Alertmanager

Edit `alertmanager/alertmanager.yml` to enable Discord, Telegram, or Slack alerts:

### Discord
```yaml
receivers:
  - name: 'default-receiver'
    webhook_configs:
      - url: 'https://discord.com/api/webhooks/YOUR_WEBHOOK_URL/slack'
        send_resolved: true
```

### Telegram
```yaml
receivers:
  - name: 'default-receiver'
    telegram_configs:
      - bot_token: 'YOUR_BOT_TOKEN'
        chat_id: 123456789
        send_resolved: true
```

Commit and push to GitHub; Dokploy will auto-deploy or you can click Redeploy!

---

## 🛡️ Oracle Cloud Free Tier Reclamation Note
Running this real observability stack with continuous metric collection, log ingestion, and uptime monitoring provides legitimate baseline utilization without requiring artificial stress loops. Combine this with your real applications to keep your instance active and productive.
