# Spring Boot Interview Questions and Answers - Complete Guide

## Table of Contents
1. [Spring Boot Fundamentals](#spring-boot-fundamentals)
2. [Auto-Configuration](#auto-configuration)
3. [Spring Boot Starters](#spring-boot-starters)
4. [Application Properties & Configuration](#application-properties--configuration)
5. [Spring Boot Annotations](#spring-boot-annotations)
6. [Dependency Injection & IoC](#dependency-injection--ioc)
7. [Spring Boot REST API](#spring-boot-rest-api)
8. [Exception Handling](#exception-handling)
9. [Spring Boot Data & JPA](#spring-boot-data--jpa)
10. [Spring Boot Security](#spring-boot-security)
11. [Spring Boot Actuator](#spring-boot-actuator)
12. [Testing in Spring Boot](#testing-in-spring-boot)
13. [Profiles & Environment](#profiles--environment)
14. [Logging](#logging)
15. [Caching](#caching)
16. [Spring Boot with Microservices](#spring-boot-with-microservices)
17. [Performance & Optimization](#performance--optimization)
18. [Deployment & DevOps](#deployment--devops)
19. [Advanced Topics](#advanced-topics)
20. [Scenario-Based Questions](#scenario-based-questions)

---

## Spring Boot Fundamentals

### Q1: What is Spring Boot? How is it different from Spring Framework?

**Answer:**
Spring Boot is an opinionated framework built on top of Spring Framework that simplifies the development of production-ready Spring applications.

| Aspect | Spring Framework | Spring Boot |
|--------|-----------------|-------------|
| **Configuration** | Extensive XML/Java config required | Auto-configuration, minimal setup |
| **Dependencies** | Manual dependency management | Starters with compatible versions |
| **Embedded Server** | External server needed | Embedded Tomcat/Jetty/Undertow |
| **Production Ready** | Manual setup needed | Built-in Actuator, metrics |
| **Development Speed** | Slower setup | Rapid application development |
| **Opinionated** | Flexible, no defaults | Opinionated with sensible defaults |

**Key Features of Spring Boot:**
1. **Auto-Configuration**: Automatically configures application based on dependencies
2. **Starter Dependencies**: Pre-packaged dependency descriptors
3. **Embedded Servers**: No need to deploy WAR files
4. **Production-Ready Features**: Health checks, metrics, externalized config
5. **No XML Configuration**: Java-based configuration preferred
6. **Spring Boot CLI**: Command-line tool for quick prototyping

---

### Q2: Explain the Spring Boot application startup process.

**Answer:**
When a Spring Boot application starts, it goes through several phases:

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

**Startup Process:**

1. **SpringApplication Instantiation**
   - Create SpringApplication instance
   - Detect application type (Servlet, Reactive, None)
   - Load ApplicationContextInitializers
   - Load ApplicationListeners

2. **Run Method Execution**
   - Create and configure Environment
   - Print Banner
   - Create ApplicationContext
   - Prepare context (apply initializers)
   - Refresh context (bean creation)
   - Call runners (CommandLineRunner, ApplicationRunner)

3. **Bean Lifecycle**
   ```
   Instantiation → Populate Properties → BeanNameAware → 
   BeanFactoryAware → ApplicationContextAware → 
   PreInitialization (BeanPostProcessor) → InitializingBean → 
   Custom init-method → PostInitialization → Bean Ready
   ```

**Startup Events (in order):**
```java
ApplicationStartingEvent          // Application starts
ApplicationEnvironmentPreparedEvent // Environment ready
ApplicationContextInitializedEvent  // Context created
ApplicationPreparedEvent           // Before refresh
ContextRefreshedEvent             // Context refreshed
ApplicationStartedEvent           // After refresh
ApplicationReadyEvent             // Application ready
```

---

### Q3: What is @SpringBootApplication annotation?

**Answer:**
`@SpringBootApplication` is a convenience annotation that combines three annotations:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration  // Same as @Configuration
@EnableAutoConfiguration  // Enables auto-configuration
@ComponentScan           // Scans for components
public @interface SpringBootApplication {
    // ...
}
```

**Components:**

| Annotation | Purpose |
|------------|---------|
| `@SpringBootConfiguration` | Marks class as configuration source (specialized @Configuration) |
| `@EnableAutoConfiguration` | Enables Spring Boot's auto-configuration mechanism |
| `@ComponentScan` | Scans for @Component, @Service, @Repository, @Controller in package and sub-packages |

**Customization:**
```java
@SpringBootApplication(
    scanBasePackages = {"com.example.app", "com.example.lib"},
    exclude = {DataSourceAutoConfiguration.class}
)
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

---

### Q4: What is the difference between @Component, @Service, @Repository, and @Controller?

**Answer:**
All are specializations of `@Component` with semantic differences:

| Annotation | Purpose | Special Behavior |
|------------|---------|------------------|
| `@Component` | Generic stereotype | Basic component scanning |
| `@Service` | Business logic layer | No additional behavior (semantic only) |
| `@Repository` | Data access layer | Exception translation (PersistenceExceptionTranslationPostProcessor) |
| `@Controller` | Web controller | Handles HTTP requests, returns view names |
| `@RestController` | REST API controller | @Controller + @ResponseBody |

```java
@Component
public class GenericComponent { }

@Service
public class UserService {
    public User findUser(Long id) { ... }
}

@Repository
public class UserRepository {
    // DataAccessException translation enabled
    public User findById(Long id) { ... }
}

@Controller
public class WebController {
    @GetMapping("/page")
    public String showPage(Model model) {
        return "viewName";  // Returns view
    }
}

@RestController  // @Controller + @ResponseBody
public class ApiController {
    @GetMapping("/api/users")
    public List<User> getUsers() {
        return userService.findAll();  // Returns JSON
    }
}
```

---

### Q5: Explain the Spring Bean lifecycle.

**Answer:**

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPRING BEAN LIFECYCLE                        │
├─────────────────────────────────────────────────────────────────┤
│  1. Instantiation (Constructor)                                 │
│  2. Populate Properties (Dependency Injection)                  │
│  3. BeanNameAware.setBeanName()                                │
│  4. BeanFactoryAware.setBeanFactory()                          │
│  5. ApplicationContextAware.setApplicationContext()             │
│  6. @PostConstruct / InitializingBean.afterPropertiesSet()     │
│  7. Custom init-method                                          │
│  8. Bean Ready for Use                                          │
│  ...                                                            │
│  9. @PreDestroy / DisposableBean.destroy()                     │
│  10. Custom destroy-method                                      │
└─────────────────────────────────────────────────────────────────┘
```

**Code Example:**
```java
@Component
public class MyBean implements InitializingBean, DisposableBean,
        BeanNameAware, ApplicationContextAware {
    
    private String beanName;
    private ApplicationContext context;
    
    // 1. Constructor
    public MyBean() {
        System.out.println("1. Constructor called");
    }
    
    // 2. Setter injection
    @Autowired
    public void setDependency(SomeDependency dep) {
        System.out.println("2. Dependencies injected");
    }
    
    // 3. BeanNameAware
    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("3. BeanNameAware: " + name);
    }
    
    // 5. ApplicationContextAware
    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        this.context = ctx;
        System.out.println("5. ApplicationContextAware");
    }
    
    // 6. @PostConstruct
    @PostConstruct
    public void postConstruct() {
        System.out.println("6. @PostConstruct");
    }
    
    // 6. InitializingBean
    @Override
    public void afterPropertiesSet() {
        System.out.println("6. afterPropertiesSet");
    }
    
    // 7. Custom init method
    @Bean(initMethod = "customInit")
    public void customInit() {
        System.out.println("7. Custom init method");
    }
    
    // 9. @PreDestroy
    @PreDestroy
    public void preDestroy() {
        System.out.println("9. @PreDestroy");
    }
    
    // 9. DisposableBean
    @Override
    public void destroy() {
        System.out.println("9. DisposableBean.destroy");
    }
}
```

---

### Q6: What are the different bean scopes in Spring?

**Answer:**

| Scope | Description | Use Case |
|-------|-------------|----------|
| `singleton` (default) | One instance per Spring container | Stateless services |
| `prototype` | New instance each request | Stateful beans |
| `request` | One instance per HTTP request | Web-specific, request data |
| `session` | One instance per HTTP session | User session data |
| `application` | One instance per ServletContext | Shared across sessions |
| `websocket` | One instance per WebSocket session | WebSocket handling |

```java
// Singleton (default)
@Component
@Scope("singleton")
public class SingletonBean { }

// Prototype
@Component
@Scope("prototype")
public class PrototypeBean { }

// Request scope
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, 
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestScopedBean { }

// Session scope
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION,
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class SessionScopedBean { }

// Using @RequestScope and @SessionScope shortcuts
@Component
@RequestScope
public class RequestBean { }

@Component
@SessionScope
public class SessionBean { }
```

**Proxy Mode for Shorter-Lived Scopes:**
```java
@Service
public class SingletonService {
    
    @Autowired
    private RequestScopedBean requestBean; // Needs proxy!
    
    // Without proxy, singleton would hold reference to 
    // one request bean forever. Proxy delegates to 
    // current request's instance.
}
```

---

## Auto-Configuration

### Q7: How does Spring Boot Auto-Configuration work?

**Answer:**
Auto-configuration automatically configures Spring application based on JAR dependencies on the classpath.

**Mechanism:**

1. **@EnableAutoConfiguration** triggers the process
2. **spring.factories** file lists auto-configuration classes
3. **Conditional annotations** determine if configuration applies

**Location of Auto-Configurations:**
```
META-INF/spring.factories (Spring Boot 2.x)
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports (Spring Boot 3.x)
```

**Example spring.factories:**
```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.MyAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

**How Auto-Configuration Class Works:**
```java
@Configuration
@ConditionalOnClass(DataSource.class)  // Only if DataSource on classpath
@ConditionalOnMissingBean(DataSource.class)  // Only if no DataSource bean defined
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource(DataSourceProperties properties) {
        return DataSourceBuilder.create()
            .url(properties.getUrl())
            .username(properties.getUsername())
            .password(properties.getPassword())
            .build();
    }
}
```

---

### Q8: Explain the @Conditional annotations in Spring Boot.

**Answer:**
Conditional annotations control when beans/configurations are loaded.

| Annotation | Condition |
|------------|-----------|
| `@ConditionalOnClass` | Class present on classpath |
| `@ConditionalOnMissingClass` | Class NOT present on classpath |
| `@ConditionalOnBean` | Bean exists in context |
| `@ConditionalOnMissingBean` | Bean does NOT exist |
| `@ConditionalOnProperty` | Property has specific value |
| `@ConditionalOnResource` | Resource exists |
| `@ConditionalOnWebApplication` | Web application context |
| `@ConditionalOnNotWebApplication` | NOT web application |
| `@ConditionalOnExpression` | SpEL expression evaluates true |
| `@ConditionalOnJava` | Specific Java version |
| `@ConditionalOnCloudPlatform` | Specific cloud platform |

```java
@Configuration
public class ConditionalConfig {
    
    // Only if MySQL driver on classpath
    @Bean
    @ConditionalOnClass(name = "com.mysql.cj.jdbc.Driver")
    public DataSource mysqlDataSource() {
        return new MysqlDataSource();
    }
    
    // Only if no DataSource bean exists
    @Bean
    @ConditionalOnMissingBean(DataSource.class)
    public DataSource defaultDataSource() {
        return new EmbeddedDatabaseBuilder().build();
    }
    
    // Only if property is true
    @Bean
    @ConditionalOnProperty(name = "feature.cache.enabled", 
                          havingValue = "true",
                          matchIfMissing = false)
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager();
    }
    
    // Only in web application
    @Bean
    @ConditionalOnWebApplication
    public WebFilter myWebFilter() {
        return new MyWebFilter();
    }
    
    // SpEL expression
    @Bean
    @ConditionalOnExpression("${feature.enabled:true} and ${app.mode} == 'production'")
    public ProductionService productionService() {
        return new ProductionService();
    }
}
```

---

### Q9: How to create a custom Auto-Configuration?

**Answer:**

**Step 1: Create Configuration Class**
```java
@Configuration
@ConditionalOnClass(MyService.class)
@EnableConfigurationProperties(MyServiceProperties.class)
public class MyServiceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyServiceProperties properties) {
        return new MyService(properties.getUrl(), properties.getTimeout());
    }
}
```

**Step 2: Create Properties Class**
```java
@ConfigurationProperties(prefix = "myservice")
public class MyServiceProperties {
    private String url = "http://localhost:8080";
    private int timeout = 5000;
    
    // Getters and setters
}
```

**Step 3: Register in spring.factories (Spring Boot 2.x)**
```properties
# src/main/resources/META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.autoconfigure.MyServiceAutoConfiguration
```

**Step 3: Register in imports file (Spring Boot 3.x)**
```
# src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
com.example.autoconfigure.MyServiceAutoConfiguration
```

**Step 4: Use in application.properties**
```properties
myservice.url=https://api.example.com
myservice.timeout=10000
```

---

### Q10: How to disable specific Auto-Configuration?

**Answer:**

**Method 1: Using @SpringBootApplication**
```java
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class,
    SecurityAutoConfiguration.class
})
public class MyApplication { }
```

**Method 2: Using @EnableAutoConfiguration**
```java
@EnableAutoConfiguration(exclude = {
    DataSourceAutoConfiguration.class
})
public class MyApplication { }
```

**Method 3: Using application.properties**
```properties
spring.autoconfigure.exclude=\
  org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
  org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
```

**Method 4: Using excludeName (for classes not on classpath)**
```java
@SpringBootApplication(excludeName = {
    "org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration"
})
public class MyApplication { }
```

---

## Spring Boot Starters

### Q11: What are Spring Boot Starters? List the important ones.

**Answer:**
Starters are dependency descriptors that bundle related dependencies with compatible versions.

**Benefits:**
- Simplified dependency management
- Version compatibility guaranteed
- Reduces Maven/Gradle configuration
- Follows convention over configuration

**Important Starters:**

| Starter | Purpose |
|---------|---------|
| `spring-boot-starter` | Core starter, logging, auto-config |
| `spring-boot-starter-web` | Web applications, embedded Tomcat |
| `spring-boot-starter-webflux` | Reactive web applications |
| `spring-boot-starter-data-jpa` | JPA with Hibernate |
| `spring-boot-starter-data-mongodb` | MongoDB support |
| `spring-boot-starter-data-redis` | Redis support |
| `spring-boot-starter-security` | Spring Security |
| `spring-boot-starter-oauth2-client` | OAuth2 client |
| `spring-boot-starter-oauth2-resource-server` | OAuth2 resource server |
| `spring-boot-starter-actuator` | Production monitoring |
| `spring-boot-starter-test` | Testing (JUnit, Mockito, etc.) |
| `spring-boot-starter-validation` | Bean validation |
| `spring-boot-starter-cache` | Caching support |
| `spring-boot-starter-aop` | Aspect-oriented programming |
| `spring-boot-starter-mail` | Email support |
| `spring-boot-starter-thymeleaf` | Thymeleaf templates |
| `spring-boot-starter-batch` | Batch processing |
| `spring-boot-starter-amqp` | RabbitMQ messaging |
| `spring-boot-starter-kafka` | Apache Kafka |

```xml
<!-- pom.xml example -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```

---

### Q12: How to create a custom Spring Boot Starter?

**Answer:**
Custom starters follow the naming convention: `{name}-spring-boot-starter`

**Project Structure:**
```
my-service-spring-boot-starter/
├── my-service-spring-boot-autoconfigure/
│   ├── src/main/java/
│   │   └── com/example/autoconfigure/
│   │       ├── MyServiceAutoConfiguration.java
│   │       └── MyServiceProperties.java
│   └── src/main/resources/
│       └── META-INF/
│           └── spring.factories
└── my-service-spring-boot-starter/
    └── pom.xml (depends on autoconfigure module)
```

**Autoconfigure Module:**
```java
// MyServiceProperties.java
@ConfigurationProperties(prefix = "my.service")
public class MyServiceProperties {
    private boolean enabled = true;
    private String apiKey;
    private String endpoint = "https://api.default.com";
    
    // Getters and setters
}

// MyServiceAutoConfiguration.java
@Configuration
@ConditionalOnClass(MyServiceClient.class)
@ConditionalOnProperty(prefix = "my.service", name = "enabled", 
                       havingValue = "true", matchIfMissing = true)
@EnableConfigurationProperties(MyServiceProperties.class)
@AutoConfigureAfter(WebClientAutoConfiguration.class)
public class MyServiceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public MyServiceClient myServiceClient(MyServiceProperties props) {
        return new MyServiceClient(props.getEndpoint(), props.getApiKey());
    }
}
```

**spring.factories:**
```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.autoconfigure.MyServiceAutoConfiguration
```

**Starter pom.xml:**
```xml
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>my-service-spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>my-service-client</artifactId>
    </dependency>
</dependencies>
```

---

## Application Properties & Configuration

### Q13: What are the different ways to externalize configuration in Spring Boot?

**Answer:**
Spring Boot supports configuration from multiple sources (in order of precedence, highest first):

1. **Command Line Arguments**
   ```bash
   java -jar app.jar --server.port=9090
   ```

2. **SPRING_APPLICATION_JSON**
   ```bash
   SPRING_APPLICATION_JSON='{"server":{"port":9090}}' java -jar app.jar
   ```

3. **ServletConfig/ServletContext Parameters**

4. **JNDI Attributes**

5. **Java System Properties**
   ```bash
   java -Dserver.port=9090 -jar app.jar
   ```

6. **OS Environment Variables**
   ```bash
   export SERVER_PORT=9090
   ```

7. **RandomValuePropertySource** (random.*)

8. **Profile-specific application-{profile}.properties**

9. **application.properties/application.yml**

10. **@PropertySource annotations**

11. **Default Properties (SpringApplication.setDefaultProperties)**

---

### Q14: Explain @ConfigurationProperties annotation.

**Answer:**
`@ConfigurationProperties` binds external configuration to a Java object with type safety.

```java
// Properties class
@ConfigurationProperties(prefix = "app.mail")
@Validated  // Enable validation
public class MailProperties {
    
    @NotBlank
    private String host;
    
    @Min(1) @Max(65535)
    private int port = 25;
    
    @NotBlank
    private String username;
    
    private String password;
    
    private Security security = new Security();
    
    // Nested class
    public static class Security {
        private boolean enabled = false;
        private String protocol = "TLS";
        
        // Getters and setters
    }
    
    // Getters and setters
}
```

**Enable Configuration Properties:**
```java
// Method 1: On main class
@SpringBootApplication
@EnableConfigurationProperties(MailProperties.class)
public class MyApplication { }

// Method 2: On configuration class
@Configuration
@EnableConfigurationProperties(MailProperties.class)
public class AppConfig { }

// Method 3: Component scanning (Spring Boot 2.2+)
@ConfigurationProperties(prefix = "app.mail")
@Component  // or @Configuration
public class MailProperties { }

// Method 4: Using @ConfigurationPropertiesScan
@SpringBootApplication
@ConfigurationPropertiesScan("com.example.config")
public class MyApplication { }
```

**application.yml:**
```yaml
app:
  mail:
    host: smtp.example.com
    port: 587
    username: user@example.com
    password: secret
    security:
      enabled: true
      protocol: TLS
```

**Using the Properties:**
```java
@Service
public class MailService {
    
    private final MailProperties mailProperties;
    
    public MailService(MailProperties mailProperties) {
        this.mailProperties = mailProperties;
    }
    
    public void sendMail() {
        System.out.println("Host: " + mailProperties.getHost());
        System.out.println("Security: " + mailProperties.getSecurity().isEnabled());
    }
}
```

---

### Q15: What is the difference between @Value and @ConfigurationProperties?

**Answer:**

| Aspect | @Value | @ConfigurationProperties |
|--------|--------|--------------------------|
| **Binding** | Single property | Multiple related properties |
| **Type Safety** | Limited | Full type safety |
| **Validation** | Manual | Integrated with @Validated |
| **Nested Objects** | Not supported | Fully supported |
| **Relaxed Binding** | Limited | Full support |
| **Meta-data** | None | IDE autocomplete support |
| **Immutability** | Supports final fields | Supports records (2.6+) |

```java
// @Value approach
@Component
public class ValueExample {
    
    @Value("${app.name}")
    private String appName;
    
    @Value("${app.timeout:5000}")  // With default
    private int timeout;
    
    @Value("${app.servers}")  // Comma-separated list
    private List<String> servers;
    
    @Value("#{${app.map}}")  // SpEL for maps
    private Map<String, String> map;
    
    @Value("#{systemProperties['user.home']}")  // SpEL expression
    private String userHome;
}

// @ConfigurationProperties approach (recommended)
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    private int timeout = 5000;
    private List<String> servers;
    private Map<String, String> map;
    
    // Getters and setters
}
```

**Relaxed Binding Examples:**
```properties
# All these bind to myProperty field
myProperty=value
my-property=value      # Kebab case (recommended)
my_property=value      # Underscore
MY_PROPERTY=value      # Upper case (env variables)
myPROPERTY=value       # Mixed
```

---

### Q16: How does Spring Boot handle YAML configuration?

**Answer:**
YAML (YAML Ain't Markup Language) provides a hierarchical, human-readable configuration format.

**application.yml:**
```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: ${DB_PASSWORD}  # Environment variable
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

# List of values
app:
  servers:
    - server1.example.com
    - server2.example.com
    - server3.example.com
  
  # Map
  regions:
    us: United States
    uk: United Kingdom
    in: India
    
# Multi-document (profiles)
---
spring:
  config:
    activate:
      on-profile: dev
server:
  port: 8081
  
---
spring:
  config:
    activate:
      on-profile: prod
server:
  port: 80
```

**YAML vs Properties:**

| YAML | Properties |
|------|------------|
| Hierarchical structure | Flat key-value pairs |
| More readable for nested config | Simple for flat config |
| Supports multi-document | One file per profile |
| Supports lists/maps naturally | Indexed lists (list[0]) |
| Whitespace-sensitive | Not whitespace-sensitive |

---

## Spring Boot Annotations

### Q17: Explain the important Spring Boot annotations.

**Answer:**

**Core Annotations:**

```java
// Main Application
@SpringBootApplication  // Combines @Configuration, @EnableAutoConfiguration, @ComponentScan

// Configuration
@Configuration          // Declares class as config source
@Bean                  // Declares a bean
@Import                // Import other configuration classes
@ImportResource        // Import XML configuration
@PropertySource        // Load additional properties file

// Component Scanning
@Component             // Generic component
@Service               // Business layer
@Repository            // Data layer (exception translation)
@Controller            // Web controller
@RestController        // @Controller + @ResponseBody

// Dependency Injection
@Autowired             // Inject dependency
@Qualifier             // Specify bean by name
@Primary               // Preferred bean when multiple exist
@Lazy                  // Lazy initialization

// Request Mapping
@RequestMapping        // Map HTTP requests
@GetMapping            // GET requests
@PostMapping           // POST requests
@PutMapping            // PUT requests
@DeleteMapping         // DELETE requests
@PatchMapping          // PATCH requests

// Request Parameters
@RequestParam          // Query parameters
@PathVariable          // URI path variables
@RequestBody           // Request body (JSON)
@RequestHeader         // HTTP headers
@CookieValue          // Cookie values

// Response
@ResponseBody          // Return value as response body
@ResponseStatus        // Set HTTP status code

// Validation
@Valid                 // Trigger validation
@Validated             // Spring's validation variant

// Configuration Properties
@ConfigurationProperties  // Bind properties to POJO
@EnableConfigurationProperties
@Value                 // Inject single property

// Conditional
@ConditionalOnClass    // If class on classpath
@ConditionalOnBean     // If bean exists
@ConditionalOnProperty // If property matches
@ConditionalOnMissingBean

// Profiles
@Profile               // Activate for specific profile

// Async
@EnableAsync           // Enable async processing
@Async                 // Mark method as async

// Scheduling
@EnableScheduling      // Enable scheduling
@Scheduled             // Define scheduled task

// Caching
@EnableCaching         // Enable caching
@Cacheable             // Cache method result
@CacheEvict            // Remove from cache
@CachePut              // Update cache

// Transaction
@Transactional         // Transaction management

// Testing
@SpringBootTest        // Load full context for testing
@MockBean              // Mock a bean
@DataJpaTest           // Test JPA repositories
@WebMvcTest            // Test web layer
```

---

### Q18: What is @Autowired and what are the injection types?

**Answer:**
`@Autowired` enables automatic dependency injection by Spring container.

**Injection Types:**

**1. Constructor Injection (Recommended)**
```java
@Service
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // @Autowired optional for single constructor (Spring 4.3+)
    public UserService(UserRepository userRepository, 
                       EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}
```
**Benefits:**
- Immutable dependencies (final fields)
- Required dependencies obvious
- Easy unit testing
- Prevents circular dependencies

**2. Setter Injection**
```java
@Service
public class UserService {
    
    private UserRepository userRepository;
    
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```
**Use Case:** Optional dependencies that can be changed

**3. Field Injection (Not Recommended)**
```java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
}
```
**Issues:**
- Hidden dependencies
- Harder to test
- Null possible at runtime
- Cannot be final

**Handling Multiple Beans:**
```java
@Service
public class NotificationService {
    
    // By name
    @Autowired
    @Qualifier("emailSender")
    private MessageSender messageSender;
    
    // Primary bean (if one marked @Primary)
    @Autowired
    private MessageSender sender;
    
    // Inject all implementations
    @Autowired
    private List<MessageSender> allSenders;
    
    // Inject as Map
    @Autowired
    private Map<String, MessageSender> senderMap;
}
```

---

## Dependency Injection & IoC

### Q19: What is Inversion of Control (IoC) and Dependency Injection (DI)?

**Answer:**

**Inversion of Control (IoC):**
A design principle where the control of object creation and lifecycle is inverted from the application to the framework (Spring container).

**Dependency Injection (DI):**
A pattern implementing IoC where dependencies are provided ("injected") rather than created by the dependent object.

**Without DI (Tight Coupling):**
```java
public class UserService {
    // Creates its own dependency - tightly coupled
    private UserRepository repository = new UserRepositoryImpl();
    
    public User findUser(Long id) {
        return repository.findById(id);
    }
}
```

**With DI (Loose Coupling):**
```java
public class UserService {
    // Dependency injected - loosely coupled
    private final UserRepository repository;
    
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
    
    public User findUser(Long id) {
        return repository.findById(id);
    }
}
```

**Benefits of DI:**
1. **Loose Coupling**: Components don't know implementation details
2. **Testability**: Easy to inject mocks for testing
3. **Flexibility**: Swap implementations without code changes
4. **Maintainability**: Single responsibility, clear dependencies
5. **Reusability**: Components can be reused in different contexts

---

### Q20: What is the difference between BeanFactory and ApplicationContext?

**Answer:**

| Feature | BeanFactory | ApplicationContext |
|---------|-------------|-------------------|
| **Bean Loading** | Lazy (on demand) | Eager (at startup) |
| **Enterprise Features** | Basic | Full support |
| **Event Publishing** | No | Yes |
| **Internationalization** | No | Yes (MessageSource) |
| **AOP Support** | Manual | Automatic |
| **Resource Loading** | Basic | Enhanced |
| **Annotation Support** | Limited | Full support |
| **Use Case** | Resource-constrained | Standard applications |

```java
// BeanFactory (rarely used directly)
BeanFactory factory = new XmlBeanFactory(
    new ClassPathResource("beans.xml"));
MyBean bean = factory.getBean(MyBean.class);

// ApplicationContext (preferred)
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
MyBean bean = context.getBean(MyBean.class);

// Spring Boot provides
// - AnnotationConfigServletWebServerApplicationContext (Web)
// - AnnotationConfigReactiveWebServerApplicationContext (Reactive)
// - AnnotationConfigApplicationContext (Non-web)
```

**ApplicationContext Implementations:**
- `AnnotationConfigApplicationContext` - Java config
- `ClassPathXmlApplicationContext` - XML from classpath
- `FileSystemXmlApplicationContext` - XML from file system
- `GenericWebApplicationContext` - Web applications

---

### Q21: How to resolve circular dependencies in Spring?

**Answer:**
Circular dependency: Bean A depends on Bean B, and Bean B depends on Bean A.

**Problem:**
```java
@Service
public class ServiceA {
    @Autowired
    private ServiceB serviceB;  // Needs ServiceB
}

@Service
public class ServiceB {
    @Autowired
    private ServiceA serviceA;  // Needs ServiceA
}
// Result: BeanCurrentlyInCreationException
```

**Solutions:**

**1. Redesign (Best Solution)**
```java
// Extract common logic to new service
@Service
public class CommonService {
    // Shared logic
}

@Service
public class ServiceA {
    @Autowired
    private CommonService commonService;
}

@Service
public class ServiceB {
    @Autowired
    private CommonService commonService;
}
```

**2. Use Setter Injection**
```java
@Service
public class ServiceA {
    private ServiceB serviceB;
    
    @Autowired
    public void setServiceB(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}
```

**3. Use @Lazy**
```java
@Service
public class ServiceA {
    private final ServiceB serviceB;
    
    public ServiceA(@Lazy ServiceB serviceB) {
        this.serviceB = serviceB;  // Injects proxy, resolved later
    }
}
```

**4. Use ObjectFactory/Provider**
```java
@Service
public class ServiceA {
    private final ObjectFactory<ServiceB> serviceBFactory;
    
    public ServiceA(ObjectFactory<ServiceB> serviceBFactory) {
        this.serviceBFactory = serviceBFactory;
    }
    
    public void doSomething() {
        ServiceB serviceB = serviceBFactory.getObject();
    }
}
```

**5. Use ApplicationContext**
```java
@Service
public class ServiceA implements ApplicationContextAware {
    private ApplicationContext context;
    
    @Override
    public void setApplicationContext(ApplicationContext context) {
        this.context = context;
    }
    
    public void doSomething() {
        ServiceB serviceB = context.getBean(ServiceB.class);
    }
}
```

**Note:** Spring Boot 2.6+ disables circular dependencies by default. Enable with:
```properties
spring.main.allow-circular-references=true
```

---

## Spring Boot REST API

### Q22: How to create a REST API in Spring Boot?

**Answer:**

```java
// Entity
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank
    private String name;
    
    @Email
    private String email;
    
    // Constructors, getters, setters
}

// DTO
public class UserDTO {
    private Long id;
    private String name;
    private String email;
    
    // Constructors, getters, setters
}

// Repository
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByNameContaining(String name);
}

// Service
@Service
@Transactional
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public List<User> findAll() {
        return userRepository.findAll();
    }
    
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
    }
    
    public User create(User user) {
        return userRepository.save(user);
    }
    
    public User update(Long id, User userDetails) {
        User user = findById(id);
        user.setName(userDetails.getName());
        user.setEmail(userDetails.getEmail());
        return userRepository.save(user);
    }
    
    public void delete(Long id) {
        User user = findById(id);
        userRepository.delete(user);
    }
}

// REST Controller
@RestController
@RequestMapping("/api/v1/users")
@Validated
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        return ResponseEntity.ok(userService.findAll());
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        User created = userService.create(user);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();
        return ResponseEntity.created(location).body(created);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(
            @PathVariable Long id, 
            @Valid @RequestBody User user) {
        return ResponseEntity.ok(userService.update(id, user));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

### Q23: Explain @RequestParam vs @PathVariable vs @RequestBody.

**Answer:**

| Annotation | Source | Use Case |
|------------|--------|----------|
| `@PathVariable` | URL path | Resource identification |
| `@RequestParam` | Query string | Filtering, pagination |
| `@RequestBody` | Request body | Creating/updating resources |

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    // @PathVariable - from URL path
    // GET /api/products/123
    @GetMapping("/{id}")
    public Product getById(@PathVariable Long id) {
        return productService.findById(id);
    }
    
    // Multiple path variables
    // GET /api/products/category/electronics/item/laptop
    @GetMapping("/category/{category}/item/{item}")
    public Product getProduct(
            @PathVariable String category,
            @PathVariable String item) {
        return productService.find(category, item);
    }
    
    // @RequestParam - from query string
    // GET /api/products?name=phone&minPrice=100&maxPrice=1000
    @GetMapping
    public List<Product> search(
            @RequestParam(required = false) String name,
            @RequestParam(defaultValue = "0") double minPrice,
            @RequestParam(defaultValue = "10000") double maxPrice,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return productService.search(name, minPrice, maxPrice, page, size);
    }
    
    // @RequestBody - from request body (JSON/XML)
    // POST /api/products
    // Body: {"name": "Phone", "price": 599.99}
    @PostMapping
    public Product create(@Valid @RequestBody ProductRequest request) {
        return productService.create(request);
    }
    
    // Combining all three
    // PUT /api/products/123?notify=true
    // Body: {"name": "Updated Phone", "price": 699.99}
    @PutMapping("/{id}")
    public Product update(
            @PathVariable Long id,
            @RequestParam(defaultValue = "false") boolean notify,
            @Valid @RequestBody ProductRequest request) {
        Product product = productService.update(id, request);
        if (notify) {
            notificationService.sendUpdateNotification(product);
        }
        return product;
    }
    
    // @RequestHeader - from HTTP headers
    @GetMapping("/secure")
    public Product getSecure(
            @RequestHeader("Authorization") String auth,
            @RequestHeader(value = "X-Request-Id", required = false) String requestId) {
        // Process with headers
        return product;
    }
}
```

---

### Q24: What is ResponseEntity and when to use it?

**Answer:**
`ResponseEntity` represents the entire HTTP response: status code, headers, and body.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // Return with status and body
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        if (user == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(user);
    }
    
    // Custom status code
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User created = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
    
    // With headers
    @GetMapping("/download/{id}")
    public ResponseEntity<byte[]> downloadFile(@PathVariable Long id) {
        byte[] content = fileService.getContent(id);
        
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
        headers.setContentDispositionFormData("attachment", "file.pdf");
        headers.setContentLength(content.length);
        
        return new ResponseEntity<>(content, headers, HttpStatus.OK);
    }
    
    // Created with location header
    @PostMapping
    public ResponseEntity<User> create(@RequestBody User user) {
        User created = userService.save(user);
        
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();
            
        return ResponseEntity.created(location).body(created);
    }
    
    // No content
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
    
    // Builder pattern
    @GetMapping("/custom")
    public ResponseEntity<User> customResponse() {
        return ResponseEntity
            .status(HttpStatus.OK)
            .header("X-Custom-Header", "value")
            .contentType(MediaType.APPLICATION_JSON)
            .lastModified(Instant.now())
            .cacheControl(CacheControl.maxAge(30, TimeUnit.MINUTES))
            .body(user);
    }
}
```

**When to use ResponseEntity vs returning object directly:**
- Use `ResponseEntity` when you need control over status/headers
- Return object directly for simple 200 OK responses

---

### Q25: How to implement pagination and sorting in Spring Boot?

**Answer:**

```java
// Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    // With Pageable
    Page<Product> findByCategory(String category, Pageable pageable);
    
    // With Slice (more efficient for infinite scroll)
    Slice<Product> findByNameContaining(String name, Pageable pageable);
}

// Service
@Service
public class ProductService {
    
    private final ProductRepository repository;
    
    public Page<Product> getProducts(int page, int size, String sortBy, String direction) {
        Sort sort = direction.equalsIgnoreCase("desc") 
            ? Sort.by(sortBy).descending()
            : Sort.by(sortBy).ascending();
            
        Pageable pageable = PageRequest.of(page, size, sort);
        return repository.findAll(pageable);
    }
    
    public Page<Product> getProductsByCategory(String category, Pageable pageable) {
        return repository.findByCategory(category, pageable);
    }
}

// Controller
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    private final ProductService productService;
    
    // Manual pagination parameters
    @GetMapping
    public ResponseEntity<Page<Product>> getProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "asc") String direction) {
        
        Page<Product> products = productService.getProducts(page, size, sortBy, direction);
        return ResponseEntity.ok(products);
    }
    
    // Using Pageable directly (recommended)
    @GetMapping("/v2")
    public ResponseEntity<Page<Product>> getProductsV2(
            @PageableDefault(size = 20, sort = "name") Pageable pageable) {
        return ResponseEntity.ok(productService.findAll(pageable));
    }
    
    // GET /api/products/v2?page=0&size=20&sort=name,desc&sort=price,asc
}

// DTO for page response
public class PageResponse<T> {
    private List<T> content;
    private int pageNumber;
    private int pageSize;
    private long totalElements;
    private int totalPages;
    private boolean first;
    private boolean last;
    
    public static <T> PageResponse<T> from(Page<T> page) {
        PageResponse<T> response = new PageResponse<>();
        response.content = page.getContent();
        response.pageNumber = page.getNumber();
        response.pageSize = page.getSize();
        response.totalElements = page.getTotalElements();
        response.totalPages = page.getTotalPages();
        response.first = page.isFirst();
        response.last = page.isLast();
        return response;
    }
}

// Multiple sort criteria
@GetMapping("/sorted")
public Page<Product> getSorted() {
    Sort sort = Sort.by(
        Sort.Order.desc("price"),
        Sort.Order.asc("name")
    );
    return repository.findAll(PageRequest.of(0, 10, sort));
}
```

**application.properties:**
```properties
# Customize parameter names
spring.data.web.pageable.page-parameter=page
spring.data.web.pageable.size-parameter=size
spring.data.web.sort.sort-parameter=sort
spring.data.web.pageable.default-page-size=20
spring.data.web.pageable.max-page-size=100
```

---

## Exception Handling

### Q26: How to handle exceptions globally in Spring Boot?

**Answer:**
Use `@ControllerAdvice` or `@RestControllerAdvice` for centralized exception handling.

```java
// Custom Exceptions
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

public class BusinessException extends RuntimeException {
    private final String errorCode;
    
    public BusinessException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
    
    public String getErrorCode() {
        return errorCode;
    }
}

// Error Response DTO
public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    private Map<String, String> validationErrors;
    
    // Builder pattern or constructors
    
    public static ErrorResponse of(HttpStatus status, String message, String path) {
        ErrorResponse response = new ErrorResponse();
        response.timestamp = LocalDateTime.now();
        response.status = status.value();
        response.error = status.getReasonPhrase();
        response.message = message;
        response.path = path;
        return response;
    }
}

// Global Exception Handler
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    // Handle specific exception
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleResourceNotFound(
            ResourceNotFoundException ex,
            HttpServletRequest request) {
        log.error("Resource not found: {}", ex.getMessage());
        return ErrorResponse.of(
            HttpStatus.NOT_FOUND, 
            ex.getMessage(), 
            request.getRequestURI()
        );
    }
    
    // Handle validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationErrors(
            MethodArgumentNotValidException ex,
            HttpServletRequest request) {
        
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> 
            errors.put(error.getField(), error.getDefaultMessage())
        );
        
        ErrorResponse response = ErrorResponse.of(
            HttpStatus.BAD_REQUEST,
            "Validation failed",
            request.getRequestURI()
        );
        response.setValidationErrors(errors);
        return response;
    }
    
    // Handle constraint violations (path/query params)
    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleConstraintViolation(
            ConstraintViolationException ex,
            HttpServletRequest request) {
        
        Map<String, String> errors = new HashMap<>();
        ex.getConstraintViolations().forEach(cv -> 
            errors.put(cv.getPropertyPath().toString(), cv.getMessage())
        );
        
        ErrorResponse response = ErrorResponse.of(
            HttpStatus.BAD_REQUEST,
            "Constraint violation",
            request.getRequestURI()
        );
        response.setValidationErrors(errors);
        return response;
    }
    
    // Handle business exceptions
    @ExceptionHandler(BusinessException.class)
    @ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
    public ErrorResponse handleBusinessException(
            BusinessException ex,
            HttpServletRequest request) {
        log.warn("Business error [{}]: {}", ex.getErrorCode(), ex.getMessage());
        return ErrorResponse.of(
            HttpStatus.UNPROCESSABLE_ENTITY,
            ex.getMessage(),
            request.getRequestURI()
        );
    }
    
    // Handle all other exceptions
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleAllExceptions(
            Exception ex,
            HttpServletRequest request) {
        log.error("Unexpected error", ex);
        return ErrorResponse.of(
            HttpStatus.INTERNAL_SERVER_ERROR,
            "An unexpected error occurred",
            request.getRequestURI()
        );
    }
}
```

---

### Q27: What is the difference between @ControllerAdvice and @RestControllerAdvice?

**Answer:**

| @ControllerAdvice | @RestControllerAdvice |
|-------------------|----------------------|
| For traditional MVC controllers | For REST controllers |
| Needs @ResponseBody on methods | @ResponseBody implied |
| Can return views | Returns JSON/XML |
| General purpose | Convenience for REST APIs |

```java
// @ControllerAdvice - for web MVC
@ControllerAdvice
public class WebExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public String handleNotFound(Model model, Exception ex) {
        model.addAttribute("message", ex.getMessage());
        return "error/404";  // Returns view
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseBody  // Needed for JSON response
    public ErrorResponse handleException(Exception ex) {
        return new ErrorResponse(ex.getMessage());
    }
}

// @RestControllerAdvice - for REST APIs
@RestControllerAdvice
public class ApiExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(Exception ex) {
        return new ErrorResponse(ex.getMessage());  // JSON automatically
    }
}

// Scoped to specific controllers
@RestControllerAdvice(
    basePackages = "com.example.api",
    assignableTypes = {UserController.class, OrderController.class},
    annotations = RestController.class
)
public class ScopedExceptionHandler { }
```

---

## Spring Boot Data & JPA

### Q28: What is Spring Data JPA? Explain repository hierarchy.

**Answer:**
Spring Data JPA simplifies data access layer by providing repository abstractions over JPA.

**Repository Hierarchy:**
```
Repository (marker interface)
    │
    └── CrudRepository<T, ID>
            │   - save(), findById(), findAll(), delete(), count()
            │
            └── PagingAndSortingRepository<T, ID>
                    │   - findAll(Pageable), findAll(Sort)
                    │
                    └── JpaRepository<T, ID>
                            - saveAll(), flush(), deleteInBatch()
                            - getOne() [deprecated] / getReferenceById()
```

```java
// Basic Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Derived query methods
    List<User> findByName(String name);
    List<User> findByNameAndEmail(String name, String email);
    List<User> findByNameContainingIgnoreCase(String name);
    List<User> findByAgeGreaterThan(int age);
    List<User> findByActiveTrue();
    List<User> findByCreatedAtBetween(LocalDate start, LocalDate end);
    List<User> findTop5ByOrderByCreatedAtDesc();
    
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
    long countByActive(boolean active);
    
    // With pagination and sorting
    Page<User> findByActive(boolean active, Pageable pageable);
    List<User> findByName(String name, Sort sort);
    
    // Custom JPQL query
    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain")
    List<User> findByEmailDomain(@Param("domain") String domain);
    
    // Native SQL query
    @Query(value = "SELECT * FROM users WHERE age > :age", nativeQuery = true)
    List<User> findOlderThan(@Param("age") int age);
    
    // Modifying query
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
    int deactivateInactiveUsers(@Param("date") LocalDate date);
    
    // Projection
    List<UserProjection> findByActive(boolean active);
    
    // Stream results
    @QueryHints(@QueryHint(name = HINT_FETCH_SIZE, value = "25"))
    Stream<User> streamAllByActiveTrue();
}

// Projection interface
public interface UserProjection {
    Long getId();
    String getName();
    String getEmail();
}

// DTO Projection
public interface UserSummary {
    String getName();
    
    @Value("#{target.firstName + ' ' + target.lastName}")
    String getFullName();
}
```

---

### Q29: Explain N+1 problem and how to solve it.

**Answer:**
The N+1 problem occurs when fetching a collection results in N additional queries for each element.

**Problem:**
```java
@Entity
public class Author {
    @Id
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "author", fetch = FetchType.LAZY)
    private List<Book> books;
}

// This causes N+1 problem
List<Author> authors = authorRepository.findAll();  // 1 query
for (Author author : authors) {
    author.getBooks().size();  // N queries (one per author)
}
```

**Solutions:**

**1. JOIN FETCH (JPQL)**
```java
@Query("SELECT a FROM Author a JOIN FETCH a.books")
List<Author> findAllWithBooks();
```

**2. @EntityGraph**
```java
@EntityGraph(attributePaths = {"books"})
List<Author> findAll();

// Named entity graph
@Entity
@NamedEntityGraph(
    name = "Author.withBooks",
    attributeNodes = @NamedAttributeNode("books")
)
public class Author { }

@EntityGraph(value = "Author.withBooks")
List<Author> findByNameContaining(String name);
```

**3. @BatchSize (Hibernate)**
```java
@Entity
public class Author {
    @OneToMany(mappedBy = "author")
    @BatchSize(size = 25)  // Fetch 25 collections at a time
    private List<Book> books;
}

// Or globally in application.properties
spring.jpa.properties.hibernate.default_batch_fetch_size=25
```

**4. Subselect Fetching**
```java
@Entity
public class Author {
    @OneToMany(mappedBy = "author")
    @Fetch(FetchMode.SUBSELECT)  // One additional query for all
    private List<Book> books;
}
```

**5. DTO Projection**
```java
@Query("""
    SELECT new com.example.dto.AuthorWithBookCount(
        a.id, a.name, COUNT(b)
    )
    FROM Author a LEFT JOIN a.books b
    GROUP BY a.id, a.name
    """)
List<AuthorWithBookCount> findAuthorsWithBookCount();
```

---

### Q30: What is the difference between FetchType.LAZY and FetchType.EAGER?

**Answer:**

| LAZY | EAGER |
|------|-------|
| Load on access | Load immediately with parent |
| Better performance | May cause performance issues |
| Risk of LazyInitializationException | No lazy loading issues |
| Default for @OneToMany, @ManyToMany | Default for @OneToOne, @ManyToOne |
| Proxy created | Actual data fetched |

```java
@Entity
public class User {
    
    @ManyToOne(fetch = FetchType.LAZY)  // Loaded when accessed
    private Department department;
    
    @OneToMany(fetch = FetchType.EAGER)  // Loaded with User
    private List<Order> orders;
}

// LazyInitializationException example
@Service
public class UserService {
    
    @Transactional(readOnly = true)
    public User getUser(Long id) {
        return userRepository.findById(id).orElseThrow();
    }
}

// In controller (outside transaction)
User user = userService.getUser(1L);
user.getDepartment().getName();  // LazyInitializationException!
```

**Solutions for LazyInitializationException:**

**1. Keep transaction open (not recommended for large data)**
```java
spring.jpa.open-in-view=true  // Default, but discouraged
```

**2. Initialize in service layer**
```java
@Transactional(readOnly = true)
public User getUser(Long id) {
    User user = userRepository.findById(id).orElseThrow();
    Hibernate.initialize(user.getDepartment());  // Force load
    return user;
}
```

**3. Fetch eagerly for specific query**
```java
@Query("SELECT u FROM User u JOIN FETCH u.department WHERE u.id = :id")
Optional<User> findByIdWithDepartment(@Param("id") Long id);
```

**4. Use DTO**
```java
@Query("SELECT new com.example.UserDTO(u.id, u.name, d.name) " +
       "FROM User u JOIN u.department d WHERE u.id = :id")
UserDTO findUserDTOById(@Param("id") Long id);
```

---

### Q31: Explain @Transactional annotation and its attributes.

**Answer:**

```java
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.DEFAULT,
    timeout = 30,
    readOnly = false,
    rollbackFor = {Exception.class},
    noRollbackFor = {EntityNotFoundException.class}
)
public void performOperation() { }
```

**Propagation Types:**

| Type | Behavior |
|------|----------|
| `REQUIRED` (default) | Join existing or create new |
| `REQUIRES_NEW` | Always create new (suspend existing) |
| `SUPPORTS` | Join if exists, non-transactional if not |
| `NOT_SUPPORTED` | Execute non-transactionally (suspend if exists) |
| `MANDATORY` | Must have existing, error if none |
| `NEVER` | Must NOT have existing, error if exists |
| `NESTED` | Nested transaction (savepoint) |

```java
@Service
public class OrderService {
    
    @Autowired
    private PaymentService paymentService;
    
    @Transactional  // REQUIRED by default
    public void createOrder(Order order) {
        orderRepository.save(order);
        
        // Uses same transaction
        paymentService.processPayment(order);
    }
}

@Service
public class PaymentService {
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void processPayment(Order order) {
        // New transaction - if this fails, order still commits
        paymentRepository.save(payment);
    }
}
```

**Isolation Levels:**

| Level | Dirty Read | Non-Repeatable | Phantom |
|-------|------------|----------------|---------|
| `READ_UNCOMMITTED` | Yes | Yes | Yes |
| `READ_COMMITTED` | No | Yes | Yes |
| `REPEATABLE_READ` | No | No | Yes |
| `SERIALIZABLE` | No | No | No |

**Important Notes:**
```java
@Service
public class UserService {
    
    // Self-invocation doesn't work!
    @Transactional
    public void methodA() {
        methodB();  // @Transactional on methodB is IGNORED
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void methodB() {
        // Called from same class - proxy not used
    }
    
    // Solution: Extract to another service or use self-injection
}

// readOnly optimization
@Transactional(readOnly = true)
public List<User> getAllUsers() {
    return userRepository.findAll();  // Hibernate flush disabled
}
```

---

## Spring Boot Security

### Q32: How does Spring Security work in Spring Boot?

**Answer:**
Spring Security provides authentication, authorization, and protection against common attacks.

**Architecture:**
```
HTTP Request
     │
     ▼
FilterChain (DelegatingFilterProxy → FilterChainProxy)
     │
     ▼
Security Filters (15+ filters)
     │
     ├── SecurityContextPersistenceFilter
     ├── UsernamePasswordAuthenticationFilter
     ├── BasicAuthenticationFilter
     ├── ExceptionTranslationFilter
     └── FilterSecurityInterceptor
     │
     ▼
Authentication Manager
     │
     ▼
Authentication Provider(s)
     │
     ▼
UserDetailsService
     │
     ▼
Controller
```

**Basic Configuration (Spring Boot 3.x / Spring Security 6.x):**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())  // Disable for APIs
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );
        
        return http.build();
    }
    
    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.builder()
            .username("user")
            .password(passwordEncoder().encode("password"))
            .roles("USER")
            .build();
            
        UserDetails admin = User.builder()
            .username("admin")
            .password(passwordEncoder().encode("admin"))
            .roles("ADMIN")
            .build();
            
        return new InMemoryUserDetailsManager(user, admin);
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

---

### Q33: Explain JWT authentication in Spring Boot.

**Answer:**

```java
// JWT Utility Class
@Component
public class JwtTokenProvider {
    
    @Value("${jwt.secret}")
    private String jwtSecret;
    
    @Value("${jwt.expiration}")
    private long jwtExpiration;
    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList()));
            
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + jwtExpiration))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }
    
    public String getUsernameFromToken(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody()
            .getSubject();
    }
    
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }
    
    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(jwtSecret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}

// JWT Authentication Filter
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    @Autowired
    private JwtTokenProvider tokenProvider;
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) 
            throws ServletException, IOException {
        
        String token = extractToken(request);
        
        if (token != null && tokenProvider.validateToken(token)) {
            String username = tokenProvider.getUsernameFromToken(token);
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            
            UsernamePasswordAuthenticationToken authentication =
                new UsernamePasswordAuthenticationToken(
                    userDetails, null, userDetails.getAuthorities());
            authentication.setDetails(
                new WebAuthenticationDetailsSource().buildDetails(request));
            
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String extractToken(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}

// Security Configuration with JWT
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Autowired
    private JwtAuthenticationFilter jwtAuthFilter;
    
    @Autowired
    private AuthenticationProvider authenticationProvider;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authenticationProvider(authenticationProvider)
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}

// Auth Controller
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    
    @Autowired
    private AuthenticationManager authenticationManager;
    
    @Autowired
    private JwtTokenProvider tokenProvider;
    
    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@RequestBody LoginRequest request) {
        Authentication authentication = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getUsername(), request.getPassword())
        );
        
        SecurityContextHolder.getContext().setAuthentication(authentication);
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        String token = tokenProvider.generateToken(userDetails);
        
        return ResponseEntity.ok(new AuthResponse(token));
    }
}
```

---

### Q34: What is @PreAuthorize and method-level security?

**Answer:**

```java
// Enable method security
@Configuration
@EnableMethodSecurity  // Spring Security 6.x
// @EnableGlobalMethodSecurity(prePostEnabled = true)  // Older versions
public class MethodSecurityConfig { }

