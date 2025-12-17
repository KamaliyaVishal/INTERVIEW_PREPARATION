# Hibernate, JPA & Spring Data Interview Guide
## Complete Questions, Answers & Code Examples

---

## 📑 Table of Contents

1. [JPA Fundamentals](#1-jpa-fundamentals)
2. [Hibernate Basics](#2-hibernate-basics)
3. [Entity Mapping](#3-entity-mapping)
4. [Relationships](#4-relationships)
5. [Inheritance Mapping](#5-inheritance-mapping)
6. [JPQL & Criteria API](#6-jpql--criteria-api)
7. [Fetching Strategies](#7-fetching-strategies)
8. [Caching](#8-caching)
9. [Transactions](#9-transactions)
10. [Spring Data JPA](#10-spring-data-jpa)
11. [Query Methods](#11-query-methods)
12. [Projections](#12-projections)
13. [Specifications](#13-specifications)
14. [Auditing](#14-auditing)
15. [Performance Optimization](#15-performance-optimization)
16. [Common Problems & Solutions](#16-common-problems--solutions)
17. [Best Practices](#17-best-practices)

---

## 1. JPA Fundamentals

### Q1: What is JPA?
**Answer:** JPA (Java Persistence API) is a **specification** (not an implementation) that defines how to persist data in Java applications. It provides a standard way to map Java objects to relational database tables (ORM - Object-Relational Mapping).

```
┌─────────────────────────────────────────────────────────────────────┐
│                        JPA ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                    Java Application                          │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│                              ▼                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                  JPA Specification                           │  │
│   │    (javax.persistence / jakarta.persistence)                 │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│          ┌───────────────────┼───────────────────┐                 │
│          ▼                   ▼                   ▼                 │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐           │
│   │  Hibernate  │    │  EclipseLink│    │  OpenJPA    │           │
│   │(Implementation)  │(Implementation)  │(Implementation)          │
│   └─────────────┘    └─────────────┘    └─────────────┘           │
│          │                   │                   │                 │
│          └───────────────────┼───────────────────┘                 │
│                              ▼                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                      JDBC                                    │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│                              ▼                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                    Database                                  │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Q2: What is the difference between JPA and Hibernate?

| Aspect | JPA | Hibernate |
|--------|-----|-----------|
| Type | Specification (Interface) | Implementation |
| Package | `javax.persistence` / `jakarta.persistence` | `org.hibernate` |
| Query Language | JPQL | HQL (superset of JPQL) |
| Caching | Defines L1, L2 cache concept | Provides actual implementation |
| Vendor Lock-in | ❌ No (portable) | ✅ Yes (if using Hibernate-specific features) |
| Features | Standard features | Additional features (Criteria, Filters, etc.) |

### Q3: What are the main components of JPA?

```java
// 1. Entity - POJO mapped to database table
@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    // getters, setters
}

// 2. EntityManager - Main interface for CRUD operations
@PersistenceContext
private EntityManager entityManager;

// 3. EntityManagerFactory - Creates EntityManager instances
EntityManagerFactory emf = Persistence.createEntityManagerFactory("myPU");
EntityManager em = emf.createEntityManager();

// 4. Persistence Unit - Configuration in persistence.xml
// 5. JPQL - Query language
```

### Q4: What is EntityManager and its lifecycle?

```java
// EntityManager Methods
public interface EntityManager {
    // Persist - Make entity managed and persistent
    void persist(Object entity);
    
    // Merge - Merge detached entity
    <T> T merge(T entity);
    
    // Remove - Remove entity
    void remove(Object entity);
    
    // Find - Find by primary key
    <T> T find(Class<T> entityClass, Object primaryKey);
    
    // Get Reference (Lazy loading)
    <T> T getReference(Class<T> entityClass, Object primaryKey);
    
    // Create Query
    Query createQuery(String qlString);
    
    // Flush - Sync with database
    void flush();
    
    // Refresh - Reload from database
    void refresh(Object entity);
    
    // Detach - Remove from persistence context
    void detach(Object entity);
    
    // Clear - Clear persistence context
    void clear();
    
    // Contains - Check if managed
    boolean contains(Object entity);
}
```

---

## 2. Hibernate Basics

### Q5: What is Hibernate and its advantages?

**Answer:** Hibernate is an ORM (Object-Relational Mapping) framework and the most popular JPA implementation.

**Advantages:**
- ✅ Reduces boilerplate JDBC code
- ✅ Database independence
- ✅ Automatic table creation (DDL)
- ✅ Caching support (L1, L2)
- ✅ Lazy loading
- ✅ Transaction management
- ✅ HQL/JPQL for queries
- ✅ Automatic dirty checking

### Q6: Explain Hibernate Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         HIBERNATE ARCHITECTURE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                        Java Application                              │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                    SessionFactory (Thread-safe)                      │  │
│   │    • One per database                                                │  │
│   │    • Heavy-weight object                                             │  │
│   │    • Created once at startup                                         │  │
│   │    • Holds second-level cache                                        │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                      Session (Not thread-safe)                       │  │
│   │    • Short-lived                                                     │  │
│   │    • One per transaction/request                                     │  │
│   │    • Wraps JDBC connection                                           │  │
│   │    • First-level cache (mandatory)                                   │  │
│   │    • Gateway to persistence context                                  │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│          ┌─────────────────────────┼─────────────────────────┐             │
│          ▼                         ▼                         ▼             │
│   ┌─────────────┐         ┌─────────────┐         ┌─────────────┐         │
│   │ Transaction │         │    Query    │         │  Criteria   │         │
│   └─────────────┘         └─────────────┘         └─────────────┘         │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                        JDBC / Connection Pool                        │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                            Database                                  │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Q7: What are the entity states/lifecycle in Hibernate?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ENTITY LIFECYCLE STATES                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                              new Entity()                                   │
│                                   │                                         │
│                                   ▼                                         │
│                          ┌───────────────┐                                  │
│                          │   TRANSIENT   │                                  │
│                          │               │                                  │
│                          │ • Not in DB   │                                  │
│                          │ • No ID       │                                  │
│                          │ • Not managed │                                  │
│                          └───────┬───────┘                                  │
│                                  │                                          │
│                    persist() / save()                                       │
│                                  │                                          │
│                                  ▼                                          │
│   ┌──────────────────────────────────────────────────────────────────────┐ │
│   │                    PERSISTENT (MANAGED)                               │ │
│   │                                                                       │ │
│   │  • Has ID              • Changes auto-synced with DB                  │ │
│   │  • In Session          • Dirty checking enabled                       │ │
│   │  • In First-level cache                                               │ │
│   │                                                                       │ │
│   │  ┌─────────────────────────────────────────────────────────────────┐ │ │
│   │  │  persist(), save(), update(), merge(), saveOrUpdate(), lock()   │ │ │
│   │  │  find(), get(), load(), query results                           │ │ │
│   │  └─────────────────────────────────────────────────────────────────┘ │ │
│   └──────────────────────────────────────────────────────────────────────┘ │
│          │                    │                         │                   │
│   remove()/delete()    detach()/evict()          session.close()           │
│          │              clear()/close()                 │                   │
│          ▼                    │                         │                   │
│   ┌─────────────┐             │                         │                   │
│   │   REMOVED   │             │                         │                   │
│   │             │             ▼                         ▼                   │
│   │ • Scheduled │      ┌─────────────────────────────────────┐             │
│   │   for delete│      │            DETACHED                 │             │
│   └─────────────┘      │                                     │             │
│          │             │  • Has ID                           │             │
│   commit/flush         │  • Not in Session                   │             │
│          │             │  • Changes NOT tracked              │             │
│          ▼             │  • Can be re-attached with merge()  │             │
│   ┌─────────────┐      └─────────────────────────────────────┘             │
│   │   DELETED   │                     │                                    │
│   │  from DB    │              merge() / update()                          │
│   └─────────────┘                     │                                    │
│                                       └──────► Back to PERSISTENT          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```java
// State Examples
public class EntityStateExample {
    
    @Autowired
    private EntityManager em;
    
    public void demonstrateStates() {
        // 1. TRANSIENT - New object, not associated with session
        Employee emp = new Employee();
        emp.setName("John");
        // emp is TRANSIENT
        
        // 2. PERSISTENT - After persist/save
        em.persist(emp);
        // emp is now PERSISTENT (managed)
        
        emp.setName("John Doe");  // Change will be auto-synced
        
        // 3. DETACHED - After session close/clear/detach
        em.detach(emp);
        // emp is now DETACHED
        
        emp.setName("Jane");  // This change won't be tracked
        
        // 4. Re-attach with merge
        Employee mergedEmp = em.merge(emp);
        // mergedEmp is PERSISTENT, emp is still DETACHED
        
        // 5. REMOVED - After remove
        em.remove(mergedEmp);
        // mergedEmp is now REMOVED (will be deleted on flush)
    }
}
```

### Q8: What is the difference between get() and load()?

| Feature | `get()` / `find()` | `load()` / `getReference()` |
|---------|-------------------|----------------------------|
| Database hit | Immediate | Lazy (on property access) |
| Returns | Actual entity or `null` | Proxy object |
| If not found | Returns `null` | Throws `ObjectNotFoundException` |
| Use case | When entity might not exist | When sure entity exists |

```java
// get() - Immediate database hit
Employee emp1 = session.get(Employee.class, 1L);  // SELECT executed
if (emp1 == null) {
    System.out.println("Not found");
}

// load() - Returns proxy, lazy loading
Employee emp2 = session.load(Employee.class, 1L);  // No SELECT yet
System.out.println(emp2.getName());  // SELECT executed here

// JPA equivalents
Employee emp3 = entityManager.find(Employee.class, 1L);      // Like get()
Employee emp4 = entityManager.getReference(Employee.class, 1L);  // Like load()
```

### Q9: What is the difference between save(), persist(), update(), merge(), and saveOrUpdate()?

| Method | Return | ID Assignment | Detached Entity | Transient Entity |
|--------|--------|--------------|-----------------|------------------|
| `save()` | Generated ID | Immediate | ❌ Exception | ✅ Works |
| `persist()` | void | May delay | ❌ Exception | ✅ Works |
| `update()` | void | N/A | ✅ Works | ❌ Exception |
| `merge()` | Merged entity | N/A | ✅ Works | ✅ Works |
| `saveOrUpdate()` | void | Immediate | ✅ Works | ✅ Works |

```java
// save() - Hibernate specific, returns ID
Serializable id = session.save(employee);

// persist() - JPA standard, void return
entityManager.persist(employee);

// update() - Reattach detached entity
session.update(detachedEmployee);

// merge() - Returns new managed instance
Employee managed = entityManager.merge(detachedEmployee);

// saveOrUpdate() - Save or update based on ID
session.saveOrUpdate(employee);
```

---

## 3. Entity Mapping

### Q10: Explain basic entity annotations

```java
import jakarta.persistence.*;

@Entity  // Marks class as entity
@Table(
    name = "employees",
    schema = "hr",
    uniqueConstraints = @UniqueConstraint(columnNames = {"email"}),
    indexes = @Index(name = "idx_dept", columnList = "department_id")
)
public class Employee {
    
    @Id  // Primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "employee_id")
    private Long id;
    
    @Column(
        name = "first_name",
        nullable = false,
        length = 50,
        unique = false,
        updatable = true,
        insertable = true
    )
    private String firstName;
    
    @Column(name = "last_name", nullable = false)
    private String lastName;
    
    @Column(unique = true)
    private String email;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal salary;
    
    @Temporal(TemporalType.DATE)  // For java.util.Date
    private Date hireDate;
    
    // For Java 8+ date types, no @Temporal needed
    private LocalDate birthDate;
    private LocalDateTime createdAt;
    
    @Enumerated(EnumType.STRING)  // Store as string (recommended)
    private EmployeeStatus status;
    
    @Lob  // Large object (CLOB/BLOB)
    private String biography;
    
    @Lob
    private byte[] profilePicture;
    
    @Transient  // Not persisted
    private String tempData;
    
    @Basic(fetch = FetchType.LAZY)  // Lazy load this field
    @Column(columnDefinition = "TEXT")
    private String longDescription;
    
    @Version  // Optimistic locking
    private Long version;
    
    // Embedded object
    @Embedded
    private Address address;
    
    // Getters and setters
}

@Embeddable
public class Address {
    private String street;
    private String city;
    
    @Column(name = "zip_code")
    private String zipCode;
}
```

### Q11: What are the ID generation strategies?

```java
// 1. IDENTITY - Auto-increment (MySQL, SQL Server, PostgreSQL)
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
// INSERT executed immediately to get ID

// 2. SEQUENCE - Database sequence (Oracle, PostgreSQL)
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "emp_seq")
@SequenceGenerator(
    name = "emp_seq",
    sequenceName = "employee_sequence",
    initialValue = 1,
    allocationSize = 50  // Pre-allocate 50 IDs for performance
)
private Long id;

// 3. TABLE - Simulates sequence using table
@Id
@GeneratedValue(strategy = GenerationType.TABLE, generator = "emp_table_gen")
@TableGenerator(
    name = "emp_table_gen",
    table = "id_generator",
    pkColumnName = "gen_name",
    valueColumnName = "gen_value",
    pkColumnValue = "employee_id",
    allocationSize = 50
)
private Long id;

// 4. AUTO - Let Hibernate choose based on database
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;

// 5. UUID - Universally unique identifier
@Id
@GeneratedValue(generator = "UUID")
@GenericGenerator(name = "UUID", strategy = "org.hibernate.id.UUIDGenerator")
@Column(updatable = false, nullable = false)
private UUID id;

// 6. Composite Key with @EmbeddedId
@Entity
public class OrderItem {
    @EmbeddedId
    private OrderItemId id;
}

@Embeddable
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;
    
    // equals() and hashCode() required!
}

// 7. Composite Key with @IdClass
@Entity
@IdClass(OrderItemId.class)
public class OrderItem {
    @Id
    private Long orderId;
    
    @Id
    private Long productId;
}
```

---

## 4. Relationships

### Q12: Explain relationship mappings with examples

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         RELATIONSHIP TYPES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ONE-TO-ONE (1:1)              ONE-TO-MANY (1:N)                           │
│   ┌───────┐    ┌───────┐        ┌───────┐    ┌───────┐                     │
│   │ User  │───►│Profile│        │ Dept  │───►│ Emp   │                     │
│   └───────┘    └───────┘        │       │    │ Emp   │                     │
│                                 │       │    │ Emp   │                     │
│                                 └───────┘    └───────┘                     │
│                                                                             │
│   MANY-TO-ONE (N:1)             MANY-TO-MANY (M:N)                         │
│   ┌───────┐    ┌───────┐        ┌───────┐    ┌───────┐                     │
│   │ Emp   │───►│ Dept  │        │Student│◄──►│Course │                     │
│   │ Emp   │    │       │        │Student│    │Course │                     │
│   │ Emp   │    │       │        │Student│    │Course │                     │
│   └───────┘    └───────┘        └───────┘    └───────┘                     │
│                                    Uses join table                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### @OneToOne Relationship

```java
// Unidirectional OneToOne
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "profile_id", referencedColumnName = "id")
    private UserProfile profile;
}

@Entity
public class UserProfile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String bio;
}

// Bidirectional OneToOne
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
}

@Entity
public class UserProfile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToOne(mappedBy = "profile")  // Not the owner
    private User user;
}

// Shared Primary Key
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToOne(cascade = CascadeType.ALL, mappedBy = "user")
    @PrimaryKeyJoinColumn
    private UserProfile profile;
}

@Entity
public class UserProfile {
    @Id
    private Long id;  // Same as User's id
    
    @OneToOne
    @MapsId
    @JoinColumn(name = "id")
    private User user;
}
```

### @OneToMany and @ManyToOne Relationship

```java
// Bidirectional OneToMany / ManyToOne (Recommended)
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @OneToMany(
        mappedBy = "department",
        cascade = CascadeType.ALL,
        orphanRemoval = true,
        fetch = FetchType.LAZY
    )
    private List<Employee> employees = new ArrayList<>();
    
    // Helper methods for bidirectional sync
    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }
    
    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);
    }
}

@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToOne(fetch = FetchType.LAZY)  // LAZY recommended
    @JoinColumn(name = "department_id")
    private Department department;
}

// Unidirectional OneToMany with JoinColumn (not recommended)
@Entity
public class Department {
    @OneToMany(cascade = CascadeType.ALL)
    @JoinColumn(name = "department_id")  // Column in Employee table
    private List<Employee> employees = new ArrayList<>();
}
```

### @ManyToMany Relationship

```java
// Bidirectional ManyToMany
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_course",  // Join table name
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
    
    // Helper methods
    public void addCourse(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }
    
    public void removeCourse(Course course) {
        courses.remove(course);
        course.getStudents().remove(this);
    }
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}

// ManyToMany with extra columns (use entity for join table)
@Entity
public class Enrollment {
    @EmbeddedId
    private EnrollmentId id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("studentId")
    private Student student;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("courseId")
    private Course course;
    
    private LocalDate enrollmentDate;
    private String grade;
}

@Embeddable
public class EnrollmentId implements Serializable {
    private Long studentId;
    private Long courseId;
    
    // equals() and hashCode()
}
```

### Q13: What is the difference between mappedBy and @JoinColumn?

| Aspect | @JoinColumn | mappedBy |
|--------|-------------|----------|
| Location | Owner side (has FK) | Inverse side |
| Foreign Key | Defines FK column | Does not create FK |
| Relationship | Controls relationship | References owner's field |
| Required | On owner side | On non-owner side |

```java
// Owner side - has @JoinColumn
@Entity
public class Employee {
    @ManyToOne
    @JoinColumn(name = "department_id")  // FK column
    private Department department;
}

// Inverse side - has mappedBy
@Entity
public class Department {
    @OneToMany(mappedBy = "department")  // References field name
    private List<Employee> employees;
}
```

---

## 5. Inheritance Mapping

### Q14: What are the inheritance strategies in JPA?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      INHERITANCE STRATEGIES                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. SINGLE_TABLE (Default)     2. TABLE_PER_CLASS                          │
│  ┌─────────────────────────┐   ┌─────────────┐  ┌─────────────┐            │
│  │     payment             │   │full_time_emp│  │part_time_emp│            │
│  ├─────────────────────────┤   ├─────────────┤  ├─────────────┤            │
│  │ id                      │   │ id          │  │ id          │            │
│  │ amount                  │   │ name        │  │ name        │            │
│  │ dtype (discriminator)   │   │ salary      │  │ hourly_rate │            │
│  │ card_number (nullable)  │   └─────────────┘  └─────────────┘            │
│  │ bank_name (nullable)    │                                               │
│  └─────────────────────────┘                                               │
│                                                                             │
│  3. JOINED                                                                  │
│  ┌─────────────┐                                                           │
│  │  employee   │                                                           │
│  ├─────────────┤                                                           │
│  │ id          │                                                           │
│  │ name        │                                                           │
│  └──────┬──────┘                                                           │
│         │                                                                   │
│    ┌────┴────┐                                                             │
│    ▼         ▼                                                             │
│  ┌─────────────┐  ┌─────────────┐                                          │
│  │full_time_emp│  │part_time_emp│                                          │
│  ├─────────────┤  ├─────────────┤                                          │
│  │ id (FK)     │  │ id (FK)     │                                          │
│  │ salary      │  │ hourly_rate │                                          │
│  └─────────────┘  └─────────────┘                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```java
// 1. SINGLE_TABLE Strategy (Default) - Best performance, nullable columns
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "payment_type", discriminatorType = DiscriminatorType.STRING)
public abstract class Payment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private BigDecimal amount;
}

