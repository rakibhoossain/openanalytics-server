# OpenAnalytics Server Deployment

This repository contains the standalone, production-ready Docker Compose deployment configuration for **OpenAnalytics**.

All application images are pre-compiled and hosted on Docker Hub under `rakibhoossain/openanalytics-*`, so deploying on a lightweight VPS (e.g. 2 vCPU, 4GB RAM) requires **zero local compiling, zero Go/Python builds, and zero OOM risks**.

---

## 📁 Repository Structure

```text
openanalytics-server/
├── clickhouse/
│   ├── init/
│   │   └── 01_create_tables.sql     # Pre-loaded ClickHouse database schema & tables
│   └── users.d/
│       └── z-users.xml              # User configuration & access control
├── data/
│   ├── models/                      # Pre-trained ML model JSON weights
│   └── geo/                         # Auto-populated GeoIP MaxMind database
├── nginx/
│   └── openanalytics.conf           # Production Nginx reverse-proxy configuration
├── .env.example                     # Environment variables template
├── docker-compose.yml               # Production Compose file (prebuilt images)
└── .gitignore                       # Ignores persistent data folders & secrets
```

---

## 🚀 Setup & Deployment Guide

### Step 1: Push this directory to your GitHub

On your local machine:
```bash
cd /Users/rakib/Projects/analytics/openanalytics-server
git init
git add .
git commit -m "feat: initial openanalytics production deployment bundle"
git branch -M main

# Add your GitHub repository remote
git remote add origin git@github.com:<your-user>/openanalytics-server.git
git push -u origin main
```

---

### Step 2: Clone and Start on Server (`91.99.83.171`)

SSH into your remote server:
```bash
ssh root@91.99.83.171
```

Clone the repository:
```bash
git clone git@github.com:<your-user>/openanalytics-server.git /opt/openanalytics-server
cd /opt/openanalytics-server
```

Create and verify your `.env` file:
```bash
cp .env.example .env
# Edit .env if you need to adjust database passwords or ports
nano .env
```

Pull the pre-built Docker images from Docker Hub:
```bash
docker compose pull
```

Start the stack in detached mode:
```bash
docker compose up -d
```

Verify that all services are healthy and running:
```bash
docker compose ps
```

To inspect service logs:
```bash
# View all logs
docker compose logs -f

# View a specific service log
docker compose logs -f worker
docker compose logs -f ingest
docker compose logs -f query
docker compose logs -f ml-worker
docker compose logs -f ml-retrain
```

---

### Step 3: Configure Nginx Reverse Proxy (Optional / Recommended)

Copy the included Nginx configuration into your system Nginx:
```bash
sudo cp nginx/openanalytics.conf /etc/nginx/sites-available/openanalytics.conf
sudo ln -s /etc/nginx/sites-available/openanalytics.conf /etc/nginx/sites-enabled/
```

Edit your domain name in `/etc/nginx/sites-available/openanalytics.conf`:
```nginx
server_name analytics.yourdomain.com;
```

Test and reload Nginx:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

Enable SSL using Let's Encrypt Certbot:
```bash
sudo certbot --nginx -d analytics.yourdomain.com
```

---

## 🔌 Architecture & Port Reference

| Service | Port | Description |
| :--- | :--- | :--- |
| **Kafka** | `91.99.83.171:9092` | Existing host Kafka broker |
| **ClickHouse HTTP** | `8123` | OLAP query interface |
| **ClickHouse TCP** | `9000` | High-speed native TCP client interface |
| **PostgreSQL 18** | `5432` | Relational storage & CAPI sync |
| **Redis 7** | `6380` | Real-time cache, counters & keyspace notifications |
| **Ingest Engine** | `8080` | High-throughput tracking API (`/api/v1/track`) |
| **Query Engine** | `8081` | Analytics queries & WebSocket feed (`/ws`) |
| **Worker** | Internal | Kafka-to-ClickHouse stream batcher |
| **ML Worker** | Internal | Real-time purchase intent scorer |
| **ML Retrain** | Internal | Automated scheduled model retraining daemon |

---

## 🔄 Updating to New Versions

When you build and push new versions from your local Mac via `build_and_push.sh`:
```bash
cd /opt/openanalytics-server
docker compose pull
docker compose up -d
```
Docker will restart only the updated containers without data loss.