// Service with method-level security
@Service
public class UserService {
    
    // Only ADMIN can access
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long userId) { }
    
    // ADMIN or owner can access
    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public User getUser(Long userId) { }
    
    // Check permission
    @PreAuthorize("hasPermission(#document, 'write')")
    public void updateDocument(Document document) { }
    
    // Post authorization - check after method execution
    @PostAuthorize("returnObject.owner == authentication.name")
    public Resource getResource(Long id) { }
    
    // Filter collections
    @PreFilter("filterObject.owner == authentication.name")
    public void deleteResources(List<Resource> resources) { }
    
    @PostFilter("filterObject.owner == authentication.name")
    public List<Resource> getAllResources() { }
    
    // Complex expressions
    @PreAuthorize("""
        hasRole('ADMIN') or 
        (hasRole('USER') and @userService.isOwner(#id, authentication.principal.id))
        """)
    public void complexCheck(Long id) { }
    
    // Using @Secured (simpler, less flexible)
    @Secured({"ROLE_ADMIN", "ROLE_SUPER_ADMIN"})
    public void adminOnlyMethod() { }
    
    // Using @RolesAllowed (JSR-250)
    @RolesAllowed({"ADMIN"})
    public void jsr250Method() { }
}
```

---

## Spring Boot Actuator

### Q35: What is Spring Boot Actuator? What endpoints does it provide?

**Answer:**
Spring Boot Actuator provides production-ready features for monitoring and managing applications.

**Add Dependency:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**Important Endpoints:**

| Endpoint | Description |
|----------|-------------|
| `/actuator/health` | Application health status |
| `/actuator/info` | Application information |
| `/actuator/metrics` | Application metrics |
| `/actuator/loggers` | View/modify logger levels |
| `/actuator/env` | Environment properties |
| `/actuator/beans` | All Spring beans |
| `/actuator/mappings` | Request mappings |
| `/actuator/threaddump` | Thread dump |
| `/actuator/heapdump` | Heap dump file |
| `/actuator/prometheus` | Prometheus metrics |
| `/actuator/shutdown` | Graceful shutdown (disabled by default) |
| `/actuator/caches` | Application caches |
| `/actuator/scheduledtasks` | Scheduled tasks |
| `/actuator/httptrace` | HTTP request traces |

**Configuration:**
```properties
# Expose endpoints
management.endpoints.web.exposure.include=health,info,metrics,loggers
# Or expose all
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=shutdown