@Entity
@DiscriminatorValue("CREDIT_CARD")
public class CreditCardPayment extends Payment {
    private String cardNumber;
    private String expiryDate;
}

@Entity
@DiscriminatorValue("BANK_TRANSFER")
public class BankTransferPayment extends Payment {
    private String bankName;
    private String accountNumber;
}

// 2. JOINED Strategy - Normalized, JOINs needed
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
}

@Entity
@PrimaryKeyJoinColumn(name = "employee_id")
public class FullTimeEmployee extends Employee {
    private BigDecimal salary;
}

@Entity
@PrimaryKeyJoinColumn(name = "employee_id")
public class PartTimeEmployee extends Employee {
    private BigDecimal hourlyRate;
}

// 3. TABLE_PER_CLASS Strategy - No JOINs for single class, UNION for polymorphic
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Vehicle {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)  // Cannot use IDENTITY
    private Long id;
    
    private String manufacturer;
}

@Entity
public class Car extends Vehicle {
    private int numberOfDoors;
}

@Entity
public class Motorcycle extends Vehicle {
    private boolean hasSidecar;
}
```

| Strategy | Pros | Cons | Use When |
|----------|------|------|----------|
| SINGLE_TABLE | Fast queries, no JOINs | Nullable columns, less normalized | Simple hierarchy, performance critical |
| JOINED | Normalized, no nulls | Slower (JOINs), complex | Complex hierarchy, data integrity |
| TABLE_PER_CLASS | No JOINs for concrete class | UNION for polymorphic queries | Rarely querying base class |

---

## 6. JPQL & Criteria API

### Q15: What is JPQL and how is it different from SQL?

| Aspect | JPQL | SQL |
|--------|------|-----|
| Operates on | Entities/Objects | Tables/Columns |
| Case sensitivity | Entity/field names case-sensitive | Database dependent |
| Syntax | Uses entity names | Uses table names |
| Portability | Database independent | Database specific |

```java
// JPQL Examples

