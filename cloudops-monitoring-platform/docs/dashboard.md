# CloudOps Infrastructure Monitoring Dashboard

## Overview

The **CloudOps Infrastructure Monitoring** dashboard provides a centralized view of host infrastructure, Docker container resources, and Prometheus target availability.

The monitoring stack is:

```text
Node Exporter ──────┐
                    │
                    ├──> Prometheus ───> Grafana
                    │
cAdvisor ───────────┘
```

Node Exporter provides host-level metrics.

cAdvisor provides Docker container-level metrics.

Prometheus collects and stores the metrics.

Grafana queries Prometheus using PromQL and visualizes the results.

---

# Dashboard Settings

Dashboard name:

```text
CloudOps Infrastructure Monitoring
```

Default time range:

```text
Last 1 hour
```

Refresh interval:

```text
30 seconds
```

The 30-second refresh provides reasonably current infrastructure information without creating unnecessary query traffic.

---

# Host Metrics

## 1. CPU Utilization

### What it measures

The percentage of CPU capacity being used by the host.

### PromQL

```promql
100 - (
  avg by(instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)
```

### Why it matters

High CPU utilization can indicate:

- CPU-intensive workloads
- overloaded applications
- inefficient processes
- insufficient CPU resources

### How to interpret

A value near 0% indicates low CPU utilization.

A sustained high value indicates that the host is under significant CPU load.

The query calculates CPU usage by measuring the rate at which CPU time is not spent in the idle state.

---

# 2. Memory Utilization

### What it measures

The percentage of host memory currently being used.

### PromQL

```promql
100 * (
  1 -
  (
    node_memory_MemAvailable_bytes
    /
    node_memory_MemTotal_bytes
  )
)
```

### Why it matters

Memory pressure can cause:

- application performance degradation
- swapping
- out-of-memory conditions
- container instability

### How to interpret

Higher percentages indicate greater memory consumption.

Sustained high memory utilization should be investigated together with application and container memory usage.

---

# 3. Disk Utilization

### What it measures

The percentage of filesystem storage currently used.

### PromQL

```promql
100 * (
  1 -
  (
    node_filesystem_avail_bytes{
      fstype!="tmpfs",
      mountpoint!~"/boot.*"
    }
    /
    node_filesystem_size_bytes{
      fstype!="tmpfs",
      mountpoint!~"/boot.*"
    }
  )
)
```

### Why it matters

Full disks can cause:

- application failures
- database failures
- logging failures
- container problems
- system instability

### How to interpret

A value close to 100% means the filesystem is nearly full.

The mountpoint is included in the legend so different filesystems can be distinguished.

---

# 4. Network Receive Rate

### What it measures

The rate at which the host receives network traffic.

### PromQL

```promql
sum by(instance) (
  rate(
    node_network_receive_bytes_total{
      device!="lo"
    }[5m]
  )
)
```

### Why it matters

Network receive traffic helps identify:

- incoming application traffic
- large downloads
- unusual traffic spikes
- network-intensive workloads

### How to interpret

The unit is bytes per second.

Spikes represent periods of increased incoming traffic.

---

# 5. Network Transmit Rate

### What it measures

The rate at which the host transmits network traffic.

### PromQL

```promql
sum by(instance) (
  rate(
    node_network_transmit_bytes_total{
      device!="lo"
    }[5m]
  )
)
```

### Why it matters

Transmit traffic helps identify:

- outgoing application traffic
- data transfers
- backups
- network-intensive services

### How to interpret

The unit is bytes per second.

Sustained increases can indicate increased outbound traffic.

---

# 6. System Load

### What it measures

The one-minute system load average.

### PromQL

```promql
node_load1
```

### Why it matters

System load indicates how much work is waiting for CPU or other system resources.

### How to interpret

Load should be interpreted relative to the number of CPU cores.

For example, a load of 4 on a 16-core system has a different meaning from a load of 4 on a 2-core system.

Load should therefore be considered together with CPU utilization.

---

# Container Metrics

Container metrics are provided by cAdvisor.

## 7. Container CPU Usage

### What it measures

CPU consumption of Docker containers.

### PromQL

```promql
sum by(name) (
  rate(
    container_cpu_usage_seconds_total{
      name!=""
    }[5m]
  )
) * 100
```