# Change actuator base path
management.endpoints.web.base-path=/manage

# Health endpoint details
management.endpoint.health.show-details=always
management.endpoint.health.show-components=always

# Enable shutdown endpoint
management.endpoint.shutdown.enabled=true

# Custom port for actuator
management.server.port=8081

# Info endpoint
info.app.name=@project.name@
info.app.version=@project.version@
info.app.description=@project.description@
```

**Custom Health Indicator:**
```java
@Component
public class CustomHealthIndicator implements HealthIndicator {
    
    @Override
    public Health health() {
        boolean isHealthy = checkExternalService();
        
        if (isHealthy) {
            return Health.up()
                .withDetail("service", "Available")
                .withDetail("version", "1.0")
                .build();
        }
        
        return Health.down()
            .withDetail("service", "Unavailable")
            .withDetail("error", "Connection failed")
            .build();
    }
    
    private boolean checkExternalService() {
        // Check database, external API, etc.
        return true;
    }
}

// Using ReactiveHealthIndicator for reactive apps
@Component
public class ReactiveCustomHealthIndicator implements ReactiveHealthIndicator {
    
    @Override
    public Mono<Health> health() {
        return checkService()
            .map(status -> Health.up().withDetail("status", status).build())
            .onErrorResume(ex -> Mono.just(Health.down(ex).build()));
    }
}
```

**Custom Metrics:**
```java
@Component
public class CustomMetrics {
    