// Basic SELECT
String jpql = "SELECT e FROM Employee e WHERE e.salary > :salary";
List<Employee> employees = entityManager.createQuery(jpql, Employee.class)
    .setParameter("salary", 50000.0)
    .getResultList();

// SELECT with JOIN
String jpql = "SELECT e FROM Employee e JOIN e.department d WHERE d.name = :deptName";

// SELECT specific fields (returns Object[])
String jpql = "SELECT e.name, e.salary FROM Employee e";
List<Object[]> results = entityManager.createQuery(jpql).getResultList();

// SELECT with DTO (Constructor Expression)
String jpql = "SELECT NEW com.example.dto.EmployeeDTO(e.id, e.name, e.salary) FROM Employee e";
List<EmployeeDTO> dtos = entityManager.createQuery(jpql, EmployeeDTO.class).getResultList();

// Aggregate functions
String jpql = "SELECT d.name, COUNT(e), AVG(e.salary) FROM Employee e JOIN e.department d GROUP BY d.name";

// Subquery
String jpql = "SELECT e FROM Employee e WHERE e.salary > (SELECT AVG(e2.salary) FROM Employee e2)";

// Named Query (defined on entity)
@Entity
@NamedQuery(name = "Employee.findByDepartment", 
            query = "SELECT e FROM Employee e WHERE e.department.name = :deptName")
