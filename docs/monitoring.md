# Monitoring & Observability

## Overview

The monitoring stack provides metrics, logs, dashboards, and alerting
for the Bookmark Manager DevOps homelab.

## Architecture

```text
                    ┌─────────────────┐
                    │     Grafana     │
                    │ Dashboard/Logs  │
                    └────────┬────────┘
                             │
                 ┌───────────┴───────────┐
                 │                       │
           ┌─────▼─────┐           ┌─────▼─────┐
           │ Prometheus│           │    Loki   │
           │  Metrics  │           │   Logs    │
           └─────┬─────┘           └─────▲─────┘
                 │                       │
        ┌────────┼────────┐              │
        │        │        │              │
   Node Exporter cAdvisor API /metrics   Alloy
        │        │        │              │
      VM101    VM101    VM101           VM101
```

## Components

- Prometheus — metrics collection and alert evaluation
- Grafana — dashboards and log visualization
- Loki — centralized log storage
- Grafana Alloy — Docker log collection
- Node Exporter — host-level metrics
- cAdvisor — container-level metrics
- Alertmanager/Grafana Alerting — alert handling

## Monitored Infrastructure

| Host | IP | Role |
|---|---|---|
| lab-app-01 | 192.168.50.10 | Application |
| lab-devops-01 | 192.168.50.20 | Jenkins / CI |
| monitoring | 192.168.50.30 | Monitoring |

## Application Metrics

The Bookmark Manager API exposes Prometheus metrics through:

```text
/metrics
```

Metrics include:

- HTTP request count
- HTTP request duration
- Node.js process metrics
- Event loop metrics
- Heap and memory metrics
- Garbage collection metrics

HTTP request metrics use:

- `method`
- `route`
- `status_code`

The application uses route templates rather than raw URLs to avoid
high-cardinality metric labels.

## Grafana Dashboard

The dashboard is organized into:

### SERVICE OVERVIEW

- API
- PostgreSQL
- Worker
- Nginx
- Infrastructure

### HOST INFRASTRUCTURE

- CPU usage
- Memory usage
- Disk usage
- Network traffic

### CONTAINER HEALTH

- Container CPU
- Container memory
- Container network
- Container restarts

### APPLICATIONS

- API Request Rate
- API Latency (P95)
- HTTP Request Rate by Status
- API Error Rate

### LOGS & EVENTS

- Application Logs

## Alerting

The initial alert monitors API availability.

### Bookmark API Down

```promql
up{job="bookmark-api"}
```

Condition:

```text
value < 1
```

Evaluation:

```text
Every 1 minute
Pending: 1 minute
```

Labels:

```text
service = bookmark-api
severity = warning
```

The alert was tested through the following lifecycle:

```text
Normal
  ↓
API stopped
  ↓
Pending
  ↓
Firing
  ↓
API restored
  ↓
Normal
```

## Evidence

- [Prometheus targets](evidence/monitoring-targets.png)
- [Grafana dashboard](evidence/monitoring-dashboard.png)
- [Application logs](evidence/monitoring-logs.png)
- [API alert firing](evidence/alert-api-firing.png)
- [API alert resolved](evidence/alert-api-resolved.png)

## Operational Notes

The current environment uses a self-signed TLS certificate for the
application endpoint.

Prometheus therefore uses TLS certificate verification disabled for the
internal API metrics scrape.

Email notification is configured as a Grafana contact point, but SMTP is
not configured yet.

This is intentionally left as a future improvement.

## Future Improvements

- SMTP/email notification
- Additional infrastructure alerts
- Alertmanager routing
- Long-term metrics retention
- Dashboard variables
- TLS certificate management
- Monitoring stack hardening