# Microservices Interview Questions & Answers - Comprehensive Guide

## Table of Contents
1. [Core Concepts](#1-core-concepts)
2. [Architecture & Design Patterns](#2-architecture--design-patterns)
3. [Communication Patterns](#3-communication-patterns)
4. [Data Management](#4-data-management)
5. [Service Discovery & Registry](#5-service-discovery--registry)
6. [API Gateway](#6-api-gateway)
7. [Security](#7-security)
8. [Resilience & Fault Tolerance](#8-resilience--fault-tolerance)
9. [Monitoring, Logging & Observability](#9-monitoring-logging--observability)
10. [Containerization & Orchestration](#10-containerization--orchestration)
11. [Testing Strategies](#11-testing-strategies)
12. [Deployment Strategies](#12-deployment-strategies)
13. [Event-Driven Architecture](#13-event-driven-architecture)
14. [Configuration Management](#14-configuration-management)
15. [Performance & Scalability](#15-performance--scalability)
16. [Real-World Scenarios](#16-real-world-scenarios)

---

## 1. Core Concepts

### Q1: What are Microservices?
**Answer:**
Microservices is an architectural style that structures an application as a collection of loosely coupled, independently deployable services. Each service:
- Is built around a specific business capability
- Runs in its own process
- Communicates via lightweight protocols (HTTP/REST, gRPC, messaging)
- Can be developed, deployed, and scaled independently
- Has its own database (Database per Service pattern)

**Key Characteristics:**
- Single Responsibility
- Autonomous
- Decentralized Data Management
- Failure Isolation
- Technology Heterogeneity

---

### Q2: What is the difference between Monolithic and Microservices Architecture?

**Answer:**

| Aspect | Monolithic | Microservices |
|--------|-----------|---------------|
| **Deployment** | Single deployable unit | Multiple independent services |
| **Scaling** | Scale entire application | Scale individual services |
| **Technology** | Single technology stack | Polyglot (multiple technologies) |
| **Database** | Shared database | Database per service |
| **Team Structure** | Cross-functional teams | Small, focused teams |
| **Failure Impact** | Entire system affected | Isolated failures |
| **Development Speed** | Slower for large apps | Faster parallel development |
| **Complexity** | Simpler initially | Higher operational complexity |
| **Testing** | Easier end-to-end | Requires integration testing |

---

### Q3: What are the advantages of Microservices?

**Answer:**
1. **Independent Deployability** - Deploy services without affecting others
2. **Technology Diversity** - Use best-fit technology for each service
3. **Scalability** - Scale individual services based on demand
4. **Fault Isolation** - Failures don't cascade across the system
5. **Team Autonomy** - Small teams can own entire services
6. **Faster Time to Market** - Parallel development and deployment
7. **Easy Maintenance** - Smaller codebases are easier to understand
8. **Reusability** - Services can be reused across applications

---

### Q4: What are the disadvantages/challenges of Microservices?

**Answer:**
1. **Distributed System Complexity** - Network latency, message serialization
2. **Data Consistency** - Maintaining consistency across services
3. **Service Discovery** - Finding and connecting services
4. **Testing Complexity** - Integration and end-to-end testing challenges
5. **Operational Overhead** - More services to deploy, monitor, manage
6. **Network Reliability** - Handling partial failures
7. **Debugging Difficulty** - Tracing requests across services
8. **Team Coordination** - API contracts and versioning

---

### Q5: When should you NOT use Microservices?

**Answer:**
- **Small applications** - Overhead outweighs benefits
- **Startups with unclear domain** - Boundaries may change frequently
- **Team lacks experience** - DevOps, distributed systems expertise needed
- **Tight coupling requirements** - Strong consistency needs
- **Limited infrastructure** - No container orchestration, CI/CD pipeline
- **Rapid prototyping** - Monolith is faster to build initially

**Rule of Thumb:** Start with a well-structured monolith, extract microservices when needed.

---

### Q6: What is Domain-Driven Design (DDD) and how does it relate to Microservices?

**Answer:**
DDD is a software development approach that focuses on modeling software based on the business domain.

**Key DDD Concepts for Microservices:**

1. **Bounded Context** - A logical boundary within which a domain model is defined
   - Each microservice should align with a bounded context
   - Clear separation of concerns

2. **Ubiquitous Language** - Common vocabulary between developers and domain experts

3. **Aggregates** - Cluster of domain objects treated as a single unit
   - Defines transaction boundaries

4. **Domain Events** - Significant occurrences in the domain
   - Enable event-driven communication

5. **Context Mapping** - Relationships between bounded contexts

```java
// Example: Order Aggregate in Order Service
@Aggregate
public class Order {
    @AggregateIdentifier
    private OrderId orderId;
    private CustomerId customerId;
    private List<OrderItem> items;
    private OrderStatus status;
    
    public void addItem(Product product, int quantity) {
        // Business logic
        apply(new OrderItemAddedEvent(orderId, product.getId(), quantity));
    }
}
```

---

### Q7: What is the Single Responsibility Principle in Microservices?

**Answer:**
Each microservice should have one, and only one, reason to change. It should:
- Focus on a single business capability
- Have a well-defined scope and boundary
- Be owned by a single team

**Example:**
Instead of a single "User Service" handling:
- User registration
- Authentication
- User preferences
- Notifications

Split into:
- **Identity Service** - Registration, profile management
- **Authentication Service** - Login, tokens, sessions
- **Preferences Service** - User settings
- **Notification Service** - Email, SMS, push notifications

---

## 2. Architecture & Design Patterns

### Q8: Explain the API Gateway Pattern

**Answer:**
An API Gateway is a single entry point for all client requests that handles:

**Responsibilities:**
- Request routing to appropriate services
- Authentication and authorization
- Rate limiting and throttling
- Request/Response transformation
- Load balancing
- Caching
- SSL termination
- API versioning

```
┌─────────────┐
│   Clients   │
└──────┬──────┘
       │
┌──────▼──────┐
│ API Gateway │
└──────┬──────┘
       │
┌──────┼──────┬──────────┐
▼      ▼      ▼          ▼
┌────┐ ┌────┐ ┌────┐ ┌────────┐
│Svc1│ │Svc2│ │Svc3│ │Svc N...│
└────┘ └────┘ └────┘ └────────┘
```

**Popular Implementations:**
- Kong
- Netflix Zuul / Spring Cloud Gateway
- AWS API Gateway
- NGINX
- Traefik

**Example - Spring Cloud Gateway:**
```java
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("order-service", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .addRequestHeader("X-Request-Source", "gateway"))
                .uri("lb://order-service"))
            .route("user-service", r -> r
                .path("/api/users/**")
                .filters(f -> f.stripPrefix(1))
                .uri("lb://user-service"))
            .build();
    }
}
```

---

### Q9: What is the Backend for Frontend (BFF) Pattern?

**Answer:**
BFF pattern creates dedicated backend services for each frontend application (web, mobile, IoT).

**Why use BFF:**
- Different clients have different data requirements
- Mobile needs optimized payloads
- Web may need more detailed data
- Reduces client-side complexity

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│   Web    │  │  Mobile  │  │   IoT    │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │              │
┌────▼─────┐  ┌────▼─────┐  ┌────▼─────┐
│ Web BFF  │  │Mobile BFF│  │ IoT BFF  │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │              │
     └─────────────┼──────────────┘
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
       ┌─────┐  ┌─────┐  ┌─────┐
       │Svc A│  │Svc B│  │Svc C│
       └─────┘  └─────┘  └─────┘
```

---

### Q10: Explain the Strangler Fig Pattern

**Answer:**
A migration pattern for gradually replacing a monolithic application with microservices.

**Steps:**
1. Identify a feature to migrate
2. Implement the feature as a new microservice
3. Route traffic from monolith to new service
4. Repeat until monolith is fully replaced

**Implementation with API Gateway:**
```yaml
# Kong configuration example
services:
  - name: legacy-monolith
    url: http://legacy-app:8080
    
  - name: new-order-service
    url: http://order-service:8081

routes:
  # New service handles order endpoints
  - name: orders-route
    service: new-order-service
    paths:
      - /api/orders
    
  # Everything else goes to monolith
  - name: legacy-route
    service: legacy-monolith
    paths:
      - /
```

**Benefits:**
- Low risk migration
- Incremental progress
- Easy rollback
- Continuous delivery during migration

---

### Q11: What is the Sidecar Pattern?

**Answer:**
The Sidecar pattern deploys helper components alongside the main service in the same host/pod.

**Use Cases:**
- Logging and monitoring agents
- Configuration management
- Network proxies (Service Mesh)
- Security (mTLS, auth)

```
┌─────────────────────────────────┐
│           Pod/Host              │
│  ┌─────────────┐ ┌───────────┐  │
│  │    Main     │ │  Sidecar  │  │
│  │   Service   │◄┤  (Envoy)  │  │
│  └─────────────┘ └───────────┘  │
└─────────────────────────────────┘
```

**Example - Kubernetes Sidecar:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: order-service
spec:
  containers:
    - name: order-service
      image: order-service:latest
      ports:
        - containerPort: 8080
    
    - name: envoy-sidecar
      image: envoyproxy/envoy:latest
      ports:
        - containerPort: 9901
      volumeMounts:
        - name: envoy-config
          mountPath: /etc/envoy
```

---

### Q12: Explain the Ambassador Pattern

**Answer:**
Ambassador pattern creates a helper service that handles network-related tasks on behalf of the main service.

**Responsibilities:**
- Retries with exponential backoff
- Circuit breaking
- Routing
- Logging
- Monitoring

**Difference from Sidecar:**
- Ambassador specifically handles outbound connections
- Sidecar is more general-purpose

```
┌──────────────────────────────────────┐
│               Pod                    │
│  ┌──────────────┐  ┌──────────────┐  │
│  │    Main      │  │  Ambassador  │  │
│  │   Service    │──▶   (Envoy)   │──────▶ External
│  └──────────────┘  └──────────────┘  │    Services
└──────────────────────────────────────┘
```

---

### Q13: What is the Saga Pattern?

**Answer:**
Saga pattern manages distributed transactions across multiple microservices without using distributed ACID transactions.

**Types:**

**1. Choreography-based Saga:**
- Services communicate through events
- No central coordinator
- Each service knows what to do next

```
Order Service ──▶ Payment Service ──▶ Inventory Service ──▶ Shipping Service
      │                  │                    │                    │
      ▼                  ▼                    ▼                    ▼
  Order Created    Payment Processed    Stock Reserved      Order Shipped
      │                  │                    │                    │
      └──────────────────┴────────────────────┴────────────────────┘
                    (Compensating transactions on failure)
```

**2. Orchestration-based Saga:**
- Central coordinator (Saga Orchestrator)
- Orchestrator tells services what to do

```java
// Saga Orchestrator Example
@Saga
public class OrderSaga {
    
    @StartSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(OrderCreatedEvent event) {
        // Step 1: Reserve Payment
        commandGateway.send(new ReservePaymentCommand(
            event.getOrderId(), 
            event.getAmount()
        ));
    }
    
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(PaymentReservedEvent event) {
        // Step 2: Reserve Inventory
        commandGateway.send(new ReserveInventoryCommand(
            event.getOrderId(),
            event.getItems()
        ));
    }
    
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(PaymentFailedEvent event) {
        // Compensating transaction
        commandGateway.send(new CancelOrderCommand(event.getOrderId()));
    }
    
    @EndSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(OrderCompletedEvent event) {
        // Saga completed successfully
    }
}
```

**Compensating Transactions:**
Each step must have a compensating action to undo changes:
| Step | Action | Compensation |
|------|--------|--------------|
| 1 | Create Order | Cancel Order |
| 2 | Reserve Payment | Refund Payment |
| 3 | Reserve Inventory | Release Inventory |
| 4 | Ship Order | Return Order |

---

### Q14: Explain CQRS Pattern

**Answer:**
**Command Query Responsibility Segregation (CQRS)** separates read and write operations into different models.

**Components:**
- **Command Model** - Handles create, update, delete operations
- **Query Model** - Optimized for read operations

```
┌─────────────────────────────────────────────────────┐
│                     Client                          │
└──────────────────────┬──────────────────────────────┘
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
    ┌───────────┐             ┌───────────┐
    │  Command  │             │   Query   │
    │   Side    │             │   Side    │
    └─────┬─────┘             └─────┬─────┘
          │                         │
    ┌─────▼─────┐             ┌─────▼─────┐
    │  Write    │   Events    │   Read    │
    │  Model    │────────────▶│   Model   │
    │  (SQL)    │             │  (NoSQL)  │
    └───────────┘             └───────────┘
```

**Example Implementation:**
```java
// Command Side
@Service
public class OrderCommandService {
    
    private final OrderRepository writeRepository;
    private final EventPublisher eventPublisher;
    
    @Transactional
    public OrderId createOrder(CreateOrderCommand command) {
        Order order = new Order(command.getCustomerId(), command.getItems());
        writeRepository.save(order);
        
        // Publish event for read model sync
        eventPublisher.publish(new OrderCreatedEvent(order));
        return order.getId();
    }
}

// Query Side
@Service
public class OrderQueryService {
    
    private final OrderReadRepository readRepository;
    
    public OrderDTO getOrder(OrderId orderId) {
        return readRepository.findById(orderId)
            .map(this::toDTO)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
    }
    
    public List<OrderSummaryDTO> getOrdersByCustomer(CustomerId customerId) {
        return readRepository.findByCustomerId(customerId);
    }
}

// Event Handler for Read Model Sync
@EventHandler
public class OrderProjection {
    
    private final OrderReadRepository readRepository;
    
    @EventHandler
    public void on(OrderCreatedEvent event) {
        OrderReadModel readModel = new OrderReadModel(
            event.getOrderId(),
            event.getCustomerId(),
            event.getTotalAmount(),
            event.getStatus()
        );
        readRepository.save(readModel);
    }
}
```

**Benefits:**
- Optimized read/write models
- Better scalability
- Flexibility in data storage
- Improved performance

---

### Q15: What is Event Sourcing?

**Answer:**
Event Sourcing stores state as a sequence of events rather than current state.

**Instead of:**
```json
{
  "orderId": "123",
  "status": "SHIPPED",
  "total": 150.00
}
```

**Store as:**
```json
[
  {"type": "OrderCreated", "orderId": "123", "customerId": "C1", "timestamp": "..."},
  {"type": "ItemAdded", "orderId": "123", "productId": "P1", "price": 100.00},
  {"type": "ItemAdded", "orderId": "123", "productId": "P2", "price": 50.00},
  {"type": "PaymentReceived", "orderId": "123", "amount": 150.00},
  {"type": "OrderShipped", "orderId": "123", "trackingNumber": "TRK123"}
]
```

**Implementation:**
```java
public class Order {
    private OrderId id;
    private OrderStatus status;
    private List<OrderItem> items;
    private Money total;
    
    // Rebuild state from events
    public static Order recreate(List<DomainEvent> events) {
        Order order = new Order();
        events.forEach(order::apply);
        return order;
    }
    
    private void apply(DomainEvent event) {
        if (event instanceof OrderCreatedEvent) {
            this.id = ((OrderCreatedEvent) event).getOrderId();
            this.status = OrderStatus.CREATED;
        } else if (event instanceof ItemAddedEvent) {
            ItemAddedEvent e = (ItemAddedEvent) event;
            this.items.add(new OrderItem(e.getProductId(), e.getPrice()));
            this.total = this.total.add(e.getPrice());
        } else if (event instanceof OrderShippedEvent) {
            this.status = OrderStatus.SHIPPED;
        }
    }
}
```

**Benefits:**
- Complete audit trail
- Temporal queries (state at any point)
- Event replay for debugging
- Natural fit with CQRS

**Challenges:**
- Event schema evolution
- Snapshot management for performance
- Eventual consistency

---

## 3. Communication Patterns

### Q16: What are the different types of inter-service communication?

**Answer:**

**1. Synchronous Communication:**

| Protocol | Use Case | Pros | Cons |
|----------|----------|------|------|
| REST/HTTP | CRUD operations, simple requests | Simple, widely supported | Coupling, latency |
| gRPC | High-performance, polyglot | Fast, typed contracts | Complexity, browser support |
| GraphQL | Flexible queries | Client-driven queries | Caching complexity |

**2. Asynchronous Communication:**

| Pattern | Use Case | Pros | Cons |
|---------|----------|------|------|
| Message Queue | Task processing, decoupling | Loose coupling, resilient | Complexity, debugging |
| Pub/Sub | Event broadcasting | Scalable, decoupled | Message ordering |
| Event Streaming | Real-time data, event sourcing | Replay, audit trail | Storage costs |

---

### Q17: Explain REST vs gRPC

**Answer:**

| Aspect | REST | gRPC |
|--------|------|------|
| **Protocol** | HTTP/1.1 | HTTP/2 |
| **Format** | JSON/XML | Protocol Buffers |
| **Contract** | OpenAPI/Swagger | .proto files |
| **Performance** | Good | Excellent |
| **Streaming** | Limited | Bidirectional |
| **Browser Support** | Native | Requires grpc-web |
| **Learning Curve** | Low | Medium |

**gRPC Example:**
```protobuf
// order.proto
syntax = "proto3";

package order;

service OrderService {
    rpc CreateOrder(CreateOrderRequest) returns (OrderResponse);
    rpc GetOrder(GetOrderRequest) returns (OrderResponse);
    rpc StreamOrderUpdates(GetOrderRequest) returns (stream OrderUpdate);
}

message CreateOrderRequest {
    string customer_id = 1;
    repeated OrderItem items = 2;
}

message OrderItem {
    string product_id = 1;
    int32 quantity = 2;
    double price = 3;
}

message OrderResponse {
    string order_id = 1;
    string status = 2;
    double total = 3;
}
```

**Java gRPC Implementation:**
```java
@GrpcService
public class OrderGrpcService extends OrderServiceGrpc.OrderServiceImplBase {
    
    private final OrderService orderService;
    
    @Override
    public void createOrder(CreateOrderRequest request, 
                           StreamObserver<OrderResponse> responseObserver) {
        Order order = orderService.create(
            request.getCustomerId(),
            request.getItemsList()
        );
        
        OrderResponse response = OrderResponse.newBuilder()
            .setOrderId(order.getId())
            .setStatus(order.getStatus().name())
            .setTotal(order.getTotal())
            .build();
            
        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }
}
```

---

### Q18: Explain Message Queues vs Event Streaming

**Answer:**

**Message Queues (RabbitMQ, ActiveMQ, SQS):**
- Messages consumed once and removed
- Point-to-point or pub/sub
- Best for task distribution

```java
// RabbitMQ Producer
@Service
public class OrderEventPublisher {
    
    private final RabbitTemplate rabbitTemplate;
    
    public void publishOrderCreated(Order order) {
        OrderCreatedMessage message = new OrderCreatedMessage(
            order.getId(),
            order.getCustomerId(),
            order.getTotal()
        );
        rabbitTemplate.convertAndSend("order.exchange", "order.created", message);
    }
}

// RabbitMQ Consumer
@Component
public class OrderEventConsumer {
    
    @RabbitListener(queues = "order.created.queue")
    public void handleOrderCreated(OrderCreatedMessage message) {
        // Process order
        log.info("Processing order: {}", message.getOrderId());
    }
}
```

**Event Streaming (Kafka, Pulsar):**
- Events persisted in log
- Multiple consumers can read
- Replay capability
- Best for event sourcing, real-time analytics

```java
// Kafka Producer
@Service
public class OrderEventPublisher {
    
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    public void publishOrderCreated(Order order) {
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getId(),
            order.getCustomerId(),
            order.getTotal(),
            Instant.now()
        );
        kafkaTemplate.send("order-events", order.getId(), event);
    }
}

// Kafka Consumer
@Component
public class OrderEventConsumer {
    
    @KafkaListener(topics = "order-events", groupId = "inventory-service")
    public void handleOrderEvent(OrderEvent event) {
        if (event instanceof OrderCreatedEvent) {
            inventoryService.reserveStock((OrderCreatedEvent) event);
        }
    }
}
```

---

### Q19: What is the difference between Pub/Sub and Message Queue?

**Answer:**

```
Message Queue (Point-to-Point):
┌──────────┐     ┌─────────┐     ┌──────────┐
│ Producer │────▶│  Queue  │────▶│ Consumer │ (One consumer gets message)
└──────────┘     └─────────┘     └──────────┘

Pub/Sub (Publish-Subscribe):
                              ┌─────────────┐
                         ┌───▶│ Subscriber1 │
┌───────────┐  ┌───────┐ │    └─────────────┘
│ Publisher │─▶│ Topic │─┤    ┌─────────────┐
└───────────┘  └───────┘ ├───▶│ Subscriber2 │ (All subscribers get message)
                         │    └─────────────┘
                         │    ┌─────────────┐
                         └───▶│ Subscriber3 │
                              └─────────────┘
```

| Aspect | Message Queue | Pub/Sub |
|--------|--------------|---------|
| Delivery | One consumer | All subscribers |
| Use Case | Task distribution | Event broadcasting |
| Coupling | Tighter | Looser |
| Ordering | Preserved | Per partition/subscriber |

---

### Q20: How do you handle API Versioning in Microservices?

**Answer:**

**Strategies:**

**1. URI Versioning:**
```
GET /api/v1/orders
GET /api/v2/orders
```

**2. Header Versioning:**
```
GET /api/orders
Accept: application/vnd.company.order.v1+json
```

**3. Query Parameter:**
```
GET /api/orders?version=1
```

**4. Content Negotiation:**
```
GET /api/orders
Content-Type: application/json; version=1
```

**Best Practices:**
```java
@RestController
@RequestMapping("/api")
public class OrderController {
    
    // Version 1
    @GetMapping(value = "/v1/orders/{id}")
    public OrderV1DTO getOrderV1(@PathVariable String id) {
        Order order = orderService.findById(id);
        return OrderV1Mapper.toDTO(order);
    }
    
    // Version 2 - with additional fields
    @GetMapping(value = "/v2/orders/{id}")
    public OrderV2DTO getOrderV2(@PathVariable String id) {
        Order order = orderService.findById(id);
        return OrderV2Mapper.toDTO(order);
    }
    
    // Header-based versioning
    @GetMapping(value = "/orders/{id}", headers = "X-API-Version=1")
    public OrderV1DTO getOrderByHeaderV1(@PathVariable String id) {
        return getOrderV1(id);
    }
}
```

**Semantic Versioning for APIs:**
- **Major (v1 → v2):** Breaking changes
- **Minor (v1.1 → v1.2):** New features, backward compatible
- **Patch (v1.1.1 → v1.1.2):** Bug fixes

---

## 4. Data Management

### Q21: What is the Database per Service Pattern?

**Answer:**
Each microservice has its own private database that only it can access.

```
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│ Order Service │  │ User Service  │  │Inventory Svc  │
└───────┬───────┘  └───────┬───────┘  └───────┬───────┘
        │                  │                  │
        ▼                  ▼                  ▼
   ┌─────────┐        ┌─────────┐        ┌─────────┐
   │Order DB │        │ User DB │        │Stock DB │
   │(Postgres)│       │ (MySQL) │        │(MongoDB)│
   └─────────┘        └─────────┘        └─────────┘
```

**Benefits:**
- Loose coupling
- Independent scaling
- Technology flexibility
- Failure isolation

**Challenges:**
- Distributed transactions
- Data consistency
- Cross-service queries

---

### Q22: How do you handle Distributed Transactions?

**Answer:**

**Approaches:**

**1. Saga Pattern** (Preferred)
- Sequence of local transactions
- Compensating transactions for rollback

**2. Two-Phase Commit (2PC)**
- Coordinator-based
- Strong consistency but blocking

**3. Eventual Consistency**
- Accept temporary inconsistency
- Reconciliation mechanisms

**Saga Implementation Example:**
```java
@Service
public class CreateOrderSaga {
    
    private final OrderService orderService;
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    
    public OrderResult execute(CreateOrderCommand command) {
        Order order = null;
        PaymentReservation payment = null;
        InventoryReservation inventory = null;
        
        try {
            // Step 1: Create pending order
            order = orderService.createPending(command);
            
            // Step 2: Reserve payment
            payment = paymentService.reserve(
                order.getId(), 
                command.getPaymentDetails(), 
                order.getTotal()
            );
            
            // Step 3: Reserve inventory
            inventory = inventoryService.reserve(
                order.getId(), 
                command.getItems()
            );
            
            // Step 4: Confirm order
            order = orderService.confirm(order.getId());
            paymentService.confirm(payment.getId());
            inventoryService.confirm(inventory.getId());
            
            return OrderResult.success(order);
            
        } catch (Exception e) {
            // Compensating transactions
            if (inventory != null) inventoryService.cancel(inventory.getId());
            if (payment != null) paymentService.cancel(payment.getId());
            if (order != null) orderService.cancel(order.getId());
            
            return OrderResult.failure(e.getMessage());
        }
    }
}
```

---

### Q23: How do you implement Cross-Service Queries?

**Answer:**

**Approaches:**

**1. API Composition:**
```java
@Service
public class OrderDetailsService {
    
    private final OrderServiceClient orderClient;
    private final UserServiceClient userClient;
    private final ProductServiceClient productClient;
    
    public OrderDetailsDTO getOrderDetails(String orderId) {
        // Parallel API calls
        CompletableFuture<Order> orderFuture = 
            CompletableFuture.supplyAsync(() -> orderClient.getOrder(orderId));
            
        Order order = orderFuture.join();
        
        CompletableFuture<User> userFuture = 
            CompletableFuture.supplyAsync(() -> userClient.getUser(order.getUserId()));
            
        CompletableFuture<List<Product>> productsFuture = 
            CompletableFuture.supplyAsync(() -> 
                productClient.getProducts(order.getProductIds()));
        
        User user = userFuture.join();
        List<Product> products = productsFuture.join();
        
        return new OrderDetailsDTO(order, user, products);
    }
}
```

**2. CQRS with Denormalized Read Model:**
```java
// Event handlers maintain denormalized view
@EventHandler
public class OrderDetailsProjection {
    
    private final OrderDetailsRepository repository;
    
    @EventHandler
    public void on(OrderCreatedEvent event) {
        OrderDetailsView view = new OrderDetailsView(event.getOrderId());
        view.setOrderDate(event.getCreatedAt());
        repository.save(view);
    }
    
    @EventHandler
    public void on(UserUpdatedEvent event) {
        // Update all orders for this user
        repository.updateUserDetails(
            event.getUserId(), 
            event.getUserName(), 
            event.getEmail()
        );
    }
}
```

**3. Event-Driven Data Replication:**
Services maintain local copies of needed data from other services.

---

### Q24: What is the Outbox Pattern?

**Answer:**
The Outbox Pattern ensures reliable event publishing with database transactions.

**Problem:** 
- Saving to database and publishing event are not atomic
- If publish fails after save, events are lost

**Solution:**
1. Write events to an "outbox" table in the same transaction
2. Separate process reads outbox and publishes events

```
┌─────────────────────────────────────────────┐
│              Order Service                  │
│  ┌────────────┐    ┌─────────────────────┐  │
│  │   Order    │    │   Outbox Table      │  │
│  │   Table    │    │  ┌───────────────┐  │  │
│  │            │    │  │ event_id      │  │  │
│  │            │    │  │ event_type    │  │  │
│  │            │    │  │ payload       │  │  │
│  │            │    │  │ created_at    │  │  │
│  └────────────┘    │  │ published     │  │  │
│                    │  └───────────────┘  │  │
└─────────────────────────────────────────────┘
         │                    │
         │  Same Transaction  │
         └────────────────────┘
                    │
            ┌───────▼───────┐
            │ Outbox Poller │
            │ / Debezium    │
            └───────┬───────┘
                    │
            ┌───────▼───────┐
            │    Kafka      │
            └───────────────┘
```

**Implementation:**
```java
@Service
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final OutboxRepository outboxRepository;
    
    @Transactional
    public Order createOrder(CreateOrderCommand command) {
        // Save order
        Order order = new Order(command);
        orderRepository.save(order);
        
        // Save event to outbox (same transaction)
        OutboxEvent event = new OutboxEvent(
            UUID.randomUUID(),
            "OrderCreated",
            JsonUtils.toJson(new OrderCreatedEvent(order)),
            Instant.now(),
            false
        );
        outboxRepository.save(event);
        
        return order;
    }
}

// Outbox Poller
@Scheduled(fixedDelay = 1000)
public void publishOutboxEvents() {
    List<OutboxEvent> events = outboxRepository.findUnpublished();
    
    for (OutboxEvent event : events) {
        try {
            kafkaTemplate.send("order-events", event.getPayload());
            event.setPublished(true);
            outboxRepository.save(event);
        } catch (Exception e) {
            log.error("Failed to publish event: {}", event.getId(), e);
        }
    }
}
```

---

### Q25: Explain the CAP Theorem and its relevance to Microservices

**Answer:**

**CAP Theorem states a distributed system can only guarantee 2 of 3:**
- **C**onsistency - All nodes see the same data
- **A**vailability - Every request gets a response
- **P**artition Tolerance - System works despite network partitions

```
           Consistency
              /\
             /  \
            /    \
           /  CP  \
          /________\
         /\        /\
        /  \  CA  /  \
       / AP \    /    \
      /______\  /______\
   Availability    Partition
                   Tolerance
```

**In Microservices:**
- Network partitions are inevitable
- Must choose between CP or AP

**CP Systems (Consistency + Partition Tolerance):**
- Choose consistency over availability
- Example: MongoDB (with write concern), Consul
- Use for: Financial transactions, inventory

**AP Systems (Availability + Partition Tolerance):**
- Choose availability over consistency
- Example: Cassandra, DynamoDB
- Use for: Social feeds, analytics

**Practical Approach - Eventual Consistency:**
```java
// Accept eventual consistency
@Service
public class InventoryService {
    
    // Optimistic update with conflict detection
    public void updateStock(String productId, int quantity, long version) {
        Product product = repository.findById(productId);
        
        if (product.getVersion() != version) {
            throw new OptimisticLockingException("Data was modified");
        }
        
        product.setQuantity(quantity);
        product.setVersion(version + 1);
        repository.save(product);
        
        // Publish event for other services
        eventPublisher.publish(new StockUpdatedEvent(productId, quantity));
    }
}
```

---

## 5. Service Discovery & Registry

### Q26: What is Service Discovery? Explain different patterns.

**Answer:**

Service Discovery enables services to find and communicate with each other dynamically.

**Types:**

**1. Client-Side Discovery:**
```
┌────────────────────────────────────────────────────┐
│                     Client                         │
│  1. Query Registry  ┌──────────────────────────┐   │
│  ─────────────────▶ │   Service Registry       │   │
│                     │   (Eureka/Consul)        │   │
│  2. Get Instances   └──────────────────────────┘   │
│  ◀─────────────────                                │
│                                                    │
│  3. Load Balance & Call                            │
│  ─────────────────────────────────────────────▶    │
│                        ┌─────────┐                 │
│                        │ Service │                 │
│                        │Instance │                 │
│                        └─────────┘                 │
└────────────────────────────────────────────────────┘
```

**2. Server-Side Discovery:**
```
┌──────────┐     ┌──────────────┐     ┌──────────────┐
│  Client  │────▶│ Load Balancer│────▶│   Service    │
└──────────┘     │   (queries   │     │   Instance   │
                 │   registry)  │     └──────────────┘
                 └──────────────┘
                        │
                        ▼
                 ┌──────────────┐
                 │   Registry   │
                 └──────────────┘
```

**Popular Tools:**
- **Eureka** (Netflix) - Client-side
- **Consul** (HashiCorp) - Both patterns
- **Kubernetes** - Server-side (DNS + kube-proxy)
- **Zookeeper** - Coordination service

---

### Q27: How does Eureka Service Discovery work?

**Answer:**

**Components:**
- **Eureka Server** - Registry that maintains service instances
- **Eureka Client** - Services that register and discover

**Server Configuration:**
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
# application.yml - Eureka Server
server:
  port: 8761

eureka:
  client:
    registerWithEureka: false
    fetchRegistry: false
  server:
    enableSelfPreservation: true
    evictionIntervalTimerInMs: 60000
```

**Client Configuration:**
```java
@SpringBootApplication
@EnableDiscoveryClient
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

```yaml
# application.yml - Eureka Client
spring:
  application:
    name: order-service

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  instance:
    preferIpAddress: true
    leaseRenewalIntervalInSeconds: 30
    leaseExpirationDurationInSeconds: 90
```

**Using Discovery Client:**
```java
@Service
public class OrderService {
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    @Autowired
    @LoadBalanced
    private RestTemplate restTemplate;
    
    // Manual discovery
    public List<ServiceInstance> getPaymentServiceInstances() {
        return discoveryClient.getInstances("payment-service");
    }
    
    // Using load-balanced RestTemplate
    public PaymentResponse processPayment(PaymentRequest request) {
        return restTemplate.postForObject(
            "http://payment-service/api/payments",
            request,
            PaymentResponse.class
        );
    }
}

@Configuration
public class RestTemplateConfig {
    
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

---

### Q28: Compare Eureka vs Consul vs Kubernetes Service Discovery

**Answer:**

| Feature | Eureka | Consul | Kubernetes |
|---------|--------|--------|------------|
| **Type** | Client-side | Both | Server-side |
| **Health Check** | Client heartbeat | Agent-based, flexible | Liveness/Readiness probes |
| **Consistency** | AP (eventual) | CP by default | Eventual (DNS) |
| **Key-Value Store** | No | Yes | ConfigMaps/Secrets |
| **Multi-DC** | Zone awareness | Native | Federation |
| **Language** | Java-focused | Polyglot | Any |
| **DNS Support** | No | Yes | Native |

**Kubernetes Service Discovery:**
```yaml
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

---
# Other services can access via:
# http://order-service (same namespace)
# http://order-service.orders-namespace.svc.cluster.local (cross-namespace)
```

---

## 6. API Gateway

### Q29: What are the responsibilities of an API Gateway?

**Answer:**

**Core Responsibilities:**

1. **Routing**
   - Route requests to appropriate services
   - Path-based, header-based routing

2. **Authentication & Authorization**
   - JWT validation
   - OAuth2 integration
   - API key management

3. **Rate Limiting & Throttling**
   - Prevent abuse
   - Fair usage policies

4. **Load Balancing**
   - Distribute traffic across instances

5. **Request/Response Transformation**
   - Protocol translation
   - Payload transformation

6. **Caching**
   - Response caching
   - Reduce backend load

7. **Circuit Breaking**
   - Prevent cascade failures

8. **Monitoring & Logging**
   - Request tracing
   - Metrics collection

---

### Q30: Implement Rate Limiting in Spring Cloud Gateway

**Answer:**

```java
@Configuration
public class RateLimiterConfig {
    
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest()
                .getHeaders()
                .getFirst("X-User-Id") != null ?
                exchange.getRequest().getHeaders().getFirst("X-User-Id") :
                exchange.getRequest().getRemoteAddress().getAddress().getHostAddress()
        );
    }
    
    @Bean
    public RedisRateLimiter redisRateLimiter() {
        return new RedisRateLimiter(10, 20); // 10 req/sec, burst of 20
    }
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder,
                                            RedisRateLimiter rateLimiter,
                                            KeyResolver userKeyResolver) {
        return builder.routes()
            .route("order-service-rate-limited", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    .requestRateLimiter(c -> c
                        .setRateLimiter(rateLimiter)
                        .setKeyResolver(userKeyResolver)
                        .setStatusCode(HttpStatus.TOO_MANY_REQUESTS))
                    .retry(config -> config
                        .setRetries(3)
                        .setStatuses(HttpStatus.SERVICE_UNAVAILABLE)))
                .uri("lb://order-service"))
            .build();
    }
}
```

**YAML Configuration:**
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
                key-resolver: "#{@userKeyResolver}"
            - name: CircuitBreaker
              args:
                name: orderServiceCB
                fallbackUri: forward:/fallback/orders
```

---

## 7. Security

### Q31: How do you implement Authentication in Microservices?

**Answer:**

**Common Patterns:**

**1. API Gateway Authentication (Recommended):**
```
┌──────────┐     ┌─────────────┐     ┌───────────┐
│  Client  │────▶│ API Gateway │────▶│  Service  │
│          │     │ (validates  │     │ (trusts   │
│          │     │  JWT token) │     │  gateway) │
└──────────┘     └─────────────┘     └───────────┘
                        │
                        ▼
                 ┌─────────────┐
                 │ Auth Server │
                 │ (Keycloak)  │
                 └─────────────┘
```

**2. Token-Based Authentication (JWT):**
```java
// Auth Service - Token Generation
@Service
public class AuthService {
    
    @Value("${jwt.secret}")
    private String jwtSecret;
    
    public String generateToken(User user) {
        return Jwts.builder()
            .setSubject(user.getId())
            .claim("roles", user.getRoles())
            .claim("email", user.getEmail())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + 3600000))
            .signWith(SignatureAlgorithm.HS256, jwtSecret)
            .compact();
    }
}

// Gateway - JWT Filter
@Component
public class JwtAuthenticationFilter implements GatewayFilter {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = extractToken(exchange.getRequest());
        
        if (token == null) {
            return unauthorized(exchange);
        }
        
        try {
            Claims claims = Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token)
                .getBody();
            
            // Add user info to headers for downstream services
            ServerHttpRequest request = exchange.getRequest().mutate()
                .header("X-User-Id", claims.getSubject())
                .header("X-User-Roles", claims.get("roles").toString())
                .build();
                
            return chain.filter(exchange.mutate().request(request).build());
            
        } catch (JwtException e) {
            return unauthorized(exchange);
        }
    }
}
```

**3. Service-to-Service Authentication (mTLS):**
```yaml
# Kubernetes mTLS with Istio
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
```

---

### Q32: Explain OAuth2 and OpenID Connect for Microservices

**Answer:**

**OAuth2 Flows for Microservices:**

**1. Authorization Code Flow (Web Apps):**
```
┌──────┐                              ┌─────────┐
│Client│                              │Auth Svr │
└──┬───┘                              └────┬────┘
   │ 1. Redirect to /authorize             │
   │─────────────────────────────────────▶│
   │                                       │
   │ 2. User authenticates                 │
   │                                       │
   │ 3. Redirect with authorization code   │
   │◀─────────────────────────────────────│
   │                                       │
   │ 4. Exchange code for tokens           │
   │─────────────────────────────────────▶│
   │                                       │
   │ 5. Access Token + Refresh Token       │
   │◀─────────────────────────────────────│
```

**2. Client Credentials Flow (Service-to-Service):**
```java
@Service
public class PaymentServiceClient {
    
    private final WebClient webClient;
    private final OAuth2AuthorizedClientManager clientManager;
    
    public PaymentServiceClient(WebClient.Builder builder,
                                OAuth2AuthorizedClientManager clientManager) {
        this.webClient = builder
            .baseUrl("http://payment-service")
            .filter(new ServletOAuth2AuthorizedClientExchangeFilterFunction(clientManager))
            .build();
        this.clientManager = clientManager;
    }
    
    public Mono<PaymentResponse> processPayment(PaymentRequest request) {
        return webClient.post()
            .uri("/api/payments")
            .attributes(clientRegistrationId("payment-service-client"))
            .bodyValue(request)
            .retrieve()
            .bodyToMono(PaymentResponse.class);
    }
}
```

```yaml
# application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          payment-service-client:
            client-id: order-service
            client-secret: ${CLIENT_SECRET}
            authorization-grant-type: client_credentials
            scope: payment:write
        provider:
          payment-service-client:
            token-uri: http://auth-server/oauth2/token
```

**OpenID Connect (OIDC):**
- Extension of OAuth2 for authentication
- Provides ID Token (identity information)
- Standard claims: sub, name, email, etc.

```java
// Resource Server Configuration
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated())
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .jwtAuthenticationConverter(jwtAuthConverter())));
        return http.build();
    }
    
    @Bean
    public JwtAuthenticationConverter jwtAuthConverter() {
        JwtGrantedAuthoritiesConverter converter = new JwtGrantedAuthoritiesConverter();
        converter.setAuthoritiesClaimName("roles");
        converter.setAuthorityPrefix("ROLE_");
        
        JwtAuthenticationConverter authConverter = new JwtAuthenticationConverter();
        authConverter.setJwtGrantedAuthoritiesConverter(converter);
        return authConverter;
    }
}
```

---

### Q33: How do you secure inter-service communication?

**Answer:**

**1. Mutual TLS (mTLS):**
Both client and server authenticate each other with certificates.

```java
// Spring Boot with mTLS
@Configuration
public class TlsConfig {
    
    @Bean
    public RestTemplate restTemplate() throws Exception {
        SSLContext sslContext = SSLContextBuilder.create()
            .loadKeyMaterial(
                new ClassPathResource("client.p12").getURL(),
                "password".toCharArray(),
                "password".toCharArray())
            .loadTrustMaterial(
                new ClassPathResource("truststore.jks").getURL(),
                "password".toCharArray())
            .build();
        
        HttpClient httpClient = HttpClients.custom()
            .setSSLContext(sslContext)
            .build();
        
        HttpComponentsClientHttpRequestFactory factory = 
            new HttpComponentsClientHttpRequestFactory(httpClient);
        
        return new RestTemplate(factory);
    }
}
```

**2. Service Mesh (Istio):**
```yaml
# Automatic mTLS with Istio
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT

---
# Authorization Policy
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: order-service-policy
spec:
  selector:
    matchLabels:
      app: order-service
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/default/sa/api-gateway"]
      to:
        - operation:
            methods: ["GET", "POST"]
            paths: ["/api/orders/*"]
```

**3. JWT Propagation:**
```java
@Component
public class JwtPropagatingInterceptor implements ClientHttpRequestInterceptor {
    
    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body,
                                        ClientHttpRequestExecution execution) throws IOException {
        // Get JWT from security context
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth instanceof JwtAuthenticationToken) {
            String token = ((JwtAuthenticationToken) auth).getToken().getTokenValue();
            request.getHeaders().setBearerAuth(token);
        }
        return execution.execute(request, body);
    }
}
```

---

## 8. Resilience & Fault Tolerance

### Q34: What is the Circuit Breaker Pattern? Explain its states.

**Answer:**

Circuit Breaker prevents cascade failures by stopping requests to failing services.

**States:**

```
         Success                    Timeout
    ┌───────────────┐          ┌───────────────┐
    │               │          │               │
    ▼               │          ▼               │
┌───────┐      ┌────┴────┐      ┌──────────┐
│CLOSED │─────▶│  OPEN   │─────▶│HALF-OPEN │
└───────┘      └─────────┘      └──────────┘
   │  ▲          Failures         │      │
   │  │          threshold        │      │
   │  │                           │      │
   │  └───────────────────────────┘      │
   │         Success                     │
   │                                     │
   └─────────────────────────────────────┘
             Failures continue
```

**States Explained:**
- **CLOSED:** Normal operation, requests flow through
- **OPEN:** Requests fail immediately, no calls to service
- **HALF-OPEN:** Limited requests allowed to test recovery

**Resilience4j Implementation:**
```java
@Configuration
public class CircuitBreakerConfig {
    
    @Bean
    public CircuitBreaker paymentServiceCircuitBreaker(CircuitBreakerRegistry registry) {
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(50)                    // Open if 50% failures
            .waitDurationInOpenState(Duration.ofSeconds(30))
            .slidingWindowType(SlidingWindowType.COUNT_BASED)
            .slidingWindowSize(10)                       // Based on last 10 calls
            .minimumNumberOfCalls(5)                     // Min calls before evaluating
            .permittedNumberOfCallsInHalfOpenState(3)    // Calls in half-open
            .automaticTransitionFromOpenToHalfOpenEnabled(true)
            .build();
        
        return registry.circuitBreaker("paymentService", config);
    }
}

@Service
public class PaymentService {
    
    private final CircuitBreaker circuitBreaker;
    private final PaymentClient paymentClient;
    
    public PaymentResponse processPayment(PaymentRequest request) {
        return circuitBreaker.executeSupplier(() -> 
            paymentClient.process(request)
        );
    }
    
    // With fallback
    public PaymentResponse processPaymentWithFallback(PaymentRequest request) {
        return Try.ofSupplier(
            CircuitBreaker.decorateSupplier(circuitBreaker, 
                () -> paymentClient.process(request)))
            .recover(throwable -> PaymentResponse.pending(request.getId()))
            .get();
    }
}
```

**Annotation-Based:**
```java
@Service
public class PaymentService {
    
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService")
    @TimeLimiter(name = "paymentService")
    public CompletableFuture<PaymentResponse> processPayment(PaymentRequest request) {
        return CompletableFuture.supplyAsync(() -> paymentClient.process(request));
    }
    
    public CompletableFuture<PaymentResponse> paymentFallback(PaymentRequest request, 
                                                               Throwable t) {
        log.warn("Payment service unavailable, returning pending status", t);
        return CompletableFuture.completedFuture(
            PaymentResponse.pending(request.getId()));
    }
}
```

---

### Q35: Explain Bulkhead Pattern

**Answer:**

Bulkhead pattern isolates resources to prevent cascade failures (like compartments in a ship).

**Types:**

**1. Thread Pool Bulkhead:**
```java
@Configuration
public class BulkheadConfig {
    
    @Bean
    public ThreadPoolBulkhead paymentBulkhead(ThreadPoolBulkheadRegistry registry) {
        ThreadPoolBulkheadConfig config = ThreadPoolBulkheadConfig.custom()
            .maxThreadPoolSize(10)
            .coreThreadPoolSize(5)
            .queueCapacity(20)
            .keepAliveDuration(Duration.ofSeconds(20))
            .build();
        
        return registry.bulkhead("paymentService", config);
    }
}

@Service
public class PaymentService {
    
    @Bulkhead(name = "paymentService", type = Bulkhead.Type.THREADPOOL)
    public CompletableFuture<PaymentResponse> processPayment(PaymentRequest request) {
        return CompletableFuture.supplyAsync(() -> paymentClient.process(request));
    }
}
```

**2. Semaphore Bulkhead:**
```java
@Bean
public Bulkhead inventoryBulkhead(BulkheadRegistry registry) {
    BulkheadConfig config = BulkheadConfig.custom()
        .maxConcurrentCalls(25)
        .maxWaitDuration(Duration.ofMillis(500))
        .build();
    
    return registry.bulkhead("inventoryService", config);
}
```

**YAML Configuration:**
```yaml
resilience4j:
  bulkhead:
    instances:
      paymentService:
        maxConcurrentCalls: 25
        maxWaitDuration: 500ms
  thread-pool-bulkhead:
    instances:
      inventoryService:
        maxThreadPoolSize: 10
        coreThreadPoolSize: 5
        queueCapacity: 20
```

---

### Q36: What is the Retry Pattern? How to implement exponential backoff?

**Answer:**

```java
@Configuration
public class RetryConfig {
    
    @Bean
    public Retry paymentRetry(RetryRegistry registry) {
        RetryConfig config = RetryConfig.custom()
            .maxAttempts(3)
            .waitDuration(Duration.ofMillis(500))
            .retryOnException(e -> e instanceof ServiceUnavailableException)
            .retryExceptions(IOException.class, TimeoutException.class)
            .ignoreExceptions(BusinessException.class)
            .build();
        
        return registry.retry("paymentService", config);
    }
    
    // With exponential backoff
    @Bean
    public Retry paymentRetryExponential(RetryRegistry registry) {
        RetryConfig config = RetryConfig.custom()
            .maxAttempts(4)
            .intervalFunction(IntervalFunction.ofExponentialBackoff(
                Duration.ofMillis(500),  // Initial interval
                2.0,                      // Multiplier
                Duration.ofSeconds(10)    // Max interval
            ))
            .build();
        
        return registry.retry("paymentServiceExp", config);
    }
    
    // With jitter (randomization)
    @Bean
    public Retry paymentRetryJitter(RetryRegistry registry) {
        RetryConfig config = RetryConfig.custom()
            .maxAttempts(4)
            .intervalFunction(IntervalFunction.ofExponentialRandomBackoff(
                Duration.ofMillis(500),
                2.0,
                0.5  // Randomization factor
            ))
            .build();
        
        return registry.retry("paymentServiceJitter", config);
    }
}
```

**Annotation-Based with YAML:**
```yaml
resilience4j:
  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 500ms
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
        retryExceptions:
          - java.io.IOException
          - java.util.concurrent.TimeoutException
        ignoreExceptions:
          - com.example.BusinessException
```

---

### Q37: Explain Timeout and Rate Limiter patterns

**Answer:**

**Timeout Pattern:**
```java
@Service
public class PaymentService {
    
    @TimeLimiter(name = "paymentService")
    @CircuitBreaker(name = "paymentService", fallbackMethod = "fallback")
    public CompletableFuture<PaymentResponse> processPayment(PaymentRequest request) {
        return CompletableFuture.supplyAsync(() -> {
            return paymentClient.process(request); // May take long time
        });
    }
}
```

```yaml
resilience4j:
  timelimiter:
    instances:
      paymentService:
        timeoutDuration: 2s
        cancelRunningFuture: true
```

**Rate Limiter Pattern:**
```java
@Configuration
public class RateLimiterConfig {
    
    @Bean
    public RateLimiter apiRateLimiter(RateLimiterRegistry registry) {
        RateLimiterConfig config = RateLimiterConfig.custom()
            .limitForPeriod(100)                    // 100 requests
            .limitRefreshPeriod(Duration.ofSeconds(1)) // per second
            .timeoutDuration(Duration.ofMillis(500))   // wait time
            .build();
        
        return registry.rateLimiter("apiService", config);
    }
}

@RestController
public class OrderController {
    
    @RateLimiter(name = "apiService", fallbackMethod = "rateLimitFallback")
    @GetMapping("/api/orders")
    public List<Order> getOrders() {
        return orderService.findAll();
    }
    
    public List<Order> rateLimitFallback(Throwable t) {
        throw new TooManyRequestsException("Rate limit exceeded. Try again later.");
    }
}
```

---

## 9. Monitoring, Logging & Observability

### Q38: What is Distributed Tracing? How does it work?

**Answer:**

Distributed Tracing tracks requests across multiple services.

**Key Concepts:**
- **Trace:** End-to-end journey of a request
- **Span:** Single unit of work within a trace
- **Trace Context:** Propagated headers (trace-id, span-id)

```
Trace ID: abc123
│
├─ Span 1: API Gateway (span-id: s1, parent: none)
│  ├─ Span 2: Auth Service (span-id: s2, parent: s1)
│  └─ Span 3: Order Service (span-id: s3, parent: s1)
│     ├─ Span 4: Inventory Service (span-id: s4, parent: s3)
│     └─ Span 5: Payment Service (span-id: s5, parent: s3)
```

**Implementation with Spring Cloud Sleuth & Zipkin:**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

```yaml
spring:
  sleuth:
    sampler:
      probability: 1.0  # Sample 100% of traces
  zipkin:
    base-url: http://zipkin:9411
    sender:
      type: web
```

**OpenTelemetry Implementation:**
```java
@Configuration
public class TracingConfig {
    
    @Bean
    public OpenTelemetry openTelemetry() {
        Resource resource = Resource.getDefault()
            .merge(Resource.create(Attributes.of(
                ResourceAttributes.SERVICE_NAME, "order-service")));
        
        SdkTracerProvider tracerProvider = SdkTracerProvider.builder()
            .addSpanProcessor(BatchSpanProcessor.builder(
                OtlpGrpcSpanExporter.builder()
                    .setEndpoint("http://jaeger:4317")
                    .build())
                .build())
            .setResource(resource)
            .build();
        
        return OpenTelemetrySdk.builder()
            .setTracerProvider(tracerProvider)
            .setPropagators(ContextPropagators.create(
                W3CTraceContextPropagator.getInstance()))
            .buildAndRegisterGlobal();
    }
}

@Service
public class OrderService {
    
    private final Tracer tracer;
    
    public Order createOrder(CreateOrderCommand command) {
        Span span = tracer.spanBuilder("createOrder")
            .setAttribute("customerId", command.getCustomerId())
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            // Business logic
            Order order = processOrder(command);
            span.setAttribute("orderId", order.getId());
            return order;
        } catch (Exception e) {
            span.recordException(e);
            span.setStatus(StatusCode.ERROR, e.getMessage());
            throw e;
        } finally {
            span.end();
        }
    }
}
```

---

### Q39: How do you implement Centralized Logging in Microservices?

**Answer:**

**ELK Stack (Elasticsearch, Logstash, Kibana):**

```
┌───────────┐  ┌───────────┐  ┌───────────┐
│ Service 1 │  │ Service 2 │  │ Service 3 │
└─────┬─────┘  └─────┬─────┘  └─────┬─────┘
      │              │              │
      └──────────────┼──────────────┘
                     │ (Logs)
              ┌──────▼──────┐
              │  Logstash   │
              │  or Fluentd │
              └──────┬──────┘
                     │
              ┌──────▼──────┐
              │Elasticsearch│
              └──────┬──────┘
                     │
              ┌──────▼──────┐
              │   Kibana    │
              └─────────────┘
```

**Structured Logging with JSON:**
```java
// logback-spring.xml
<configuration>
    <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeMdcKeyName>traceId</includeMdcKeyName>
            <includeMdcKeyName>spanId</includeMdcKeyName>
            <includeMdcKeyName>userId</includeMdcKeyName>
            <customFields>{"service":"order-service","env":"prod"}</customFields>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="JSON"/>
    </root>
</configuration>
```

**Adding Context to Logs:**
```java
@Component
public class LoggingFilter implements WebFilter {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        String requestId = exchange.getRequest()
            .getHeaders()
            .getFirst("X-Request-Id");
        
        if (requestId == null) {
            requestId = UUID.randomUUID().toString();
        }
        
        MDC.put("requestId", requestId);
        MDC.put("userId", extractUserId(exchange));
        
        return chain.filter(exchange)
            .doFinally(signal -> MDC.clear());
    }
}

@Service
public class OrderService {
    
    private static final Logger log = LoggerFactory.getLogger(OrderService.class);
    
    public Order createOrder(CreateOrderCommand command) {
        log.info("Creating order for customer: {}", command.getCustomerId());
        
        try {
            Order order = processOrder(command);
            log.info("Order created successfully: {}", order.getId());
            return order;
        } catch (Exception e) {
            log.error("Failed to create order: {}", e.getMessage(), e);
            throw e;
        }
    }
}
```

**Output:**
```json
{
  "@timestamp": "2024-01-15T10:30:45.123Z",
  "level": "INFO",
  "message": "Order created successfully: ORD-123",
  "service": "order-service",
  "traceId": "abc123def456",
  "spanId": "789xyz",
  "requestId": "req-001",
  "userId": "user-42",
  "logger": "com.example.OrderService"
}
```

---

### Q40: How do you implement Health Checks in Microservices?

**Answer:**

**Spring Boot Actuator:**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: always
      probes:
        enabled: true
  health:
    livenessstate:
      enabled: true
    readinessstate:
      enabled: true
```

**Custom Health Indicators:**
```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    
    private final DataSource dataSource;
    
    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            if (conn.isValid(1)) {
                return Health.up()
                    .withDetail("database", "PostgreSQL")
                    .withDetail("connection", "valid")
                    .build();
            }
        } catch (SQLException e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
        return Health.down().build();
    }
}

@Component
public class ExternalServiceHealthIndicator implements HealthIndicator {
    
    private final WebClient webClient;
    
    @Override
    public Health health() {
        try {
            ResponseEntity<String> response = webClient.get()
                .uri("/health")
                .retrieve()
                .toEntity(String.class)
                .block(Duration.ofSeconds(5));
            
            if (response.getStatusCode().is2xxSuccessful()) {
                return Health.up()
                    .withDetail("paymentService", "available")
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("paymentService", "unavailable")
                .withDetail("error", e.getMessage())
                .build();
        }
        return Health.unknown().build();
    }
}
```

**Kubernetes Probes:**
```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: order-service
      image: order-service:latest
      livenessProbe:
        httpGet:
          path: /actuator/health/liveness
          port: 8080
        initialDelaySeconds: 30
        periodSeconds: 10
        failureThreshold: 3
      readinessProbe:
        httpGet:
          path: /actuator/health/readiness
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 3
      startupProbe:
        httpGet:
          path: /actuator/health
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 10
        failureThreshold: 30
```

---

### Q41: Explain the difference between Metrics, Logs, and Traces

**Answer:**

| Aspect | Metrics | Logs | Traces |
|--------|---------|------|--------|
| **Purpose** | Measure system behavior | Record events | Track request flow |
| **Format** | Numeric (counters, gauges) | Text/JSON | Spans with context |
| **Cardinality** | Low | High | Medium |
| **Query** | Time-series aggregations | Full-text search | Trace ID lookup |
| **Tools** | Prometheus, Graphite | ELK, Loki | Jaeger, Zipkin |
| **Use Case** | Alerting, dashboards | Debugging, audit | Performance analysis |

**Implementation Example:**

```java
@RestController
@RequiredArgsConstructor
public class OrderController {
    
    private final OrderService orderService;
    private final MeterRegistry meterRegistry;
    private final Tracer tracer;
    private static final Logger log = LoggerFactory.getLogger(OrderController.class);
    
    @PostMapping("/api/orders")
    public ResponseEntity<Order> createOrder(@RequestBody CreateOrderRequest request) {
        // Start span (Trace)
        Span span = tracer.spanBuilder("createOrder").startSpan();
        
        // Increment counter (Metric)
        meterRegistry.counter("orders.created.total").increment();
        
        Timer.Sample sample = Timer.start(meterRegistry);
        
        try (Scope scope = span.makeCurrent()) {
            // Log event
            log.info("Received order request for customer: {}", request.getCustomerId());
            
            Order order = orderService.create(request);
            
            // Record success metrics
            sample.stop(meterRegistry.timer("orders.create.duration", 
                "status", "success"));
            
            // Log success
            log.info("Order created: {}", order.getId());
            
            span.setAttribute("orderId", order.getId());
            return ResponseEntity.ok(order);
            
        } catch (Exception e) {
            // Record failure metrics
            meterRegistry.counter("orders.created.errors").increment();
            sample.stop(meterRegistry.timer("orders.create.duration", 
                "status", "error"));
            
            // Log error
            log.error("Failed to create order: {}", e.getMessage(), e);
            
            // Record exception in trace
            span.recordException(e);
            span.setStatus(StatusCode.ERROR);
            
            throw e;
        } finally {
            span.end();
        }
    }
}
```

---

## 10. Containerization & Orchestration

### Q42: How do you containerize a Microservice with Docker?

**Answer:**

**Multi-Stage Dockerfile (Java):**
```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Security: Non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Copy artifact
COPY --from=build /app/target/*.jar app.jar

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# JVM tuning for containers
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

EXPOSE 8080
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

**Docker Compose for Local Development:**
```yaml
version: '3.8'

services:
  order-service:
    build: ./order-service
    ports:
      - "8081:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - DATABASE_URL=jdbc:postgresql://postgres:5432/orders
      - KAFKA_BOOTSTRAP_SERVERS=kafka:9092
    depends_on:
      postgres:
        condition: service_healthy
      kafka:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - microservices-network
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  payment-service:
    build: ./payment-service
    ports:
      - "8082:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
    networks:
      - microservices-network

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=orders
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=secret
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d orders"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - microservices-network

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
    depends_on:
      - zookeeper
    networks:
      - microservices-network

networks:
  microservices-network:
    driver: bridge

volumes:
  postgres-data:
```

---

### Q43: Explain Kubernetes basics for Microservices deployment

**Answer:**

**Key Kubernetes Objects:**

**1. Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
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
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: url
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
```

**2. Service:**
```yaml
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

**3. ConfigMap & Secret:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: order-service-config
data:
  application.yml: |
    server:
      port: 8080
    spring:
      profiles:
        active: kubernetes

---
apiVersion: v1
kind: Secret
metadata:
  name: db-secrets
type: Opaque
data:
  url: cG9zdGdyZXNxbDovL3VzZXI6cGFzc0Bwb3N0Z3Jlczo1NDMyL29yZGVycw==
  password: c2VjcmV0cGFzc3dvcmQ=
```

**4. Horizontal Pod Autoscaler:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

**5. Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: microservices-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /orders
            pathType: Prefix
            backend:
              service:
                name: order-service
                port:
                  number: 80
          - path: /users
            pathType: Prefix
            backend:
              service:
                name: user-service
                port:
                  number: 80
```

---

### Q44: What is a Service Mesh? Explain Istio.

**Answer:**

A Service Mesh is a dedicated infrastructure layer that handles service-to-service communication.

**Features:**
- Traffic management
- Security (mTLS)
- Observability
- Policy enforcement

**Istio Architecture:**
```
┌────────────────────────────────────────────────────────┐
│                     Control Plane                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │  Pilot   │  │ Citadel  │  │  Galley  │             │
│  │(traffic) │  │(security)│  │ (config) │             │
│  └──────────┘  └──────────┘  └──────────┘             │
└────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────┼─────────────────────────────┐
│                   Data Plane                          │
│  ┌─────────────────┐         ┌─────────────────┐      │
│  │ Pod             │         │ Pod             │      │
│  │ ┌─────────────┐ │         │ ┌─────────────┐ │      │
│  │ │   Service   │ │         │ │   Service   │ │      │
│  │ └─────────────┘ │         │ └─────────────┘ │      │
│  │ ┌─────────────┐ │ ──────▶ │ ┌─────────────┐ │      │
│  │ │Envoy Sidecar│ │         │ │Envoy Sidecar│ │      │
│  │ └─────────────┘ │         │ └─────────────┘ │      │
│  └─────────────────┘         └─────────────────┘      │
└───────────────────────────────────────────────────────┘
```

**Istio Configuration Examples:**

**Traffic Management - Canary Deployment:**
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
    - order-service
  http:
    - match:
        - headers:
            canary:
              exact: "true"
      route:
        - destination:
            host: order-service
            subset: v2
    - route:
        - destination:
            host: order-service
            subset: v1
          weight: 90
        - destination:
            host: order-service
            subset: v2
          weight: 10

---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: order-service
spec:
  host: order-service
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

**Circuit Breaker:**
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: payment-service
spec:
  host: payment-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        h2UpgradePolicy: UPGRADE
        http1MaxPendingRequests: 100
        http2MaxRequests: 1000
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 60s
      maxEjectionPercent: 50
```

---

## 11. Testing Strategies

### Q45: Explain the Testing Pyramid for Microservices

**Answer:**

```
                    /\
                   /  \
                  / E2E\        <- Few, Slow, Expensive
                 /──────\
                /        \
               /Integration\    <- Some, Medium
              /──────────────\
             /                \
            /   Unit Tests     \  <- Many, Fast, Cheap
           /────────────────────\
```

**Testing Types:**

**1. Unit Tests:**
```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    
    @Mock
    private OrderRepository orderRepository;
    
    @Mock
    private EventPublisher eventPublisher;
    
    @InjectMocks
    private OrderService orderService;
    
    @Test
    void shouldCreateOrder() {
        // Given
        CreateOrderCommand command = new CreateOrderCommand(
            "customer-1", 
            List.of(new OrderItem("product-1", 2, Money.of(50)))
        );
        
        when(orderRepository.save(any(Order.class)))
            .thenAnswer(inv -> {
                Order order = inv.getArgument(0);
                order.setId("order-123");
                return order;
            });
        
        // When
        Order order = orderService.create(command);
        
        // Then
        assertThat(order.getId()).isEqualTo("order-123");
        assertThat(order.getTotal()).isEqualTo(Money.of(100));
        verify(eventPublisher).publish(any(OrderCreatedEvent.class));
    }
}
```

**2. Integration Tests:**
```java
@SpringBootTest
@Testcontainers
class OrderRepositoryIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("orders")
        .withUsername("test")
        .withPassword("test");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Test
    void shouldPersistAndRetrieveOrder() {
        // Given
        Order order = new Order("customer-1", List.of(
            new OrderItem("product-1", 2, Money.of(50))));
        
        // When
        Order saved = orderRepository.save(order);
        Order found = orderRepository.findById(saved.getId()).orElseThrow();
        
        // Then
        assertThat(found.getCustomerId()).isEqualTo("customer-1");
        assertThat(found.getItems()).hasSize(1);
    }
}
```

**3. Contract Tests (Consumer-Driven):**
```java
// Consumer side (Order Service)
@SpringBootTest
@AutoConfigureStubRunner(
    stubsMode = StubRunnerProperties.StubsMode.LOCAL,
    ids = "com.example:payment-service:+:stubs:8085")
class PaymentServiceContractTest {
    
    @Autowired
    private PaymentClient paymentClient;
    
    @Test
    void shouldProcessPayment() {
        // Given
        PaymentRequest request = new PaymentRequest("order-123", Money.of(100));
        
        // When - calls stub generated from contract
        PaymentResponse response = paymentClient.process(request);
        
        // Then
        assertThat(response.getStatus()).isEqualTo("APPROVED");
    }
}

// Provider side (Payment Service) - Pact
@Provider("payment-service")
@PactFolder("pacts")
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class PaymentServiceProviderTest {
    
    @LocalServerPort
    private int port;
    
    @BeforeEach
    void setup(PactVerificationContext context) {
        context.setTarget(new HttpTestTarget("localhost", port));
    }
    
    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void verifyPact(PactVerificationContext context) {
        context.verifyInteraction();
    }
    
    @State("payment can be processed")
    void setupPaymentState() {
        // Setup test data
    }
}
```

**4. Component Tests:**
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class OrderServiceComponentTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");
    
    @Container
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.5.0"));
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @MockBean
    private PaymentClient paymentClient; // Mock external service
    
    @Test
    void shouldCreateOrderEndToEnd() {
        // Given
        when(paymentClient.process(any()))
            .thenReturn(new PaymentResponse("APPROVED"));
        
        CreateOrderRequest request = new CreateOrderRequest(
            "customer-1",
            List.of(new OrderItemRequest("product-1", 2))
        );
        
        // When
        ResponseEntity<OrderResponse> response = restTemplate.postForEntity(
            "/api/orders",
            request,
            OrderResponse.class
        );
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody().getStatus()).isEqualTo("CONFIRMED");
    }
}
```

**5. End-to-End Tests:**
```java
@SpringBootTest
@Testcontainers
class OrderFlowE2ETest {
    
    @Container
    static DockerComposeContainer<?> environment = 
        new DockerComposeContainer<>(new File("docker-compose-test.yml"))
            .withExposedService("order-service", 8080)
            .withExposedService("payment-service", 8081)
            .withExposedService("inventory-service", 8082)
            .waitingFor("order-service", Wait.forHealthcheck());
    
    private WebClient webClient;
    
    @BeforeEach
    void setup() {
        String orderServiceUrl = String.format("http://%s:%d",
            environment.getServiceHost("order-service", 8080),
            environment.getServicePort("order-service", 8080));
        webClient = WebClient.create(orderServiceUrl);
    }
    
    @Test
    void shouldCompleteOrderFlow() {
        // Create order
        OrderResponse order = webClient.post()
            .uri("/api/orders")
            .bodyValue(new CreateOrderRequest("customer-1", items))
            .retrieve()
            .bodyToMono(OrderResponse.class)
            .block();
        
        // Wait for async processing
        await().atMost(Duration.ofSeconds(30))
            .untilAsserted(() -> {
                OrderResponse status = webClient.get()
                    .uri("/api/orders/{id}", order.getId())
                    .retrieve()
                    .bodyToMono(OrderResponse.class)
                    .block();
                assertThat(status.getStatus()).isEqualTo("SHIPPED");
            });
    }
}
```

---

### Q46: How do you test asynchronous messaging in Microservices?

**Answer:**

```java
@SpringBootTest
@EmbeddedKafka(partitions = 1, topics = {"order-events"})
class KafkaIntegrationTest {
    
    @Autowired
    private KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    @Autowired
    private EmbeddedKafkaBroker embeddedKafka;
    
    @SpyBean
    private OrderEventConsumer orderEventConsumer;
    
    @Test
    void shouldConsumeOrderCreatedEvent() throws Exception {
        // Given
        OrderCreatedEvent event = new OrderCreatedEvent(
            "order-123", 
            "customer-1", 
            Money.of(100)
        );
        
        // When
        kafkaTemplate.send("order-events", event.getOrderId(), event).get();
        
        // Then
        verify(orderEventConsumer, timeout(5000))
            .handleOrderCreated(argThat(e -> 
                e.getOrderId().equals("order-123")));
    }
}

// RabbitMQ Testing
@SpringBootTest
@Testcontainers
class RabbitMQIntegrationTest {
    
    @Container
    static RabbitMQContainer rabbit = new RabbitMQContainer("rabbitmq:3.12-management");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.rabbitmq.host", rabbit::getHost);
        registry.add("spring.rabbitmq.port", rabbit::getAmqpPort);
    }
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @Autowired
    private OrderEventConsumer consumer;
    
    @Test
    void shouldProcessMessageFromQueue() {
        // Given
        OrderCreatedMessage message = new OrderCreatedMessage("order-123");
        
        // When
        rabbitTemplate.convertAndSend("order.exchange", "order.created", message);
        
        // Then
        await().atMost(Duration.ofSeconds(5))
            .untilAsserted(() -> 
                verify(consumer).handleOrderCreated(any()));
    }
}
```

---

## 12. Deployment Strategies

### Q47: Explain different deployment strategies for Microservices

**Answer:**

**1. Rolling Deployment:**
```
Time 0: [v1] [v1] [v1] [v1]
Time 1: [v2] [v1] [v1] [v1]
Time 2: [v2] [v2] [v1] [v1]
Time 3: [v2] [v2] [v2] [v1]
Time 4: [v2] [v2] [v2] [v2]
```

```yaml
# Kubernetes Rolling Update
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Extra pods during update
      maxUnavailable: 0  # Always maintain capacity
```

**2. Blue-Green Deployment:**
```
┌─────────────────────────────────────────────┐
│                Load Balancer                │
└─────────────────┬───────────────────────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
   ┌────▼────┐         ┌────▼────┐
   │  Blue   │         │  Green  │
   │  (v1)   │  ───►   │  (v2)   │
   │ Active  │         │  Idle   │
   └─────────┘         └─────────┘
```

```yaml
# Kubernetes Blue-Green with Service
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
    version: blue  # Switch to 'green' for deployment
  ports:
    - port: 80
      targetPort: 8080

---
# Blue Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
      version: blue
  template:
    metadata:
      labels:
        app: order-service
        version: blue
    spec:
      containers:
        - name: order-service
          image: order-service:1.0.0

---
# Green Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
      version: green
  template:
    metadata:
      labels:
        app: order-service
        version: green
    spec:
      containers:
        - name: order-service
          image: order-service:2.0.0
```

**3. Canary Deployment:**
```
┌─────────────────────────────────────────────┐
│                Load Balancer                │
│            (Traffic Splitting)              │
└─────────────────┬───────────────────────────┘
                  │
        ┌─────────┴─────────┐
        │ 95%               │ 5%
   ┌────▼────┐         ┌────▼────┐
   │ Stable  │         │ Canary  │
   │  (v1)   │         │  (v2)   │
   └─────────┘         └─────────┘
```

```yaml
# Istio Canary Deployment
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
    - order-service
  http:
    - route:
        - destination:
            host: order-service
            subset: stable
          weight: 95
        - destination:
            host: order-service
            subset: canary
          weight: 5

---
# Argo Rollouts Canary
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: order-service
spec:
  replicas: 10
  strategy:
    canary:
      steps:
        - setWeight: 5
        - pause: {duration: 1h}
        - setWeight: 20
        - pause: {duration: 1h}
        - setWeight: 50
        - pause: {duration: 1h}
        - setWeight: 100
      canaryMetadata:
        labels:
          role: canary
      stableMetadata:
        labels:
          role: stable
```

**4. A/B Testing Deployment:**
```yaml
# Route based on user attributes
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
    - order-service
  http:
    - match:
        - headers:
            x-user-group:
              exact: "beta-testers"
      route:
        - destination:
            host: order-service
            subset: v2
    - route:
        - destination:
            host: order-service
            subset: v1
```

---

### Q48: What is GitOps? How does it relate to Microservices deployment?

**Answer:**

GitOps is a deployment methodology where Git is the single source of truth for declarative infrastructure and applications.

**Principles:**
1. Declarative configuration
2. Version controlled
3. Automated apply
4. Continuous reconciliation

**GitOps Workflow:**
```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Developer   │────▶│   Git Repo   │────▶│  CI Pipeline │
│  (Push code) │     │  (manifests) │     │   (Build)    │
└──────────────┘     └──────┬───────┘     └──────┬───────┘
                            │                     │
                     ┌──────▼───────┐      ┌──────▼───────┐
                     │ Config Repo  │◀─────│ Update Image │
                     │ (K8s YAML)   │      │    Tag       │
                     └──────┬───────┘      └──────────────┘
                            │
                     ┌──────▼───────┐
                     │   ArgoCD /   │
                     │   Flux       │
                     │  (Operator)  │
                     └──────┬───────┘
                            │
                     ┌──────▼───────┐
                     │  Kubernetes  │
                     │   Cluster    │
                     └──────────────┘
```

**ArgoCD Application:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: order-service
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/k8s-manifests.git
    targetRevision: HEAD
    path: services/order-service
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
```

**Flux Configuration:**
```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: order-service
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/company/k8s-manifests
  ref:
    branch: main

---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: order-service
  namespace: flux-system
spec:
  interval: 5m
  path: ./services/order-service
  prune: true
  sourceRef:
    kind: GitRepository
    name: order-service
  healthChecks:
    - apiVersion: apps/v1
      kind: Deployment
      name: order-service
      namespace: production
```

---

## 13. Event-Driven Architecture

### Q49: Explain Event-Driven Architecture patterns

**Answer:**

**1. Event Notification:**
```java
// Simple notification - receiver fetches data
@Service
public class OrderService {
    
    private final EventPublisher eventPublisher;
    
    public Order createOrder(CreateOrderCommand cmd) {
        Order order = orderRepository.save(new Order(cmd));
        
        // Notify others - minimal data
        eventPublisher.publish(new OrderCreatedNotification(order.getId()));
        
        return order;
    }
}

// Consumer fetches full data if needed
@EventHandler
public class InventoryService {
    
    private final OrderClient orderClient;
    
    @EventHandler
    public void on(OrderCreatedNotification notification) {
        Order order = orderClient.getOrder(notification.getOrderId());
        reserveInventory(order.getItems());
    }
}
```

**2. Event-Carried State Transfer:**
```java
// Event contains all necessary data
public class OrderCreatedEvent {
    private String orderId;
    private String customerId;
    private List<OrderItem> items;
    private Money totalAmount;
    private Address shippingAddress;
    private Instant createdAt;
}

// Consumer has all data needed - no callback
@EventHandler
public class InventoryService {
    
    @EventHandler
    public void on(OrderCreatedEvent event) {
        // All data available in event
        event.getItems().forEach(item -> 
            reserveStock(item.getProductId(), item.getQuantity()));
    }
}

@EventHandler
public class NotificationService {
    
    @EventHandler
    public void on(OrderCreatedEvent event) {
        // Can send notification with all details
        emailService.sendOrderConfirmation(
            event.getCustomerId(),
            event.getOrderId(),
            event.getItems(),
            event.getTotalAmount()
        );
    }
}
```

**3. Domain Events vs Integration Events:**
```java
// Domain Event - internal to bounded context
public class OrderPlacedDomainEvent implements DomainEvent {
    private final Order order;
    private final Instant occurredAt;
    
    // Used within Order Service for internal processing
}

// Integration Event - published to other services
public class OrderPlacedIntegrationEvent implements IntegrationEvent {
    private String orderId;
    private String customerId;
    private BigDecimal amount;
    private String currency;
    private List<OrderItemDTO> items;
    
    // Stable contract for external services
}

@Service
public class OrderService {
    
    private final DomainEventPublisher domainEventPublisher;
    private final IntegrationEventPublisher integrationEventPublisher;
    
    @Transactional
    public Order placeOrder(CreateOrderCommand cmd) {
        Order order = Order.create(cmd);
        orderRepository.save(order);
        
        // Internal event
        domainEventPublisher.publish(new OrderPlacedDomainEvent(order));
        
        // External event (through outbox)
        integrationEventPublisher.publish(
            OrderPlacedIntegrationEvent.from(order));
        
        return order;
    }
}
```

---

### Q50: Explain Event Streaming with Apache Kafka

**Answer:**

**Kafka Architecture:**
```
┌─────────────────────────────────────────────────────────────┐
│                       Kafka Cluster                          │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              Topic: order-events                         │ │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐              │ │
│  │  │Partition 0│ │Partition 1│ │Partition 2│              │ │
│  │  │ [0,1,2,3] │ │ [0,1,2]   │ │ [0,1,2,3,4]│             │ │
│  │  └───────────┘ └───────────┘ └───────────┘              │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  Consumer Groups:                                            │
│  ┌─────────────────────┐  ┌─────────────────────┐           │
│  │ inventory-service   │  │ notification-service│           │
│  │ [C1: P0] [C2: P1,P2]│  │ [C1: P0,P1,P2]      │          │
│  └─────────────────────┘  └─────────────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

**Producer Configuration:**
```java
@Configuration
public class KafkaProducerConfig {
    
    @Bean
    public ProducerFactory<String, OrderEvent> producerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka:9092");
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        
        // Reliability settings
        config.put(ProducerConfig.ACKS_CONFIG, "all");
        config.put(ProducerConfig.RETRIES_CONFIG, 3);
        config.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
        
        // Performance settings
        config.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        config.put(ProducerConfig.LINGER_MS_CONFIG, 5);
        config.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");
        
        return new DefaultKafkaProducerFactory<>(config);
    }
    
    @Bean
    public KafkaTemplate<String, OrderEvent> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}

@Service
public class OrderEventPublisher {
    
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    public CompletableFuture<SendResult<String, OrderEvent>> publish(OrderEvent event) {
        return kafkaTemplate.send(
            "order-events",
            event.getOrderId(),  // Key for partitioning
            event
        ).whenComplete((result, ex) -> {
            if (ex != null) {
                log.error("Failed to publish event: {}", event.getOrderId(), ex);
            } else {
                log.info("Event published to partition {} offset {}",
                    result.getRecordMetadata().partition(),
                    result.getRecordMetadata().offset());
            }
        });
    }
}
```

**Consumer Configuration:**
```java
@Configuration
@EnableKafka
public class KafkaConsumerConfig {
    
    @Bean
    public ConsumerFactory<String, OrderEvent> consumerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka:9092");
        config.put(ConsumerConfig.GROUP_ID_CONFIG, "inventory-service");
        config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        
        // Reliability settings
        config.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        config.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 100);
        
        return new DefaultKafkaConsumerFactory<>(config);
    }
    
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, OrderEvent> 
            kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, OrderEvent> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3);  // Parallel consumers
        factory.getContainerProperties().setAckMode(AckMode.MANUAL);
        
        // Error handling
        factory.setCommonErrorHandler(new DefaultErrorHandler(
            new DeadLetterPublishingRecoverer(kafkaTemplate),
            new FixedBackOff(1000L, 3)
        ));
        
        return factory;
    }
}

@Component
public class InventoryEventConsumer {
    
    @KafkaListener(
        topics = "order-events",
        groupId = "inventory-service",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void handleOrderEvent(
            @Payload OrderEvent event,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
            @Header(KafkaHeaders.OFFSET) long offset,
            Acknowledgment acknowledgment) {
        
        try {
            log.info("Processing event from partition {} offset {}", partition, offset);
            
            if (event instanceof OrderCreatedEvent) {
                inventoryService.reserveStock((OrderCreatedEvent) event);
            } else if (event instanceof OrderCancelledEvent) {
                inventoryService.releaseStock((OrderCancelledEvent) event);
            }
            
            acknowledgment.acknowledge();
            
        } catch (Exception e) {
            log.error("Failed to process event", e);
            // Will be retried or sent to DLQ
            throw e;
        }
    }
}
```

**Kafka Streams for Real-Time Processing:**
```java
@Configuration
@EnableKafkaStreams
public class KafkaStreamsConfig {
    
    @Bean
    public KStream<String, OrderEvent> orderProcessingStream(
            StreamsBuilder streamsBuilder) {
        
        KStream<String, OrderEvent> orderStream = streamsBuilder
            .stream("order-events", Consumed.with(Serdes.String(), orderEventSerde));
        
        // Filter and branch
        Map<String, KStream<String, OrderEvent>> branches = orderStream
            .split(Named.as("order-"))
            .branch((key, event) -> event instanceof OrderCreatedEvent, 
                Branched.as("created"))
            .branch((key, event) -> event instanceof OrderCancelledEvent, 
                Branched.as("cancelled"))
            .defaultBranch(Branched.as("other"));
        
        // Process created orders - aggregate by customer
        branches.get("order-created")
            .groupBy((key, event) -> ((OrderCreatedEvent) event).getCustomerId())
            .windowedBy(TimeWindows.ofSizeWithNoGrace(Duration.ofHours(1)))
            .aggregate(
                CustomerOrderStats::new,
                (key, event, stats) -> stats.add((OrderCreatedEvent) event),
                Materialized.as("customer-order-stats")
            )
            .toStream()
            .to("customer-analytics");
        
        // Join with customer data
        GlobalKTable<String, Customer> customers = streamsBuilder
            .globalTable("customers");
        
        orderStream
            .join(customers,
                (orderId, event) -> event.getCustomerId(),
                (event, customer) -> new EnrichedOrderEvent(event, customer))
            .to("enriched-orders");
        
        return orderStream;
    }
}
```

---

## 14. Configuration Management

### Q51: How do you manage configuration in Microservices?

**Answer:**

**1. Spring Cloud Config Server:**
```java
// Config Server
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

```yaml
# Config Server - application.yml
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/company/config-repo
          search-paths: '{application}'
          default-label: main
          clone-on-start: true
        encrypt:
          enabled: true

encrypt:
  key: ${ENCRYPT_KEY}
```

**Git Repository Structure:**
```
config-repo/
├── application.yml           # Shared config
├── order-service/
│   ├── application.yml       # Default
│   ├── application-dev.yml   # Dev profile
│   └── application-prod.yml  # Prod profile
├── payment-service/
│   ├── application.yml
│   └── application-prod.yml
└── inventory-service/
    └── application.yml
```

**Client Configuration:**
```yaml
# bootstrap.yml
spring:
  application:
    name: order-service
  cloud:
    config:
      uri: http://config-server:8888
      fail-fast: true
      retry:
        initial-interval: 1000
        max-attempts: 6
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}
```

**2. Dynamic Configuration Refresh:**
```java
@RefreshScope
@RestController
public class OrderController {
    
    @Value("${order.max-items:10}")
    private int maxItems;
    
    @Value("${order.discount-enabled:false}")
    private boolean discountEnabled;
    
    @GetMapping("/api/orders/config")
    public Map<String, Object> getConfig() {
        return Map.of(
            "maxItems", maxItems,
            "discountEnabled", discountEnabled
        );
    }
}

// Trigger refresh via actuator
// POST /actuator/refresh
```

**3. Kubernetes ConfigMaps & Secrets:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: order-service-config
data:
  application.yml: |
    server:
      port: 8080
    order:
      max-items: 20
      discount-enabled: true
    spring:
      datasource:
        url: jdbc:postgresql://postgres:5432/orders

---
apiVersion: v1
kind: Secret
metadata:
  name: order-service-secrets
type: Opaque
stringData:
  database-password: supersecret
  jwt-secret: myverysecretkey

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  template:
    spec:
      containers:
        - name: order-service
          image: order-service:latest
          env:
            - name: SPRING_DATASOURCE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: order-service-secrets
                  key: database-password
          volumeMounts:
            - name: config
              mountPath: /config
              readOnly: true
      volumes:
        - name: config
          configMap:
            name: order-service-config
```

**4. HashiCorp Vault for Secrets:**
```java
@Configuration
public class VaultConfig {
    
    @Bean
    public VaultTemplate vaultTemplate() {
        VaultEndpoint endpoint = VaultEndpoint.create("vault", 8200);
        TokenAuthentication auth = new TokenAuthentication("s.xxxxx");
        return new VaultTemplate(endpoint, auth);
    }
}

@Service
public class SecretService {
    
    private final VaultTemplate vaultTemplate;
    
    public String getDatabasePassword() {
        VaultResponse response = vaultTemplate.read("secret/data/order-service");
        return (String) response.getData().get("database-password");
    }
}
```

```yaml
# Spring Cloud Vault
spring:
  cloud:
    vault:
      uri: https://vault:8200
      authentication: KUBERNETES
      kubernetes:
        role: order-service
        kubernetes-path: kubernetes
      kv:
        enabled: true
        backend: secret
        default-context: order-service
```

---

## 15. Performance & Scalability

### Q52: How do you scale Microservices?

**Answer:**

**1. Horizontal Pod Autoscaler (HPA):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "1000"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
        - type: Pods
          value: 4
          periodSeconds: 15
      selectPolicy: Max
```

**2. Vertical Pod Autoscaler (VPA):**
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: order-service-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: order-service
        minAllowed:
          cpu: 100m
          memory: 128Mi
        maxAllowed:
          cpu: 2
          memory: 2Gi
```

**3. KEDA (Kubernetes Event-Driven Autoscaling):**
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: order-service-scaler
spec:
  scaleTargetRef:
    name: order-service
  pollingInterval: 15
  cooldownPeriod: 300
  minReplicaCount: 1
  maxReplicaCount: 50
  triggers:
    - type: kafka
      metadata:
        bootstrapServers: kafka:9092
        consumerGroup: order-service
        topic: order-events
        lagThreshold: "100"
    - type: prometheus
      metadata:
        serverAddress: http://prometheus:9090
        metricName: http_requests_total
        threshold: "1000"
        query: sum(rate(http_requests_total{service="order-service"}[2m]))
```

---

### Q53: How do you implement Caching in Microservices?

**Answer:**

**1. Local Cache (Caffeine):**
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .initialCapacity(100)
            .maximumSize(1000)
            .expireAfterWrite(Duration.ofMinutes(10))
            .recordStats());
        return cacheManager;
    }
}

@Service
public class ProductService {
    
    @Cacheable(value = "products", key = "#productId")
    public Product getProduct(String productId) {
        return productRepository.findById(productId).orElseThrow();
    }
    
    @CacheEvict(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        return productRepository.save(product);
    }
    
    @CacheEvict(value = "products", allEntries = true)
    public void clearCache() {
        // Clears all cached products
    }
}
```

**2. Distributed Cache (Redis):**
```java
@Configuration
@EnableCaching
public class RedisCacheConfig {
    
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(30))
            .serializeKeysWith(SerializationPair.fromSerializer(
                new StringRedisSerializer()))
            .serializeValuesWith(SerializationPair.fromSerializer(
                new GenericJackson2JsonRedisSerializer()))
            .disableCachingNullValues();
        
        Map<String, RedisCacheConfiguration> cacheConfigs = Map.of(
            "products", config.entryTtl(Duration.ofHours(1)),
            "users", config.entryTtl(Duration.ofMinutes(15)),
            "orders", config.entryTtl(Duration.ofMinutes(5))
        );
        
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .withInitialCacheConfigurations(cacheConfigs)
            .build();
    }
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(
            RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
}

@Service
public class OrderService {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    // Cache-Aside Pattern
    public Order getOrder(String orderId) {
        String key = "order:" + orderId;
        
        // Check cache
        Order cached = (Order) redisTemplate.opsForValue().get(key);
        if (cached != null) {
            return cached;
        }
        
        // Load from DB
        Order order = orderRepository.findById(orderId).orElseThrow();
        
        // Cache with TTL
        redisTemplate.opsForValue().set(key, order, Duration.ofMinutes(30));
        
        return order;
    }
    
    // Write-Through Pattern
    @Transactional
    public Order updateOrder(Order order) {
        Order saved = orderRepository.save(order);
        redisTemplate.opsForValue().set(
            "order:" + order.getId(), 
            saved, 
            Duration.ofMinutes(30)
        );
        return saved;
    }
}
```

**3. HTTP Caching:**
```java
@RestController
public class ProductController {
    
    @GetMapping("/api/products/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable String id) {
        Product product = productService.getProduct(id);
        
        return ResponseEntity.ok()
            .cacheControl(CacheControl.maxAge(1, TimeUnit.HOURS))
            .eTag(product.getVersion().toString())
            .body(product);
    }
    
    @GetMapping("/api/products/{id}")
    public ResponseEntity<Product> getProductConditional(
            @PathVariable String id,
            @RequestHeader(value = "If-None-Match", required = false) String ifNoneMatch) {
        
        Product product = productService.getProduct(id);
        String etag = "\"" + product.getVersion() + "\"";
        
        if (etag.equals(ifNoneMatch)) {
            return ResponseEntity.status(HttpStatus.NOT_MODIFIED).build();
        }
        
        return ResponseEntity.ok()
            .eTag(etag)
            .cacheControl(CacheControl.maxAge(1, TimeUnit.HOURS))
            .body(product);
    }
}
```

---

## 16. Real-World Scenarios

### Q54: Design an E-Commerce Order Processing System

**Answer:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            E-Commerce Architecture                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────┐                                                            │
│  │   Clients    │                                                            │
│  │ (Web/Mobile) │                                                            │
│  └──────┬───────┘                                                            │
│         │                                                                    │
│  ┌──────▼───────┐                                                            │
│  │ API Gateway  │  (Kong/Spring Cloud Gateway)                               │
│  │ - Auth       │                                                            │
│  │ - Rate Limit │                                                            │
│  │ - Routing    │                                                            │
│  └──────┬───────┘                                                            │
│         │                                                                    │
│  ┌──────┴──────────────────────────────────────────────────────────────┐    │
│  │                         Service Mesh (Istio)                         │    │
│  │                                                                      │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐  │    │
│  │  │   Order     │  │  Payment    │  │  Inventory  │  │  Shipping  │  │    │
│  │  │  Service    │  │  Service    │  │  Service    │  │  Service   │  │    │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └─────┬──────┘  │    │
│  │         │                │                │               │         │    │
│  │         ▼                ▼                ▼               ▼         │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐  │    │
│  │  │  Order DB   │  │ Payment DB  │  │Inventory DB │  │Shipping DB │  │    │
│  │  │ (PostgreSQL)│  │  (MySQL)    │  │ (MongoDB)   │  │(PostgreSQL)│  │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └────────────┘  │    │
│  │                                                                      │    │
│  └──────────────────────────────┬───────────────────────────────────────┘    │
│                                 │                                            │
│                     ┌───────────▼───────────┐                                │
│                     │   Apache Kafka        │                                │
│                     │  (Event Streaming)    │                                │
│                     └───────────────────────┘                                │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                       Supporting Services                             │   │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌─────────────────┐    │   │
│  │  │ Eureka    │  │  Config   │  │  Zipkin   │  │   Prometheus +  │    │   │
│  │  │ (Registry)│  │  Server   │  │ (Tracing) │  │   Grafana       │    │   │
│  │  └───────────┘  └───────────┘  └───────────┘  └─────────────────┘    │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Order Saga Implementation:**
```java
@Saga
public class OrderSaga {
    
    private static final Logger log = LoggerFactory.getLogger(OrderSaga.class);
    
    @Autowired
    private transient CommandGateway commandGateway;
    
    private String orderId;
    private String paymentId;
    private String inventoryReservationId;
    private String shipmentId;
    
    @StartSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(OrderCreatedEvent event) {
        this.orderId = event.getOrderId();
        log.info("Saga started for order: {}", orderId);
        
        // Step 1: Reserve payment
        commandGateway.send(new ReservePaymentCommand(
            UUID.randomUUID().toString(),
            orderId,
            event.getCustomerId(),
            event.getTotalAmount()
        ));
    }
    
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(PaymentReservedEvent event) {
        this.paymentId = event.getPaymentId();
        log.info("Payment reserved: {}", paymentId);
        
        // Step 2: Reserve inventory
        commandGateway.send(new ReserveInventoryCommand(
            UUID.randomUUID().toString(),
            orderId,
            event.getItems()
        ));
    }
    
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(InventoryReservedEvent event) {
        this.inventoryReservationId = event.getReservationId();
        log.info("Inventory reserved: {}", inventoryReservationId);
        
        // Step 3: Create shipment
        commandGateway.send(new CreateShipmentCommand(
            UUID.randomUUID().toString(),
            orderId,
            event.getShippingAddress()
        ));
    }
    
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(ShipmentCreatedEvent event) {
        this.shipmentId = event.getShipmentId();
        log.info("Shipment created: {}", shipmentId);
        
        // Step 4: Confirm order
        commandGateway.send(new ConfirmOrderCommand(orderId));
    }
    
    @EndSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(OrderConfirmedEvent event) {
        log.info("Order saga completed successfully: {}", orderId);
    }
    
    // Compensating Transactions
    
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(PaymentFailedEvent event) {
        log.warn("Payment failed for order: {}", orderId);
        commandGateway.send(new CancelOrderCommand(orderId, "Payment failed"));
    }
    
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(InventoryNotAvailableEvent event) {
        log.warn("Inventory not available for order: {}", orderId);
        
        // Compensate: Release payment
        if (paymentId != null) {
            commandGateway.send(new ReleasePaymentCommand(paymentId));
        }
        
        commandGateway.send(new CancelOrderCommand(orderId, "Inventory not available"));
    }
    
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(ShipmentFailedEvent event) {
        log.warn("Shipment failed for order: {}", orderId);
        
        // Compensate: Release inventory and payment
        if (inventoryReservationId != null) {
            commandGateway.send(new ReleaseInventoryCommand(inventoryReservationId));
        }
        if (paymentId != null) {
            commandGateway.send(new ReleasePaymentCommand(paymentId));
        }
        
        commandGateway.send(new CancelOrderCommand(orderId, "Shipment failed"));
    }
    
    @EndSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(OrderCancelledEvent event) {
        log.info("Order saga ended with cancellation: {}", orderId);
    }
}
```

---

### Q55: How do you handle database migrations in Microservices?

**Answer:**

**1. Flyway Migrations:**
```java
// Automatic migration on startup
@Configuration
public class FlywayConfig {
    
    @Bean
    public Flyway flyway(DataSource dataSource) {
        return Flyway.configure()
            .dataSource(dataSource)
            .locations("classpath:db/migration")
            .baselineOnMigrate(true)
            .validateOnMigrate(true)
            .outOfOrder(false)
            .build();
    }
}
```

```
src/main/resources/db/migration/
├── V1__Create_orders_table.sql
├── V2__Add_status_column.sql
├── V3__Create_order_items_table.sql
└── V4__Add_customer_index.sql
```

```sql
-- V1__Create_orders_table.sql
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- V2__Add_status_column.sql
ALTER TABLE orders 
ADD COLUMN status VARCHAR(20) DEFAULT 'PENDING';

CREATE INDEX idx_orders_status ON orders(status);

-- V3__Create_order_items_table.sql
CREATE TABLE order_items (
    id UUID PRIMARY KEY,
    order_id UUID REFERENCES orders(id),
    product_id VARCHAR(50) NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10,2) NOT NULL
);
```

**2. Backward-Compatible Migrations (Expand-Contract Pattern):**
```sql
-- Phase 1: Expand (Add new column)
-- V5__Add_email_column.sql
ALTER TABLE customers 
ADD COLUMN email_new VARCHAR(255);

-- Phase 2: Migrate data
-- V6__Migrate_email_data.sql
UPDATE customers SET email_new = email WHERE email_new IS NULL;

-- Phase 3: Code uses both columns during transition

-- Phase 4: Contract (Remove old column) - after all services updated
-- V7__Remove_old_email_column.sql
ALTER TABLE customers DROP COLUMN email;
ALTER TABLE customers RENAME COLUMN email_new TO email;
ALTER TABLE customers ALTER COLUMN email SET NOT NULL;
```

**3. Kubernetes Job for Migrations:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: order-service-migration
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
        - name: flyway
          image: flyway/flyway:9
          args:
            - migrate
          env:
            - name: FLYWAY_URL
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: url
            - name: FLYWAY_USER
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: username
            - name: FLYWAY_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: password
          volumeMounts:
            - name: migrations
              mountPath: /flyway/sql
      volumes:
        - name: migrations
          configMap:
            name: order-service-migrations
      restartPolicy: Never
  backoffLimit: 3
```

---

### Q56: How do you handle service-to-service authentication without user context?

**Answer:**

**1. Machine-to-Machine OAuth2 (Client Credentials):**
```java
@Configuration
public class OAuth2ClientConfig {
    
    @Bean
    public OAuth2AuthorizedClientManager authorizedClientManager(
            ClientRegistrationRepository clientRegistrationRepository,
            OAuth2AuthorizedClientService authorizedClientService) {
        
        OAuth2AuthorizedClientProvider authorizedClientProvider =
            OAuth2AuthorizedClientProviderBuilder.builder()
                .clientCredentials()
                .build();
        
        AuthorizedClientServiceOAuth2AuthorizedClientManager authorizedClientManager =
            new AuthorizedClientServiceOAuth2AuthorizedClientManager(
                clientRegistrationRepository, 
                authorizedClientService
            );
        
        authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);
        return authorizedClientManager;
    }
    
    @Bean
    public WebClient webClient(OAuth2AuthorizedClientManager authorizedClientManager) {
        ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2Client =
            new ServletOAuth2AuthorizedClientExchangeFilterFunction(authorizedClientManager);
        
        oauth2Client.setDefaultClientRegistrationId("payment-service");
        
        return WebClient.builder()
            .apply(oauth2Client.oauth2Configuration())
            .build();
    }
}
```

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          payment-service:
            client-id: order-service
            client-secret: ${PAYMENT_SERVICE_CLIENT_SECRET}
            authorization-grant-type: client_credentials
            scope: payment.write,payment.read
        provider:
          payment-service:
            token-uri: http://auth-server/oauth2/token
```

**2. Service Account Token (Kubernetes):**
```java
@Component
public class ServiceAccountAuthenticator {
    
    private final Path tokenPath = Paths.get(
        "/var/run/secrets/kubernetes.io/serviceaccount/token");
    
    public String getServiceAccountToken() throws IOException {
        return Files.readString(tokenPath);
    }
}

@Component
public class InternalAuthInterceptor implements ClientHttpRequestInterceptor {
    
    private final ServiceAccountAuthenticator authenticator;
    
    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body,
                                        ClientHttpRequestExecution execution) 
            throws IOException {
        request.getHeaders().set("Authorization", 
            "Bearer " + authenticator.getServiceAccountToken());
        request.getHeaders().set("X-Service-Name", "order-service");
        return execution.execute(request, body);
    }
}
```

**3. mTLS with Certificate-Based Identity:**
```yaml
# Istio AuthorizationPolicy
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: payment-service-policy
  namespace: production
spec:
  selector:
    matchLabels:
      app: payment-service
  action: ALLOW
  rules:
    - from:
        - source:
            principals:
              - "cluster.local/ns/production/sa/order-service"
              - "cluster.local/ns/production/sa/refund-service"
      to:
        - operation:
            methods: ["POST", "GET"]
            paths: ["/api/payments/*"]
```

---

### Q57: How do you implement feature flags in Microservices?

**Answer:**

```java
// Feature Flag Configuration
@Configuration
public class FeatureFlagConfig {
    
    @Bean
    public Unleash unleash() {
        return new DefaultUnleash(
            UnleashConfig.builder()
                .appName("order-service")
                .instanceId("order-service-1")
                .unleashAPI("http://unleash:4242/api")
                .apiKey("${UNLEASH_API_KEY}")
                .synchronousFetchOnInitialisation(true)
                .build()
        );
    }
}

@Service
public class OrderService {
    
    private final Unleash unleash;
    private final OrderRepository orderRepository;
    private final NewPricingEngine newPricingEngine;
    private final LegacyPricingEngine legacyPricingEngine;
    
    public Order createOrder(CreateOrderCommand command) {
        Order order = new Order(command);
        
        // Feature flag check
        if (unleash.isEnabled("new-pricing-engine", 
                UnleashContext.builder()
                    .userId(command.getCustomerId())
                    .addProperty("region", command.getRegion())
                    .build())) {
            
            order.setTotal(newPricingEngine.calculate(order.getItems()));
            log.info("Using new pricing engine for order: {}", order.getId());
        } else {
            order.setTotal(legacyPricingEngine.calculate(order.getItems()));
        }
        
        // Gradual rollout feature
        if (unleash.isEnabled("instant-checkout", 
                UnleashContext.builder()
                    .userId(command.getCustomerId())
                    .build())) {
            order.enableInstantCheckout();
        }
        
        return orderRepository.save(order);
    }
}

// Custom Feature Flag with Spring
@Component
public class FeatureFlags {
    
    @Value("${features.new-pricing-engine:false}")
    private boolean newPricingEngineEnabled;
    
    @Value("${features.new-pricing-engine.rollout-percentage:0}")
    private int rolloutPercentage;
    
    public boolean isNewPricingEngineEnabled(String customerId) {
        if (!newPricingEngineEnabled) {
            return false;
        }
        
        // Deterministic rollout based on customer ID
        int bucket = Math.abs(customerId.hashCode() % 100);
        return bucket < rolloutPercentage;
    }
}
```

```yaml
# ConfigMap for feature flags
apiVersion: v1
kind: ConfigMap
metadata:
  name: feature-flags
data:
  features.new-pricing-engine: "true"
  features.new-pricing-engine.rollout-percentage: "25"
  features.instant-checkout: "false"
```

---

### Q58: Explain Idempotency in Microservices

**Answer:**

**What is Idempotency?**
An operation is idempotent if performing it multiple times produces the same result as performing it once.

**Implementation Strategies:**

**1. Idempotency Key:**
```java
@RestController
public class PaymentController {
    
    private final PaymentService paymentService;
    private final IdempotencyStore idempotencyStore;
    
    @PostMapping("/api/payments")
    public ResponseEntity<PaymentResponse> createPayment(
            @RequestHeader("Idempotency-Key") String idempotencyKey,
            @RequestBody PaymentRequest request) {
        
        // Check if request was already processed
        Optional<PaymentResponse> cached = idempotencyStore.get(idempotencyKey);
        if (cached.isPresent()) {
            return ResponseEntity.ok(cached.get());
        }
        
        // Process payment
        PaymentResponse response = paymentService.process(request);
        
        // Store result with TTL
        idempotencyStore.store(idempotencyKey, response, Duration.ofHours(24));
        
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}

@Component
public class RedisIdempotencyStore implements IdempotencyStore {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    @Override
    public Optional<PaymentResponse> get(String key) {
        Object value = redisTemplate.opsForValue().get("idempotency:" + key);
        return Optional.ofNullable((PaymentResponse) value);
    }
    
    @Override
    public void store(String key, PaymentResponse response, Duration ttl) {
        redisTemplate.opsForValue().set("idempotency:" + key, response, ttl);
    }
}
```

**2. Database-Level Idempotency:**
```java
@Service
public class OrderService {
    
    @Transactional
    public Order createOrder(String requestId, CreateOrderCommand command) {
        // Check if order already exists with this request ID
        Optional<Order> existing = orderRepository.findByRequestId(requestId);
        if (existing.isPresent()) {
            return existing.get();
        }
        
        Order order = new Order(command);
        order.setRequestId(requestId);
        
        try {
            return orderRepository.save(order);
        } catch (DataIntegrityViolationException e) {
            // Concurrent request - fetch existing
            return orderRepository.findByRequestId(requestId)
                .orElseThrow(() -> new IllegalStateException("Order not found"));
        }
    }
}

// Entity with unique constraint
@Entity
@Table(name = "orders", 
    uniqueConstraints = @UniqueConstraint(columnNames = "request_id"))
public class Order {
    @Id
    private String id;
    
    @Column(name = "request_id", unique = true, nullable = false)
    private String requestId;
}
```

**3. Event Deduplication:**
```java
@Component
public class IdempotentEventHandler {
    
    private final ProcessedEventRepository processedEventRepository;
    
    @KafkaListener(topics = "order-events")
    @Transactional
    public void handleOrderEvent(
            @Payload OrderEvent event,
            @Header(KafkaHeaders.RECEIVED_MESSAGE_KEY) String key,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
            @Header(KafkaHeaders.OFFSET) long offset) {
        
        String eventId = String.format("%s-%d-%d", key, partition, offset);
        
        // Check if already processed
        if (processedEventRepository.existsById(eventId)) {
            log.info("Skipping duplicate event: {}", eventId);
            return;
        }
        
        // Process event
        processEvent(event);
        
        // Mark as processed
        processedEventRepository.save(new ProcessedEvent(eventId, Instant.now()));
    }
}
```

---

### Q59: How do you implement Rate Limiting across multiple service instances?

**Answer:**

**1. Distributed Rate Limiting with Redis:**
```java
@Component
public class DistributedRateLimiter {
    
    private final StringRedisTemplate redisTemplate;
    
    // Sliding Window Rate Limiter
    public boolean isAllowed(String key, int maxRequests, Duration window) {
        String redisKey = "ratelimit:" + key;
        long now = System.currentTimeMillis();
        long windowStart = now - window.toMillis();
        
        // Lua script for atomic operation
        String luaScript = """
            redis.call('ZREMRANGEBYSCORE', KEYS[1], 0, ARGV[1])
            local count = redis.call('ZCARD', KEYS[1])
            if count < tonumber(ARGV[2]) then
                redis.call('ZADD', KEYS[1], ARGV[3], ARGV[3])
                redis.call('EXPIRE', KEYS[1], ARGV[4])
                return 1
            end
            return 0
            """;
        
        Long result = redisTemplate.execute(
            RedisScript.of(luaScript, Long.class),
            List.of(redisKey),
            String.valueOf(windowStart),
            String.valueOf(maxRequests),
            String.valueOf(now),
            String.valueOf(window.toSeconds())
        );
        
        return result != null && result == 1;
    }
    
    // Token Bucket with Redis
    public boolean tryAcquire(String key, int bucketSize, int refillRate) {
        String luaScript = """
            local tokens = redis.call('GET', KEYS[1])
            local lastRefill = redis.call('GET', KEYS[2])
            local now = tonumber(ARGV[1])
            
            if tokens == false then
                tokens = ARGV[2]
                lastRefill = now
            else
                tokens = tonumber(tokens)
                lastRefill = tonumber(lastRefill)
                local elapsed = now - lastRefill
                local newTokens = math.floor(elapsed / 1000 * ARGV[3])
                tokens = math.min(tonumber(ARGV[2]), tokens + newTokens)
                if newTokens > 0 then
                    lastRefill = now
                end
            end
            
            if tokens > 0 then
                redis.call('SET', KEYS[1], tokens - 1)
                redis.call('SET', KEYS[2], lastRefill)
                redis.call('EXPIRE', KEYS[1], 60)
                redis.call('EXPIRE', KEYS[2], 60)
                return 1
            end
            return 0
            """;
        
        Long result = redisTemplate.execute(
            RedisScript.of(luaScript, Long.class),
            List.of("bucket:" + key, "lastRefill:" + key),
            String.valueOf(System.currentTimeMillis()),
            String.valueOf(bucketSize),
            String.valueOf(refillRate)
        );
        
        return result != null && result == 1;
    }
}

@Component
public class RateLimitFilter implements WebFilter {
    
    private final DistributedRateLimiter rateLimiter;
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        String clientId = extractClientId(exchange);
        
        if (!rateLimiter.isAllowed(clientId, 100, Duration.ofMinutes(1))) {
            exchange.getResponse().setStatusCode(HttpStatus.TOO_MANY_REQUESTS);
            exchange.getResponse().getHeaders()
                .add("Retry-After", "60");
            return exchange.getResponse().setComplete();
        }
        
        return chain.filter(exchange);
    }
}
```

**2. API Gateway Rate Limiting (Spring Cloud Gateway):**
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 100
                redis-rate-limiter.burstCapacity: 200
                redis-rate-limiter.requestedTokens: 1
                key-resolver: "#{@apiKeyResolver}"

---
# Different limits for different tiers
spring:
  cloud:
    gateway:
      routes:
        - id: premium-tier
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
            - Header=X-Tier, premium
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 1000
                redis-rate-limiter.burstCapacity: 2000
                
        - id: standard-tier
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 100
                redis-rate-limiter.burstCapacity: 200
```

---

### Q60: What are the best practices for Microservices design?

**Answer:**

**1. Design Principles:**
```
┌─────────────────────────────────────────────────────────────────┐
│                   Microservices Best Practices                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Single Responsibility                                        │
│     ├── One service = One business capability                   │
│     └── Clear bounded context                                   │
│                                                                  │
│  2. Database per Service                                         │
│     ├── No shared databases                                     │
│     ├── API for data access                                     │
│     └── Eventual consistency accepted                           │
│                                                                  │
│  3. Smart Endpoints, Dumb Pipes                                  │
│     ├── Business logic in services                              │
│     ├── Simple communication channels                           │
│     └── Avoid ESB-style integration                             │
│                                                                  │
│  4. Design for Failure                                           │
│     ├── Circuit breakers                                        │
│     ├── Retries with backoff                                    │
│     ├── Timeouts                                                │
│     ├── Fallbacks                                               │
│     └── Bulkheads                                               │
│                                                                  │
│  5. Decentralized Governance                                     │
│     ├── Team autonomy                                           │
│     ├── Polyglot persistence                                    │
│     └── Polyglot programming                                    │
│                                                                  │
│  6. Infrastructure Automation                                    │
│     ├── CI/CD pipelines                                         │
│     ├── Infrastructure as Code                                  │
│     └── Container orchestration                                 │
│                                                                  │
│  7. Observability                                                │
│     ├── Centralized logging                                     │
│     ├── Distributed tracing                                     │
│     ├── Metrics and alerting                                    │
│     └── Health checks                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**2. API Design Guidelines:**
```java
// Good: RESTful, resource-oriented, versioned
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    
    @GetMapping
    public Page<OrderSummaryDTO> listOrders(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) OrderStatus status) {
        return orderService.findAll(PageRequest.of(page, size), status);
    }
    
    @GetMapping("/{orderId}")
    public OrderDTO getOrder(@PathVariable String orderId) {
        return orderService.findById(orderId);
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public OrderDTO createOrder(
            @RequestHeader("Idempotency-Key") String idempotencyKey,
            @Valid @RequestBody CreateOrderRequest request) {
        return orderService.create(idempotencyKey, request);
    }
    
    @PatchMapping("/{orderId}")
    public OrderDTO updateOrder(
            @PathVariable String orderId,
            @Valid @RequestBody UpdateOrderRequest request) {
        return orderService.update(orderId, request);
    }
    
    @DeleteMapping("/{orderId}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void cancelOrder(@PathVariable String orderId) {
        orderService.cancel(orderId);
    }
}

// Use problem details for errors (RFC 7807)
@ExceptionHandler(OrderNotFoundException.class)
public ResponseEntity<ProblemDetail> handleNotFound(OrderNotFoundException ex) {
    ProblemDetail problem = ProblemDetail.forStatusAndDetail(
        HttpStatus.NOT_FOUND,
        ex.getMessage()
    );
    problem.setType(URI.create("https://api.example.com/errors/order-not-found"));
    problem.setTitle("Order Not Found");
    problem.setProperty("orderId", ex.getOrderId());
    
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(problem);
}
```

**3. Service Template:**
```java
// Standard service structure
order-service/
├── src/main/java/com/example/order/
│   ├── OrderServiceApplication.java
│   ├── api/
│   │   ├── controller/
│   │   ├── dto/
│   │   └── mapper/
│   ├── domain/
│   │   ├── model/
│   │   ├── event/
│   │   ├── repository/
│   │   └── service/
│   ├── infrastructure/
│   │   ├── config/
│   │   ├── persistence/
│   │   └── messaging/
│   └── integration/
│       └── client/
├── src/main/resources/
│   ├── application.yml
│   ├── application-dev.yml
│   ├── application-prod.yml
│   └── db/migration/
├── src/test/
│   ├── java/
│   │   ├── unit/
│   │   ├── integration/
│   │   └── contract/
│   └── resources/
├── Dockerfile
├── docker-compose.yml
├── kubernetes/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
└── pom.xml
```

---

## Summary: Key Takeaways for Interviews

### Technical Knowledge Checklist:
- [ ] Core microservices concepts and trade-offs
- [ ] Design patterns (API Gateway, Circuit Breaker, Saga, CQRS)
- [ ] Communication patterns (REST, gRPC, async messaging)
- [ ] Data management (Database per service, eventual consistency)
- [ ] Service discovery and registration
- [ ] Security (OAuth2, JWT, mTLS)
- [ ] Resilience patterns (Circuit Breaker, Retry, Bulkhead)
- [ ] Observability (Logging, Tracing, Metrics)
- [ ] Containerization and Kubernetes
- [ ] Event-driven architecture and Kafka
- [ ] Testing strategies
- [ ] Deployment strategies (Blue-Green, Canary)

### Common Interview Questions Categories:
1. **Architecture decisions** - When to use microservices vs monolith
2. **Problem-solving** - How to handle distributed transactions
3. **Trade-offs** - CAP theorem, consistency vs availability
4. **Debugging** - How to trace requests across services
5. **Scaling** - Horizontal vs vertical scaling strategies
6. **Security** - Service-to-service authentication
7. **Real-world scenarios** - Design an e-commerce system

### Interview Tips:
1. **Be specific** - Use concrete examples from experience
2. **Discuss trade-offs** - No solution is perfect
3. **Consider scale** - Think about millions of users
4. **Mention tools** - Show practical knowledge (Kubernetes, Kafka, etc.)
5. **Address failure** - Always consider what can go wrong
6. **Talk about testing** - Testing distributed systems is critical
7. **Security first** - Always mention security considerations

---

**Document Version:** 2.0  
**Last Updated:** December 2024  
**Topics Covered:** 60+ Interview Questions with detailed answers and code examples