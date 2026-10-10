# CloudOps Infrastructure Monitoring Platform

Welcome to the CloudOps Infrastructure Monitoring Platform. This is a robust, locally runnable project designed to demonstrate production-grade monitoring, alerting, and observability practices using industry-standard open-source tools.

## Overview

This project provisions a complete monitoring stack using Docker Compose. It leverages Prometheus for time-series metrics collection and alerting, and Grafana for powerful visualizations. The platform monitors both the host machine and the running containers, simulating a real-world infrastructure environment.

**Key Technologies:** Docker, Docker Compose, Prometheus, Grafana, Node Exporter, cAdvisor, Bash.

## Project Structure

This repository is structured to separate configuration, automation, and documentation, mirroring a professional DevOps repository:

```text
.
├── alerts/               # Prometheus alerting rules and configuration
│   └── (e.g., rules.yml)
├── architecture/         # Architecture diagrams and system design documentation
├── dashboards/           # Pre-configured Grafana dashboards (JSON format)
├── docs/                 # Detailed documentation (troubleshooting, guides)
├── grafana/              # Grafana provisioning configurations (datasources, dashboards setup)
├── prometheus/           # Prometheus main configuration files (prometheus.yml)
├── screenshots/          # Images for documentation showcasing the working platform
├── scripts/              # Bash scripts for automation, setup, and load generation (testing alerts)
├── docker-compose.yml    # The declarative definition of the entire monitoring stack
├── PROJECT_PLAN.md       # Strategic roadmap, objectives, and interview talking points
└── README.md             # This file (Project entrypoint and instructions)
```

## Purpose of Directories

*   **`alerts/`**: Contains the rules that define when an alert should be triggered (e.g., when CPU usage > 80% for 5 minutes). Keeping these separate from the main Prometheus config improves maintainability.
*   **`architecture/`**: Visualizing the system is crucial. This folder will hold diagrams (e.g., Mermaid files, exported PNGs) explaining how components interact.
*   **`dashboards/`**: Holds the JSON representations of our Grafana dashboards. These are stored as code so they can be version-controlled and automatically loaded on startup.
*   **`docs/`**: A place for in-depth technical documentation. If a troubleshooting guide gets too long for the README, it goes here.
*   **`grafana/`**: Contains the provisioning setup. Instead of clicking through the Grafana UI to add Prometheus as a data source, we define it here as code (Configuration as Code).
*   **`prometheus/`**: Holds `prometheus.yml`, which tells Prometheus *what* to monitor (the scrape targets) and *how often*.
*   **`scripts/`**: Automation is key in DevOps. This folder will contain helper scripts (e.g., `generate_cpu_load.sh`) used to test our alerting system.

## Getting Started

*(Note: The project is currently in the planning phase. Deployment instructions will be added in Phase 2 once the `docker-compose.yml` is fully implemented.)*

### Prerequisites
- Docker Engine installed and running
- Docker Compose v2 installed
- Git

## Next Steps
Please refer to `PROJECT_PLAN.md` to see the roadmap and current progress of the project phases.