@NamedQueries({
    @NamedQuery(name = "Employee.findAll", query = "SELECT e FROM Employee e"),
    @NamedQuery(name = "Employee.findActive", query = "SELECT e FROM Employee e WHERE e.active = true")
})
public class Employee { }

// Using named query
List<Employee> emps = entityManager.createNamedQuery("Employee.findByDepartment", Employee.class)
    .setParameter("deptName", "IT")
    .getResultList();

// UPDATE and DELETE (Bulk operations)
int updated = entityManager.createQuery("UPDATE Employee e SET e.salary = e.salary * 1.1 WHERE e.department.id = :deptId")
    .setParameter("deptId", 1L)
    .executeUpdate();

int deleted = entityManager.createQuery("DELETE FROM Employee e WHERE e.active = false")
    .executeUpdate();

// Pagination
List<Employee> page = entityManager.createQuery("SELECT e FROM Employee e ORDER BY e.name", Employee.class)
    .setFirstResult(0)    // Offset
    .setMaxResults(10)    // Limit
    .getResultList();
```

### Q16: Explain Criteria API

```java
// Criteria API - Type-safe queries

@Autowired
private EntityManager em;

public List<Employee> findEmployeesByCriteria(String name, BigDecimal minSalary, Long deptId) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Employee> cq = cb.createQuery(Employee.class);
    Root<Employee> employee = cq.from(Employee.class);
    
    // Build predicates dynamically
    List<Predicate> predicates = new ArrayList<>();
    
    if (name != null) {
        predicates.add(cb.like(cb.lower(employee.get("name")), "%" + name.toLowerCase() + "%"));
    }
    
    if (minSalary != null) {
        predicates.add(cb.greaterThanOrEqualTo(employee.get("salary"), minSalary));
    }
    
    if (deptId != null) {
        Join<Employee, Department> dept = employee.join("department");
        predicates.add(cb.equal(dept.get("id"), deptId));
    }
    
    cq.where(predicates.toArray(new Predicate[0]));
    cq.orderBy(cb.asc(employee.get("name")));
    
    return em.createQuery(cq).getResultList();
}

// Criteria with aggregation
public List<Object[]> getDepartmentStats() {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class);
    Root<Employee> emp = cq.from(Employee.class);
    Join<Employee, Department> dept = emp.join("department");
    
    cq.multiselect(
        dept.get("name"),
        cb.count(emp),
        cb.avg(emp.get("salary")),
        cb.max(emp.get("salary"))
    );
    cq.groupBy(dept.get("name"));
    cq.having(cb.gt(cb.count(emp), 5L));
    
    return em.createQuery(cq).getResultList();
}

// Using Metamodel (compile-time safety)
// First generate metamodel classes with annotation processor
public List<Employee> findByNameTypeSafe(String name) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Employee> cq = cb.createQuery(Employee.class);
    Root<Employee> emp = cq.from(Employee.class);
    
    // Employee_ is the generated metamodel class
    cq.where(cb.equal(emp.get(Employee_.name), name));
    
    return em.createQuery(cq).getResultList();
}
```

---

## 7. Fetching Strategies

### Q17: What is the difference between Lazy and Eager loading?

| Aspect | LAZY (Default for collections) | EAGER |
|--------|-------------------------------|-------|
| Loading time | On first access | Immediately with parent |
| Performance | Better for large collections | Can cause performance issues |
| N+1 Problem | Can occur | Avoided but may load too much |
| Default for | @OneToMany, @ManyToMany | @ManyToOne, @OneToOne |

```java
@Entity
public class Employee {
    
    @ManyToOne(fetch = FetchType.LAZY)  // Override default EAGER
    private Department department;
    
    @OneToMany(fetch = FetchType.LAZY)  // Default
    private List<Task> tasks;
    
    @OneToOne(fetch = FetchType.LAZY)  // Override default EAGER
    private EmployeeDetails details;
}

// Forcing eager loading with JPQL
String jpql = "SELECT e FROM Employee e JOIN FETCH e.department JOIN FETCH e.tasks";

