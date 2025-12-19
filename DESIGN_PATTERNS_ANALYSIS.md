# Design Patterns Analysis - Enterprise Java Codebase

This document provides a comprehensive analysis of the design patterns used throughout the enterprise Java microservices codebase.

---

## Table of Contents
1. [Creational Patterns](#1-creational-patterns)
   - [1.1 Singleton Pattern](#11-singleton-pattern)
   - [1.2 Factory Pattern](#12-factory-pattern)
   - [1.3 Builder Pattern](#13-builder-pattern)
2. [Structural Patterns](#2-structural-patterns)
   - [2.1 Adapter Pattern](#21-adapter-pattern)
   - [2.2 Decorator Pattern](#22-decorator-pattern)
   - [2.3 Proxy Pattern](#23-proxy-pattern)
   - [2.4 Facade/Wrapper Pattern](#24-facadewrapper-pattern)
3. [Behavioral Patterns](#3-behavioral-patterns)
   - [3.1 Strategy Pattern](#31-strategy-pattern)
   - [3.2 Template Method Pattern](#32-template-method-pattern)
   - [3.3 Chain of Responsibility Pattern](#33-chain-of-responsibility-pattern)
   - [3.4 Observer/Listener Pattern](#34-observerlistener-pattern)
   - [3.5 Interceptor Pattern](#35-interceptor-pattern)
   - [3.6 Validator Pattern](#36-validator-pattern)
4. [Enterprise/Architectural Patterns](#4-enterprisearchitectural-patterns)
   - [4.1 Repository Pattern](#41-repository-pattern)
   - [4.2 Service Layer Pattern](#42-service-layer-pattern)
   - [4.3 DTO (Data Transfer Object) Pattern](#43-dto-data-transfer-object-pattern)
   - [4.4 Mapper Pattern](#44-mapper-pattern)
   - [4.5 MVC Pattern](#45-mvc-model-view-controller-pattern)
   - [4.6 Layered Architecture Pattern](#46-layered-architecture-pattern)
   - [4.7 Thread-Local Context Pattern](#47-thread-local-context-pattern)
   - [4.8 Helper/Utility Pattern](#48-helperutility-pattern)
5. [Spring Framework Patterns](#5-spring-framework-patterns)
   - [5.1 Dependency Injection Pattern](#51-dependency-injection-pattern)
   - [5.2 Configuration Pattern](#52-configuration-pattern)
   - [5.3 Thread Pool Pattern](#53-thread-pool-pattern)

---

## 1. Creational Patterns

Creational patterns deal with object creation mechanisms, trying to create objects in a manner suitable to the situation.

---

### 1.1 Singleton Pattern

**Purpose:** Ensures a class has only one instance and provides a global point of access to it.

**Usage in Codebase:**

#### Application-Level Singletons:
- `ApplicationCache` - Provides centralized caching and configuration management
- `CacheManagement` - Manages application-wide caching
- `DataSourceConfig` - Database configuration singleton
- `QuartzHelper` - Scheduler singleton instance
- `ThreadManager` - Thread pool management singleton

#### Spring-Managed Singletons:
Spring Framework manages singleton beans by default with `@Component`, `@Service`, `@Repository`, `@Controller` annotations.

**Examples:**
```java
// ApplicationCache.java - Traditional Singleton
public static ApplicationCache getInstance() {
    return SpringApplicationContextProvider.getBeanByClassType(ApplicationCache.class);
}

// Spring-managed singleton
@Service
public class DocumentMetadataServiceImpl implements DocumentMetadataService {
    // Single instance managed by Spring container
}

@Component
public class EventPublisherFactory {
    // Single instance for factory
}
```

**Benefits:**
- Centralized configuration management
- Consistent state across the application
- Memory efficiency for shared resources
- Resource efficiency with single instance reducing memory footprint
- Managed lifecycle by Spring handles creation and destruction

---

### 1.2 Factory Pattern

**Purpose:** Creates objects without exposing the instantiation logic to the client. The Factory pattern lets a class defer instantiation to subclasses.

**Usage in Codebase:**

**Location:** Various factory classes throughout the codebase

**Factory Classes:**
- `AccessManagerFactory` - Creates access manager instances based on security level
- `EventPublisherFactory` - Creates event publishers based on action type
- `MetaDataFactory` - Creates metadata objects
- `CacheFactory` - Creates different cache types
- `ItemDataTypeMapperFactory` - Creates mappers based on column types
- `MonitoredCustomFileItemFactory` - File upload handling

**Examples:**
```java
// EventPublisherFactory.java
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

// ItemDataTypeMapperFactory - creates appropriate mapper based on type
itemDataTypeMapperFactory.getMapperByTypeId(Integer.parseInt(column.getColumnType()))
    .mapDTOItemsInDataItems(column, columns, criteria);
```

**Benefits:**
- **Loose Coupling:** Client code doesn't need to know concrete implementation classes
- **Single Responsibility:** Object creation logic is centralized
- **Open/Closed Principle:** New implementations can be added without modifying factory code
- Enables runtime type selection

---

### 1.3 Builder Pattern

**Purpose:** Separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

**Usage in Codebase:**

**Location:** Throughout the codebase using Lombok's `@Builder` annotation and manual builders

**Builder Usages:**
- `ISheetColumnFilterQueryExpression.builder()` - Query expression building
- `IsheetItemRequest.builder()` - iSheet item requests
- `CreateEditIsheetItemDTO` - DTO construction
- `DocumentForm` - Document form construction
- Various DTOs with Lombok `@Builder` annotation

**Examples:**
```java
// Query Expression Builder
ISheetColumnFilterQueryExpression iSheetColumnFilterQueryExpression = 
    ISheetColumnFilterQueryExpression.builder()
        .sheetID(sheetId)
        .viewID(viewId)
        .userID(userId)
        .localeID(localeId)
        .build();

// Domain Object with Lombok Builder
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

**Benefits:**
- **Readable Code:** Named parameters make object construction clear
- **Immutability:** Can create immutable objects with many fields
- **Flexible:** Optional fields can be omitted
- Handles optional parameters elegantly

---

## 2. Structural Patterns

Structural patterns deal with object composition, creating relationships between objects to form larger structures.

---

### 2.1 Adapter Pattern

**Purpose:** Converts the interface of a class into another interface clients expect. Allows classes with incompatible interfaces to work together.

**Usage in Codebase:**

**Location:** Repository adapters, XML adapters, data adapters

**Adapter Classes:**
- `CustomPageConfigurationRepositoryAdapter` - Adapts JPA repository to domain repository
- `CacheDataAdapter` - Adapts cache data formats
- `UserMapAdapter` - User data adaptation
- `RightsMapAdapter` - Rights management adaptation
- `FileMapAdapter` - File handling adaptation
- XML adapters for serialization (`@XmlJavaTypeAdapter`)

**Examples:**
```java
// Repository Adapter - adapts JPA to domain interface
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

// XML Adapter for serialization
@XmlJavaTypeAdapter(UserMapAdapter.class)
private Map<String, String> userMap;
```

**Benefits:**
- **Decoupling:** Domain layer doesn't depend on JPA entities
- **Clean Architecture:** Separates persistence concerns from business logic
- **Flexibility:** Can change underlying persistence mechanism without affecting domain
- Legacy system integration
- Third-party library integration

---

### 2.2 Decorator Pattern

**Purpose:** Attaches additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

**Usage in Codebase:**

**Location:** Task decorators, mapper decorators

**Decorator Classes:**
- `TenantAwareTaskDecorator` - Adds tenant context to async tasks
- `CommonTaskDecorator` - Decorates async tasks with context propagation
- `SheetItemMapperDecorator` - Extends mapper functionality

**Examples:**
```java
// Tenant-Aware Task Decorator
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

// MapStruct Decorator
@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
@DecoratedWith(SheetItemMapperDecorator.class)
public interface SheetItemMapper {
    SheetItem mapDTOtoDomain(GetIsheetItemDTO item, @Context IsheetViewItemsCriteria criteria);
}

public abstract class SheetItemMapperDecorator implements SheetItemMapper {
    @Autowired
    @Qualifier("delegate")
    private SheetItemMapper delegate;
    
    public SheetItem mapDTOtoDomain(GetIsheetItemDTO item, @Context IsheetViewItemsCriteria criteria) {
        SheetItem sheetItem = delegate.mapDTOtoDomain(item, criteria);
        // Add additional behavior
        return sheetItem;
    }
}
```

**Benefits:**
- **Context Propagation:** Ensures tenant context is propagated to async threads
- **Non-Invasive:** Adds functionality without modifying existing code
- **Composable:** Multiple decorators can be chained together
- Follows single responsibility principle

---

### 2.3 Proxy Pattern

**Purpose:** Provides a surrogate or placeholder for another object to control access to it.

**Usage in Codebase:**

**Location:** Feign clients for microservice communication

**Proxy Classes:**
- `IsheetProxy` - Proxy for iSheet service
- `SystemCacheProxy` - System cache service proxy
- `AuthProxy` - Authentication service proxy
- `UserDetailsProxy` - User details service proxy
- `SiteDetailsProxy` - Site details service proxy
- `ColumnProxy`, `ItemProxy`, `RuleProxy` - Metadata service proxies

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

**Benefits:**
- **Abstraction:** Hides complexity of HTTP communication
- **Declarative:** Simple interface definitions for remote calls
- **Centralized Configuration:** Error handling, retries configured in one place

---

### 2.4 Facade/Wrapper Pattern

**Purpose:** Provides a simplified interface to a complex subsystem.

**Usage in Codebase:**

**Location:** Module wrappers, document wrappers

**Wrapper Classes:**
- `DocumentWrapper` - Simplifies document operations
- `ISheetModuleWrapper` - Wraps iSheet module functionality
- `SheetWrapper` - Sheet operations facade

**Example:**
```java
// DocumentWrapper provides simplified document API
public class DocumentWrapper {
    // Wraps complex document operations into simpler methods
    public DocumentDBO getDocument(int docId, User user);
    public void insertDocument(DocumentDBO document);
    public void updateDocument(DocumentDBO document);
}
```

**Benefits:**
- Simplified API for complex operations
- Reduces coupling between subsystems
- Single entry point for functionality

---

## 3. Behavioral Patterns

Behavioral patterns deal with communication between objects, defining how objects interact and distribute responsibility.

---

### 3.1 Strategy Pattern

**Purpose:** Defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

**Usage in Codebase:**

**Location:** Backoff strategies, validation strategies, service implementations

**Strategy Implementations:**

**Backoff Strategies:**
```java
// BackoffStrategy.java - Interface
public interface BackoffStrategy {
    default void reset() {
        // noop if not applicable
    }
    long nextBackoffInMillis();
}

// ExponentialBackoffStrategy.java - Concrete Strategy
public class ExponentialBackoffStrategy implements BackoffStrategy {
    private final long minDelayInMillis;
    private final long maxDelayInMillis;
    private final double multiplier;
    
    @Override
    public long nextBackoffInMillis() {
        // Exponential backoff calculation with jitter
    }
}

// FixedBackoffStrategy.java - Concrete Strategy
public class FixedBackoffStrategy implements BackoffStrategy {
    // Fixed delay implementation
}
```

**Other Strategy Usages:**
- `TTVFileViewServiceImpl` - Strategy for TTV file viewing
- Different validation strategies in `IsheetRuleValidator`
- Multiple service implementations for different contexts

**Benefits:**
- **Flexibility:** Easy to switch between different strategies
- **Testability:** Each strategy can be tested independently
- **Runtime Selection:** Strategies can be changed at runtime based on configuration
- Eliminates conditional statements

---

### 3.2 Template Method Pattern

**Purpose:** Defines the skeleton of an algorithm in a method, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps without changing the algorithm's structure.

**Usage in Codebase:**

**Location:** Access manager handlers

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
- `DocumentModuleAccess`
- `FolderShareAccess`
- `SiteAdminAccess`

**Benefits:**
- **Code Reuse:** Common algorithm steps are in the base class
- **Consistent Behavior:** All access checks follow the same structure
- **Extensibility:** New access types only implement specific checks

---

### 3.3 Chain of Responsibility Pattern

**Purpose:** Passes requests along a chain of handlers. Each handler decides either to process the request or pass it to the next handler in the chain.

**Usage in Codebase:**

**Location:** Security filters, access control handlers

**Filter Chain:**
- `SecurityFilter` - API security filtering
- `AuthenticationFilter` - Authentication processing
- `SingleSignOnFilter` - SSO processing
- `CSSFilter` - Content security filtering
- `PreventRecycleRequestAccessFilter` - Request validation
- `DisableUrlSessionFilter` - Session management

**Access Control Chain:**
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

**Filter Example:**
```java
public class SecurityFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        // Security validation
        // Pass to next filter in chain
        chain.doFilter(request, response);
    }
}
```

**Benefits:**
- **Flexible Security:** Multiple access checks can be composed
- **Single Responsibility:** Each handler focuses on one check
- **Decoupling:** Handlers don't know about other handlers in chain
- Modular security implementation

---

### 3.4 Observer/Listener Pattern

**Purpose:** Defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified automatically.

**Usage in Codebase:**

**Location:** Session listeners, event publishers, message listeners

**Listener Classes:**
- `TomcatSessionListener` - Session lifecycle events
- `SessionListenerService` - Session attribute changes
- `HazelcastSessionListener` - Distributed session events
- `HazelcastMemberInitListener` - Cluster member events
- `ShutdownListenerImpl` - Application shutdown events
- `ConfirmListenerImpl` - Message confirmation events
- `UploadListener` - File upload progress events
- `ASBCacheListener` - Azure Service Bus cache events

**Examples:**
```java
// Session Listener
@Component
public class TomcatSessionListener implements HttpSessionListener {
    @Override
    public void sessionCreated(HttpSessionEvent event) {
        // Handle session creation
    }
    
    @Override
    public void sessionDestroyed(HttpSessionEvent event) {
        // Handle session destruction
    }
}

// Event Publisher Interface
public interface EventPublisher {
    @Async(ExecutorConstant.EVENT_EXECUTOR)
    @Transactional(isolation = Isolation.READ_UNCOMMITTED, propagation = Propagation.REQUIRED)
    void publishEvent(EventDTO data);
    
    ActionType getActionType();
}

// ASB Cache Listener
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

**Benefits:**
- **Loose Coupling:** Publishers don't need to know about subscribers
- **Async Processing:** Events can be processed asynchronously
- **Scalability:** Easy to add new event listeners without modifying publishers
- Dynamic event handling

---

### 3.5 Interceptor Pattern

**Purpose:** Intercepts and processes calls before they reach the target.

**Usage in Codebase:**

**Location:** HTTP interceptors, authentication interceptors

**Interceptor Classes:**
- `CustomRequestResponseHandlerInterceptor` - HTTP request/response handling
- `BearerAuthenticationInterceptor` - Authentication token interception
- `ProcessExecuteAndWaitInterceptor` - Async processing
- `GriffinExecuteAndWaitInterceptor` - Griffin-specific interception

**Example:**
```java
public class CustomRequestResponseHandlerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // Pre-processing logic
        return true;
    }
    
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, 
            Object handler, ModelAndView modelAndView) {
        // Post-processing logic
    }
}
```

**Benefits:**
- Cross-cutting concerns handling
- Request/response modification
- Authentication and logging

---

### 3.6 Validator Pattern

**Purpose:** Validates data before processing.

**Usage in Codebase:**

**Location:** Item validators, rule validators, permission validators

**Validator Classes:**
- `IsheetItemRuleValidator` - iSheet item validation
- `IsheetRuleValidator` - Rule validation
- `ISheetValidator` - Sheet validation
- `ISheetColumnPermissionValidator` - Column permission checks
- `ISheetImportValidatorDetails` - Import validation
- `TaskValidationUtil` - Task validation

**Example:**
```java
public class IsheetRuleValidator {
    public ValidationResult validate(IsheetRule rule) {
        // Validation logic
        return validationResult;
    }
}
```

**Benefits:**
- Centralized validation logic
- Reusable validation rules
- Separation from business logic

---

## 4. Enterprise/Architectural Patterns

Enterprise patterns address common challenges in enterprise application development.

---

### 4.1 Repository Pattern

**Purpose:** Abstracts data access logic and provides a collection-like interface for accessing domain objects. Mediates between the domain and data mapping layers.

**Usage in Codebase:**

**Location:** `*-repository` modules

**Repository Classes:**
- `DocumentRepository` - Document data access
- `TaskRepository` - Task data access
- `SynclyRepository` - Syncly data access
- Various repositories in `com.os.isheet.module.repository`

**Examples:**
```java
// Repository Interface
@Repository
public interface DocumentRepository {
    Optional<Document> findById(int id);
    List<Document> saveAll(List<Document> documentList);
    Document save(Document document);
    boolean exists(int id);
    // ... more data access methods
}

// Task Repository
@Repository
public class TaskRepository {
    public Task findById(int taskId);
    public List<Task> findByUserId(int userId);
    public void save(Task task);
    public void delete(int taskId);
}
```

**Benefits:**
- **Separation of Concerns:** Domain logic is isolated from data access logic
- **Testability:** Easy to mock repositories for unit testing
- **Flexibility:** Can switch database implementations without affecting business logic
- **Cleaner Code:** Centralizes database queries in one place

---

### 4.2 Service Layer Pattern

**Purpose:** Defines an application's boundary with a layer of services that establishes a set of available operations and coordinates the application's response.

**Usage in Codebase:**

**Structure:**
```
service/
├── DocumentService.java          (Interface)
├── impl/
│   └── DocumentServiceImpl.java  (Implementation)
```

**Service Interfaces and Implementations:**
- `IsheetItemService` → `IsheetItemServiceImpl`
- `TaskService` → `TaskServiceImpl`
- `DocumentService` → `DocumentServiceImpl`
- `IsheetRuleService` → `IsheetRuleServiceImpl`
- `WorkflowService` → `WorkflowServiceImpl`
- `DocumentMetadataService` → `DocumentMetadataServiceImpl`

**Examples:**
```java
// Service Interface
public interface IsheetItemService {
    IsheetViewItemsObject getISheetItems(IsheetViewItemsCriteria criteria);
    IsheetItemResponseDBO createSheetItem(IsheetItemRequest request);
    IsheetEditItemResponseDBO updateSheetItem(IsheetEditItemRequest request);
}

// Service Implementation
@Service
public class IsheetItemServiceImpl implements IsheetItemService {
    @Override
    public IsheetViewItemsObject getISheetItems(IsheetViewItemsCriteria criteria) {
        // Implementation
    }
}

// Metadata Service
public interface DocumentMetadataService {
    void updateMetadata(int documentId, MetadataDTO metadata);
    MetadataDTO getMetadata(int documentId);
}

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

**Benefits:**
- **Business Logic Encapsulation:** All business rules in one place
- **Transaction Management:** Services define transaction boundaries
- **Testability:** Easy to mock for unit testing
- Interface-based programming

---

### 4.3 DTO (Data Transfer Object) Pattern

**Purpose:** Carries data between processes to reduce the number of method calls and separate internal domain models from external APIs.

**Usage in Codebase:**

**Location:** Throughout all service modules

**DTO Classes:**
- `GetIsheetRequestDTO`, `GetIsheetResponseDTO`
- `ASPConfigurationMessageDTO`, `ASPEventDTO`
- `EventDTO`, `SmartFolderRuleDTO`
- `CreateEditIsheetItemDTO`
- Request/Response DTOs for all API operations

**Example:**
```java
public class CreateEditIsheetItemDTO {
    private Integer sheetID;
    private Integer itemID;
    private List<ColumnDataDTO> columns;
    // Getters and setters
}
```

**Benefits:**
- **API Stability:** Internal changes don't affect external contracts
- **Security:** Only expose necessary fields
- **Optimization:** Send only required data over network
- Decouples layers

---

### 4.4 Mapper Pattern

**Purpose:** Transforms data between different object models (e.g., entities to DTOs, domain objects to persistence entities).

**Usage in Codebase:**

**Location:** Converter classes, MapStruct mappers

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

// Sheet Item Mapper
@Mapper(componentModel = "spring")
public interface SheetItemMapper {
    @Mapping(target = "sheetID", expression = "java(criteria.getSheetID())")
    @Mapping(target = "itemID", source = "item.itemId")
    SheetItem mapDTOtoDomain(GetIsheetItemDTO item, @Context IsheetViewItemsCriteria criteria);
}
```

**Manual Converters:**
```java
public class CustomPageConfigurationJpaConverter implements Converter<...> {
    public CustomPageConfiguration to(CustomPageConfigurationEntity entity) { ... }
    public CustomPageConfigurationEntity from(CustomPageConfiguration domain) { ... }
}
```

**Benefits:**
- **Separation of Concerns:** Conversion logic is isolated
- **Type Safety:** Compile-time checking of mappings
- **Maintainability:** Changes in one model don't affect others
- Reduced boilerplate code

---

### 4.5 MVC (Model-View-Controller) Pattern

**Purpose:** Separates application into three interconnected components.

**Usage in Codebase:**
- **Controllers:** `IsheetItemController`, `TaskController`, `DocumentResource`
- **Services:** Business logic layer
- **Models/Domains:** Entity classes in domain packages

**Example:**
```java
@Controller
public class IsheetItemController {
    @Autowired
    private IsheetItemService isheetItemService;
    
    @GetMapping("/items")
    public ResponseEntity<IsheetViewItemsObject> getItems(@RequestParam int sheetId) {
        return ResponseEntity.ok(isheetItemService.getISheetItems(criteria));
    }
}
```

---

### 4.6 Layered Architecture Pattern

**Purpose:** Organizes the application into horizontal layers where each layer has a specific role and responsibility.

**Usage in Codebase:**

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

**Benefits:**
- **Separation of Concerns:** Each layer has specific responsibility
- **Testability:** Layers can be tested independently
- **Maintainability:** Changes in one layer don't affect others
- **Team Scalability:** Different teams can work on different layers

---

### 4.7 Thread-Local Context Pattern

**Purpose:** Provides a way to store data that is specific to a particular thread, ensuring each thread has its own copy of the data.

**Usage in Codebase:**

**Location:** Multi-tenant context management

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

**Benefits:**
- **Multi-Tenancy Support:** Each request knows which tenant it belongs to
- **Thread Safety:** No shared mutable state between threads
- **Transparent Access:** Context available anywhere in the call stack

---

### 4.8 Helper/Utility Pattern

**Purpose:** Provides common functionality across the application.

**Usage in Codebase:**

**Helper Classes:**
- `SiteHelper` - Site-related utilities
- `IsheetItemHelper` - iSheet item utilities
- `DocumentUtils` - Document utilities
- `TaskHelper` - Task utilities
- `WorkflowUtils` - Workflow utilities
- `APIMessageCodeHelper` - API message handling

**Benefits:**
- Code reusability
- Centralized common logic
- Reduced duplication

---

## 5. Spring Framework Patterns

Patterns specific to Spring Framework usage.

---

### 5.1 Dependency Injection Pattern

**Purpose:** Injects dependencies rather than creating them inside classes.

**Usage in Codebase:**
- `@Autowired` - Field/constructor injection
- `@Component` - Component scanning
- `@Service` - Service layer beans
- `@Repository` - Data access beans
- `@Configuration` - Configuration classes

**Example:**
```java
@Service
public class IsheetItemServiceImpl implements IsheetItemService {
    @Autowired
    private IsheetColumnService isheetColumnService;
    
    @Autowired
    private TaskRepository taskRepository;
}
```

**Benefits:**
- Loose coupling
- Easier testing
- Centralized configuration

---

### 5.2 Configuration Pattern

**Purpose:** Centralizes application configuration.

**Usage in Codebase:**

**Configuration Classes:**
- `DataSourceConfig` - Database configuration
- `TomcatConfig` - Server configuration
- `HazelcastConfiguration` - Cache configuration
- `SpringSessionConfig` - Session configuration
- `WorkflowTopicConfig` - Messaging configuration

**Example:**
```java
@Configuration
public class DataSourceConfig {
    @Bean(name = "defaultDataSource")
    @Primary
    public DataSource defaultDataSource() {
        HikariDataSource hikariDataSource = createAndConfigureDataSource(url, jdbcUser, jdbcPassword);
        return hikariDataSource;
    }
}
```

---

### 5.3 Thread Pool Pattern

**Purpose:** Manages a pool of worker threads for concurrent operations.

**Usage in Codebase:**

**Thread Pool Classes:**
- `ThreadManager` - Centralized thread pool management
- `BackGroundTaskHolder` - Background task execution
- `SendBulkMessageToWorkflowThread` - Async messaging

**Example:**
```java
ExecutorService threadPool = ThreadManager.getInstance().getCachedThreadPool("TaskName");
threadPool.execute(new BackgroundTask());
```

**Benefits:**
- Resource management
- Concurrent processing
- Controlled parallelism

---

## Summary Table

| Pattern | Category | Primary Location | Key Benefit |
|---------|----------|------------------|-------------|
| Singleton | Creational | `ApplicationCache`, Spring beans | Single instance guarantee |
| Factory | Creational | `EventPublisherFactory`, `AccessManagerFactory` | Object creation encapsulation |
| Builder | Creational | Lombok `@Builder`, DTOs | Complex object construction |
| Adapter | Structural | `*RepositoryAdapter` classes | Interface compatibility |
| Decorator | Structural | `TenantAwareTaskDecorator`, MapStruct | Dynamic behavior extension |
| Proxy | Structural | Feign Client interfaces | Remote service abstraction |
| Facade/Wrapper | Structural | `DocumentWrapper`, `SheetWrapper` | Simplified interface |
| Strategy | Behavioral | `BackoffStrategy` implementations | Algorithm interchangeability |
| Template Method | Behavioral | `AccessManagerHandler` | Algorithm skeleton with customization |
| Chain of Responsibility | Behavioral | Access control handlers, Filters | Request processing chain |
| Observer/Listener | Behavioral | Event publishers/listeners | Loose coupling via events |
| Interceptor | Behavioral | HTTP interceptors | Cross-cutting concerns |
| Validator | Behavioral | `*Validator` classes | Data validation |
| Repository | Enterprise | `*Repository` interfaces | Data access abstraction |
| Service Layer | Enterprise | `*ServiceImpl` classes | Business logic encapsulation |
| DTO | Enterprise | `*DTO` classes | Data transfer between layers |
| Mapper | Enterprise | MapStruct interfaces | Object transformation |
| MVC | Architectural | Controllers, Services, Models | Application structure |
| Layered Architecture | Architectural | Module structure | Separation of concerns |
| Thread-Local Context | Enterprise | `TenantContext` | Thread-specific storage |
| Dependency Injection | Spring | `@Autowired`, `@Service` | Loose coupling |
| Configuration | Spring | `@Configuration` classes | Centralized config |
| Thread Pool | Spring | `ThreadManager` | Concurrent processing |

---

## Architectural Patterns

In addition to design patterns, the codebase follows these architectural patterns:

1. **Microservices Architecture:** Application split into independently deployable services
2. **Multi-Tenant Architecture:** Single instance serves multiple tenants via `MultiTenantRoutingDataSource`
3. **Event-Driven Architecture:** Services communicate via Azure Service Bus
4. **Clean Architecture:** Clear separation between domain, application, and infrastructure layers

---

## Recommendations

1. **Pattern Consistency:** Continue using established patterns consistently across new features
2. **Documentation:** Add pattern annotations/comments for maintainability
3. **Testing:** Leverage patterns like DI and Repository for better testability
4. **Evolution:** Consider introducing Command/Query patterns for complex operations

---

*Document generated: December 2024*
*Enterprise Java Microservices Codebase Analysis*
