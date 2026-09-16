# Monitoring Stack

Centralized monitoring and observability stack for the DevOps homelab.

This repository contains the monitoring infrastructure used to observe the application, containers, hosts, logs, and operational alerts across the homelab environment.

## Architecture

```text
                         ┌──────────────────────┐
                         │        Grafana       │
                         │ Dashboard & Explore  │
                         └──────────▲───────────┘
                                    │
                         ┌──────────┴───────────┐
                         │      Prometheus      │
                         │       Metrics        │
                         └───────▲───────▲──────┘
                                 │       │
                    ┌────────────┘       └─────────────┐
                    │                                  │
              Node Exporter                       cAdvisor
                    │                                  │
        ┌───────────┼───────────┐                      │
        │           │           │                      │
       VM101       VM102       VM103                  VM101
     192.168.50.10  .20         .30             Container metrics
        │           │           │
        │           │           └── Monitoring host
        │           └────────────── Jenkins host
        └────────────────────────── Application host
                                      │
                                      ├── API
                                      ├── Worker
                                      ├── PostgreSQL
                                      └── Nginx

                         ┌──────────────────────┐
                         │         Loki         │
                         │        Logs          │
                         └──────────▲───────────┘
                                    │
                              Log collectors

                         ┌──────────────────────┐
                         │     Alertmanager     │
                         │   Alert routing      │
                         └──────────▲───────────┘
                                    │
                                Prometheus
```

## Components

| Component | Purpose | Location |
|---|---|---|
| Prometheus | Metrics collection and alert evaluation | VM103 |
| Grafana | Metrics and log visualization | VM103 |
| Alertmanager | Alert routing and notification | VM103 |
| Loki | Centralized log storage | VM103 |
| Node Exporter | Host-level metrics | VM101, VM102, VM103 |
| cAdvisor | Docker container metrics | VM101 |
| API metrics | Application-level metrics | VM101 |

## Homelab Targets

| Host | IP | Role |
|---|---|---|
| VM101 | `192.168.50.10` | Application / Docker host |
| VM102 | `192.168.50.20` | Jenkins / CI server |
| VM103 | `192.168.50.30` | Monitoring server |

### VM101

Application services:

- Bookmark Manager API
- Background Worker
- PostgreSQL
- Nginx

Monitoring targets:

- Host metrics through Node Exporter
- Container metrics through cAdvisor
- API metrics through `/metrics`
- Application and infrastructure logs

### VM102

Monitoring target:

- Host CPU
- Memory
- Disk
- Network
- System availability

### VM103

Monitoring services:

- Prometheus
- Grafana
- Loki
- Alertmanager

VM103 also monitors itself through Node Exporter.

## Monitoring Scope

### Host Monitoring

Node Exporter provides host-level metrics including:

- CPU utilization
- Memory utilization
- Disk usage
- Disk I/O
- Network traffic
- System load
- Uptime
- Filesystem usage

### Container Monitoring

cAdvisor provides container-level metrics for the application host:

- CPU usage
- Memory usage
- Network traffic
- Disk I/O
- Container resource usage
- Container availability

### API Monitoring

The application will expose Prometheus-compatible metrics for:

- Request count
- HTTP status codes
- Error rate
- Request latency
- Request duration
- Application availability

Example metrics endpoint:

```text
/metrics
```

### Log Monitoring

Loki will provide centralized log storage and Grafana will be used to search and correlate logs with metrics.

Expected log sources include:

- API
- Worker
- Nginx
- PostgreSQL
- Host services

## Alerting

Prometheus rules will be used to detect operational conditions and Alertmanager will handle alert routing.

Initial alert scope:

- API unavailable
- Monitoring target unavailable
- Container unavailable
- High CPU utilization
- High memory utilization
- High disk utilization
- Increased API 5xx errors
- High API latency

Alert thresholds will be defined based on operational requirements and validated in the homelab.

## Repository Structure

```text
monitoring-stack/
├── alertmanager/
│   └── alertmanager.yml
├── grafana/
├── loki/
│   └── config.yml
├── prometheus/
│   ├── prometheus.yml
│   └── rules/
├── docker-compose.yml
├── .gitignore
└── README.md
```

## Deployment

The monitoring stack runs on VM103 using Docker Compose.

Configuration is maintained in Git and deployed to the monitoring server.

```text
GitHub
   │
   ▼
VM103
   │
   ▼
Docker Compose
   │
   ├── Prometheus
   ├── Grafana
   ├── Loki
   └── Alertmanager
```

Host-level exporters are deployed on their respective monitoring targets.

## Planned Dashboard

The Grafana dashboard will provide an operational overview covering:

```text
Infrastructure
├── VM101
├── VM102
└── VM103

Containers
├── bookmark-api
├── bookmark-worker
├── bookmark-postgres
└── bookmark-nginx

API
├── Availability
├── Requests
├── Error rate
└── Latency

Logs
├── API
├── Worker
├── Nginx
└── PostgreSQL

Alerts
├── Availability
├── Resource utilization
├── API errors
└── API latency
```

## Implementation Roadmap

- [x] Monitoring VM provisioned
- [x] Docker installed on VM103
- [x] Repository initialized
- [x] Prometheus configuration
- [x] Grafana configuration
- [x] Alertmanager configuration
- [x] Loki configuration
- [ ] Deploy monitoring stack to VM103
- [ ] Node Exporter on VM101
- [ ] Node Exporter on VM102
- [ ] Node Exporter on VM103
- [ ] cAdvisor on VM101
- [ ] API `/metrics` endpoint
- [ ] Prometheus scrape configuration
- [ ] Loki log collection
- [ ] Alert rules
- [ ] Alert notification channel
- [ ] Grafana dashboards
- [ ] Alert testing
- [ ] Monitoring evidence
- [ ] Stage 7 documentation

## Design Principles

### Git as Source of Truth

Monitoring configuration is maintained in Git rather than manually edited on the monitoring server.

### Separation of Responsibilities

The monitoring server is separated from the application and CI/CD servers.

```text
VM101 → Application
VM102 → CI/CD
VM103 → Monitoring
```

### Reproducibility

The monitoring stack is deployed using Docker Compose so the environment can be recreated consistently.

### Resource Awareness

VM103 has limited resources, so monitoring retention and component configuration are intentionally kept lightweight for the homelab environment.

## Related Project

Application repository:

`bookmark-manager-devops`

The application repository contains the Bookmark Manager API, Docker configuration, CI/CD pipeline, deployment configuration, networking, TLS, and security hardening.