// Forcing eager loading with EntityGraph
@EntityGraph(attributePaths = {"department", "tasks"})
List<Employee> findAll();
```

### Q18: What is the N+1 Problem and how to solve it?

```java
// N+1 Problem Example
// 1 query for departments + N queries for employees (one per department)
List<Department> departments = em.createQuery("SELECT d FROM Department d", Department.class)
    .getResultList();

for (Department dept : departments) {
    // Each access triggers a new query!
    System.out.println(dept.getEmployees().size());
}

// Solution 1: JOIN FETCH
String jpql = "SELECT d FROM Department d JOIN FETCH d.employees";
List<Department> departments = em.createQuery(jpql, Department.class).getResultList();

// Solution 2: Entity Graph
@Entity
@NamedEntityGraph(
    name = "Department.withEmployees",
    attributeNodes = @NamedAttributeNode("employees")
)
public class Department { }

// Using entity graph
EntityGraph<?> graph = em.getEntityGraph("Department.withEmployees");
List<Department> departments = em.createQuery("SELECT d FROM Department d", Department.class)
    .setHint("javax.persistence.loadgraph", graph)
    .getResultList();

// Solution 3: @BatchSize (Hibernate specific)
@Entity
public class Department {
    @OneToMany(mappedBy = "department")
    @BatchSize(size = 25)  // Load in batches of 25
    private List<Employee> employees;
}

// Solution 4: Subselect fetching (Hibernate)
@Entity
public class Department {
    @OneToMany(mappedBy = "department")
    @Fetch(FetchMode.SUBSELECT)  // Uses subquery
    private List<Employee> employees;
}
```

---

## 8. Caching

### Q19: Explain First-Level and Second-Level Cache

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           HIBERNATE CACHING                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                     FIRST-LEVEL CACHE (L1)                           │  │
│   │                                                                       │  │
│   │  • Session/EntityManager scoped                                       │  │
│   │  • Enabled by default (mandatory)                                     │  │
│   │  • Cannot be disabled                                                 │  │
│   │  • Automatic - no configuration needed                                │  │
│   │  • Cleared when session closes                                        │  │
│   │                                                                       │  │
│   │  Session 1          Session 2          Session 3                      │  │
│   │  ┌─────────┐        ┌─────────┐        ┌─────────┐                   │  │
│   │  │ L1 Cache│        │ L1 Cache│        │ L1 Cache│                   │  │
│   │  └─────────┘        └─────────┘        └─────────┘                   │  │
│   │                                                                       │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                    SECOND-LEVEL CACHE (L2)                           │  │
│   │                                                                       │  │
│   │  • SessionFactory scoped (shared across sessions)                     │  │
│   │  • Disabled by default                                                │  │
│   │  • Needs explicit configuration                                       │  │
│   │  • Providers: EHCache, Infinispan, Hazelcast                          │  │
│   │  • Stores entity data, not entity objects                             │  │
│   │                                                                       │  │
│   │  ┌───────────────────────────────────────────────────────────────┐   │  │
│   │  │                    Shared L2 Cache                             │   │  │
│   │  │   Entity Cache │ Collection Cache │ Query Cache                │   │  │
│   │  └───────────────────────────────────────────────────────────────┘   │  │
│   │                                                                       │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                          DATABASE                                    │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```java
// First-Level Cache Example
Session session = sessionFactory.openSession();
Employee emp1 = session.get(Employee.class, 1L);  // DB query
Employee emp2 = session.get(Employee.class, 1L);  // From L1 cache (no query)
System.out.println(emp1 == emp2);  // true (same object)
session.close();

// Clear L1 cache
session.clear();  // Clear all
session.evict(employee);  // Clear specific entity

// Second-Level Cache Configuration
// 1. Add dependency (e.g., EHCache)
// 2. Configure in application.properties:
/*
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory
spring.jpa.properties.hibernate.cache.use_query_cache=true
*/

// 3. Enable caching on entity
@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Employee {
    // ...
    
    @OneToMany(mappedBy = "employee")
    @Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    private List<Task> tasks;  // Collection cache
}

// Cache concurrency strategies:
// READ_ONLY - Never modified, safest
// NONSTRICT_READ_WRITE - Rarely modified, eventual consistency
// READ_WRITE - Modified, strict consistency
// TRANSACTIONAL - Full transaction support (JTA required)

// Query Cache
Query query = em.createQuery("SELECT e FROM Employee e WHERE e.department.id = :deptId")
    .setParameter("deptId", 1L)
    .setHint("org.hibernate.cacheable", true);  // Enable query cache
```

---

## 9. Transactions

### Q20: Explain transaction management in JPA/Hibernate

```java
// Programmatic Transaction Management
EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();

try {
    tx.begin();
    
    Employee emp = new Employee();
    emp.setName("John");
    em.persist(emp);
    
    tx.commit();
} catch (Exception e) {
    if (tx.isActive()) {
        tx.rollback();
    }
    throw e;
} finally {
    em.close();
}

// Declarative Transaction Management (Spring)
@Service
@Transactional  // Class-level: applies to all methods
public class EmployeeService {
    
    @Autowired
    private EmployeeRepository repository;
    
    @Transactional(readOnly = true)  // Read-only transaction
    public Employee findById(Long id) {
        return repository.findById(id).orElse(null);
    }
    
    @Transactional(
        propagation = Propagation.REQUIRED,
        isolation = Isolation.READ_COMMITTED,
        timeout = 30,
        rollbackFor = Exception.class,
        noRollbackFor = NotFoundException.class
    )
    public Employee save(Employee employee) {
        return repository.save(employee);
    }
}
```

### Q21: What are the transaction propagation types?

```java
// REQUIRED (Default) - Use existing or create new
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
    // Uses existing transaction or creates new one
}

// REQUIRES_NEW - Always create new, suspend existing
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodB() {
    // Always creates new transaction
    // Existing transaction is suspended
}

// SUPPORTS - Use existing if available, otherwise non-transactional
@Transactional(propagation = Propagation.SUPPORTS)

// NOT_SUPPORTED - Execute non-transactionally, suspend existing
@Transactional(propagation = Propagation.NOT_SUPPORTED)

// MANDATORY - Must run within existing transaction
@Transactional(propagation = Propagation.MANDATORY)

// NEVER - Must NOT run within transaction
@Transactional(propagation = Propagation.NEVER)

