<p align="center">
  <img src="https://img.shields.io/badge/Microservices-FF6B6B?style=for-the-badge&logo=docker&logoColor=white" alt="Microservices"/>
</p>

<h1 align="center">🔗 Microservices Interview Guide</h1>

<p align="center">
  <a href="README.md">← Back to Main Guide</a>
</p>

---

## 📚 Section Overview

| # | Topic | Difficulty |
|---|-------|------------|
| 1 | [Microservices Fundamentals](#1-microservices-fundamentals) | ⭐⭐ |
| 2 | [Design Patterns](#2-microservices-design-patterns) | ⭐⭐⭐⭐ |
| 3 | [Service Communication](#3-service-communication) | ⭐⭐⭐ |
| 4 | [Service Discovery](#4-service-discovery) | ⭐⭐⭐ |
| 5 | [API Gateway](#5-api-gateway) | ⭐⭐⭐ |
| 6 | [Configuration Management](#6-configuration-management) | ⭐⭐ |
| 7 | [Security](#7-microservices-security) | ⭐⭐⭐⭐ |
| 8 | [Data Management](#8-data-management) | ⭐⭐⭐⭐ |
| 9 | [Resilience Patterns](#9-resilience-patterns) | ⭐⭐⭐⭐ |
| 10 | [Observability](#10-observability) | ⭐⭐⭐ |
| 11 | [Containerization](#11-containerization--orchestration) | ⭐⭐⭐ |
| 12 | [CI/CD](#12-cicd-for-microservices) | ⭐⭐⭐ |
| 13 | [Event-Driven Architecture](#13-event-driven-architecture) | ⭐⭐⭐⭐ |

---

## 1. Microservices Fundamentals

### 🔷 Monolithic vs Microservices

| Aspect | Monolithic | Microservices |
|--------|------------|---------------|
| **Deployment** | Single unit | Independent services |
| **Scaling** | Scale entire app | Scale individual services |
| **Technology** | Single stack | Polyglot possible |
| **Development** | Large teams, long cycles | Small teams, fast cycles |
| **Failure** | Can affect entire app | Isolated failures |
| **Database** | Single shared | Database per service |

### 🔷 When to Use Microservices

✅ **Use When:**
- Large, complex applications
- Need independent scaling
- Multiple teams
- Different tech requirements
- Frequent deployments

❌ **Avoid When:**
- Small, simple applications
- Small team
- Tight deadlines
- Limited DevOps experience

---

## 2. Microservices Design Patterns

### 🔷 Pattern Categories

```
┌────────────────────────────────────────────────────────────────────┐
│                   MICROSERVICES PATTERNS                           │
├───────────────────┬────────────────────────────────────────────────┤
│   DECOMPOSITION   │  By Business Capability, By Subdomain (DDD)    │
├───────────────────┼────────────────────────────────────────────────┤
│     DATABASE      │  Database per Service, Shared Database         │
├───────────────────┼────────────────────────────────────────────────┤
│   COMMUNICATION   │  API Gateway, BFF, Service Mesh                │
├───────────────────┼────────────────────────────────────────────────┤
│  DATA MANAGEMENT  │  Saga, CQRS, Event Sourcing                    │
├───────────────────┼────────────────────────────────────────────────┤
│    RELIABILITY    │  Circuit Breaker, Bulkhead, Retry              │
├───────────────────┼────────────────────────────────────────────────┤
│    DEPLOYMENT     │  Sidecar, Ambassador                           │
└───────────────────┴────────────────────────────────────────────────┘
```

---

## 3. Service Communication

### 🔷 Synchronous vs Asynchronous

| Type | Protocol | Use Case | Pros | Cons |
|------|----------|----------|------|------|
| **Sync** | REST, gRPC | Real-time queries | Simple, immediate | Tight coupling |
| **Async** | Kafka, RabbitMQ | Events, commands | Loose coupling, resilient | Complex, eventual consistency |

### 🔷 REST vs gRPC

| Aspect | REST | gRPC |
|--------|------|------|
| **Protocol** | HTTP/1.1, JSON | HTTP/2, Protocol Buffers |
| **Performance** | Slower | ~10x faster |
| **Streaming** | Limited | Full bidirectional |
| **Browser Support** | ✅ Native | ❌ Requires proxy |
| **Use Case** | Public APIs | Internal microservices |

---

## 4. Service Discovery

```
┌────────────────────────────────────────────────────────────────────┐
│                      SERVICE DISCOVERY                             │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌─────────────┐         ┌─────────────┐         ┌─────────────┐   │
│  │  Service A  │─────────│  Registry   │─────────│  Service B  │   │
│  │             │ register│   (Eureka)  │ discover│             │   │
│  └─────────────┘         └─────────────┘         └─────────────┘   │
│        │                       │                        │          │
│        │                       │                        │          │
│        └───────────── Heartbeat ────────────────────────┘          │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 5. API Gateway

### 🔷 Responsibilities

| Function | Description |
|----------|-------------|
| **Routing** | Route requests to appropriate service |
| **Load Balancing** | Distribute traffic |
| **Authentication** | Verify JWT tokens |
| **Rate Limiting** | Prevent abuse |
| **Request Transformation** | Modify requests/responses |
| **Aggregation** | Combine multiple service calls |

---

## 6. Configuration Management

```yaml
# Config Server: application.yml for order-service
order-service:
  inventory:
    url: ${INVENTORY_URL:http://inventory-service}
    timeout: 5000
  payment:
    url: ${PAYMENT_URL:http://payment-service}
```

---

## 7. Microservices Security

### 🔷 OAuth2 Flow

```
┌────────────────────────────────────────────────────────────────────┐
│                      OAuth2 + JWT FLOW                             │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌────────┐       ┌──────────┐       ┌─────────────┐               │
│  │ Client │──1.───│  Auth    │       │  Resource   │               │
│  │        │◄──2.──│  Server  │       │  Server     │               │
│  │        │──────────3.───────────────│             │              │
│  │        │◄─────────4.───────────────│             │              │
│  └────────┘       └──────────┘       └─────────────┘               │
│                                                                    │
│  1. Request token with credentials                                 │
│  2. Return JWT token                                               │
│  3. Request resource with JWT in header                            │
│  4. Validate JWT & return resource                                 │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 8. Data Management

### 🔷 8.1 Database per Service Pattern

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DATABASE PER SERVICE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐        │
│  │ Order Service │    │ User Service  │    │ Product Svc   │        │
│  └───────┬───────┘    └───────┬───────┘    └───────┬───────┘        │
│          │                    │                    │                │
│          ▼                    ▼                    ▼                │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐        │
│  │  Orders DB    │    │   Users DB    │    │  Products DB  │        │
│  │  (PostgreSQL) │    │   (MongoDB)   │    │   (MySQL)     │        │
│  └───────────────┘    └───────────────┘    └───────────────┘        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 🔷 8.2 Saga Pattern

| Type | Description | Use Case |
|------|-------------|----------|
| **Choreography** | Services emit events, others react | Simple workflows |
| **Orchestration** | Central orchestrator manages flow | Complex workflows |

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SAGA - CHOREOGRAPHY                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Order Service ──── Order Created Event ────► Inventory Service     │
│       │                                              │              │
│       │                                              ▼              │
│       │                                    Stock Reserved Event     │
│       │                                              │              │
│       ▼                                              ▼              │
│  Order Confirmed ◄───────────────────────── Payment Service         │
│                                                                     │
│  On Failure: Compensating transactions (rollback)                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 🔷 8.3 CQRS (Command Query Responsibility Segregation)

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CQRS PATTERN                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                      ┌───────────────┐                              │
│                      │     Client    │                              │
│                      └───────┬───────┘                              │
│                    ┌─────────┴─────────┐                            │
│                    ▼                   ▼                            │
│           ┌─────────────┐      ┌─────────────┐                      │
│           │  Commands   │      │   Queries   │                      │
│           │  (Write)    │      │   (Read)    │                      │
│           └──────┬──────┘      └──────┬──────┘                      │
│                  │                    │                             │
│                  ▼                    ▼                             │
│           ┌─────────────┐      ┌─────────────┐                      │
│           │  Write DB   │─sync─│  Read DB    │                      │
│           │ (Normalized)│──────│(Denormalized)│                     │
│           └─────────────┘      └─────────────┘                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 🔷 8.4 Event Sourcing

```java
// Instead of storing current state, store all events
public class Account {
    private List<Event> events = new ArrayList<>();
    
    public void deposit(BigDecimal amount) {
        events.add(new MoneyDeposited(id, amount, Instant.now()));
    }
    
    public void withdraw(BigDecimal amount) {
        events.add(new MoneyWithdrawn(id, amount, Instant.now()));
    }
    
    // Rebuild state by replaying events
    public BigDecimal getBalance() {
        return events.stream()
            .reduce(BigDecimal.ZERO, (balance, event) -> {
                if (event instanceof MoneyDeposited) return balance.add(event.amount);
                if (event instanceof MoneyWithdrawn) return balance.subtract(event.amount);
                return balance;
            }, BigDecimal::add);
    }
}
```

---

## 9. Resilience Patterns

### 🔷 9.1 Circuit Breaker

```
┌─────────────────────────────────────────────────────────────────────┐
│                     CIRCUIT BREAKER STATES                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│      ┌────────────────────────────────────────────────────┐         │
│      │                                                    │         │
│      ▼                                                    │         │
│  ┌────────┐   failure threshold   ┌────────┐   timeout   │          │
│  │ CLOSED │─────────────────────► │  OPEN  │─────────────┘          │
│  │        │                       │        │                        │
│  │ Normal │                       │ Reject │                        │
│  │  flow  │                       │  all   │                        │
│  └────────┘                       └───┬────┘                        │
│      ▲                                │                             │
│      │                                ▼                             │
│      │       success           ┌───────────┐                        │
│      └─────────────────────────│ HALF-OPEN │                        │
│                                │           │                        │
│                                │ Test some │                        │
│                                │ requests  │                        │
│                                └───────────┘                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

```java
// Resilience4j Circuit Breaker
@CircuitBreaker(name = "inventoryService", fallbackMethod = "fallback")
@Retry(name = "inventoryService")
@TimeLimiter(name = "inventoryService")
public CompletableFuture<Inventory> checkInventory(String productId) {
    return CompletableFuture.supplyAsync(() -> 
        inventoryClient.check(productId));
}

public CompletableFuture<Inventory> fallback(String productId, Exception ex) {
    return CompletableFuture.completedFuture(Inventory.unknown(productId));
}
```

### 🔷 9.2 Resilience Patterns Summary

| Pattern | Purpose | Implementation |
|---------|---------|----------------|
| **Circuit Breaker** | Fail fast when service is down | Resilience4j, Hystrix |
| **Retry** | Retry failed requests | `@Retry`, exponential backoff |
| **Timeout** | Limit wait time | `@TimeLimiter` |
| **Bulkhead** | Isolate failures | Thread pool isolation |
| **Rate Limiter** | Limit request rate | Token bucket algorithm |

---

## 10. Observability

### 🔷 Three Pillars of Observability

```
┌─────────────────────────────────────────────────────────────────────┐
│                    THREE PILLARS OF OBSERVABILITY                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐        │
│  │     LOGGING     │ │     METRICS     │ │     TRACING     │        │
│  │                 │ │                 │ │                 │        │
│  │  📝 What        │ │  📊 How much   │ │  🔍 Where       │        │
│  │  happened?      │ │  and how fast?  │ │  did it go?     │        │
│  │                 │ │                 │ │                 │        │
│  │  ELK Stack      │ │  Prometheus     │ │  Zipkin/Jaeger  │        │
│  │  (Elasticsearch │ │  Grafana        │ │  Sleuth         │        │
│  │  Logstash       │ │  Micrometer     │ │                 │        │
│  │  Kibana)        │ │                 │ │                 │        │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 🔷 Correlation IDs

```java
// Add correlation ID to all requests
@Component
public class CorrelationIdFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                         FilterChain chain) throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        String correlationId = req.getHeader("X-Correlation-ID");
        
        if (correlationId == null) {
            correlationId = UUID.randomUUID().toString();
        }
        
        MDC.put("correlationId", correlationId);
        chain.doFilter(request, response);
        MDC.remove("correlationId");
    }
}
```

---

## 11. Containerization & Orchestration

### 🔷 11.1 Docker

```dockerfile
# Multi-stage Dockerfile for Spring Boot
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./gradlew bootJar

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  order-service:
    build: ./order-service
    ports:
      - "8081:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - DB_HOST=postgres
    depends_on:
      - postgres
      - kafka
  
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: orders
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 🔷 11.2 Kubernetes Basics

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: order-service:1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
---
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

---

## 12. CI/CD for Microservices

### 🔷 Deployment Strategies

| Strategy | Description | Risk | Rollback |
|----------|-------------|------|----------|
| **Rolling Update** | Gradual replacement | Low | Slow |
| **Blue-Green** | Switch between environments | Medium | Instant |
| **Canary** | Route % of traffic to new version | Low | Fast |
| **Feature Flags** | Toggle features in code | Lowest | Instant |

```
┌─────────────────────────────────────────────────────────────────────┐
│                      DEPLOYMENT STRATEGIES                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  BLUE-GREEN:                                                        │
│  ┌──────────┐                    ┌──────────┐                       │
│  │  Blue    │  ←── Traffic ──►   │  Green   │                       │
│  │  (v1.0)  │      Switch        │  (v1.1)  │                       │
│  └──────────┘                    └──────────┘                       │
│                                                                     │
│  CANARY:                                                            │
│  ┌──────────┐  ◄─── 90% traffic                                     │
│  │  Stable  │                                                       │
│  │  (v1.0)  │                                                       │
│  └──────────┘                                                       │
│  ┌──────────┐  ◄─── 10% traffic                                     │
│  │  Canary  │                                                       │
│  │  (v1.1)  │                                                       │
│  └──────────┘                                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 13. Event-Driven Architecture

### 🔷 Event Types

| Type | Purpose | Example |
|------|---------|---------|
| **Domain Events** | Business state changes | `OrderPlaced`, `PaymentReceived` |
| **Integration Events** | Cross-service communication | `InventoryUpdated` |
| **Notification Events** | Inform about changes | `UserEmailChanged` |

### 🔷 Message Brokers

```
┌─────────────────────────────────────────────────────────────────────┐
│                       KAFKA vs RABBITMQ                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  KAFKA:                           RABBITMQ:                         │
│  • Log-based (persistent)         • Queue-based                     │
│  • High throughput                 • Flexible routing               │
│  • Replay messages                 • Message acknowledgment         │
│  • Partitioning                    • Dead letter queues             │
│  • Consumer groups                 • Priority queues                │
│                                                                     │
│  Use for:                          Use for:                         │
│  • Event streaming                 • Task queues                    │
│  • Log aggregation                 • Request/Reply                  │
│  • High-volume events              • Complex routing                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

<p align="center">
  <a href="README.md">← Back to Main Guide</a>
</p>

