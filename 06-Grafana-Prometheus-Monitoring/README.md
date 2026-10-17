# Exercise 6: Real-Time Operations Monitoring and Alerting (Prometheus + Grafana)

## Objective
As a DevOps engineer at ZAPPTTO (a fast-paced delivery service), build a monitoring setup that gives real-time insights into delivery performance: simulate delivery metrics with Python, collect them with Prometheus, raise alerts, and visualise them on a Grafana dashboard.

## Files
| File | Purpose |
|------|---------|
| `delivery_metrics.py` | Python app that simulates delivery metrics every second and exposes them at `/metrics` on port 8000 |
| `Dockerfile` | Builds the delivery metrics app into a container image |
| `prometheus.yml` | Prometheus scrape config (scrapes itself and the delivery service) |
| `alert_rules.yml` | Alert rules: HighPendingDeliveries and HighAverageDeliveryTime |
| `grafana-dashboard.json` | Grafana dashboard with 4 panels (import via Dashboards → New → Import) |
| `Jenkinsfile` | Jenkins pipeline that builds and runs the whole monitoring stack |

## Metrics
| Metric | Type | Meaning |
|--------|------|---------|
| `total_deliveries` | Gauge | Total number of deliveries |
| `pending_deliveries` | Gauge | Number of pending deliveries |
| `on_the_way_deliveries` | Gauge | Number of deliveries on the way |
| `average_delivery_time` | Summary | Delivery time in seconds (`_sum` / `_count` gives the average) |

## Steps and Commands

### 1. Run the app locally and check metrics
```bash
sudo apt install -y python3-prometheus-client
python3 delivery_metrics.py
```
Open `http://localhost:8000/metrics`.

### 2. Build the app image and create a network
```bash
docker network create monitoring
docker build -t delivery_metrics .
```

### 3. Run the app, Prometheus and Grafana
```bash
docker run -d --name delivery_metrics --network monitoring -p 8000:8000 delivery_metrics

docker run -d --name prometheus --network monitoring -p 9090:9090 \
  -v "$(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml" \
  -v "$(pwd)/alert_rules.yml:/etc/prometheus/alert_rules.yml" \
  prom/prometheus

docker run -d --name grafana --network monitoring -p 3000:3000 grafana/grafana

docker ps
```

### 4. Verify in Prometheus
- Targets: `http://localhost:9090/targets`
- Alerts: `http://localhost:9090/alerts`

### 5. Set up Grafana
- Open `http://localhost:3000` and log in with `admin` / `admin` (skip the password change)
- Connections → Data sources → Add → Prometheus → URL `http://prometheus:9090` → Save & test
- Dashboards → New → Import → paste `grafana-dashboard.json` → Load → Import

## Changes from the Lab Manual (Windows + Docker Desktop)
- The manual uses `--network=host` and the Linux host IP `172.17.0.1`, which don't work with Docker Desktop on Windows. Instead, all three containers run on a user-defined network (`monitoring`) and reach each other by container name (`delivery_metrics:8000`, `prometheus:9090`).
- `scrape_interval` set to 5s so graphs fill up faster.
- `prometheus-client` installed with `apt` because newer Ubuntu blocks system-wide `pip install`.
- The Jenkinsfile uses files from the Git checkout instead of a hard-coded local path.

## Output Screenshots

### Delivery metrics app running
![App Running](screenshots/1.png)

### Metrics endpoint
![Metrics Endpoint](screenshots/2.png)

### App, Prometheus and Grafana containers running
![Docker PS](screenshots/3.png)

### Prometheus targets UP
![Prometheus Targets](screenshots/4.png)

### Prometheus alert firing
![Prometheus Alerts](screenshots/5.png)

### Grafana connected to Prometheus
![Grafana Data Source](screenshots/6.png)

### Grafana delivery monitoring dashboard
![Grafana Dashboard](screenshots/7.png)

## Learnings
- `start_http_server` from `prometheus_client` automatically creates the `/metrics` endpoint.
- Metric types: **Counter** (only goes up), **Gauge** (goes up and down), **Histogram** (bucketed distribution), **Summary** (count/sum/quantiles).
- Prometheus **scrapes** (pulls) metrics from targets at a fixed interval and evaluates **alert rules** on them.
- Grafana uses Prometheus as a **data source** and visualises queries as dashboard panels.

## Jenkins Pipeline (Step 5 of the manual)

The whole monitoring stack is deployed automatically by a Jenkins pipeline defined in `Jenkinsfile`.

### Setup
Jenkins runs in Docker with access to the host's Docker engine:
~~~bash
docker run -d --name jenkins -u root -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts

docker exec jenkins bash -c "apt-get update && apt-get install -y docker-cli"
docker exec jenkins git config --global --add safe.directory '*'
~~~

### Pipeline job
- New Item → `Delivery-Monitoring-Pipeline` → **Pipeline**
- Pipeline script from SCM → Git → `https://github.com/mridulaachar3/devops-lab.git`, branch `*/main`
- Script Path: `06-Grafana-Prometheus-Monitoring/Jenkinsfile`

### Stages
| Stage | What it does |
|-------|-------------|
| Pre-check Docker | Checks Docker is installed and the daemon is running |
| Setup Workspace | Lists the files checked out from Git |
| Build Docker Image | Builds `delivery_metrics` and `delivery_prometheus` images |
| Create Network | Creates the `monitoring` Docker network |
| Run Application | Starts the delivery metrics app |
| Run Prometheus & Grafana | Starts Prometheus and Grafana |
| Verify | Checks containers are up and Prometheus can read the delivery metrics |

> Note: Prometheus config files are copied into a small custom image (`Dockerfile.prometheus`) instead of being bind-mounted, because paths inside the Jenkins container don't exist on the Docker host.

### Jenkins pipeline configuration
![Pipeline Config](screenshots/8.png)

### All pipeline stages passed
![Pipeline Stages](screenshots/9.png)

### Jenkins logs: successful pipeline build
![Jenkins Console](screenshots/10.png)

## Step 6: Simulate Alerts

`delivery_metrics.py` was modified to increase pending deliveries above the alert threshold:
~~~python
pending = random.randint(50, 100)  # Increase range to exceed alert threshold
~~~
The app and Prometheus were then restarted:
~~~bash
docker rm -f delivery_metrics prometheus grafana
docker build -t delivery_metrics .
docker build -t delivery_prometheus -f Dockerfile.prometheus .
docker run -d --name delivery_metrics --network monitoring -p 8000:8000 delivery_metrics
docker run -d --name prometheus --network monitoring -p 9090:9090 delivery_prometheus
docker run -d --name grafana --network monitoring -p 3000:3000 grafana/grafana
~~~
Result: `pending_deliveries` jumps to 50-100 and both alerts fire. (The file in this repo was set back to `randint(10, 20)` afterwards.)

### Pending range changed to 50-100
![Code Change](screenshots/11.png)

### Prometheus query graph: pending_deliveries
![Prometheus Graph](screenshots/12.png)

### Both alerts firing
![Alerts Firing](screenshots/13.png)