// NESTED - Nested transaction with savepoint
@Transactional(propagation = Propagation.NESTED)
```

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     PROPAGATION BEHAVIOR                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Propagation        │ No TX Exists      │ TX Exists                       │
│   ─────────────────────────────────────────────────────────────────────     │
│   REQUIRED           │ Create new        │ Use existing                    │
│   REQUIRES_NEW       │ Create new        │ Create new (suspend existing)   │
│   SUPPORTS           │ Non-TX            │ Use existing                    │
│   NOT_SUPPORTED      │ Non-TX            │ Non-TX (suspend existing)       │
│   MANDATORY          │ Exception         │ Use existing                    │
│   NEVER              │ Non-TX            │ Exception                       │
│   NESTED             │ Create new        │ Create nested (savepoint)       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. Spring Data JPA

### Q22: What is Spring Data JPA?

**Answer:** Spring Data JPA is a Spring module that makes it easy to implement JPA-based repositories. It reduces boilerplate code by providing repository interfaces with built-in CRUD operations.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     SPRING DATA JPA ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                        Your Repository                               │  │
│   │            public interface UserRepository                           │  │
│   │            extends JpaRepository<User, Long>                         │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │ extends                               │
│                                    ▼                                       │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │               JpaRepository<T, ID>                                   │  │
│   │    flush(), saveAndFlush(), deleteInBatch(), findAll(Example)        │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │ extends                               │
│                                    ▼                                       │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │          PagingAndSortingRepository<T, ID>                           │  │
│   │    findAll(Sort), findAll(Pageable)                                  │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │ extends                               │
│                                    ▼                                       │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │               CrudRepository<T, ID>                                  │  │
│   │    save(), findById(), findAll(), count(), delete(), existsById()    │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │ extends                               │
│                                    ▼                                       │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                  Repository<T, ID>                                   │  │
│   │                    (Marker interface)                                │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```java
// Basic Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    // Inherits: save, findById, findAll, delete, count, etc.
}

// Usage
@Service
public class EmployeeService {
    
    @Autowired
    private EmployeeRepository repository;
    
    public Employee save(Employee employee) {
        return repository.save(employee);  // INSERT or UPDATE
    }
    
    public Optional<Employee> findById(Long id) {
        return repository.findById(id);
    }
    
    public List<Employee> findAll() {
        return repository.findAll();
    }
    
    public List<Employee> findAllSorted() {
        return repository.findAll(Sort.by("name").ascending());
    }
    
    public Page<Employee> findAllPaged(int page, int size) {
        return repository.findAll(PageRequest.of(page, size));
    }
    
    public void delete(Employee employee) {
        repository.delete(employee);
    }
    
    public boolean exists(Long id) {
        return repository.existsById(id);
    }
    
    public long count() {
        return repository.count();
    }
}
```

---

## 11. Query Methods

### Q23: How do derived query methods work?

```java
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    
    // Find by single property
    List<Employee> findByName(String name);
    Optional<Employee> findByEmail(String email);
    
    // Find by multiple properties
    List<Employee> findByNameAndDepartment(String name, Department dept);
    List<Employee> findByNameOrEmail(String name, String email);
    
    // Comparison operators
    List<Employee> findBySalaryGreaterThan(BigDecimal salary);
    List<Employee> findBySalaryLessThanEqual(BigDecimal salary);
    List<Employee> findBySalaryBetween(BigDecimal min, BigDecimal max);
    List<Employee> findByAgeIn(Collection<Integer> ages);
    
    // String matching
    List<Employee> findByNameLike(String pattern);        // LIKE pattern
    List<Employee> findByNameContaining(String keyword);  // LIKE %keyword%
    List<Employee> findByNameStartingWith(String prefix); // LIKE prefix%
    List<Employee> findByNameEndingWith(String suffix);   // LIKE %suffix
    List<Employee> findByNameIgnoreCase(String name);
    
    // Null checks
    List<Employee> findByManagerIsNull();
    List<Employee> findByManagerIsNotNull();
    
    // Boolean
    List<Employee> findByActiveTrue();
    List<Employee> findByActiveFalse();
    
    // Date
    List<Employee> findByHireDateAfter(LocalDate date);
    List<Employee> findByHireDateBefore(LocalDate date);
    
    // Nested property
    List<Employee> findByDepartmentName(String deptName);
    List<Employee> findByAddressCityIn(List<String> cities);
    
    // Ordering
    List<Employee> findByDepartmentOrderBySalaryDesc(Department dept);
    List<Employee> findByActiveOrderByNameAscSalaryDesc(boolean active);
    
    // Limiting results
    Employee findFirstByOrderBySalaryDesc();
    List<Employee> findTop5ByOrderBySalaryDesc();
    List<Employee> findFirst10ByDepartmentOrderBySalaryDesc(Department dept);
    
    // Distinct
    List<Employee> findDistinctByDepartmentName(String deptName);
    
    // Count, Exists, Delete
    long countByDepartmentName(String deptName);
    boolean existsByEmail(String email);
    void deleteByDepartment(Department dept);
    long deleteByActivefalse();
    
    // With Pageable and Sort
    Page<Employee> findByDepartment(Department dept, Pageable pageable);
    List<Employee> findByDepartment(Department dept, Sort sort);
    Slice<Employee> findByActive(boolean active, Pageable pageable);
}
```

### Q24: How to use @Query annotation?

```java
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    
    // JPQL Query
    @Query("SELECT e FROM Employee e WHERE e.salary > :salary")
    List<Employee> findHighEarners(@Param("salary") BigDecimal salary);
    
    // JPQL with JOIN
    @Query("SELECT e FROM Employee e JOIN e.department d WHERE d.name = :deptName")
    List<Employee> findByDepartmentName(@Param("deptName") String deptName);
    
    // JPQL with named parameters
    @Query("SELECT e FROM Employee e WHERE e.name = :name AND e.active = :active")
    List<Employee> findByNameAndActive(@Param("name") String name, @Param("active") boolean active);
    
    // JPQL with positional parameters
    @Query("SELECT e FROM Employee e WHERE e.name = ?1 AND e.salary > ?2")
    List<Employee> findByNameAndMinSalary(String name, BigDecimal minSalary);
    
    // Native SQL Query
    @Query(value = "SELECT * FROM employees WHERE department_id = ?1", nativeQuery = true)
    List<Employee> findByDepartmentIdNative(Long deptId);
    
    // Native SQL with pagination
    @Query(
        value = "SELECT * FROM employees WHERE active = true",
        countQuery = "SELECT COUNT(*) FROM employees WHERE active = true",
        nativeQuery = true
    )
    Page<Employee> findActiveNative(Pageable pageable);
    
    // Update query
    @Modifying
    @Query("UPDATE Employee e SET e.salary = e.salary * :multiplier WHERE e.department.id = :deptId")
    int updateSalaryByDepartment(@Param("deptId") Long deptId, @Param("multiplier") BigDecimal multiplier);
    
    // Delete query
    @Modifying
    @Query("DELETE FROM Employee e WHERE e.active = false")
    int deleteInactiveEmployees();
    
    // SpEL expressions
    @Query("SELECT e FROM #{#entityName} e WHERE e.department.id = :deptId")
    List<Employee> findByDeptWithSpEL(@Param("deptId") Long deptId);
    
    // Projection with DTO
    @Query("SELECT new com.example.dto.EmployeeSummary(e.id, e.name, e.salary) FROM Employee e")
    List<EmployeeSummary> findAllSummaries();
}
```

---

## 12. Projections

### Q25: What are projections in Spring Data JPA?

```java
// 1. Interface-based Closed Projection
public interface EmployeeNameOnly {
    String getName();
    String getEmail();
}