### Why it matters

This identifies containers consuming significant CPU resources.

### How to interpret

Higher values indicate greater CPU consumption.

The container name is shown in the legend.

A container that continuously consumes excessive CPU should be investigated.

---

# 8. Container Memory Usage

### What it measures

Working memory used by each Docker container.

### PromQL

```promql
sum by(name) (
  container_memory_working_set_bytes{
    name!=""
  }
)
```

### Why it matters

Container memory monitoring helps identify:

- memory-heavy containers
- memory leaks
- resource imbalance
- containers approaching memory limits

### How to interpret

The unit is bytes and Grafana automatically formats the value into KB, MB, GB, etc.

A continuously increasing memory value can be an indication that the application should be investigated for abnormal memory growth.

---

# 9. Container Network Receive

### What it measures

Network traffic received by Docker containers.

### PromQL

```promql
sum by(name) (
  rate(
    container_network_receive_bytes_total{
      name!=""
    }[5m]
  )
)
```

### Why it matters

This helps identify containers receiving significant network traffic.

It can be useful when investigating:

- API traffic
- database traffic
- service-to-service communication
- unexpected traffic spikes

### How to interpret

The unit is bytes per second.

The container name identifies the source of the traffic.

---

# 10. Container Network Transmit

### What it measures

Network traffic transmitted by Docker containers.

### PromQL

```promql
sum by(name) (
  rate(
    container_network_transmit_bytes_total{
      name!=""
    }[5m]
  )
)
```

### Why it matters

This helps identify containers generating significant outbound traffic.

### How to interpret

The unit is bytes per second.

Large or unexpected spikes should be investigated in relation to application behavior.

---

# 11. Container Status / Last Seen

### What it measures

The most recent timestamp at which cAdvisor observed the container.

### PromQL

```promql
container_last_seen{name!=""}
```

### Why it matters

It provides a basic container-observation signal.

If a container disappears from cAdvisor metrics, this can indicate that the container is no longer running or is no longer being monitored.

### How to interpret

Recent timestamps indicate that cAdvisor is currently observing the container.

An absent container metric can indicate that the container has stopped or disappeared from cAdvisor's metric set.

### Important limitation

cAdvisor does not provide a universal Docker restart counter.

For a production-grade restart counter, Docker Engine metrics or an additional exporter can be introduced.

Therefore this panel intentionally uses `container_last_seen` as a supported container-status/observation signal rather than pretending it is a restart counter.

---

# Prometheus Health

## 12. Target Availability / UP Status

### What it measures

The number of Prometheus scrape targets currently reporting `UP`.

### PromQL

```promql
sum(up)
```

### Why it matters

This provides a simple high-level view of monitoring health.

If a target is down, Prometheus may no longer be receiving metrics from that target.

### How to interpret

A value of:

```text
3
```

means three targets are currently reporting UP.

For this development stack, targets may include:

```text
prometheus
node-exporter
cadvisor
```

---

# Target UP Status

### PromQL

```promql
up
```

### Interpretation

Prometheus uses:

```text
1 = target is UP
0 = target is DOWN
```

The dashboard displays the job and instance in the legend.

For example:

```text
prometheus - prometheus:9090
node-exporter - node-exporter:9100
cadvisor - cadvisor:8080
```

A target reporting `0` should be investigated.

---

# Interview Explanation

The dashboard follows a standard observability architecture:

```text
Exporter
   ↓
Prometheus
   ↓
PromQL
   ↓
Grafana Panel
   ↓
Dashboard
```

Node Exporter provides host metrics.

cAdvisor provides container metrics.

Prometheus performs metric collection and storage.

Grafana provides querying and visualization.

PromQL is used to calculate rates, percentages, and aggregated metrics.

For example:

```promql
rate(metric[5m])
```

calculates the per-second rate of increase over the previous five minutes.

This is useful for monotonically increasing counters such as CPU time and network bytes.

---

# Operational Use

The dashboard can be used to investigate:

- CPU saturation
- memory pressure
- disk capacity
- network utilization
- high-load hosts
- CPU-intensive containers
- memory-intensive containers
- container network activity
- missing containers
- unavailable Prometheus targets

The dashboard is intended to provide an infrastructure-level overview rather than replace application-specific dashboards.