    private final Counter orderCounter;
    private final Timer orderProcessingTimer;
    private final AtomicInteger activeOrders;
    
    public CustomMetrics(MeterRegistry registry) {
        this.orderCounter = Counter.builder("orders.created")
            .description("Number of orders created")
            .tag("type", "online")
            .register(registry);
            
        this.orderProcessingTimer = Timer.builder("orders.processing.time")
            .description("Time to process orders")
            .register(registry);
            
        this.activeOrders = registry.gauge("orders.active", 
            new AtomicInteger(0));
    }
    
    public void orderCreated() {
        orderCounter.increment();
    }
    
    public void recordProcessingTime(Runnable task) {
        orderProcessingTimer.record(task);
    }
}
```

---

### Q36: How to secure Actuator endpoints?

**Answer:**

```java
@Configuration
@EnableWebSecurity
public class ActuatorSecurityConfig {
    
    @Bean
    public SecurityFilterChain actuatorSecurityFilterChain(HttpSecurity http) 
            throws Exception {
        http
            .securityMatcher("/actuator/**")
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/actuator/info").permitAll()
                .requestMatchers("/actuator/**").hasRole("ADMIN")
            )
            .httpBasic(Customizer.withDefaults());
        
        return http.build();
    }
}
```

**application.properties:**
```properties
# Separate security for actuator
management.server.port=8081
management.server.address=127.0.0.1

