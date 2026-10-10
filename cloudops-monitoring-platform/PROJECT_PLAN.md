# Project Plan: CloudOps Infrastructure Monitoring Platform

## Project Objective
The objective of this project is to build a robust, production-style infrastructure monitoring and alerting platform completely locally. This project serves to demonstrate practical, hands-on experience with modern DevOps and SRE (Site Reliability Engineering) tools and practices. It will showcase the ability to design, deploy, configure, and troubleshoot a containerized observability stack without relying on managed cloud services or proprietary platforms.

## Architecture
The platform follows a standard pull-based monitoring architecture:
1. **Targets (Exporters)**: Lightweight agents (`Node Exporter` for host metrics, `cAdvisor` for container metrics) expose hardware and OS-level metrics via HTTP endpoints.
2. **Scraper & Time-Series Database**: `Prometheus` periodically scrapes these endpoints, stores the metrics in a local time-series database, and evaluates alerting rules.
3. **Alert Management**: `Alertmanager` (integrated with or alongside Prometheus) handles alerts sent by client applications such as the Prometheus server. It takes care of deduplicating, grouping, and routing them to the correct receiver (e.g., email, webhook).
4. **Visualization**: `Grafana` connects to Prometheus as a data source and provides rich, interactive dashboards to visualize the infrastructure's health and performance.
5. **Infrastructure as Code**: The entire stack is orchestrated using `Docker Compose`, ensuring reproducibility, isolation, and easy local deployment.

## Technology Stack
*   **Containerization**: Docker, Docker Compose
*   **Operating System**: Linux (via Docker containers)
*   **Monitoring & TSDB**: Prometheus (Pinned version)
*   **Visualization**: Grafana (Pinned version)
*   **Metrics Exporters**: 
    *   Node Exporter (Host metrics)
    *   cAdvisor (Container metrics)
*   **Scripting**: Bash (for automation, health checks, and simulated load)
*   **Version Control**: Git / GitHub

## Expected Features
*   **Local Deployment**: One-click deployment using `docker-compose up -d`.
*   **Comprehensive Dashboards**: Grafana dashboards visualizing CPU, Memory, Disk, and Network I/O for both the host and running containers.
*   **Alerting System**: Pre-configured Prometheus alerting rules (e.g., High CPU usage, Container down, Disk space running low).
*   **Simulated Workloads**: Bash scripts to artificially generate CPU/Memory spikes to test the alerting system.
*   **Persistent Storage**: Docker volumes configured to persist Prometheus time-series data and Grafana dashboard configurations across restarts.
*   **Security Best Practices**: Principle of least privilege (running containers as non-root where possible), read-only file systems for exporters, and no hardcoded credentials.

## Phases
1.  **Phase 1: Project Initialization & Planning (Current Phase)**
    *   Set up workspace and directory structure.
    *   Draft `PROJECT_PLAN.md` and initial `README.md`.
    *   Define the architecture and technology stack.
2.  **Phase 2: Core Infrastructure Setup**
    *   Write `docker-compose.yml` defining Prometheus, Grafana, Node Exporter, and cAdvisor services.
    *   Configure network and basic volume mapping.
3.  **Phase 3: Configuration & Integration**
    *   Create `prometheus.yml` configuration to scrape targets.
    *   Configure Grafana data sources via provisioning.
4.  **Phase 4: Dashboards & Visualizations**
    *   Import/Create Grafana dashboards.
    *   Set up dashboard provisioning so dashboards load automatically on startup.
5.  **Phase 5: Alerting & Testing**
    *   Define Prometheus alerting rules (e.g., `alerts/rules.yml`).
    *   Develop bash scripts (`scripts/load_generator.sh`) to trigger alerts.
    *   Verify alerts are firing correctly in the Prometheus UI.
6.  **Phase 6: Documentation & Polish**
    *   Finalize `README.md` with detailed run instructions and troubleshooting guides.
    *   Add architecture diagrams and screenshots.
    *   Review against security and reproducible best practices.

## Testing Strategy
*   **Configuration Validation**: Use built-in tools (e.g., `promtool`) to validate Prometheus configuration files and alerting rules before deployment.
*   **Integration Testing**: Verify all containers spin up cleanly without crash loops. Ensure Prometheus successfully discovers and scrapes all defined targets.
*   **Chaos/Load Testing**: Use custom bash scripts to simulate high resource usage (CPU spikes, memory hogs) and verify that the monitoring stack captures the anomalies and triggers the expected alerts.
*   **Resiliency Testing**: Stop a monitored container and verify that an alert is generated. Restart the stack and verify that historical data and dashboard configurations are preserved.

## Placement/Interview Topics (What this project demonstrates)
*   **Docker & Docker Compose**: Understanding of container lifecycle, networking, volumes, and multi-container orchestration.
*   **Observability & Monitoring**: The difference between metrics, logs, and traces. Pull vs. Push monitoring models.
*   **PromQL**: Basic ability to query time-series data (e.g., calculating rates, percentiles).
*   **Infrastructure as Code (IaC) principles**: Defining the stack declaratively rather than setting it up manually.
*   **Linux & Bash**: Navigating file systems, managing processes, writing scripts for automation and testing.
*   **Troubleshooting**: How to debug a container that won't start, or investigate why a dashboard is showing "No Data".
*   **System Design**: Explaining the architecture of a monitoring system and how it scales.
