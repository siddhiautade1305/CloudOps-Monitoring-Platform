# Grafana Configuration

## Overview

Grafana is an open-source observability and visualization platform used to monitor and analyze metrics, logs, and traces.

In the CloudOps Monitoring Platform, Grafana provides the visualization layer for metrics collected and stored by Prometheus.

The architecture is:

```text
Node Exporter
     |
     | metrics
     v
Prometheus
     |
     | PromQL queries
     v
Grafana
     |
     v
Dashboards and Panels
```

## Grafana

Grafana provides a web-based interface for querying and visualizing observability data.

In this project, Grafana runs as a Docker container using Docker Compose.

Grafana is exposed locally on:

```text
http://localhost:3000
```

## Data Source

A data source is the backend from which Grafana retrieves data.

For this project, Prometheus is configured as the Grafana data source.

The data source is automatically provisioned using:

```text
grafana/provisioning/datasources/prometheus.yml
```

The Prometheus endpoint used by Grafana is:

```text
http://prometheus:9090
```

The hostname `prometheus` refers to the Prometheus Docker Compose service.

## Dashboard

A dashboard is a collection of visualizations used to monitor a system.

A CloudOps dashboard can contain information such as:

- CPU utilization
- Memory utilization
- Disk utilization
- Network traffic
- System uptime
- Container metrics
- Prometheus health

Dashboards provide a centralized view of infrastructure health.

## Panel

A panel is an individual visualization inside a Grafana dashboard.

Examples include:

- CPU usage graph
- Memory usage gauge
- Network traffic graph
- Disk utilization
- Request rate
- System uptime

A dashboard can contain multiple panels.

Example:

```text
+------------------------------------------------+
|              CloudOps Dashboard                |
+------------------------+-----------------------+
| CPU Usage              | Memory Usage           |
|                        |                       |
|       Graph            |       Gauge            |
+------------------------+-----------------------+
| Network Traffic        | Disk Usage             |
|                        |                       |
|       Graph            |       Gauge            |
+------------------------+-----------------------+
```

## Query

A query is a request for data from a data source.

Grafana uses PromQL when querying Prometheus.

Example:

```promql
up
```

This checks whether monitored targets are currently up.

Example:

```promql
node_cpu_seconds_total
```

This retrieves CPU-related metrics exported by Node Exporter.

Example:

```promql
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

This can be used to calculate CPU utilization from Node Exporter metrics.

## Visualization

A visualization determines how query results are displayed.

Grafana supports multiple visualization types, including:

- Time series
- Gauge
- Stat
- Bar chart
- Table
- Heatmap

For example:

```text
Prometheus
    |
    | PromQL
    v
Query result
    |
    v
Grafana Panel
    |
    v
Time Series / Gauge / Stat / Table
```

## Docker Configuration

Grafana runs using Docker Compose.

The Grafana service uses:

```yaml
grafana:
  image: grafana/grafana:latest
  container_name: grafana
  restart: unless-stopped
  ports:
    - "3000:3000"
```

Grafana data is persisted using:

```yaml
volumes:
  - grafana_data:/var/lib/grafana
```

The Prometheus data source is provisioned using:

```yaml
volumes:
  - ./grafana/provisioning/datasources/prometheus.yml:/etc/grafana/provisioning/datasources/prometheus.yml:ro
```

## Data Source Provisioning

Grafana automatically loads the Prometheus data source during startup.

Configuration:

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false
```

This avoids manually recreating the Prometheus data source after every deployment or container recreation.

## Local Access

Grafana:

```text
http://localhost:3000
```

Prometheus:

```text
http://localhost:9090
```

Node Exporter:

```text
http://localhost:9100/metrics
```

## Verification

Check running containers:

```bash
docker compose ps
```

Expected services:

```text
prometheus
node-exporter
grafana
```

Check Grafana logs:

```bash
docker compose logs grafana
```

Check Prometheus logs:

```bash
docker compose logs prometheus
```

The Grafana container should start successfully without data-source provisioning errors.

## Summary

The CloudOps Monitoring Platform uses:

```text
Prometheus
    |
    | Metrics storage and PromQL
    v
Grafana
    |
    | Dashboards
    v
Observability
```

Prometheus collects and stores metrics, while Grafana queries those metrics and presents them through dashboards and visualizations.