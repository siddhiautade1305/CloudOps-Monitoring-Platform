# Prometheus: Core Concepts & Architecture

Welcome to the heart of the monitoring platform. If you are learning Site Reliability Engineering (SRE) or DevOps, Prometheus is one of the most critical tools you can master. This guide breaks down how it works.

## 1. What is Prometheus?
Prometheus is an open-source systems monitoring and alerting toolkit originally built at SoundCloud. It is now a standalone open-source project and a core project of the Cloud Native Computing Foundation (CNCF), much like Kubernetes.

### The Time-Series Database (TSDB) Concept
Unlike traditional SQL databases (like MySQL) that store data in rows and columns, Prometheus is a **Time-Series Database**. It stores data as streams of timestamped values belonging to the same metric and the same set of labeled dimensions.
*   *Example:* Imagine recording the temperature outside every minute. The data is simply: `[Timestamp] -> [Value]`. In our case, it's `[Timestamp] -> [CPU Usage]`.

## 2. The Pull-Based Model
Most traditional monitoring systems (like StatsD or DataDog agents) use a **push-based** model: the applications send (push) their data to the central server.
Prometheus uses a **pull-based** model. 
*   The central Prometheus server actively reaches out over the network to the targets (via HTTP GET requests).
*   *Why?* This makes it highly scalable and fault-tolerant. If a target goes down, Prometheus immediately knows because the HTTP request fails. It also prevents the central server from being overwhelmed by a flood of incoming traffic during a spike.

## 3. Core Terminology

### Metrics
The actual data being recorded. (e.g., `process_cpu_seconds_total`). Metrics represent the state of a system at a specific point in time.

### Exporters
Software that takes existing metrics from a third-party system and formats them so Prometheus can read them.
*   **Node Exporter:** Translates Linux kernel stats into Prometheus metrics.
*   **cAdvisor:** Translates Docker daemon stats into Prometheus metrics.

### Targets
An endpoint that Prometheus scrapes. A target is usually defined by an IP address and a port, plus a path (which defaults to `/metrics`).
*   *Example Target:* `http://node-exporter:9100/metrics`

### Scraping & Scrape Interval
*   **Scraping:** The act of Prometheus making an HTTP request to a target to fetch the latest metrics.
*   **Scrape Interval:** How often Prometheus does this. In our project, it's set to `15s`. Every 15 seconds, Prometheus knocks on the door of cAdvisor and Node Exporter and asks, "What are your numbers now?"

### Target Health (UP / DOWN)
Because Prometheus uses a pull model, it automatically monitors the health of the targets themselves using a synthetic metric called `up`.
*   If Prometheus successfully scrapes a target, `up == 1`.
*   If the connection times out or the target is dead, `up == 0`.
This allows us to easily set an alert like: "Send an email if `up == 0` for more than 5 minutes."