public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    List<EmployeeNameOnly> findByDepartmentId(Long deptId);
}

// 2. Interface-based Open Projection (with SpEL)
public interface EmployeeFullName {
    String getFirstName();
    String getLastName();
    
    @Value("#{target.firstName + ' ' + target.lastName}")
    String getFullName();
}

// 3. Class-based Projection (DTO)
public class EmployeeDTO {
    private final String name;
    private final BigDecimal salary;
    
    public EmployeeDTO(String name, BigDecimal salary) {
        this.name = name;
        this.salary = salary;
    }
    
    // Getters
}

public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    @Query("SELECT new com.example.dto.EmployeeDTO(e.name, e.salary) FROM Employee e")
    List<EmployeeDTO> findAllDTOs();
}

// 4. Dynamic Projections
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    <T> List<T> findByDepartmentId(Long deptId, Class<T> type);
    <T> T findByEmail(String email, Class<T> type);
}

// Usage
List<EmployeeNameOnly> names = repository.findByDepartmentId(1L, EmployeeNameOnly.class);
List<EmployeeDTO> dtos = repository.findByDepartmentId(1L, EmployeeDTO.class);

// 5. Nested Projections
public interface EmployeeWithDepartment {
    String getName();
    DepartmentSummary getDepartment();
    
    interface DepartmentSummary {
        String getName();
        String getLocation();
    }
}
```

---

## 13. Specifications

### Q26: How to use Specifications for dynamic queries?

```java
// Enable JpaSpecificationExecutor
public interface EmployeeRepository extends 
        JpaRepository<Employee, Long>, 
        JpaSpecificationExecutor<Employee> {
}

// Create Specifications
public class EmployeeSpecifications {
    
    public static Specification<Employee> hasName(String name) {
        return (root, query, cb) -> 
            name == null ? null : cb.like(cb.lower(root.get("name")), "%" + name.toLowerCase() + "%");
    }
    
    public static Specification<Employee> hasSalaryGreaterThan(BigDecimal salary) {
        return (root, query, cb) -> 
            salary == null ? null : cb.greaterThanOrEqualTo(root.get("salary"), salary);
    }
    
    public static Specification<Employee> belongsToDepartment(Long deptId) {
        return (root, query, cb) -> 
            deptId == null ? null : cb.equal(root.get("department").get("id"), deptId);
    }
    
    public static Specification<Employee> isActive() {
        return (root, query, cb) -> cb.isTrue(root.get("active"));
    }
    
    public static Specification<Employee> hireDateBetween(LocalDate start, LocalDate end) {
        return (root, query, cb) -> {
            if (start == null && end == null) return null;
            if (start == null) return cb.lessThanOrEqualTo(root.get("hireDate"), end);
            if (end == null) return cb.greaterThanOrEqualTo(root.get("hireDate"), start);
            return cb.between(root.get("hireDate"), start, end);
        };
    }
}

// Usage in Service
@Service
public class EmployeeService {
    
    @Autowired
    private EmployeeRepository repository;
    
    public List<Employee> searchEmployees(EmployeeSearchCriteria criteria) {
        Specification<Employee> spec = Specification
            .where(EmployeeSpecifications.hasName(criteria.getName()))
            .and(EmployeeSpecifications.hasSalaryGreaterThan(criteria.getMinSalary()))
            .and(EmployeeSpecifications.belongsToDepartment(criteria.getDepartmentId()))
            .and(EmployeeSpecifications.isActive());
        
        return repository.findAll(spec);
    }
    
    public Page<Employee> searchEmployeesPaged(EmployeeSearchCriteria criteria, Pageable pageable) {
        Specification<Employee> spec = Specification
            .where(EmployeeSpecifications.hasName(criteria.getName()))
            .and(EmployeeSpecifications.hasSalaryGreaterThan(criteria.getMinSalary()));
        
        return repository.findAll(spec, pageable);
    }
}
```

---

## 14. Auditing

### Q27: How to implement auditing in Spring Data JPA?

```java
// 1. Enable JPA Auditing
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaConfig {
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext())
            .map(SecurityContext::getAuthentication)
            .filter(Authentication::isAuthenticated)
            .map(Authentication::getName);
    }
}

// 2. Create Base Auditable Entity
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class Auditable {
    
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @CreatedBy
    @Column(name = "created_by", updatable = false)
    private String createdBy;
    
    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;
    
    // Getters and setters
}

// 3. Extend in your entities
@Entity
public class Employee extends Auditable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    // Other fields
}

// 4. Envers for Full Audit History (Hibernate specific)
// Add dependency: hibernate-envers

@Entity
@Audited  // Enable revision tracking
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @NotAudited  // Exclude from auditing
    private String tempData;
}

// Query revisions
@Autowired
private EntityManager em;

public List<Object[]> getEmployeeRevisions(Long employeeId) {
    AuditReader auditReader = AuditReaderFactory.get(em);
    
    return auditReader.createQuery()
        .forRevisionsOfEntity(Employee.class, false, true)
        .add(AuditEntity.id().eq(employeeId))
        .getResultList();
}
```

---

## 15. Performance Optimization

### Q28: Best practices for JPA/Hibernate performance

```java
// 1. Use LAZY fetching by default
@ManyToOne(fetch = FetchType.LAZY)
private Department department;

// 2. Avoid N+1 with JOIN FETCH or EntityGraph
@EntityGraph(attributePaths = {"department", "tasks"})
List<Employee> findAll();

// 3. Use DTO projections for read-only queries
@Query("SELECT new com.example.dto.EmployeeDTO(e.id, e.name) FROM Employee e")
List<EmployeeDTO> findAllDTOs();

// 4. Use batch fetching
@BatchSize(size = 25)
@OneToMany(mappedBy = "department")
private List<Employee> employees;

// 5. Enable batch inserts/updates
// application.properties:
// spring.jpa.properties.hibernate.jdbc.batch_size=50
// spring.jpa.properties.hibernate.order_inserts=true
// spring.jpa.properties.hibernate.order_updates=true

// 6. Use @Immutable for read-only entities
@Entity
@Immutable
public class AuditLog {
    // Fields
}

// 7. Use read-only transactions
@Transactional(readOnly = true)
public List<Employee> findAll() {
    return repository.findAll();
}

// 8. Avoid fetching entire entities when not needed
@Query("SELECT e.name FROM Employee e WHERE e.id = :id")
String findNameById(@Param("id") Long id);