# Require authentication
spring.security.user.name=actuator
spring.security.user.password=secret
spring.security.user.roles=ADMIN
```

---

## Testing in Spring Boot

### Q37: Explain the different testing annotations in Spring Boot.

**Answer:**

| Annotation | Purpose | Context Loaded |
|------------|---------|----------------|
| `@SpringBootTest` | Full integration test | Full application context |
| `@WebMvcTest` | Controller layer test | Web layer only |
| `@DataJpaTest` | Repository layer test | JPA components only |
| `@WebFluxTest` | Reactive controller test | WebFlux components |
| `@JsonTest` | JSON serialization test | JSON components |
| `@RestClientTest` | REST client test | RestTemplate components |
| `@JdbcTest` | JDBC test | JDBC components |

```java
// Full Integration Test
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ApplicationIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @LocalServerPort
    private int port;
    
    @Test
    void contextLoads() {
        ResponseEntity<String> response = restTemplate
            .getForEntity("/api/health", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    }
}

// Controller Test with MockMvc
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void shouldReturnUser() throws Exception {
        User user = new User(1L, "John", "john@example.com");
        when(userService.findById(1L)).thenReturn(user);
        
        mockMvc.perform(get("/api/users/1")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John"))
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }
    
    @Test
    void shouldCreateUser() throws Exception {
        User user = new User(null, "Jane", "jane@example.com");
        User savedUser = new User(1L, "Jane", "jane@example.com");
        
        when(userService.save(any(User.class))).thenReturn(savedUser);
        
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name": "Jane", "email": "jane@example.com"}
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1));
    }
}

