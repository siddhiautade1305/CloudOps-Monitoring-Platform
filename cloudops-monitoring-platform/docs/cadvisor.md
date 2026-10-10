# cAdvisor: Docker Container Metrics

## Why cAdvisor is used

cAdvisor (Container Advisor) discovers running containers and samples their resource usage from the container runtime and Linux cgroups. It translates that information into Prometheus-format time series. It is the container-level counterpart to Node Exporter in this stack.

## Container metrics vs. host metrics

- **Node Exporter** reports the operating system visible to it: aggregate CPU, memory, network, and filesystem information for the host. With Docker Desktop, that host is generally the Linux VM that runs the Docker Engine, not the Windows desktop itself.
- **cAdvisor** reports resource usage associated with individual running containers. It does not replace host monitoring; use both exporters to distinguish a busy host from the container responsible for the load.

On Docker Desktop, cAdvisor can observe Linux containers in the Docker Engine VM. It cannot provide native Windows-process metrics, and the VM’s kernel and filesystem are not identical to those of the Windows host.

## How cAdvisor exposes metrics

cAdvisor serves Prometheus exposition text at `http://cadvisor:8080/metrics` on the Compose `monitoring` network. The service is not published on a host port. The Compose configuration grants no extra Linux capabilities, drops all capabilities, runs with a read-only root filesystem, and uses read-only mounts for the host root, cgroups, Docker data directory, and Docker socket.

The Docker socket provides runtime/container metadata. A read-only bind mount protects the mounted filesystem path but **does not make Docker API operations read-only**. Treat access to this socket as highly privileged; only the trusted cAdvisor service receives it. Avoid mounting the entire `/var/run` directory or enabling `privileged` unless a specific runtime limitation is diagnosed and there is no narrower remedy.

Useful metric families include:

| Purpose | Example cAdvisor metric | Example PromQL |
| --- | --- | --- |
| Container CPU | `container_cpu_usage_seconds_total` | `rate(container_cpu_usage_seconds_total[5m])` |
| Container memory | `container_memory_working_set_bytes` | `container_memory_working_set_bytes` |
| Network receive/transmit | `container_network_receive_bytes_total`, `container_network_transmit_bytes_total` | `rate(container_network_receive_bytes_total[5m])` |
| Container filesystem | `container_fs_usage_bytes`, `container_fs_limit_bytes` | `container_fs_usage_bytes` |
| Container start time / uptime | `container_start_time_seconds` | `time() - container_start_time_seconds` |
| Recently observed container | `container_last_seen` | `time() - container_last_seen < 60` |

Metric labels depend on the runtime and cAdvisor version. Prefer filtering by `name` or `id` only after checking the actual labels exposed by your Docker Engine. cAdvisor primarily reports *running* containers; stopped containers disappear from current cgroup observations. `container_last_seen` can help identify recently observed containers, but it is not an authoritative Docker lifecycle state such as `Exited`.

## How Prometheus scrapes cAdvisor

The `cadvisor` scrape job in `prometheus/prometheus.yml` targets `cadvisor:8080` over the shared Compose network and scrapes `/metrics` every 15 seconds (the global interval). Prometheus publishes `up{job="cadvisor"}`: `1` means the scrape succeeded and `0` means it failed. This measures exporter reachability, not the health of each application container.

To validate from the Prometheus expression browser or HTTP API, run these queries:

```promql
up{job="cadvisor"}
container_cpu_usage_seconds_total
container_memory_working_set_bytes
container_network_receive_bytes_total
container_fs_usage_bytes
container_start_time_seconds
```

For a compact CPU rate and uptime view:

```promql
rate(container_cpu_usage_seconds_total[5m])
time() - container_start_time_seconds
```

## Common troubleshooting issues

1. **Target is DOWN / connection refused**
   - Confirm the cAdvisor service is healthy and attached to `monitoring`: `docker compose ps`.
   - Check `docker compose logs cadvisor` and verify the Prometheus target is `cadvisor:8080` (not `localhost:8080`, which would refer to the Prometheus container itself).
   - Validate/reload the Prometheus configuration and allow at least one scrape interval for samples to arrive.

2. **cAdvisor starts but reports no individual Docker containers**
   - Check that the active Docker context is the expected Docker Desktop Linux Engine and that cAdvisor can read `/var/run/docker.sock` and host cgroup information under `/sys`.
   - Inspect cAdvisor logs for runtime discovery, cgroup, permission, or storage-driver errors. Docker Desktop exposes its Linux VM, not the Windows host, and its VM paths can differ from a native Linux installation.
   - Use a current pinned cAdvisor release. Older releases may fail to map Docker container IDs to storage metadata (for example, missing Docker `layerdb` paths). Do not enable `privileged` as a first response; first correct the runtime path/version and retest.

3. **CPU or memory is present but filesystem series are missing**
   - Filesystem visibility depends on the Docker storage driver and the Docker data directory being readable. Confirm the read-only `/var/lib/docker` mount corresponds to the engine’s actual data root and inspect cAdvisor logs for layer/snapshotter lookup failures.
   - Docker Desktop’s storage implementation may not expose the same metadata layout as native Linux. If cAdvisor cannot resolve filesystem usage in that engine configuration, filesystem metrics may be unavailable even while cgroup CPU/memory work; record this as a runtime limitation rather than reporting fabricated values.

4. **Metrics exist but do not have container names**
   - Inspect `/metrics` labels and cAdvisor’s Docker API discovery logs. Runtime versions and containerd-backed storage can affect names/labels. Use container IDs or cgroup IDs when names are not exposed.

5. **No network activity appears**
   - Verify the container has a network interface and generate traffic. Network counters are per container/interface and may be zero when idle.

6. **Host and container values appear similar**
   - Node Exporter measures host/VM-wide counters; cAdvisor metrics are container-scoped. Do not compare raw cumulative counters directly—use `rate()` over a time range and filter to a container where labels permit it.
