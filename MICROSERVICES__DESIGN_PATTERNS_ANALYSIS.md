# Design Patterns Analysis - HighQ Microservices

This document provides a comprehensive analysis of the design patterns used throughout the HighQ Microservices codebase.

---

## Table of Contents
1. [Repository Pattern](#1-repository-pattern)
2. [Factory Pattern](#2-factory-pattern)
3. [Strategy Pattern](#3-strategy-pattern)
4. [Adapter Pattern](#4-adapter-pattern)
5. [Decorator Pattern](#5-decorator-pattern)
6. [Proxy Pattern](#6-proxy-pattern)
7. [Template Method Pattern](#7-template-method-pattern)
8. [Chain of Responsibility Pattern](#8-chain-of-responsibility-pattern)
9. [Observer / Event-Driven Pattern](#9-observer--event-driven-pattern)
10. [Singleton Pattern](#10-singleton-pattern)
11. [Builder Pattern](#11-builder-pattern)
12. [Data Transfer Object (DTO) Pattern](#12-data-transfer-object-dto-pattern)
13. [Mapper Pattern](#13-mapper-pattern)
14. [Service Layer Pattern](#14-service-layer-pattern)
15. [Thread-Local Context Pattern](#15-thread-local-context-pattern)
16. [Layered Architecture Pattern](#16-layered-architecture-pattern)

---

## 1. Repository Pattern

### Purpose
Abstracts data access logic and provides a collection-like interface for accessing domain objects. This pattern mediates between the domain and data mapping layers using a collection-like interface.

### Usage in Codebase

**Location:** `files-repository`, `metadata-repository`, `isheet-repository`, etc.

**Example:**
```java
// files-repository/src/.../document/DocumentRepository.java
@Repository
public interface DocumentRepository {
    Optional<Document> findById(int id);
    List<Document> saveAll(List<Document> documentList);
    Document save(Document document);
    boolean exists(int id);
    // ... more data access methods
}
```

### Benefits
- **Separation of Concerns:** Domain logic is isolated from data access logic
- **Testability:** Easy to mock repositories for unit testing
- **Flexibility:** Can switch database implementations without affecting business logic
- **Cleaner Code:** Centralizes database queries in one place

---

## 2. Factory Pattern

### Purpose
Creates objects without exposing the instantiation logic to the client. The Factory pattern lets a class defer instantiation to subclasses.

### Usage in Codebase

**Location:** `files-service/src/.../event/EventPublisherFactory.java`

**Example:**
```java
@Component
public class EventPublisherFactory {
    private final Map<ActionType, EventPublisher> eventPublisherMap;

    @Autowired
    private EventPublisherFactory(List<EventPublisher> eventPublishers) {
        eventPublisherMap = eventPublishers.stream()
            .collect(Collectors.toUnmodifiableMap(
                EventPublisher::getActionType, 
                Function.identity()
            ));
    }

    public EventPublisher getEventPublisher(ActionType actionType) {
        return Optional.ofNullable(eventPublisherMap.get(actionType))
            .orElseThrow(IllegalArgumentException::new);
    }
}
```

**Also found in:**
- `AccessManagerFactory.java` - Creates access manager handlers based on security level

### Benefits
- **Loose Coupling:** Client code doesn't need to know concrete implementation classes
- **Single Responsibility:** Object creation logic is centralized
- **Open/Closed Principle:** New event publishers can be added without modifying factory code

---

## 3. Strategy Pattern

### Purpose
Defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

### Usage in Codebase

**Location:** `workflow-service/src/.../backoff/`

**Interface:**
```java
// BackoffStrategy.java
public interface BackoffStrategy {
    default void reset() {
        // noop if not applicable
    }
    long nextBackoffInMillis();
}
```

**Implementations:**
```java
// ExponentialBackoffStrategy.java
public class ExponentialBackoffStrategy implements BackoffStrategy {
    private final long minDelayInMillis;
    private final long maxDelayInMillis;
    private final double multiplier;
    
    @Override
    public long nextBackoffInMillis() {
        // Exponential backoff calculation with jitter
    }
}

// FixedBackoffStrategy.java
public class FixedBackoffStrategy implements BackoffStrategy {
    // Fixed delay implementation
}
```

### Benefits
- **Flexibility:** Easy to switch between different retry strategies
- **Testability:** Each strategy can be tested independently
- **Runtime Selection:** Strategies can be changed at runtime based on configuration

---

## 4. Adapter Pattern

### Purpose
Converts the interface of a class into another interface clients expect. Allows classes with incompatible interfaces to work together.

### Usage in Codebase

**Location:** `dashboard-builder-repository-jpa/src/.../adapter/`

**Example:**
```java
@RequiredArgsConstructor
public class CustomPageConfigurationRepositoryAdapter 
    implements CustomPageConfigurationRepository {

    private final CustomPageConfigurationJpaConverter converter;
    private final CustomPageConfigurationJpaRepository jpaRepository;

    @Override
    public CustomPageConfiguration create(CustomPageConfiguration config) {
        CustomPageConfigurationEntity entity = converter.from(config);
        return converter.to(jpaRepository.save(entity));
    }

    @Override
    public List<CustomPageConfiguration> selectAll() {
        return jpaRepository.findAll().stream()
            .map(converter::to)
            .collect(Collectors.toList());
    }
}
```

### Benefits
- **Decoupling:** Domain layer doesn't depend on JPA entities
- **Clean Architecture:** Separates persistence concerns from business logic
- **Flexibility:** Can change underlying persistence mechanism without affecting domain

---

## 5. Decorator Pattern

### Purpose
Attaches additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

### Usage in Codebase

**Location:** `highq_lib-multi-tenant-datasource/src/.../async/`

**Example:**
```java
public interface TenantAwareTaskDecorator extends TaskDecorator {}

class DefaultTenantAwareTaskDecorator implements TenantAwareTaskDecorator {
    @Override
    public @NonNull Runnable decorate(@NonNull Runnable runnable) {
        String tenantId = MultiTenantUtil.getCurrentNullableTenantId();
        return () -> {
            try {
                MultiTenantUtil.setCurrentTenantId(tenantId);
                runnable.run();
            } finally {
                MultiTenantUtil.clearTenantContext();
            }
        };
    }
}
```

**Also found in:**
- `CommonTaskDecorator.java` - Decorates async tasks with context propagation

### Benefits
- **Context Propagation:** Ensures tenant context is propagated to async threads
- **Non-Invasive:** Adds functionality without modifying existing task code
- **Composable:** Multiple decorators can be chained together

---

## 6. Proxy Pattern

### Purpose
Provides a surrogate or placeholder for another object to control access to it. In this codebase, it's implemented via Feign Clients for microservice communication.

### Usage in Codebase

**Location:** Various service modules using Feign clients

**Example:**
```java
@FeignClient(value = "isheet", url = "${isheet.url}", 
    configuration = FeignClientMetadataConfiguration.class)
public interface IsheetProxy {
    
    @PostMapping(value = "${isheet.apipath}/isheets")
    ResponseEntity<GetIsheetsResponseDTO> getIsheets(
        @RequestParam(name = "containerTypeAlias") String containerTypeAlias,
        @RequestParam(name = "containerId") String containerId,
        @RequestBody GetIsheetRequestDTO sheetDTO,
        @RequestHeader Map<String, String> headerMap);

    @GetMapping(value = "${isheet.apipath}/isheet/{isheetid}")
    ResponseEntity<GetIsheetResponseDTO> getIsheetById(
        @PathVariable(name = "isheetid") String isheetid,
        @RequestHeader Map<String, String> headerMap);
}
```

**Other Proxies:**
- `SystemCacheProxy`, `AuthProxy`, `UserDetailsProxy`, `SiteDetailsProxy`
- `ColumnProxy`, `ItemProxy`, `RuleProxy` (for metadata service)

### Benefits
- **Abstraction:** Hides complexity of HTTP communication
- **Declarative:** Simple interface definitions for remote calls
- **Centralized Configuration:** Error handling, retries configured in one place

---

## 7. Template Method Pattern

### Purpose
Defines the skeleton of an algorithm in a method, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps without changing the algorithm's structure.

### Usage in Codebase

**Location:** `files-service/src/.../security/AccessManagerHandler.java`

**Example:**
```java
@Component
@NoArgsConstructor
public abstract class AccessManagerHandler {

    // Abstract method to be implemented by subclasses
    public abstract boolean hasAccess(FilesCommonDO filesCommonDO);

    // Template method that defines the algorithm
    public final boolean checkAccess(FilesCommonDO filesCommonDO, 
            ConcurrentMap<String, String> securityAnnotationAttributes) {
        
        // Step 1: Validate security annotation attributes
        if (securityAnnotationAttributes != null && !securityAnnotationAttributes.isEmpty()) {
            for (Map.Entry<String, String> entry : securityAnnotationAttributes.entrySet()) {
                AccessAttrMapHandler handler = applicationContext.getBean(key, AccessAttrMapHandler.class);
                if (!handler.verifyAttribute(filesCommonDO).equals(value)) {
                    throw new FilesSecurityException("Access Denied");
                }
            }
        }
        // Step 2: Delegate to subclass implementation
        return hasAccess(filesCommonDO);
    }
}
```

**Concrete Implementations:**
- `DocumentModuleAccess`, `FolderShareAccess`, `SiteAdminAccess`, etc.

### Benefits
- **Code Reuse:** Common algorithm steps are in the base class
- **Consistent Behavior:** All access checks follow the same structure
- **Extensibility:** New access types only implement specific checks

---

## 8. Chain of Responsibility Pattern

### Purpose
Passes requests along a chain of handlers. Each handler decides either to process the request or pass it to the next handler in the chain.

### Usage in Codebase

**Location:** `files-service/src/.../security/access/`

**Example:**
```java
@Component(AccessConstants.DOCUMENT_MODULE_ACCESS)
public class DocumentModuleAccess extends AccessManagerHandler {

    @Autowired
    @Qualifier(AccessConstants.MODULEACCESS_CONTENT_MANAGER)
    private AccessManagerHandler moduleAccessContentManager;

    @Autowired
    @Lazy
    @Qualifier(AccessConstants.FOLDER_ACCESS)
    private AccessManagerHandler folderAccess;

    @Override
    public boolean hasAccess(FilesCommonDO filesCommonDO) {
        // Check 1: Module enabled
        if (!accessUtils.isDocumentModuleEnable(site)) {
            return false;
        }
        
        // Check 2: Delegate to content manager
        if (moduleAccessContentManager.hasAccess(filesCommonDO)) {
            return true;
        }
        
        // Check 3: Delegate to folder access
        return folderAccess.hasAccess(filesCommonDO);
    }
}
```

### Benefits
- **Flexible Security:** Multiple access checks can be composed
- **Single Responsibility:** Each handler focuses on one check
- **Decoupling:** Handlers don't know about other handlers in chain

---

## 9. Observer / Event-Driven Pattern

### Purpose
Defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified automatically.

### Usage in Codebase

**Location:** `files-service/src/.../event/`

**Publisher Interface:**
```java
public interface EventPublisher {
    @Async(ExecutorConstant.EVENT_EXECUTOR)
    @Transactional(isolation = Isolation.READ_UNCOMMITTED, propagation = Propagation.REQUIRED)
    void publishEvent(EventDTO data);
    
    ActionType getActionType();
}
```

**Event Listener (ASB Cache Listener):**
```java
@Component
public class ASBCacheListener {

    @EventListener(ApplicationReadyEvent.class)
    public void startASBTopicListenersForCache() {
        connectToASBTopic(asbConnectionString, topicName, subscriptionName);
    }

    public void messageProcessor(ServiceBusReceivedMessageContext context) {
        ServiceBusReceivedMessage message = context.getMessage();
        ASPConfigurationMessageDTO messageData = message.getBody().toObject(ASPConfigurationMessageDTO.class);
        
        if (key.filter(FilesConstants.REFRESH_DATA_CACHE::equals).isPresent()) {
            cacheManagementService.refreshDataCache(messageData.getTenantId(), ...);
        }
        context.complete();
    }
}
```

### Benefits
- **Loose Coupling:** Publishers don't need to know about subscribers
- **Async Processing:** Events can be processed asynchronously
- **Scalability:** Easy to add new event listeners without modifying publishers

---

## 10. Singleton Pattern

### Purpose
Ensures a class has only one instance and provides a global point of access to it.

### Usage in Codebase

Spring Framework manages singleton beans by default with `@Component`, `@Service`, `@Repository`, `@Controller` annotations.

**Example:**
```java
@Service
public class DocumentMetadataServiceImpl implements DocumentMetadataService {
    // Single instance managed by Spring container
}

@Component
public class EventPublisherFactory {
    // Single instance for factory
}
```

### Benefits
- **Resource Efficiency:** Single instance reduces memory footprint
- **Consistent State:** All clients use the same instance
- **Managed Lifecycle:** Spring handles creation and destruction

---

## 11. Builder Pattern

### Purpose
Separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

### Usage in Codebase

**Location:** Throughout the codebase using Lombok's `@Builder` annotation

**Example:**
```java
// Domain objects with Lombok Builder
@Builder
public class DocumentForm {
    private Integer id;
    private String name;
    private Integer folderId;
    private String status;
    // ... many more fields
}

// Usage
DocumentForm document = DocumentForm.builder()
    .id(1)
    .name("Report.pdf")
    .folderId(100)
    .status("ACTIVE")
    .build();
```

### Benefits
- **Readable Code:** Named parameters make object construction clear
- **Immutability:** Can create immutable objects with many fields
- **Flexible:** Optional fields can be omitted

---

## 12. Data Transfer Object (DTO) Pattern

### Purpose
Carries data between processes to reduce the number of method calls and separate internal domain models from external APIs.

### Usage in Codebase

**Location:** Throughout all service modules

**Examples:**
- `GetIsheetRequestDTO`, `GetIsheetResponseDTO`
- `ASPConfigurationMessageDTO`, `ASPEventDTO`
- `EventDTO`, `SmartFolderRuleDTO`

### Benefits
- **API Stability:** Internal changes don't affect external contracts
- **Security:** Only expose necessary fields
- **Optimization:** Send only required data over network

---

## 13. Mapper Pattern

### Purpose
Transforms data between different object models (e.g., entities to DTOs, domain objects to persistence entities).

### Usage in Codebase

**Location:** `files-spring-app/src/.../convertor/`

**Using MapStruct:**
```java
@Mapper(componentModel = MappingConstants.ComponentModel.SPRING)
public interface SmartFolderRuleObjectMapper {
    
    @Mapping(source = "matchAll", target = "matchAll", qualifiedByName = "booleanToByte")
    SmartFolderRuleDO mapDtoToDomain(SmartFolderRuleDTO dto);

    @Mapping(source = "matchAll", target = "matchAll", qualifiedByName = "byteToBoolean")
    SmartFolderRuleDTO mapDomainToDto(SmartFolderRuleDO domain);

    @Named("booleanToByte")
    static byte booleanToByte(boolean value) {
        return BooleanUtils.toIntegerObject(value).byteValue();
    }
}
```

**Manual Converters:**
```java
public class CustomPageConfigurationJpaConverter implements Converter<...> {
    public CustomPageConfiguration to(CustomPageConfigurationEntity entity) { ... }
    public CustomPageConfigurationEntity from(CustomPageConfiguration domain) { ... }
}
```

### Benefits
- **Separation of Concerns:** Conversion logic is isolated
- **Type Safety:** Compile-time checking of mappings
- **Maintainability:** Changes in one model don't affect others

---

## 14. Service Layer Pattern

### Purpose
Defines an application's boundary with a layer of services that establishes a set of available operations and coordinates the application's response.

### Usage in Codebase

**Structure:**
```
service/
├── DocumentService.java          (Interface)
├── impl/
│   └── DocumentServiceImpl.java  (Implementation)
```

**Example:**
```java
// Interface
public interface DocumentMetadataService {
    void updateMetadata(int documentId, MetadataDTO metadata);
    MetadataDTO getMetadata(int documentId);
}

// Implementation
@Service
public class DocumentMetadataServiceImpl implements DocumentMetadataService {
    
    @Autowired
    private DocumentRepository documentRepository;
    
    @Override
    public void updateMetadata(int documentId, MetadataDTO metadata) {
        // Business logic here
    }
}
```

### Benefits
- **Business Logic Encapsulation:** All business rules in one place
- **Transaction Management:** Services define transaction boundaries
- **Testability:** Easy to mock for unit testing

---

## 15. Thread-Local Context Pattern

### Purpose
Provides a way to store data that is specific to a particular thread, ensuring each thread has its own copy of the data.

### Usage in Codebase

**Location:** `highq_lib-multi-tenant-datasource/src/.../config/TenantContext.java`

**Example:**
```java
public class TenantContext implements Supplier<String> {
    
    private static ThreadLocal<String> currentTenant = new ThreadLocal<>();

    protected static String getCurrentTenant() {
        return currentTenant.get();
    }

    protected static void setCurrentTenant(String tenant) {
        currentTenant.set(tenant);
    }

    protected static void clearCurrentTenant() {
        currentTenant.remove();
    }

    @Override
    public String get() {
        return Optional.ofNullable(getCurrentTenant()).orElse(DEFAULT_TENANT_NAME);
    }
}
```

### Benefits
- **Multi-Tenancy Support:** Each request knows which tenant it belongs to
- **Thread Safety:** No shared mutable state between threads
- **Transparent Access:** Context available anywhere in the call stack

---

## 16. Layered Architecture Pattern

### Purpose
Organizes the application into horizontal layers where each layer has a specific role and responsibility.

### Usage in Codebase

**Structure:**
```
service-files/
├── files-client/           → API Clients (Feign)
├── files-domain/           → Domain Objects, Entities, Exceptions
├── files-repository/       → Repository Interfaces
├── files-repository-jpa/   → JPA Implementations
├── files-service/          → Business Logic
└── files-spring-app/       → Controllers, DTOs, Configuration
```

**Layer Responsibilities:**
- **Controller Layer:** HTTP request/response handling
- **Service Layer:** Business logic and orchestration
- **Repository Layer:** Data access abstraction
- **Domain Layer:** Core business entities and rules

### Benefits
- **Separation of Concerns:** Each layer has specific responsibility
- **Testability:** Layers can be tested independently
- **Maintainability:** Changes in one layer don't affect others
- **Team Scalability:** Different teams can work on different layers

---

## Summary Table

| Pattern | Primary Location | Key Benefit |
|---------|-----------------|-------------|
| Repository | `*-repository` modules | Data access abstraction |
| Factory | `EventPublisherFactory`, `AccessManagerFactory` | Object creation encapsulation |
| Strategy | `BackoffStrategy` implementations | Algorithm interchangeability |
| Adapter | `*RepositoryAdapter` classes | Interface compatibility |
| Decorator | `TenantAwareTaskDecorator` | Dynamic behavior extension |
| Proxy | Feign Client interfaces | Remote service abstraction |
| Template Method | `AccessManagerHandler` | Algorithm skeleton with customization |
| Chain of Responsibility | Access control handlers | Request processing chain |
| Observer | Event publishers/listeners | Loose coupling via events |
| Singleton | Spring-managed beans | Single instance guarantee |
| Builder | Lombok `@Builder` | Complex object construction |
| DTO | `*DTO` classes | Data transfer between layers |
| Mapper | MapStruct interfaces | Object transformation |
| Service Layer | `*ServiceImpl` classes | Business logic encapsulation |
| Thread-Local | `TenantContext` | Thread-specific storage |
| Layered Architecture | Module structure | Separation of concerns |

---

## Architectural Patterns

In addition to design patterns, the codebase follows these architectural patterns:

1. **Microservices Architecture:** Application split into independently deployable services
2. **Multi-Tenant Architecture:** Single instance serves multiple tenants via `MultiTenantRoutingDataSource`
3. **Event-Driven Architecture:** Services communicate via Azure Service Bus
4. **Clean Architecture:** Clear separation between domain, application, and infrastructure layers

---

*Document generated for HighQ Microservices codebase analysis*