// 9. Use pagination for large result sets
Page<Employee> findAll(Pageable pageable);

// 10. Use native queries for complex operations
@Query(value = "SELECT * FROM employees WHERE ...", nativeQuery = true)
List<Employee> findWithComplexQuery();

// 11. Index frequently queried columns
@Table(indexes = {
    @Index(name = "idx_email", columnList = "email"),
    @Index(name = "idx_dept_salary", columnList = "department_id, salary")
})
public class Employee { }

// 12. Use StatelessSession for bulk operations
Session session = sessionFactory.openStatelessSession();
Transaction tx = session.beginTransaction();

ScrollableResults results = session.createQuery("FROM Employee")
    .scroll(ScrollMode.FORWARD_ONLY);

while (results.next()) {
    Employee emp = (Employee) results.get(0);
    // Process
}

tx.commit();
session.close();
```

---

## 16. Common Problems & Solutions

### Q29: How to handle LazyInitializationException?

```java
// Problem: Accessing lazy collection outside of session/transaction

// Solution 1: Use @Transactional
@Transactional
public List<Employee> getDepartmentEmployees(Long deptId) {
    Department dept = departmentRepository.findById(deptId).orElseThrow();
    dept.getEmployees().size();  // Initialize collection
    return dept.getEmployees();
}

// Solution 2: Use JOIN FETCH
@Query("SELECT d FROM Department d JOIN FETCH d.employees WHERE d.id = :id")
Optional<Department> findByIdWithEmployees(@Param("id") Long id);

// Solution 3: Use EntityGraph
@EntityGraph(attributePaths = {"employees"})
Optional<Department> findById(Long id);

// Solution 4: Use Hibernate.initialize()
@Transactional
public Department getDepartmentWithEmployees(Long id) {
    Department dept = departmentRepository.findById(id).orElseThrow();
    Hibernate.initialize(dept.getEmployees());
    return dept;
}

// Solution 5: Use DTO projection (best for read-only)
@Query("SELECT new com.example.dto.DepartmentDTO(d.id, d.name, SIZE(d.employees)) FROM Department d")
List<DepartmentDTO> findAllWithEmployeeCount();
```

### Q30: How to handle OptimisticLockException?

```java
// Enable optimistic locking with @Version
@Entity
public class Employee {
    @Id
    private Long id;
    
    @Version
    private Long version;
    
    private String name;
}

// Handle the exception
@Service
public class EmployeeService {
    
    public Employee updateEmployee(Long id, String newName) {
        int maxRetries = 3;
        int retries = 0;
        
        while (retries < maxRetries) {
            try {
                Employee emp = repository.findById(id).orElseThrow();
                emp.setName(newName);
                return repository.save(emp);
            } catch (OptimisticLockException e) {
                retries++;
                if (retries >= maxRetries) {
                    throw new ConcurrentModificationException("Failed after " + maxRetries + " retries");
                }
            }
        }
        throw new RuntimeException("Unexpected");
    }
}
```

---

## 17. Best Practices

### Q31: JPA/Hibernate Best Practices Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          BEST PRACTICES                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ENTITY DESIGN                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ✅ Use LAZY fetching by default                                            │
│  ✅ Implement equals() and hashCode() using business key, not ID            │
│  ✅ Use @Version for optimistic locking                                     │
│  ✅ Prefer Set over List for @ManyToMany                                    │
│  ✅ Use helper methods for bidirectional relationships                      │
│  ✅ Make entities serializable                                              │
│                                                                             │
│  QUERIES                                                                    │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ✅ Use DTO projections for read-only queries                               │
│  ✅ Use JOIN FETCH or EntityGraph for associations                          │
│  ✅ Use pagination for large result sets                                    │
│  ✅ Use @BatchSize to avoid N+1 problem                                     │
│  ✅ Use native queries for complex operations                               │
│  ✅ Enable query statistics in development                                  │
│                                                                             │
│  TRANSACTIONS                                                               │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ✅ Keep transactions short                                                 │
│  ✅ Use @Transactional(readOnly = true) for read-only operations            │
│  ✅ Understand propagation and isolation levels                             │
│  ✅ Handle exceptions properly                                              │
│                                                                             │
│  PERFORMANCE                                                                │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ✅ Enable second-level cache for read-heavy entities                       │
│  ✅ Use batch inserts/updates                                               │
│  ✅ Index frequently queried columns                                        │
│  ✅ Monitor with hibernate.generate_statistics=true                         │
│  ✅ Use StatelessSession for bulk operations                                │
│                                                                             │
│  AVOID                                                                      │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ❌ Don't use EAGER fetching                                                │
│  ❌ Don't use CascadeType.ALL blindly                                       │
│  ❌ Don't fetch entire entities for read-only scenarios                     │
│  ❌ Don't use open-session-in-view pattern in production                    │
│  ❌ Don't ignore N+1 problem                                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📝 Quick Reference

### Common Annotations

| Annotation | Purpose |
|------------|---------|
| `@Entity` | Mark class as JPA entity |
| `@Table` | Specify table details |
| `@Id` | Mark primary key |
| `@GeneratedValue` | Auto-generate ID |
| `@Column` | Column mapping |
| `@OneToOne` | 1:1 relationship |
| `@OneToMany` | 1:N relationship |
| `@ManyToOne` | N:1 relationship |
| `@ManyToMany` | M:N relationship |
| `@JoinColumn` | Foreign key column |
| `@JoinTable` | Join table for M:N |
| `@Transient` | Not persisted |
| `@Temporal` | Date/time type |
| `@Enumerated` | Enum mapping |
| `@Lob` | Large object |
| `@Version` | Optimistic locking |
| `@Cacheable` | Second-level cache |

### Query Method Keywords

| Keyword | Sample | JPQL Snippet |
|---------|--------|--------------|
| And | findByNameAndEmail | WHERE x.name = ?1 AND x.email = ?2 |
| Or | findByNameOrEmail | WHERE x.name = ?1 OR x.email = ?2 |
| Is, Equals | findByName | WHERE x.name = ?1 |
| Between | findByAgeBetween | WHERE x.age BETWEEN ?1 AND ?2 |
| LessThan | findByAgeLessThan | WHERE x.age < ?1 |
| GreaterThan | findByAgeGreaterThan | WHERE x.age > ?1 |
| Like | findByNameLike | WHERE x.name LIKE ?1 |
| Containing | findByNameContaining | WHERE x.name LIKE %?1% |
| In | findByAgeIn | WHERE x.age IN (?1) |
| True | findByActiveTrue | WHERE x.active = TRUE |
| OrderBy | findByNameOrderByAgeDesc | ORDER BY x.age DESC |

---

*Document generated: December 2024*
*Versions: JPA 3.0, Hibernate 6.x, Spring Data JPA 3.x*

