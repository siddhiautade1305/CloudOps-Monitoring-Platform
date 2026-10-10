# System Architecture: CloudOps Monitoring Platform

This document outlines the architectural design of the local infrastructure monitoring platform. The architecture follows a modern, cloud-native approach using the **pull-based monitoring model** championed by Prometheus.

## High-Level Data Flow

1.  **Generation**: The Host OS and Docker Engine generate raw performance data.
2.  **Exporting**: "Exporters" (Node Exporter and cAdvisor) translate this raw OS/Engine data into a format Prometheus understands (HTTP endpoints exposing metrics).
3.  **Collection**: Prometheus periodically "pulls" (scrapes) these endpoints to collect the data and stores it in its local time-series database.
4.  **Visualization**: The user accesses the Grafana web interface. Grafana queries Prometheus for specific metrics using PromQL and renders them as visual charts.

---

## Component Breakdown

### 1. Prometheus (The Core Engine)
*   **What it does:** It is a Time Series Database (TSDB) and a scraper. It connects to configured target URLs, downloads the metrics, and stores them locally with timestamps. It also evaluates alerting rules.
*   **Why it is needed:** It acts as the central brain of the monitoring stack, aggregating data from multiple sources so it can be queried centrally.
*   **Data it produces/handles:** Time-series data (e.g., `cpu_usage_seconds_total{instance="node-exporter"} 12345.6`).
*   **Communication:** 
    *   *Inbound*: Queried by Grafana (PromQL).
    *   *Outbound*: Makes HTTP GET requests to Exporters.
*   **Port:** `9090` (Internal and exposed to host).

### 2. Grafana (The Visualization Layer)
*   **What it does:** Provides a rich, interactive web UI to create dashboards. It connects to data sources (like Prometheus) and translates raw metrics into graphs, gauges, and tables.
*   **Why it is needed:** Raw time-series data is difficult for humans to read. Grafana makes the data actionable and understandable at a glance.
*   **Data it produces/handles:** Dashboard JSON files, visual charts.
*   **Communication:** Sends PromQL queries via HTTP to the Prometheus API.
*   **Port:** `3000` (Exposed to host for user access).

### 3. Node Exporter (Host Metrics)
*   **What it does:** A lightweight daemon that reads low-level hardware and OS metrics directly from the host system's kernel.
*   **Why it is needed:** Prometheus cannot natively read Linux kernel stats. Node Exporter translates system stats into Prometheus HTTP metrics.
*   **Data it produces:** CPU usage, memory utilization, disk I/O, network bandwidth, filesystem space, etc., of the **underlying host machine**.
*   **Communication:** Reads `/proc` and `/sys` files on the host (mounted into the container). Scraped by Prometheus.
*   **Port:** `9100` (Internal to Docker network).

### 4. cAdvisor (Container Metrics)
*   **What it does:** Container Advisor (cAdvisor) analyzes and exposes resource usage and performance data from running containers.
*   **Why it is needed:** Node Exporter only sees the host as a whole. We need cAdvisor to see the resource breakdown *per container* (e.g., how much RAM the Prometheus container itself is using).
*   **Data it produces:** CPU, memory, network, and disk isolation metrics for every running Docker container.
*   **Communication:** Reads from the Docker Engine via the Docker socket (`/var/run/docker.sock`). Scraped by Prometheus.
*   **Port:** `8080` (Internal to Docker network).

### 5. Demo Containers (Observable Workloads)
*   **What they do:** We will deploy two lightweight containers to act as observable subjects.
    1.  **Demo App 1 (Web):** A simple Nginx web server to demonstrate network traffic and basic container metrics. Port: `80` (Internal).
    2.  **Demo App 2 (Worker):** A simple Alpine container running a bash loop or `stress` tool to artificially generate CPU load. This is crucial for testing our alerts.
*   **Why they are needed:** A monitoring system is useless without something to monitor. These containers provide realistic, fluctuating data and allow us to trigger threshold alerts on demand.

---

## Network & Security Design

*   **Docker Bridge Network:** All components will be placed on a dedicated Docker bridge network (e.g., `monitoring-net`). This allows containers to communicate with each other using DNS names (e.g., Prometheus can scrape `http://node-exporter:9100`).
*   **Least Privilege Exposure:** We will only expose ports `3000` (Grafana) and `9090` (Prometheus) to the host machine. The exporters (`9100`, `8080`) do not need to be accessible from outside the Docker network, improving security.
*   **Read-Only Mounts:** Node Exporter and cAdvisor will mount host directories (like `/proc` and the Docker socket) as *read-only* to prevent them from modifying the host system.
