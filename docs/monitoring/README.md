# Monitoring

## Overview

The system exposes operational metrics via Prometheus and visualizes them in Grafana. This gives exchange operators real-time visibility into engine health, trade throughput, risk exposure, and infrastructure status.

---

## Accessing Metrics

| Interface | URL |
|---|---|
| Prometheus metrics endpoint | `http://localhost:7002/actuator/prometheus` |
| Spring Boot health check | `http://localhost:7002/actuator/health` |
| Grafana dashboard | `http://localhost:3000` (configure Prometheus as data source) |

---

## Key Metrics

### Matching Engine Throughput

| Metric | What it tells you |
|---|---|
| Commands processed (total) | Total orders, cancellations, and admin actions processed since startup |
| Commands per second | Current throughput rate — useful for spotting load spikes |
| Command latency (P50 / P95 / P99) | How fast the engine is processing; P99 spike indicates backpressure |

### Persistence Pipeline

| Metric | What it tells you |
|---|---|
| Objects queued for persistence | How many events are waiting to be written to MongoDB |
| Objects saved | Confirmed writes — gap between queued and saved indicates write lag |
| Snapshot queue depth | How far behind the snapshot thread is; large values mean slow recovery if a restart occurs |

### Risk & Finance

| Metric | What it tells you |
|---|---|
| Insurance Fund balance | Real-time fund value — drop toward zero signals high liquidation stress |
| Fee account balance | Accumulated trading fees — tracks revenue |

---

## Operational Alerts to Configure

The following conditions are worth alerting on in Grafana:

| Condition | Suggested action |
|---|---|
| Command latency P99 > 5ms | Investigate Kafka lag or CPU pressure on the engine host |
| Persistence queue depth growing | MongoDB write throughput is insufficient — check disk or replica lag |
| Insurance Fund balance < threshold | High liquidation activity; may need to pause high-leverage products |
| Snapshot queue depth > 0 sustained | Snapshot thread is falling behind; check MongoDB write performance |

---

## Infrastructure Health

Spring Boot Actuator exposes standard health indicators for all infrastructure dependencies:

- **Kafka** — broker connectivity and consumer lag
- **MongoDB** — replica set status and write availability
- **Redis** — connection pool and memory usage

These are available at `/actuator/health` and can be scraped by Prometheus or polled by an external health check system.

---

## Deployment

Prometheus and Grafana are not included in the default Docker Compose stack but can be added as additional services. The metrics endpoint is ready to scrape with no additional configuration.

Sample Grafana dashboard JSON is available for import to visualize the above metrics out of the box.
