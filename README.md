
```markdown
# Spring-Actuator-Monitoring-Health-Dashboard

## Overview

This project is a **multi-microservice demonstration** of **Spring Boot Actuator** for **production-grade monitoring**, **health checks**, and **custom dashboards**. It showcases how Actuator provides out-of-the-box and custom endpoints to monitor application health, metrics, environment details, and operational status — crucial for maintaining reliability in production systems.

The focus is on **deep customization**: custom health indicators, detailed metrics, info contributors, and integration readiness with external tools like Prometheus, Grafana, and cloud monitoring platforms.

## Real-World Scenario (Simulated)

In production applications (e.g., e-commerce, banking, SaaS platforms):
- Services must report their operational status (DB connectivity, external APIs, disk space, etc.).
- SREs and DevOps teams rely on health checks for auto-scaling, alerting, and incident response.
- Detailed metrics and info help debug issues quickly without SSH access.
- Custom business health (e.g., "order processing queue depth") is often required beyond standard checks.

We simulate an order processing system with multiple services, each exposing rich Actuator endpoints and custom health indicators for comprehensive monitoring.

## Microservices Involved

| Service                | Responsibility                                                                 | Port  |
|------------------------|--------------------------------------------------------------------------------|-------|
| **eureka-server**      | Service discovery (Netflix Eureka)                                             | 8761  |
| **order-service**      | Main service: processes orders, custom health for queue depth                  | 8081  |
| **inventory-service**  | Checks stock, custom health for external warehouse API                         | 8082  |
| **payment-service**    | Processes payments, custom health for payment gateway connectivity             | 8083  |

All services expose full Actuator endpoints with security and customization.

## Tech Stack

- Spring Boot 3.x
- Spring Boot Actuator
- Micrometer (metrics registry)
- Custom HealthIndicators and HealthContributor
- Custom Metrics (Counter, Timer, Gauge)
- Custom InfoContributor
- Spring Security (endpoint protection)
- Spring Cloud Netflix Eureka
- Lombok
- Maven (multi-module)
- Docker & Docker Compose

## Docker Containers

```yaml
services:
  eureka-server:
    build: ./eureka-server
    ports:
      - "8761:8761"

  order-service:
    build: ./order-service
    depends_on:
      - eureka-server
    ports:
      - "8081:8081"

  inventory-service:
    build: ./inventory-service
    depends_on:
      - eureka-server
    ports:
      - "8082:8082"

  payment-service:
    build: ./payment-service
    depends_on:
      - eureka-server
    ports:
      - "8083:8083"
```

Run with: `docker-compose up --build`

## Actuator Features Implemented

| Endpoint               | Customization                                                          |
|------------------------|------------------------------------------------------------------------|
| **/actuator/health**   | Composite health with custom indicators (queue, external API)          |
| **/actuator/metrics**  | Custom business metrics (orders processed, failed payments)            |
| **/actuator/info**     | Git info, build details, custom app version, environment               |
| **/actuator/prometheus**| Prometheus scrape endpoint                                             |
| **Custom Health**      | Down if critical business component fails (e.g., payment gateway down) |

## Key Features

- Full Actuator endpoint exposure with security (management.endpoint.health.show-details=always)
- Custom HealthIndicator for business logic (e.g., queue depth > threshold → DOWN)
- Custom metrics (counters for orders, timers for processing)
- Detailed health breakdown (disk space, DB, custom checks)
- Info endpoint with build, git, and application metadata
- Prometheus-ready metrics
- Liveness and Readiness probes (for Kubernetes)
- Endpoint security configuration

## Expected Endpoints (All services - e.g., Order Service `http://localhost:8081`)

| Endpoint                          | Description                                      |
|-----------------------------------|--------------------------------------------------|
| GET `/actuator`                   | List all available endpoints                     |
| GET `/actuator/health`            | Overall status + detailed component health       |
| GET `/actuator/health/{component}`| Specific health (e.g., db, customOrderQueue)    |
| GET `/actuator/metrics`           | List all metrics                                 |
| GET `/actuator/metrics/orders.processed`| Custom business metric                        |
| GET `/actuator/info`              | Build, git, and custom application info          |
| GET `/actuator/prometheus`        | Scrape target for Prometheus                     |
| GET `/actuator/liveness`          | Liveness probe (always UP)                       |
| GET `/actuator/readiness`         | Readiness probe (depends on critical components) |

### Sample Health Response
```json
{
  "status": "UP",
  "components": {
    "db": { "status": "UP" },
    "diskSpace": { "status": "UP" },
    "orderQueue": { "status": "DOWN", "details": { "queueDepth": 1500 } },
    "paymentGateway": { "status": "UP" }
  }
}
```

## Architecture Overview

```
External Monitoring (Prometheus, Grafana, CloudWatch)
   ↓
/actuator/prometheus, /health, /metrics
   ↓
Spring Boot Services
   ├── Standard Indicators (DB, Disk, Ping)
   └── Custom Indicators (Business logic: queue, external API)
   ↓
Composite Health → UP/DOWN decision
```

**Health Flow**:
1. Actuator aggregates all HealthIndicators
2. Custom indicator checks critical business condition
3. If any DOWN → overall DOWN
4. External systems (K8s, load balancer) react

## How to Run

1. Clone repository
2. Start Docker: `docker-compose up --build`
3. Access Eureka: `http://localhost:8761`
4. Explore Actuator:
   - `http://localhost:8081/actuator/health`
   - `http://localhost:8081/actuator/metrics`
   - `http://localhost:8081/actuator/info`
5. Simulate failure (e.g., queue depth high) → health DOWN
6. View detailed component status

## Testing Monitoring

1. Normal state → health UP
2. Trigger custom failure condition → specific component DOWN → overall DOWN
3. Process orders → custom metrics increase
4. View /info → rich application metadata
5. Scrape /prometheus → valid format
6. Use with Grafana/Prometheus → visualize custom metrics

## Skills Demonstrated

- Deep customization of Spring Boot Actuator
- Implementing meaningful custom HealthIndicators
- Business-relevant metrics with Micrometer
- Production monitoring best practices
- Liveness/readiness separation
- Integration with external monitoring tools
- Secure endpoint exposure
- Operational readiness for cloud-native apps

## Future Extensions

- Integration with Prometheus + Grafana dashboards
- Alerting via webhook on health change
- Distributed tracing correlation
- Custom dashboard UI (React + Actuator API)
- Integration with Spring Boot Admin
- Dynamic health thresholds via config