// Repository Test
@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldFindByEmail() {
        User user = new User(null, "John", "john@example.com");
        entityManager.persist(user);
        entityManager.flush();
        
        Optional<User> found = userRepository.findByEmail("john@example.com");
        
        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("John");
    }
}

// Service Test with Mockito
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void shouldFindUserById() {
        User user = new User(1L, "John", "john@example.com");
        when(userRepository.findById(1L)).thenReturn(Optional.of(user));
        
        User found = userService.findById(1L);
        
        assertThat(found.getName()).isEqualTo("John");
        verify(userRepository, times(1)).findById(1L);
    }
    
    @Test
    void shouldThrowWhenUserNotFound() {
        when(userRepository.findById(99L)).thenReturn(Optional.empty());
        
        assertThrows(ResourceNotFoundException.class, 
            () -> userService.findById(99L));
    }
}
```

---

### Q38: What is @MockBean vs @Mock?

**Answer:**

| @MockBean | @Mock |
|-----------|-------|
| Spring Boot test annotation | Mockito annotation |
| Adds mock to Spring context | Pure Mockito mock |
| Replaces existing bean | No context awareness |
| Use with @SpringBootTest, @WebMvcTest | Use with @ExtendWith(MockitoExtension.class) |
| Slower (context loading) | Faster (no context) |

```java
// @MockBean - replaces bean in context
@WebMvcTest(OrderController.class)
class OrderControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean  // Replaces OrderService in context
    private OrderService orderService;
    
    @MockBean  // Replaces PaymentService in context
    private PaymentService paymentService;
    
    @Test
    void testCreateOrder() {
        when(orderService.create(any())).thenReturn(new Order(1L));
        // ...
    }
}

// @Mock - pure unit test
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    
    @Mock  // Pure Mockito mock
    private OrderRepository orderRepository;
    
    @Mock
    private PaymentGateway paymentGateway;
    
    @InjectMocks  // Inject mocks into service
    private OrderService orderService;
    
    @Test
    void testProcessOrder() {
        when(orderRepository.save(any())).thenReturn(new Order(1L));
        // ...
    }
}

// @SpyBean - partial mock
@SpringBootTest
class OrderIntegrationTest {
    
    @SpyBean  // Real bean with spy capabilities
    private OrderService orderService;
    
    @Test
    void testWithSpy() {
        // Can stub specific methods while keeping real behavior
        doReturn(mockOrder).when(orderService).findById(1L);
        
        // Real method call
        orderService.processOrder(mockOrder);
    }
}
```

---

### Q39: How to test with different profiles and configurations?

**Answer:**

```java
// Test with specific profile
@SpringBootTest
@ActiveProfiles("test")
class ProfileSpecificTest {
    // Uses application-test.properties
}

// Test with custom properties
@SpringBootTest
@TestPropertySource(properties = {
    "app.feature.enabled=true",
    "app.cache.ttl=300"
})
class CustomPropertyTest {
}

// Test with property file
@SpringBootTest
@TestPropertySource(locations = "classpath:test-config.properties")
class PropertyFileTest {
}

// Dynamic properties (e.g., for Testcontainers)
@SpringBootTest
@Testcontainers
class ContainerTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:14");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
}

// Test configuration class
@TestConfiguration
public class TestConfig {
    
    @Bean
    @Primary
    public EmailService mockEmailService() {
        return mock(EmailService.class);
    }
}

@SpringBootTest
@Import(TestConfig.class)
class WithTestConfigTest {
    @Autowired
    private EmailService emailService;  // Gets mock from TestConfig
}
```

---

### Q40: How to test REST APIs with WebTestClient?

**Answer:**

```java
// WebTestClient for reactive and non-reactive apps
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class WebTestClientTest {
    
    @Autowired
    private WebTestClient webTestClient;
    
    @Test
    void shouldGetUser() {
        webTestClient.get()
            .uri("/api/users/{id}", 1)
            .accept(MediaType.APPLICATION_JSON)
            .exchange()
            .expectStatus().isOk()
            .expectHeader().contentType(MediaType.APPLICATION_JSON)
            .expectBody()
            .jsonPath("$.name").isEqualTo("John")
            .jsonPath("$.email").exists();
    }
    
    @Test
    void shouldCreateUser() {
        User newUser = new User(null, "Jane", "jane@example.com");
        
        webTestClient.post()
            .uri("/api/users")
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(newUser)
            .exchange()
            .expectStatus().isCreated()
            .expectBody(User.class)
            .value(user -> {
                assertThat(user.getId()).isNotNull();
                assertThat(user.getName()).isEqualTo("Jane");
            });
    }
    
    @Test
    void shouldReturnNotFound() {
        webTestClient.get()
            .uri("/api/users/{id}", 999)
            .exchange()
            .expectStatus().isNotFound()
            .expectBody()
            .jsonPath("$.message").isEqualTo("User not found");
    }
    
    @Test
    void shouldReturnList() {
        webTestClient.get()
            .uri("/api/users")
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(User.class)
            .hasSize(3)
            .contains(expectedUser1, expectedUser2);
    }
}
```

---

## Profiles & Environment

### Q41: How do Spring Boot Profiles work?

**Answer:**
Profiles allow different configurations for different environments (dev, test, prod).

**Defining Profiles:**

```properties
# application.properties (default)
spring.application.name=myapp
server.port=8080

# application-dev.properties
server.port=8081
logging.level.root=DEBUG
spring.datasource.url=jdbc:h2:mem:devdb

# application-prod.properties
server.port=80
logging.level.root=WARN
spring.datasource.url=jdbc:postgresql://prod-db:5432/mydb
```

**YAML Multi-document:**
```yaml
# application.yml
spring:
  application:
    name: myapp

---
spring:
  config:
    activate:
      on-profile: dev
server:
  port: 8081
logging:
  level:
    root: DEBUG

---
spring:
  config:
    activate:
      on-profile: prod
server:
  port: 80
logging:
  level:
    root: WARN
```

**Activating Profiles:**
```bash
# Command line
java -jar app.jar --spring.profiles.active=prod

# Environment variable
export SPRING_PROFILES_ACTIVE=prod

# JVM argument
java -Dspring.profiles.active=prod -jar app.jar

# In application.properties
spring.profiles.active=dev

# Multiple profiles
spring.profiles.active=prod,aws,metrics
```

**Profile-specific Beans:**
```java
@Configuration
public class DataSourceConfig {
    
    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
    
    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:postgresql://prod-db:5432/mydb");
        return ds;
    }
    
    @Bean
    @Profile("!prod")  // Active when NOT prod
    public DataSource nonProdDataSource() {
        return new EmbeddedDatabaseBuilder().build();
    }
    
    @Bean
    @Profile({"dev", "test"})  // Multiple profiles
    public MockService mockService() {
        return new MockService();
    }
}

