## 🧭 Overview

**name-metrics-api** is a Go-based microservice designed for educational and practical DevOps purposes. It provides an API for analyzing names while serving as a real-world example of building, monitoring, and securing modern cloud-native applications. The service is fully observable — with health checks, Prometheus metrics, and Grafana dashboards — and deployable via Docker and Kubernetes.

---

## 🎯 Project Purpose

This project demonstrates how to create a **fully observable microservice** with:

* **Health checks** (`/health` endpoints for readiness and liveness)
* **Prometheus metrics** and **Grafana dashboards**
* **Structured JSON logging** for production-grade observability
* **Kubernetes-native deployment** with probes, sidecar exporters, and ServiceMonitors
* **Resilient architecture** (graceful restarts, dependency checks, init containers)

It’s a practical exercise in DevOps monitoring and logging best practices — showing how to instrument a service for visibility, reliability, and security.

---

## ⚙️ Tech Stack

| Component            | Technology                   |
| -------------------- | ---------------------------- |
| **Language**         | Go 1.22                      |
| **Framework**        | `net/http` + `gorilla/mux`   |
| **Metrics**          | Prometheus + StatsD Exporter |
| **Monitoring UI**    | Grafana                      |
| **Containerization** | Docker                       |
| **Orchestration**    | Kubernetes                   |
| **CI/CD**            | GitHub Actions (optional)    |

---

## 🚀 Features

* `/metrics` — Prometheus-compatible endpoint with StatsD exporter sidecar
* `/health` — readiness and liveness probes for Kubernetes
* `/analyze?name=<string>` — computes metrics like length, vowels, consonants, complexity
* Structured JSON logs (`{"level":"info","event":"...","ts":"..."}`)
* Grafana dashboards for CPU, RAM, Pod health, latency (p95), and error rate
* ServiceMonitor integration for kube-prometheus-stack

---

## 🧩 Architecture Summary

### Pods & Sidecars

* **Main container:** `name-metrics-api`
* **Sidecar:** `prom/statsd-exporter:v0.26.0` (exposes `/metrics` on port `9102`)

### Health Probes

| Probe     | Path      | Delay | Period | Purpose                                   |
| --------- | --------- | ----- | ------ | ----------------------------------------- |
| Readiness | `/health` | 5s    | 5s     | Checks if app is ready to receive traffic |
| Liveness  | `/health` | 30s   | 10s    | Ensures app is alive; restarts if hung    |

### Monitoring Flow

1. Prometheus scrapes metrics from StatsD exporter (port `9102`).
2. Grafana visualizes data via pre-configured dashboards.
3. Kubernetes probes ensure Pods stay healthy.
4. Structured logs flow to Loki or ELK (optional).

---

## 🧠 Why Structured Logging

Example of bad vs good logging:

**Plain text (bad):**

```
User logged in successfully
```

**Structured JSON (good):**

```json
{"level":"info","event":"user_login","user_id":123,"status":"success","ts":"2025-10-29T20:12:03Z"}
```

Advantages:

* Easy filtering and querying in Loki/ELK
* Consistent log format
* Enables automated alerts and dashboards

---

## 📊 Critical Metrics

| Type           | Example                             | Description                      |
| -------------- | ----------------------------------- | -------------------------------- |
| **Latency**    | `http_request_duration_seconds`     | Measures response time (P95/P99) |
| **Errors**     | `http_requests_total{status="500"}` | Tracks failed requests           |
| **Traffic**    | `requests_per_second`               | Shows service load               |
| **Saturation** | CPU / RAM usage                     | Detects performance bottlenecks  |

These are the **“Four Golden Signals”** of SRE: latency, traffic, errors, saturation.

---

## 🧱 Kubernetes Example

### Deployment (simplified)

```yaml
containers:
  - name: name-metrics-api
    image: name-metrics-api:latest
    ports:
      - name: http
        containerPort: 8080
    readinessProbe:
      httpGet: { path: /health, port: 8080 }
      initialDelaySeconds: 5
    livenessProbe:
      httpGet: { path: /health, port: 8080 }
      initialDelaySeconds: 30

  - name: statsd-exporter
    image: prom/statsd-exporter:v0.26.0
    ports:
      - name: metrics
        containerPort: 9102
```

### ServiceMonitor

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: name-metrics-servicemonitor
  namespace: monitoring
  labels:
    release: monitoring
spec:
  selector:
    matchLabels:
      app: name-metrics-api
  endpoints:
    - port: metrics
      path: /metrics
      interval: 15s
```

---

## 📈 Grafana Dashboards

Current panels:

* Pod Health (ready/unready)
* CPU Usage
* Memory Usage
* Request Latency (P95)
* Error Rate (% of 5xx)

Future improvements:

* Alertmanager rules for uptime or error thresholds
* User-defined latency histograms from app code
* Enhanced tracing integration

---

## 🧰 Local Setup

### Run Locally

```bash
go mod tidy
go run main.go
```

### Run with Docker

```bash
docker build -t name-metrics-api .
docker run -p 8080:8080 name-metrics-api
```

### Deploy on Kubernetes

```bash
kubectl apply -f k8s/
```

Then verify health:

```bash
kubectl port-forward svc/name-metrics-api 8080:8080
curl localhost:8080/health
```

---

## 🧪 Testing

```bash
go test ./...
```
---

## 🔒 Security & Best Practices

* Runs as non-root in container
* Minimal base image
* Proper liveness/readiness separation
* TLS-ready Ingress configuration
* Metrics secured under `/metrics`
* Configurable through environment variables

---

## 🧭 Future Enhancements

* Implement latency/error histogram exports directly from Go code
* Integrate alert rules in Prometheus
* Add OpenTelemetry tracing
* Extend Grafana dashboards with database performance metrics

---
