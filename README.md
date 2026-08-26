# Redpanda Serverless Dashboard Test Environment

This Docker Compose setup provides a local Prometheus and Grafana environment to test the new Redpanda Serverless Dashboard.

## Components

- **Prometheus**: Scrapes metrics from the Redpanda serverless cluster
- **Grafana**: Visualizes metrics using the Redpanda Serverless Dashboard

## Setup

Both the Prometheus config and the metrics credential are gitignored, so create them first:

```bash
cp prometheus.yml.example prometheus.yml
# edit prometheus.yml: set your cluster name, cluster id, region and cloud provider

mkdir -p secrets
printf 'YOUR_METRICS_PASSWORD' > secrets/metrics_password
chmod 644 secrets/metrics_password   # readable by the prometheus container user
```

Generate the metrics username/password in the Redpanda Cloud console for the cluster
you want to scrape.

## Quick Start

1. Start the services:
```bash
docker compose up -d
```

2. Access the interfaces:
   - **Grafana**: http://localhost:3000
     - Username: `admin`
     - Password: `admin`
   - **Prometheus**: http://localhost:9090

3. View the dashboard:
   - Log into Grafana (http://localhost:3000)
   - Navigate to Dashboards
   - Select "Redpanda Serverless Dashboard"

## Configuration Details

### Prometheus Configuration
`prometheus.yml` is gitignored; it holds your own cluster ID and endpoint. Create it
from the template (see Setup above). It scrapes:
- **Endpoint**: `https://<cluster-id>.console.<region>.<cloud>.ign.cloud.redpanda.com/api/cloud/prometheus/public_metrics`
- **Auth**: HTTP basic, username `prometheus`, password read from `secrets/metrics_password`
- **Scrape Interval**: 15 seconds

The password is read from a file rather than an env var because Prometheus does not
expand environment variables in its config file.

### Grafana Dashboard
The dashboard is automatically provisioned from:
https://github.com/redpanda-data/observability/blob/e4c60f3cffe31fac486e7eeda3cfad4a73652a38/grafana-dashboards/Redpanda-Serverless-Dashboard.json

## Stopping the Services

```bash
docker compose down
```

To remove all data and volumes:
```bash
docker compose down -v
```

## Troubleshooting

### Check Prometheus targets
```bash
curl -s http://localhost:9090/api/v1/targets | python3 -m json.tool
```

### View container logs
```bash
docker logs prometheus
docker logs grafana
```

### Restart services
```bash
docker compose restart
```

## File Structure

```
.
├── docker-compose.yml                          # Docker Compose configuration
├── prometheus.yml.example                      # Template for prometheus.yml
├── prometheus.yml                              # Local scrape config (gitignored)
├── secrets/
│   └── metrics_password                        # Metrics credential (gitignored)
├── grafana/
│   ├── dashboards/
│   │   └── Redpanda-Serverless-Dashboard.json  # Dashboard definition
│   └── provisioning/
│       ├── datasources/
│       │   └── datasource.yml                  # Prometheus datasource config
│       └── dashboards/
│           └── dashboard.yml                   # Dashboard provisioning config
└── README.md                                   # This file
```
