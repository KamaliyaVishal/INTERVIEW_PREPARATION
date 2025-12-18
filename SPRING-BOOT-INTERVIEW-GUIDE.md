<p align="center">
  <img src="https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white" alt="Spring Boot"/>
</p>

<h1 align="center">🍃 Spring Boot Interview Guide</h1>

<p align="center">
  <a href="README.md">← Back to Main Guide</a>
</p>

---

## 📚 Section Overview

| # | Topic | Difficulty |
|---|-------|------------|
| 1 | [Spring Core Concepts](#1-spring-core-concepts) | ⭐⭐ |
| 2 | [Spring Boot Fundamentals](#2-spring-boot-fundamentals) | ⭐⭐ |
| 3 | [Spring MVC / REST API](#3-spring-mvc--rest-api) | ⭐⭐⭐ |
| 4 | [Spring Data JPA](#4-spring-data-jpa) | ⭐⭐⭐ |
| 5 | [Spring Security](#5-spring-security) | ⭐⭐⭐⭐ |
| 6 | [Spring AOP](#6-spring-aop) | ⭐⭐⭐ |
| 7 | [Spring Boot Testing](#7-spring-boot-testing) | ⭐⭐⭐ |
| 8 | [Spring Cloud](#8-spring-cloud-microservices) | ⭐⭐⭐⭐ |
| 9 | [Caching](#9-caching) | ⭐⭐ |
| 10 | [Messaging](#10-messaging) | ⭐⭐⭐ |

---

## 1. Spring Core Concepts

### 🔷 1.1 Inversion of Control (IoC)

```java
// ❌ Without IoC: Tight coupling
public class OrderService {
    private PaymentService paymentService = new PaymentService();  // Creates dependency
}

// ✅ With IoC: Loose coupling
public class OrderService {
    private final PaymentService paymentService;
    
    @Autowired  // Spring injects the dependency
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

### 🔷 1.2 Dependency Injection Types

```java
// 1️⃣ Constructor Injection (RECOMMENDED)
@Service
public class OrderService {
    private final PaymentService paymentService;
    
    public OrderService(PaymentService paymentService) {  // @Autowired optional
        this.paymentService = paymentService;
    }
}

// 2️⃣ Setter Injection
@Service
public class OrderService {
    private PaymentService paymentService;
    
    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}

// 3️⃣ Field Injection (NOT RECOMMENDED)
@Service
public class OrderService {
    @Autowired
    private PaymentService paymentService;  // Hard to test
}
```

### 🔷 1.3 Bean Scopes

| Scope | Description | Use Case |
|-------|-------------|----------|
| **singleton** | One instance per Spring container (default) | Stateless services |
| **prototype** | New instance each time requested | Stateful beans |
| **request** | One instance per HTTP request | Web request data |
| **session** | One instance per HTTP session | User session data |
| **application** | One instance per ServletContext | App-wide config |

```java
@Component
@Scope("prototype")
public class PrototypeBean { }

@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, 
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestScopedBean { }
```

### 🔷 1.4 Bean Lifecycle

```
┌────────────────────────────────────────────────────────────────────┐
│                        BEAN LIFECYCLE                              │
├────────────────────────────────────────────────────────────────────┤
│  1. Instantiation (Constructor)                                    │
│       ↓                                                            │
│  2. Populate Properties (Dependency Injection)                     │
│       ↓                                                            │
│  3. BeanNameAware.setBeanName()                                    │
│       ↓                                                            │
│  4. BeanFactoryAware.setBeanFactory()                              │
│       ↓                                                            │
│  5. ApplicationContextAware.setApplicationContext()                │
│       ↓                                                            │
│  6. BeanPostProcessor.postProcessBeforeInitialization()            │
│       ↓                                                            │
│  7. @PostConstruct method                                          │
│       ↓                                                            │
│  8. InitializingBean.afterPropertiesSet()                          │
│       ↓                                                            │
│  9. Custom init-method                                             │
│       ↓                                                            │
│  10. BeanPostProcessor.postProcessAfterInitialization()            │
│       ↓                                                            │
│  ═══════════════ BEAN READY FOR USE ═══════════════                │
│       ↓                                                            │
│  11. @PreDestroy method                                            │
│       ↓                                                            │
│  12. DisposableBean.destroy()                                      │
│       ↓                                                            │
│  13. Custom destroy-method                                         │
└────────────────────────────────────────────────────────────────────┘
```

### 🔷 1.5 Stereotype Annotations

| Annotation | Layer | Purpose |
|------------|-------|---------|
| `@Component` | Generic | Generic Spring-managed component |
| `@Service` | Business | Business logic layer |
| `@Repository` | Data Access | DAO layer (enables exception translation) |
| `@Controller` | Presentation | MVC controller (returns views) |
| `@RestController` | Presentation | REST API controller (returns JSON/XML) |
| `@Configuration` | Config | Configuration class with @Bean methods |

---

## 2. Spring Boot Fundamentals

### 🔷 2.1 @SpringBootApplication

```java
@SpringBootApplication  // = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

### 🔷 2.2 Auto-configuration

Spring Boot automatically configures beans based on:
1. **Classpath dependencies** (e.g., H2 → DataSource)
2. **Existing beans** (won't override your custom beans)
3. **Property values** (application.properties)

```java
// Conditional auto-configuration
@Configuration
@ConditionalOnClass(DataSource.class)
@ConditionalOnMissingBean(DataSource.class)
public class DataSourceAutoConfiguration {
    // Only applied if DataSource is on classpath and no DataSource bean exists
}
```

### 🔷 2.3 Configuration Properties

```yaml
# application.yml
spring:
  profiles:
    active: dev

---
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:h2:mem:devdb
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

---
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:postgresql://prod-server:5432/proddb
```

```java
// Type-safe configuration
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {
    private String name;
    private int maxConnections;
    private List<String> servers;
    // getters and setters
}
```

### 🔷 2.4 Spring Boot Actuator

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: always
```

| Endpoint | Description |
|----------|-------------|
| `/actuator/health` | Application health status |
| `/actuator/info` | Application information |
| `/actuator/metrics` | Application metrics |
| `/actuator/env` | Environment properties |
| `/actuator/beans` | All beans in context |
| `/actuator/mappings` | All request mappings |
| `/actuator/prometheus` | Prometheus metrics |

---

## 3. Spring MVC / REST API

### 🔷 3.1 Controller Annotations

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @GetMapping
    public List<User> getAllUsers() { }
    
    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) { }
    
    @GetMapping("/search")
    public List<User> searchUsers(
        @RequestParam String name,
        @RequestParam(required = false, defaultValue = "0") int page) { }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public User createUser(@Valid @RequestBody CreateUserRequest request) { }
    
    @PutMapping("/{id}")
    public User updateUser(
        @PathVariable Long id, 
        @RequestBody UpdateUserRequest request) { }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteUser(@PathVariable Long id) { }
}
```

### 🔷 3.2 ResponseEntity

```java
@GetMapping("/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(user -> ResponseEntity.ok(user))
        .orElse(ResponseEntity.notFound().build());
}

@PostMapping
public ResponseEntity<User> createUser(@RequestBody User user) {
    User created = userService.save(user);
    URI location = ServletUriComponentsBuilder.fromCurrentRequest()
        .path("/{id}")
        .buildAndExpand(created.getId())
        .toUri();
    return ResponseEntity.created(location).body(created);
}
```

### 🔷 3.3 Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                FieldError::getDefaultMessage
            ));
        return new ErrorResponse(400, "Validation failed", errors);
    }
}
```

### 🔷 3.4 Validation

```java
public class CreateUserRequest {
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be 2-100 characters")
    private String name;
    
    @Email(message = "Invalid email format")
    @NotBlank(message = "Email is required")
    private String email;
    
    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 120, message = "Age must be at most 120")
    private Integer age;
    
    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Invalid phone number")
    private String phone;
}
```

---

## 4. Spring Data JPA

### 🔷 4.1 Entity Mapping

```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Enumerated(EnumType.STRING)
    private UserStatus status;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    @ManyToMany
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

### 🔷 4.2 Repository Methods

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Derived Query Methods
    Optional<User> findByEmail(String email);
    List<User> findByNameContainingIgnoreCase(String name);
    List<User> findByStatusAndCreatedAtAfter(UserStatus status, LocalDateTime date);
    boolean existsByEmail(String email);
    
    // JPQL Query
    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain")
    List<User> findByEmailDomain(@Param("domain") String domain);
    
    // Native Query
    @Query(value = "SELECT * FROM users WHERE status = :status", nativeQuery = true)
    List<User> findByStatusNative(@Param("status") String status);
    
    // Modifying Query
    @Modifying
    @Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") UserStatus status);
    
    // Pagination
    Page<User> findByStatus(UserStatus status, Pageable pageable);
}
```

### 🔷 4.3 N+1 Problem & Solutions

```java
// ❌ Problem: N+1 queries
List<User> users = userRepository.findAll();
users.forEach(u -> u.getOrders().size());  // Each triggers a query!

// ✅ Solution 1: Fetch Join
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllWithOrders();

// ✅ Solution 2: Entity Graph
@EntityGraph(attributePaths = {"orders", "roles"})
List<User> findAll();

// ✅ Solution 3: Batch Fetching (application.properties)
spring.jpa.properties.hibernate.default_batch_fetch_size=20
```

### 🔷 4.4 Transactions

```java
@Service
@Transactional(readOnly = true)  // Default for class
public class OrderService {
    
    @Transactional  // Read-write for this method
    public Order createOrder(OrderRequest request) {
        Order order = new Order();
        orderRepository.save(order);
        inventoryService.decreaseStock(order.getItems());
        return order;
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logOrder(Order order) {
        // Creates a new transaction, independent of parent
    }
    
    @Transactional(rollbackFor = BusinessException.class)
    public void processOrder(Long orderId) throws BusinessException {
        // Rolls back for checked exception too
    }
}
```

| Propagation | Description |
|-------------|-------------|
| `REQUIRED` | Use current or create new (default) |
| `REQUIRES_NEW` | Always create new |
| `SUPPORTS` | Use current if exists |
| `MANDATORY` | Must have existing |
| `NEVER` | Must NOT have |
| `NESTED` | Create savepoint within current |

---

## 5. Spring Security

### 🔷 5.1 Security Configuration (Spring Security 6.x)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 🔷 5.2 JWT Authentication Flow

```
┌────────────────────────────────────────────────────────────────────┐
│                    JWT AUTHENTICATION FLOW                         │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  1. Client sends POST /auth/login with credentials                 │
│                         ↓                                          │
│  2. Server validates credentials                                   │
│                         ↓                                          │
│  3. Server generates JWT token and returns it                      │
│                         ↓                                          │
│  4. Client stores token (localStorage/cookie)                      │
│                         ↓                                          │
│  5. Client sends requests with Authorization: Bearer <token>       │
│                         ↓                                          │
│  6. Server validates token on each request                         │
│                         ↓                                          │
│  7. Server extracts user info from token                           │
│                         ↓                                          │
│  8. Server processes request if valid                              │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 🔷 5.3 Method Security

```java
@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long id) { }
    
    @PreAuthorize("hasRole('ADMIN') or #id == authentication.principal.id")
    public User getUser(Long id) { }
    
    @PostAuthorize("returnObject.owner == authentication.principal.username")
    public Resource getResource(Long id) { }
    
    @PreFilter("filterObject.owner == authentication.principal.username")
    public void processResources(List<Resource> resources) { }
}
```

---

## 6. Spring AOP

### 🔷 6.1 AOP Concepts

| Term | Description |
|------|-------------|
| **Aspect** | Module containing cross-cutting concerns |
| **Advice** | Action taken at a join point |
| **Pointcut** | Expression matching join points |
| **JoinPoint** | Point in execution (method call) |
| **Weaving** | Linking aspects with objects |

### 🔷 6.2 Advice Types

```java
@Aspect
@Component
public class LoggingAspect {
    
    // Before advice
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        log.info("Entering: {}", joinPoint.getSignature().getName());
    }
    
    // After advice (always runs)
    @After("execution(* com.example.service.*.*(..))")
    public void logAfter(JoinPoint joinPoint) {
        log.info("Exiting: {}", joinPoint.getSignature().getName());
    }
    
    // After returning
    @AfterReturning(pointcut = "execution(* com.example.service.*.*(..))", 
                    returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        log.info("Method returned: {}", result);
    }
    
    // After throwing
    @AfterThrowing(pointcut = "execution(* com.example.service.*.*(..))", 
                   throwing = "ex")
    public void logAfterThrowing(JoinPoint joinPoint, Exception ex) {
        log.error("Exception in {}: {}", joinPoint.getSignature(), ex.getMessage());
    }
    
    // Around advice (most powerful)
    @Around("execution(* com.example.service.*.*(..))")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            Object result = joinPoint.proceed();
            long duration = System.currentTimeMillis() - start;
            log.info("{} executed in {} ms", joinPoint.getSignature(), duration);
            return result;
        } catch (Exception ex) {
            log.error("Exception: {}", ex.getMessage());
            throw ex;
        }
    }
}
```

---

## 7. Spring Boot Testing

### 🔷 7.1 Testing Annotations

```java
// Full integration test
@SpringBootTest
class ApplicationTests { }

// Web layer test only
@WebMvcTest(UserController.class)
class UserControllerTest { }

// JPA repository test
@DataJpaTest
class UserRepositoryTest { }

// Service layer test with specific config
@SpringBootTest(classes = {UserService.class, UserRepository.class})
class UserServiceTest { }
```

### 🔷 7.2 MockMvc Testing

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void shouldReturnUser() throws Exception {
        User user = new User(1L, "John");
        when(userService.findById(1L)).thenReturn(Optional.of(user));
        
        mockMvc.perform(get("/api/users/1")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John"));
    }
    
    @Test
    void shouldCreateUser() throws Exception {
        String json = """
            {"name": "John", "email": "john@example.com"}
            """;
        
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(json))
            .andExpect(status().isCreated());
    }
}
```

---

## 8. Spring Cloud (Microservices)

### 🔷 Key Components

| Component | Purpose | Tool |
|-----------|---------|------|
| **Service Discovery** | Register & find services | Eureka, Consul |
| **Config Server** | Centralized configuration | Spring Cloud Config |
| **API Gateway** | Entry point, routing | Spring Cloud Gateway |
| **Circuit Breaker** | Fault tolerance | Resilience4j |
| **Load Balancer** | Distribute traffic | Spring Cloud LoadBalancer |
| **Distributed Tracing** | Track requests | Sleuth + Zipkin |

---

## 9. Caching

```java
@Configuration
@EnableCaching
public class CacheConfig { }

@Service
public class UserService {
    
    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) { }
    
    @CachePut(value = "users", key = "#user.id")
    public User update(User user) { }
    
    @CacheEvict(value = "users", key = "#id")
    public void delete(Long id) { }
    
    @CacheEvict(value = "users", allEntries = true)
    public void clearCache() { }
}
```

---

## 10. Messaging

### 🔷 Kafka

```java
// Producer
@Service
public class OrderProducer {
    @Autowired
    private KafkaTemplate<String, Order> kafkaTemplate;
    
    public void send(Order order) {
        kafkaTemplate.send("orders", order.getId(), order);
    }
}

// Consumer
@Service
public class OrderConsumer {
    @KafkaListener(topics = "orders", groupId = "order-service")
    public void consume(Order order) {
        processOrder(order);
    }
}
```

---

<p align="center">
  <a href="README.md">← Back to Main Guide</a>
</p>