// Profile on class level
@Configuration
@Profile("cloud")
public class CloudConfig {
    // Only loaded when 'cloud' profile is active
}
```

---

### Q42: What is the difference between @Profile and @Conditional?

**Answer:**

| @Profile | @Conditional |
|----------|--------------|
| Simple profile-based activation | Complex condition evaluation |
| Built-in Spring annotation | Extensible mechanism |
| Based on `spring.profiles.active` | Based on any custom condition |
| Limited flexibility | Full programmatic control |

```java
// @Profile - simple profile check
@Bean
@Profile("dev")
public Service devService() { }

// @Conditional - custom condition
@Bean
@Conditional(OnLinuxCondition.class)
public Service linuxService() { }

// Custom Condition
public class OnLinuxCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, 
                          AnnotatedTypeMetadata metadata) {
        String os = context.getEnvironment().getProperty("os.name");
        return os != null && os.toLowerCase().contains("linux");
    }
}

// Combining both
@Bean
@Profile("prod")
@ConditionalOnProperty(name = "feature.new-algo.enabled", havingValue = "true")
public Algorithm newAlgorithm() { }
```

---

## Logging

### Q43: How does logging work in Spring Boot?

**Answer:**
Spring Boot uses Commons Logging internally but provides default configuration for Logback, Log4j2, or Java Util Logging.

**Default: Logback with SLF4J**

```properties
# application.properties

# Log level (TRACE, DEBUG, INFO, WARN, ERROR)
logging.level.root=INFO
logging.level.org.springframework=WARN
logging.level.com.example.myapp=DEBUG

# Log to file
logging.file.name=logs/application.log
logging.file.path=/var/logs

# File size and rotation
logging.logback.rollingpolicy.max-file-size=10MB
logging.logback.rollingpolicy.max-history=30
logging.logback.rollingpolicy.total-size-cap=1GB

# Console pattern
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n

# File pattern
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n

# Date format
logging.pattern.dateformat=yyyy-MM-dd HH:mm:ss.SSS
```

**Using Logback (logback-spring.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    
    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
    
    <springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="FILE"/>
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
    
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/app.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
</configuration>
```

**Using SLF4J in Code:**
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class UserService {
    
    private static final Logger log = LoggerFactory.getLogger(UserService.class);
    
    // Or use Lombok
    // @Slf4j on class level
    
    public User findById(Long id) {
        log.debug("Finding user by id: {}", id);
        
        try {
            User user = repository.findById(id).orElseThrow();
            log.info("Found user: {}", user.getName());
            return user;
        } catch (Exception e) {
            log.error("Error finding user with id: {}", id, e);
            throw e;
        }
    }
}
```

**Using Log4j2:**
```xml
<!-- Exclude logback, add log4j2 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

---

## Caching

### Q44: How does caching work in Spring Boot?

**Answer:**

**Enable Caching:**
```java
@SpringBootApplication
@EnableCaching
public class MyApplication { }
```

**Cache Annotations:**

```java
@Service
public class ProductService {
    
    // Cache the result
    @Cacheable(value = "products", key = "#id")
    public Product findById(Long id) {
        log.info("Fetching product from database: {}", id);
        return productRepository.findById(id).orElseThrow();
    }
    
    // Cache with condition
    @Cacheable(value = "products", key = "#id", 
               condition = "#id > 0",
               unless = "#result == null")
    public Product findByIdConditional(Long id) {
        return productRepository.findById(id).orElse(null);
    }
    
    // Update cache
    @CachePut(value = "products", key = "#product.id")
    public Product update(Product product) {
        return productRepository.save(product);
    }
    
    // Remove from cache
    @CacheEvict(value = "products", key = "#id")
    public void delete(Long id) {
        productRepository.deleteById(id);
    }
    
    // Clear entire cache
    @CacheEvict(value = "products", allEntries = true)
    public void clearCache() {
        log.info("Cache cleared");
    }
    
    // Multiple cache operations
    @Caching(
        cacheable = @Cacheable(value = "products", key = "#id"),
        evict = @CacheEvict(value = "productList", allEntries = true)
    )
    public Product findAndInvalidateList(Long id) {
        return productRepository.findById(id).orElseThrow();
    }
}
```

**Cache Configuration:**

```java
// Simple Cache (In-memory ConcurrentHashMap)
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("products", "users", "orders");
    }
}

// Caffeine Cache
@Configuration
@EnableCaching
public class CaffeineCacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .maximumSize(1000)
            .recordStats());
        return cacheManager;
    }
}

// Redis Cache
@Configuration
@EnableCaching
public class RedisCacheConfig {
    
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));
                
        return RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .withCacheConfiguration("products",
                RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofHours(1)))
            .build();
    }
}
```

**application.properties:**
```properties
# Cache type (simple, redis, caffeine, ehcache, hazelcast)
spring.cache.type=redis

# Redis
spring.redis.host=localhost
spring.redis.port=6379

# Caffeine
spring.cache.caffeine.spec=maximumSize=500,expireAfterWrite=10m
```

---

## Spring Boot with Microservices

### Q45: What Spring Cloud components support microservices?

**Answer:**

| Component | Purpose |
|-----------|---------|
| **Spring Cloud Config** | Centralized configuration management |
| **Eureka / Consul / Zookeeper** | Service discovery |
| **Spring Cloud Gateway / Zuul** | API Gateway |
| **Ribbon / Spring Cloud LoadBalancer** | Client-side load balancing |
| **Feign / OpenFeign** | Declarative REST client |
| **Resilience4j / Hystrix** | Circuit breaker, fault tolerance |
| **Spring Cloud Sleuth + Zipkin** | Distributed tracing |
| **Spring Cloud Stream** | Event-driven microservices |
| **Spring Cloud Bus** | Configuration refresh broadcast |

**Service Discovery with Eureka:**

```java
// Eureka Server
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication { }
```

```properties
# application.properties - Eureka Server
server.port=8761
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

```java
// Eureka Client (Microservice)
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication { }
```

```properties
# application.properties - Eureka Client
spring.application.name=user-service
eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
```

**OpenFeign Client:**

```java
@SpringBootApplication
@EnableFeignClients
public class OrderServiceApplication { }

@FeignClient(name = "user-service", fallback = UserClientFallback.class)
public interface UserClient {
    
    @GetMapping("/api/users/{id}")
    User getUser(@PathVariable Long id);
    
    @PostMapping("/api/users")
    User createUser(@RequestBody User user);
}

@Component
public class UserClientFallback implements UserClient {
    
    @Override
    public User getUser(Long id) {
        return new User(id, "Fallback User", "fallback@example.com");
    }
    
    @Override
    public User createUser(User user) {
        throw new ServiceUnavailableException("User service unavailable");
    }
}
```

**Circuit Breaker with Resilience4j:**

```java
@Service
public class OrderService {
    
    private final UserClient userClient;
    
    @CircuitBreaker(name = "userService", fallbackMethod = "getUserFallback")
    @Retry(name = "userService")
    @RateLimiter(name = "userService")
    public User getUser(Long userId) {
        return userClient.getUser(userId);
    }
    
    private User getUserFallback(Long userId, Exception ex) {
        log.warn("Fallback for user {}: {}", userId, ex.getMessage());
        return new User(userId, "Unknown", "unknown@example.com");
    }
}
```

```properties
# Resilience4j configuration
resilience4j.circuitbreaker.instances.userService.failure-rate-threshold=50
resilience4j.circuitbreaker.instances.userService.wait-duration-in-open-state=10s
resilience4j.circuitbreaker.instances.userService.sliding-window-size=10

resilience4j.retry.instances.userService.max-attempts=3
resilience4j.retry.instances.userService.wait-duration=1s

