# 🚀 Coolify + Laravel + Loki + Grafana Monitoring Stack

A production-ready central logging and monitoring stack tailored for **Laravel applications** deployed via **Coolify** (or stand-alone Docker) on a Virtual Private Server (VPS).

This stack automatically collects Docker container logs, parses Laravel's formatted multiline log entries, routes them through Grafana Loki, and visualizes them in Grafana with a ready-to-use dashboard.

---

## 📌 Features

- **Automated Docker Container Log Scraping**: Promtail automatically discovers all running Docker containers via the Docker socket.
- **Coolify Integration**: Retains Coolify metadata (Application ID, Application Name, Project ID) as queryable tags in Loki.
- **Laravel Multiline & Log Level Parsing**: Formats multiline stack traces cleanly and extracts log levels (`info`, `error`, `debug`, `warning`) and environments (`production`, `staging`).
- **Pre-configured Grafana Datasource**: Automatically provisions Grafana with Loki set as the default data source.
- **Custom Laravel Dashboard Included**: Includes a pre-built `laravel-dashboard.json` for immediate monitoring.
- **Lightweight & Efficient**: Optimized retention (31 days default) and memory footprint.

---

## 🏗️ Architecture Overview

```
 ┌─────────────────────────────────────────────────────────┐
 │                       VPS Server                        │
 │                                                         │
 │  ┌────────────────┐                                     │
 │  │ Laravel App    │──────┐                              │
 │  │ (Docker Container)    │ (Logs to Docker socket)       │
 │  └────────────────┘      ▼                              │
 │                  ┌───────────────┐                      │
 │                  │   Promtail    │                      │
 │                  └───────┬───────┘                      │
 │                          │ (Parses & pushes)            │
 │                          ▼                              │
 │                  ┌───────────────┐                      │
 │                  │ Grafana Loki  │                      │
 │                  └───────┬───────┘                      │
 │                          │ (Queries)                    │
 │                          ▼                              │
 │                  ┌───────────────┐                      │
 │                  │    Grafana    │ ◄── User Browser     │
 │                  └───────────────┘     (Port 3000)      │
 └─────────────────────────────────────────────────────────┘
```

1. **Promtail**: Scrapes container logs from `/var/lib/docker/containers` and relabels them using container metadata.
2. **Loki**: Stores and indexes log streams efficiently without indexing full text.
3. **Grafana**: Web interface to query logs, view dashboards, and set up alerts.

---

## 📋 Prerequisites

Before deploying to your VPS, ensure you have:
- A VPS running Linux (Ubuntu 20.04/22.04/24.04 recommended).
- **Docker** and **Docker Compose** installed (v2.0+).
- *Optional:* **Coolify** installed on your VPS if you manage applications through Coolify.
- SSH access to your VPS with `sudo` privileges.

---

## 🚀 How to Run on VPS

### Option 1: Standalone Docker Compose (Direct VPS Deployment)

Use this method if you manage your VPS directly via SSH.

#### Step 1: SSH into your VPS and Clone the Repository
```bash
ssh user@your-vps-ip
git clone https://github.com/your-username/coolify-laravel-loki-grafana.git
cd coolify-laravel-loki-grafana
```

#### Step 2: Configure Environment Variables
Copy the example environment file and update your credentials and domain configuration:
```bash
cp env.example .env
nano .env
```
Set secure values for Grafana access:
```env
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=YourStrongPasswordHere
GRAFANA_ROOT_URL=https://grafana.yourdomain.com
GRAFANA_DOMAIN=grafana.yourdomain.com
```

#### Step 3: Launch the Stack
Start all services in detached mode:
```bash
docker compose up -d
```

#### Step 4: Verify Deployment
Check that all containers (`loki`, `promtail`, `grafana`) are running and healthy:
```bash
docker compose ps
```

Access Grafana in your web browser at `http://your-vps-ip:3000` (or your configured domain).

---

### Option 2: Deploying via Coolify Dashboard

If your VPS uses Coolify as a PaaS, you can deploy this stack directly as a Service / Docker Compose resource.

1. **Log in to your Coolify Dashboard**.
2. Navigate to your **Project** and select **+ New Resource**.
3. Choose **Docker Compose**.
4. Paste the repository URL or paste the contents of `docker-compose.yml` directly.
5. In the **Environment Variables** tab, add the following variables:
   - `GRAFANA_ADMIN_USER`
   - `GRAFANA_ADMIN_PASSWORD`
   - `GRAFANA_ROOT_URL`
   - `GRAFANA_DOMAIN`
6. Click **Deploy**.

---

## 📊 Importing the Laravel Grafana Dashboard

Once Grafana is accessible:

1. Open Grafana (`http://your-vps-ip:3000`).
2. Log in with your `GRAFANA_ADMIN_USER` and `GRAFANA_ADMIN_PASSWORD`.
3. Go to **Dashboards** > **New** > **Import**.
4. Click **Upload dashboard JSON file** and select `laravel-dashboard.json` from this repository.
5. Select **Loki** as the datasource and click **Import**.

---

## 🔍 LogQL Query Cheat Sheet

You can search and filter your Laravel logs in Grafana's **Explore** tab using LogQL:

- **View all Laravel logs:**
  ```logql
  {job="docker"} |= "production."
  ```
- **Filter logs by severity level (Errors only):**
  ```logql
  {job="docker", level="error"}
  ```
- **Filter by Coolify application name:**
  ```logql
  {coolify_app_name="my-laravel-app"}
  ```
- **Search for specific text inside exception messages:**
  ```logql
  {job="docker"} |= "SQLSTATE"
  ```

---

## ⚙️ Configuration Details

| File | Description |
| :--- | :--- |
| `docker-compose.yml` | Service definitions with inline Promtail pipeline and Grafana provisioning logic. |
| `env.example` | Template for Grafana security credentials and domain bindings. |
| `loki-config.yaml` | Standalone Loki configuration file (retention, storage, filesystem paths). |
| `promtail-config.yaml` | Standalone Promtail configuration file with Docker socket discovery and multiline regex parsing. |
| `grafana-datasources.yaml` | Standalone Grafana Loki datasource configuration. |
| `laravel-dashboard.json` | Pre-built Grafana dashboard JSON model for visual monitoring. |

---

## 🛡️ Security Best Practices for VPS

- **Firewall Setup**: If exposing Grafana publicly, ensure ports `3100` (Loki) and `9080` (Promtail) are closed to the outside world using `ufw`:
  ```bash
  sudo ufw allow 3000/tcp # Grafana UI (or restrict to reverse proxy)
  sudo ufw deny 3100/tcp # Loki API
  sudo ufw deny 9080/tcp # Promtail API
  ```
- **Reverse Proxy / SSL**: Set up Nginx, Traefik, or Caddy in front of Grafana (port 3000) with Let's Encrypt SSL certificates.
- **Change Default Passwords**: Always update `GRAFANA_ADMIN_PASSWORD` in your `.env` file before running in production.

---

## 📄 License

MIT License. Free to use and modify for personal and commercial projects.
