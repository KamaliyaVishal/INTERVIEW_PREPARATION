# Design Patterns Analysis - HighQ Collaborate Codebase

This document provides a comprehensive analysis of the design patterns used throughout the HighQ Collaborate Java codebase.

---

## Table of Contents
1. [Creational Patterns](#1-creational-patterns)
2. [Structural Patterns](#2-structural-patterns)
3. [Behavioral Patterns](#3-behavioral-patterns)
4. [Enterprise/Architectural Patterns](#4-enterprisearchitectural-patterns)
5. [Spring Framework Patterns](#5-spring-framework-patterns)

---

## 1. Creational Patterns

### 1.1 Singleton Pattern
**Purpose:** Ensures a class has only one instance and provides a global point of access to it.

**Usage in Codebase:**
- `ApplicationCache` - Provides centralized caching and configuration management
- `CacheManagement` - Manages application-wide caching
- `DataSourceConfig` - Database configuration singleton
- `QuartzHelper` - Scheduler singleton instance
- `ThreadManager` - Thread pool management singleton

**Example:**
```java
// ApplicationCache.java
public static ApplicationCache getInstance()
{
    return SpringApplicationContextProvider.getBeanByClassType(ApplicationCache.class);
}
```

**Benefits:**
- Centralized configuration management
- Consistent state across the application
- Memory efficiency for shared resources

---

### 1.2 Factory Pattern
**Purpose:** Creates objects without exposing the instantiation logic to the client.

**Usage in Codebase:**
- `AccessManagerFactory` - Creates access manager instances
- `MetaDataFactory` - Creates metadata objects
- `CacheFactory` - Creates different cache types
- `ItemDataTypeMapperFactory` - Creates mappers based on column types
- `MonitoredCustomFileItemFactory` - File upload handling

**Example:**
```java
// ItemDataTypeMapperFactory - creates appropriate mapper based on type
itemDataTypeMapperFactory.getMapperByTypeId(Integer.parseInt(column.getColumnType()))
    .mapDTOItemsInDataItems(column, columns, criteria);
```

**Benefits:**
- Decouples object creation from usage
- Supports open/closed principle
- Enables runtime type selection

---

### 1.3 Builder Pattern
**Purpose:** Constructs complex objects step by step, separating construction from representation.

**Usage in Codebase:**
- `ISheetColumnFilterQueryExpression.builder()` - Query expression building
- `IsheetItemRequest.builder()` - iSheet item requests
- `CreateEditIsheetItemDTO` - DTO construction
- Various DTOs with Lombok `@Builder` annotation

**Example:**
```java
ISheetColumnFilterQueryExpression iSheetColumnFilterQueryExpression = 
    ISheetColumnFilterQueryExpression.builder()
        .sheetID(sheetId)
        .viewID(viewId)
        .userID(userId)
        .localeID(localeId)
        .build();
```

**Benefits:**
- Readable code for complex object creation
- Immutable objects support
- Optional parameters handling

---

## 2. Structural Patterns

### 2.1 Adapter Pattern
**Purpose:** Allows incompatible interfaces to work together by wrapping an object with a new interface.

**Usage in Codebase:**
- `CacheDataAdapter` - Adapts cache data formats
- `UserMapAdapter` - User data adaptation
- `RightsMapAdapter` - Rights management adaptation
- `FileMapAdapter` - File handling adaptation
- XML adapters for serialization (`@XmlJavaTypeAdapter`)

**Example:**
```java
// Seclore DMS adapters for XML marshalling
@XmlJavaTypeAdapter(UserMapAdapter.class)
private Map<String, String> userMap;
```

**Benefits:**
- Legacy system integration
- Third-party library integration
- Data format transformation

---

### 2.2 Decorator Pattern
**Purpose:** Adds new behaviors to objects dynamically by wrapping them.

**Usage in Codebase:**
- `SheetItemMapperDecorator` - Extends mapper functionality
- MapStruct `@DecoratedWith` annotation usage

**Example:**
```java
@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
@DecoratedWith(SheetItemMapperDecorator.class)
public interface SheetItemMapper {
    SheetItem mapDTOtoDomain(GetIsheetItemDTO item, @Context IsheetViewItemsCriteria criteria);
}

// Decorator implementation
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
- Extends functionality without modifying original class
- Combines multiple behaviors
- Follows single responsibility principle

---

### 2.3 Wrapper/Facade Pattern
**Purpose:** Provides a simplified interface to a complex subsystem.

**Usage in Codebase:**
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

### 2.4 Filter/Chain of Responsibility Pattern
**Purpose:** Passes requests along a chain of handlers for processing.

**Usage in Codebase:**
- `SecurityFilter` - API security filtering
- `AuthenticationFilter` - Authentication processing
- `SingleSignOnFilter` - SSO processing
- `CSSFilter` - Content security filtering
- `PreventRecycleRequestAccessFilter` - Request validation
- `DisableUrlSessionFilter` - Session management

**Example:**
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
- Request/response processing pipeline
- Cross-cutting concerns separation
- Modular security implementation

---

## 3. Behavioral Patterns

### 3.1 Observer/Listener Pattern
**Purpose:** Defines a subscription mechanism to notify multiple objects about events.

**Usage in Codebase:**
- `TomcatSessionListener` - Session lifecycle events
- `SessionListenerService` - Session attribute changes
- `HazelcastSessionListener` - Distributed session events
- `HazelcastMemberInitListener` - Cluster member events
- `ShutdownListenerImpl` - Application shutdown events
- `ConfirmListenerImpl` - Message confirmation events
- `UploadListener` - File upload progress events

**Example:**
```java
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
```

**Benefits:**
- Loose coupling between event producers and consumers
- Dynamic event handling
- Multiple listeners per event

---

### 3.2 Strategy Pattern
**Purpose:** Defines a family of algorithms and makes them interchangeable.

**Usage in Codebase:**
- `TTVFileViewServiceImpl` - Strategy for TTV file viewing
- Different validation strategies in `IsheetRuleValidator`
- Multiple service implementations for different contexts

**Benefits:**
- Algorithm selection at runtime
- Eliminates conditional statements
- Easy addition of new strategies

---

### 3.3 Interceptor Pattern
**Purpose:** Intercepts and processes calls before they reach the target.

**Usage in Codebase:**
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
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
        // Post-processing logic
    }
}
```

**Benefits:**
- Cross-cutting concerns handling
- Request/response modification
- Authentication and logging

---

### 3.4 Validator Pattern
**Purpose:** Validates data before processing.

**Usage in Codebase:**
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

### 4.1 Repository Pattern
**Purpose:** Mediates between domain and data mapping layers.

**Usage in Codebase:**
- `TaskRepository` - Task data access
- `SynclyRepository` - Syncly data access
- Various repository classes in `com.os.isheet.module.repository`

**Example:**
```java
@Repository
public class TaskRepository {
    public Task findById(int taskId);
    public List<Task> findByUserId(int userId);
    public void save(Task task);
    public void delete(int taskId);
}
```

**Benefits:**
- Abstraction of data access
- Domain-centric data operations
- Testable data layer

---

### 4.2 Service Layer Pattern
**Purpose:** Defines an application's boundary with a layer of services.

**Usage in Codebase:**
- Interface + Implementation pattern:
  - `IsheetItemService` → `IsheetItemServiceImpl`
  - `TaskService` → `TaskServiceImpl`
  - `DocumentService` → `DocumentServiceImpl`
  - `IsheetRuleService` → `IsheetRuleServiceImpl`
  - `WorkflowService` → `WorkflowServiceImpl`

**Example:**
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
```

**Benefits:**
- Business logic encapsulation
- Transaction management
- Interface-based programming

---

### 4.3 DTO (Data Transfer Object) Pattern
**Purpose:** Carries data between processes to reduce method calls.

**Usage in Codebase:**
- Extensive use throughout `com.os.api.dbo` package
- `com.os.isheet.module.dto` - iSheet DTOs
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
- Reduces network calls
- Decouples layers
- API contract definition

---

### 4.4 MVC (Model-View-Controller) Pattern
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

### 4.5 Helper/Utility Pattern
**Purpose:** Provides common functionality across the application.

**Usage in Codebase:**
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

### 5.3 Mapper Pattern (MapStruct)
**Purpose:** Maps between different object types automatically.

**Usage in Codebase:**
- `SheetItemMapper` - Maps DTOs to domain objects
- `ISheetMapper` - iSheet object mapping
- `SiteGroupMapper` - Site group mapping
- Various mappers in `com.os.isheet.module.convert.mapper`

**Example:**
```java
@Mapper(componentModel = "spring")
public interface SheetItemMapper {
    @Mapping(target = "sheetID", expression = "java(criteria.getSheetID())")
    @Mapping(target = "itemID", source = "item.itemId")
    SheetItem mapDTOtoDomain(GetIsheetItemDTO item, @Context IsheetViewItemsCriteria criteria);
}
```

**Benefits:**
- Type-safe mapping
- Reduced boilerplate code
- Compile-time validation

---

### 5.4 Thread Pool Pattern
**Purpose:** Manages a pool of worker threads for concurrent operations.

**Usage in Codebase:**
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

| Pattern | Category | Primary Use |
|---------|----------|-------------|
| Singleton | Creational | Configuration & Cache Management |
| Factory | Creational | Object Creation |
| Builder | Creational | Complex Object Construction |
| Adapter | Structural | Interface Compatibility |
| Decorator | Structural | Dynamic Behavior Extension |
| Facade/Wrapper | Structural | Simplified Interface |
| Filter | Behavioral | Request Processing Pipeline |
| Observer/Listener | Behavioral | Event Handling |
| Strategy | Behavioral | Algorithm Selection |
| Interceptor | Behavioral | Cross-cutting Concerns |
| Validator | Behavioral | Data Validation |
| Repository | Enterprise | Data Access Abstraction |
| Service Layer | Enterprise | Business Logic Encapsulation |
| DTO | Enterprise | Data Transfer |
| MVC | Architectural | Application Structure |
| Dependency Injection | Spring | Loose Coupling |
| Configuration | Spring | Centralized Config |
| Mapper | Spring/MapStruct | Object Transformation |

---

## Recommendations

1. **Pattern Consistency:** Continue using established patterns consistently across new features
2. **Documentation:** Add pattern annotations/comments for maintainability
3. **Testing:** Leverage patterns like DI and Repository for better testability
4. **Evolution:** Consider introducing Command/Query patterns for complex operations

---

*Document generated: December 2024*
*Codebase: HighQ Collaborate*