resilience4j.ratelimiter.instances.userService.limit-for-period=100
resilience4j.ratelimiter.instances.userService.limit-refresh-period=1s
```

---

### Q46: How to implement API Gateway in Spring Boot?

**Answer:**

**Spring Cloud Gateway:**

```java
@SpringBootApplication
public class GatewayApplication { }
```

```yaml
# application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1
            - AddRequestHeader=X-Request-Source, gateway
            
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
            - Method=GET,POST
          filters:
            - name: CircuitBreaker
              args:
                name: orderCircuitBreaker
                fallbackUri: forward:/fallback/orders
            - name: Retry
              args:
                retries: 3
                
        - id: product-service
          uri: http://localhost:8083
          predicates:
            - Path=/api/products/**
            - Header=X-Request-Id, \d+
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

**Custom Gateway Filter:**

```java
@Component
public class AuthenticationFilter implements GlobalFilter, Ordered {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String authHeader = exchange.getRequest().getHeaders()
            .getFirst(HttpHeaders.AUTHORIZATION);
            
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        
        // Validate token and proceed
        return chain.filter(exchange);
    }
    
    @Override
    public int getOrder() {
        return -1;  // Execute first
    }
}

// Route-specific filter
@Component
public class LoggingFilter extends AbstractGatewayFilterFactory<LoggingFilter.Config> {
    
    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            log.info("Request: {} {}", 
                exchange.getRequest().getMethod(),
                exchange.getRequest().getURI());
                
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                log.info("Response: {}", 
                    exchange.getResponse().getStatusCode());
            }));
        };
    }
    
    public static class Config { }
}
```

---

## Performance & Optimization

### Q47: How to optimize Spring Boot application performance?

**Answer:**

**1. JVM Tuning:**
```bash
java -Xms512m -Xmx2g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -XX:+UseStringDeduplication \
     -jar app.jar
```

**2. Connection Pool Configuration:**
```properties
# HikariCP (default)
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=300000
spring.datasource.hikari.connection-timeout=20000
spring.datasource.hikari.max-lifetime=1200000
```

**3. JPA/Hibernate Optimization:**
```properties
# Batch operations
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true

# Fetch size
spring.jpa.properties.hibernate.jdbc.fetch_size=100

# Disable open-in-view
spring.jpa.open-in-view=false

# Second level cache
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory
```

**4. Lazy Initialization:**
```properties
spring.main.lazy-initialization=true
```

**5. HTTP Compression:**
```properties
server.compression.enabled=true
server.compression.mime-types=application/json,application/xml,text/html,text/xml,text/plain
server.compression.min-response-size=1024
```

**6. Async Processing:**
```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
    
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class NotificationService {
    
    @Async
    public CompletableFuture<Void> sendNotification(User user) {
        // Non-blocking operation
        return CompletableFuture.completedFuture(null);
    }
}
```

**7. Virtual Threads (Java 21+):**
```properties
spring.threads.virtual.enabled=true
```

```java
@Configuration
public class VirtualThreadConfig {
    
    @Bean
    public TomcatProtocolHandlerCustomizer<?> protocolHandlerVirtualThreads() {
        return protocolHandler -> {
            protocolHandler.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
        };
    }
}
```

---

### Q48: How to monitor and troubleshoot Spring Boot applications?

**Answer:**

**1. Micrometer + Prometheus + Grafana:**

```properties
management.endpoints.web.exposure.include=prometheus,health,info,metrics
management.metrics.export.prometheus.enabled=true
management.metrics.tags.application=${spring.application.name}
```

```java
@Component
public class BusinessMetrics {
    
    private final MeterRegistry registry;
    
    public BusinessMetrics(MeterRegistry registry) {
        this.registry = registry;
        
        // Custom gauge
        Gauge.builder("orders.pending", this, BusinessMetrics::getPendingOrders)
            .description("Number of pending orders")
            .register(registry);
    }
    
    public void recordOrderProcessed(String status) {
        registry.counter("orders.processed", "status", status).increment();
    }
    
    public void recordProcessingTime(long timeMs) {
        registry.timer("orders.processing.time").record(timeMs, TimeUnit.MILLISECONDS);
    }
}
```

**2. Distributed Tracing (Spring Cloud Sleuth + Zipkin):**

```properties
spring.sleuth.sampler.probability=1.0
spring.zipkin.base-url=http://localhost:9411
spring.zipkin.sender.type=web
```

```java
@Service
@Slf4j
public class OrderService {
    
    @Autowired
    private Tracer tracer;
    
    public void processOrder(Order order) {
        Span span = tracer.currentSpan();
        span.tag("orderId", order.getId().toString());
        
        log.info("Processing order: {}", order.getId());  // Trace ID auto-added
        // Business logic
    }
}
```

**3. Thread Dump Analysis:**
```bash
# Get thread dump
curl http://localhost:8080/actuator/threaddump

# Or via JStack
jstack <pid> > thread_dump.txt
```

**4. Heap Dump Analysis:**
```bash
# Get heap dump
curl http://localhost:8080/actuator/heapdump -o heap.hprof

# Analyze with Eclipse MAT or VisualVM
```

---

## Deployment & DevOps

### Q49: How to containerize a Spring Boot application?

**Answer:**

**Dockerfile:**
```dockerfile
# Multi-stage build
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./mvnw clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

# Create non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Layered Dockerfile (Better caching):**
```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Copy layers in order of change frequency
COPY --from=builder /app/target/extracted/dependencies/ ./
COPY --from=builder /app/target/extracted/spring-boot-loader/ ./
COPY --from=builder /app/target/extracted/snapshot-dependencies/ ./
COPY --from=builder /app/target/extracted/application/ ./

ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

**Build with Spring Boot Maven Plugin:**
```bash
# Build OCI image
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=myapp:latest
```

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/mydb
    depends_on:
      - db
      - redis
      
  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      
  redis:
    image: redis:7-alpine
    
volumes:
  postgres_data:
```

**Kubernetes Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:latest
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod"
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "500m"
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
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```

---

### Q50: How to implement graceful shutdown?

**Answer:**

```properties
# Enable graceful shutdown
server.shutdown=graceful

# Timeout for graceful shutdown
spring.lifecycle.timeout-per-shutdown-phase=30s
```

```java
@Component
public class GracefulShutdown implements ApplicationListener<ContextClosedEvent> {
    
    private final ExecutorService executorService;
    
    @Override
    public void onApplicationEvent(ContextClosedEvent event) {
        log.info("Graceful shutdown initiated");
        
        // Stop accepting new requests
        // Complete in-flight requests
        executorService.shutdown();
        
        try {
            if (!executorService.awaitTermination(30, TimeUnit.SECONDS)) {
                executorService.shutdownNow();
            }
        } catch (InterruptedException e) {
            executorService.shutdownNow();
            Thread.currentThread().interrupt();
        }
        
        log.info("Graceful shutdown completed");
    }
}

// Using @PreDestroy
@Component
public class ResourceCleanup {
    
    @PreDestroy
    public void cleanup() {
        log.info("Cleaning up resources...");
        // Close connections, flush caches, etc.
    }
}
```

**Kubernetes Graceful Shutdown:**
```yaml
spec:
  containers:
    - name: myapp
      lifecycle:
        preStop:
          exec:
            command: ["sh", "-c", "sleep 10"]
  terminationGracePeriodSeconds: 60
```

---

## Advanced Topics

### Q51: What is Spring Boot Native and GraalVM?

**Answer:**
Spring Boot Native compiles applications ahead-of-time (AOT) using GraalVM, creating standalone executables.

**Benefits:**
- **Instant startup** (milliseconds vs seconds)
- **Lower memory footprint**
- **No JVM required at runtime**

**Limitations:**
- Longer build time
- Limited reflection support
- Not all libraries compatible

```xml
<!-- pom.xml -->
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
</plugin>
```

```bash
# Build native image
./mvnw -Pnative native:compile

# Run native executable
./target/myapp
```

```java
// Runtime hints for reflection
@RegisterReflectionForBinding({User.class, Order.class})
@Configuration
public class NativeConfig implements RuntimeHintsRegistrar {
    
    @Override
    public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
        hints.reflection()
            .registerType(User.class, MemberCategory.values());
    }
}
```

---

### Q52: Explain Spring Boot AOT (Ahead-of-Time) processing.

**Answer:**
AOT processing generates optimized code at build time instead of runtime.

**AOT Phases:**
1. **Source generation**: Generates Java source files
2. **Bytecode generation**: Generates class files
3. **Hint collection**: Collects reflection/resource hints

```java
// Custom AOT processing
@Configuration
public class MyAotConfiguration {
    
    @Bean
    static BeanFactoryInitializationAotProcessor myProcessor() {
        return (beanFactory, generationContext) -> {
            // Generate optimized code
            return (instanceContext) -> {
                // Custom bean initialization
            };
        };
    }
}
```

**Benefits:**
- Faster startup
- Reduced memory usage
- Better for containerized deployments

---

### Q53: What is Spring WebFlux and reactive programming?

**Answer:**
WebFlux is Spring's reactive web framework for building non-blocking applications.

```java
@RestController
@RequestMapping("/api/users")
public class ReactiveUserController {
    
    private final ReactiveUserRepository userRepository;
    
    @GetMapping
    public Flux<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    @GetMapping("/{id}")
    public Mono<ResponseEntity<User>> getUser(@PathVariable String id) {
        return userRepository.findById(id)
            .map(ResponseEntity::ok)
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public Mono<User> createUser(@RequestBody User user) {
        return userRepository.save(user);
    }
    
    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<User> streamUsers() {
        return userRepository.findAll()
            .delayElements(Duration.ofSeconds(1));
    }
}

// WebClient (reactive HTTP client)
@Service
public class ExternalApiService {
    
    private final WebClient webClient;
    
    public ExternalApiService(WebClient.Builder builder) {
        this.webClient = builder.baseUrl("https://api.example.com").build();
    }
    
    public Mono<Product> getProduct(String id) {
        return webClient.get()
            .uri("/products/{id}", id)
            .retrieve()
            .bodyToMono(Product.class)
            .timeout(Duration.ofSeconds(5))
            .onErrorResume(ex -> Mono.empty());
    }
    
    public Flux<Product> getAllProducts() {
        return webClient.get()
            .uri("/products")
            .retrieve()
            .bodyToFlux(Product.class);
    }
}
```

---

## Scenario-Based Questions

### Q54: How would you design a high-availability Spring Boot application?

**Answer:**

```
Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                        Load Balancer (Nginx/ALB)                │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│  App Instance │     │  App Instance │     │  App Instance │
│   (Pod 1)     │     │   (Pod 2)     │     │   (Pod 3)     │
└───────────────┘     └───────────────┘     └───────────────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ▼
        ┌─────────────────────────────────────────┐
        │           Redis (Session/Cache)         │
        │              (Cluster Mode)             │
        └─────────────────────────────────────────┘
                              │
        ┌─────────────────────────────────────────┐
        │         PostgreSQL (Primary)            │
        │              ↓ Replication              │
        │         PostgreSQL (Replica)            │
        └─────────────────────────────────────────┘
```

**Key Considerations:**

```java
// 1. Externalized session storage
@EnableRedisHttpSession
public class SessionConfig { }

// 2. Health checks
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        return checkDatabase() ? Health.up().build() : Health.down().build();
    }
}

// 3. Circuit breaker for external calls
@CircuitBreaker(name = "externalApi", fallbackMethod = "fallback")
public Data callExternalApi() { }

// 4. Idempotent operations
@PostMapping("/orders")
public Order createOrder(@RequestHeader("Idempotency-Key") String key, 
                         @RequestBody OrderRequest request) {
    return orderService.createIdempotent(key, request);
}
```

---

### Q55: How to handle database migrations in Spring Boot?

**Answer:**

**Using Flyway:**
```properties
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
spring.flyway.baseline-on-migrate=true
```

```sql
-- V1__Create_users_table.sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- V2__Add_status_column.sql
ALTER TABLE users ADD COLUMN status VARCHAR(20) DEFAULT 'ACTIVE';
```

**Using Liquibase:**
```properties
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.xml
```

```xml
<!-- db.changelog-master.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog">
    
    <changeSet id="1" author="developer">
        <createTable tableName="users">
            <column name="id" type="BIGINT" autoIncrement="true">
                <constraints primaryKey="true"/>
            </column>
            <column name="name" type="VARCHAR(100)">
                <constraints nullable="false"/>
            </column>
            <column name="email" type="VARCHAR(255)">
                <constraints unique="true" nullable="false"/>
            </column>
        </createTable>
    </changeSet>
    
    <changeSet id="2" author="developer">
        <addColumn tableName="users">
            <column name="status" type="VARCHAR(20)" defaultValue="ACTIVE"/>
        </addColumn>
    </changeSet>
    
</databaseChangeLog>
```

---

## Quick Reference Summary

### Essential Spring Boot Properties
```properties
# Server
server.port=8080
server.servlet.context-path=/api

# Database
spring.datasource.url=jdbc:postgresql://localhost:5432/db
spring.datasource.username=user
spring.datasource.password=pass
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false

# Logging
logging.level.root=INFO
logging.level.com.example=DEBUG

# Actuator
management.endpoints.web.exposure.include=health,info,metrics

# Profile
spring.profiles.active=dev
```

### Key Annotations Quick Reference
| Annotation | Purpose |
|------------|---------|
| `@SpringBootApplication` | Main application class |
| `@RestController` | REST API controller |
| `@Service` | Business logic layer |
| `@Repository` | Data access layer |
| `@Autowired` | Dependency injection |
| `@Transactional` | Transaction management |
| `@Cacheable` | Cache method results |
| `@Async` | Async method execution |
| `@Scheduled` | Scheduled tasks |
| `@ConfigurationProperties` | Type-safe configuration |

---

## Interview Tips

1. **Understand the internals**: Know how auto-configuration, DI, and AOP work
2. **Practical experience**: Be ready to discuss real projects and challenges
3. **Trade-offs**: Understand when to use different approaches
4. **Performance**: Know how to optimize and troubleshoot
5. **Microservices**: Understand distributed systems patterns
6. **Testing**: Be comfortable with different testing strategies
7. **Security**: Know OAuth2, JWT, and common security practices
8. **DevOps**: Understand containerization and deployment strategies

---

*Document Version: 1.0 | Last Updated: December 2024*