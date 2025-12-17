# Spring Security Interview Guide
## Complete Questions, Answers & Code Examples

---

## 📑 Table of Contents

1. [Spring Security Fundamentals](#1-spring-security-fundamentals)
2. [Architecture & Filter Chain](#2-architecture--filter-chain)
3. [Authentication](#3-authentication)
4. [Authorization](#4-authorization)
5. [Security Configuration](#5-security-configuration)
6. [Password Encoding](#6-password-encoding)
7. [Form-Based Authentication](#7-form-based-authentication)
8. [JWT Authentication](#8-jwt-authentication)
9. [OAuth2 & OpenID Connect](#9-oauth2--openid-connect)
10. [Method-Level Security](#10-method-level-security)
11. [CSRF Protection](#11-csrf-protection)
12. [CORS Configuration](#12-cors-configuration)
13. [Session Management](#13-session-management)
14. [Remember-Me](#14-remember-me)
15. [Security Headers](#15-security-headers)
16. [Custom Authentication](#16-custom-authentication)
17. [Testing Security](#17-testing-security)
18. [Common Vulnerabilities](#18-common-vulnerabilities)
19. [Best Practices](#19-best-practices)

---

## 1. Spring Security Fundamentals

### Q1: What is Spring Security?

**Answer:** Spring Security is a powerful and highly customizable authentication and access-control framework for Java applications. It is the de-facto standard for securing Spring-based applications.

**Key Features:**
- ✅ Authentication & Authorization
- ✅ Protection against common attacks (CSRF, XSS, Session Fixation)
- ✅ Integration with Spring MVC, WebFlux
- ✅ OAuth2/OpenID Connect support
- ✅ Method-level security
- ✅ LDAP, SAML, JWT support

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SPRING SECURITY OVERVIEW                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                      HTTP Request                                    │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                   Security Filter Chain                              │  │
│   │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │  │
│   │  │  CSRF    │→│  Auth    │→│ Session  │→│Exception │→│ Authz    │  │  │
│   │  │  Filter  │ │  Filter  │ │ Filter   │ │ Filter   │ │ Filter   │  │  │
│   │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘  │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                 Authentication Manager                               │  │
│   │              (Validates credentials)                                 │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │              Authorization Manager / Access Decision                 │  │
│   │           (Checks roles/permissions)                                 │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                    Secured Resource                                  │  │
│   │                 (Controller/Endpoint)                                │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Q2: What is the difference between Authentication and Authorization?

| Aspect | Authentication | Authorization |
|--------|---------------|---------------|
| **Definition** | Verifying WHO you are | Verifying WHAT you can do |
| **Process** | Login/Credentials verification | Permission/Access control |
| **Order** | First | After authentication |
| **Question** | "Are you who you claim to be?" | "Are you allowed to do this?" |
| **HTTP Status** | 401 Unauthorized | 403 Forbidden |
| **Example** | Username/Password, JWT token | Role-based access, Permissions |

```java
// Authentication - Verifying identity
Authentication authentication = authenticationManager.authenticate(
    new UsernamePasswordAuthenticationToken(username, password)
);

// Authorization - Checking permissions
@PreAuthorize("hasRole('ADMIN')")
public void adminOnly() {
    // Only users with ADMIN role can access
}
```

### Q3: What are the core components of Spring Security?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CORE COMPONENTS                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. SecurityContextHolder                                                   │
│     ├── SecurityContext                                                     │
│     │   └── Authentication                                                  │
│     │       ├── Principal (UserDetails)                                     │
│     │       ├── Credentials (password)                                      │
│     │       └── Authorities (roles/permissions)                             │
│     └── Storage Strategy (ThreadLocal, InheritableThreadLocal, Global)      │
│                                                                             │
│  2. AuthenticationManager                                                   │
│     └── ProviderManager                                                     │
│         └── AuthenticationProvider(s)                                       │
│             └── UserDetailsService                                          │
│                 └── UserDetails                                             │
│                                                                             │
│  3. SecurityFilterChain                                                     │
│     └── Multiple Filters (15+ default filters)                              │
│                                                                             │
│  4. AccessDecisionManager (deprecated in 6.x)                               │
│     └── AuthorizationManager (new in 6.x)                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```java
// SecurityContextHolder - Access current user
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
String username = auth.getName();
Collection<? extends GrantedAuthority> authorities = auth.getAuthorities();

// UserDetails - Core user interface
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    String getPassword();
    String getUsername();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}

// Authentication - Core authentication interface
public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    Object getCredentials();
    Object getDetails();
    Object getPrincipal();
    boolean isAuthenticated();
    void setAuthenticated(boolean isAuthenticated);
}
```

---

## 2. Architecture & Filter Chain

### Q4: Explain the Spring Security Filter Chain

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SECURITY FILTER CHAIN ORDER                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Order │ Filter                          │ Purpose                        │
│   ──────┼─────────────────────────────────┼────────────────────────────────│
│    1    │ ForceEagerSessionCreationFilter │ Create session eagerly         │
│    2    │ ChannelProcessingFilter         │ Redirect to HTTPS              │
│    3    │ WebAsyncManagerIntegrationFilter│ Async request support          │
│    4    │ SecurityContextHolderFilter     │ Setup SecurityContext          │
│    5    │ HeaderWriterFilter              │ Add security headers           │
│    6    │ CorsFilter                      │ CORS handling                  │
│    7    │ CsrfFilter                      │ CSRF protection                │
│    8    │ LogoutFilter                    │ Handle logout                  │
│    9    │ OAuth2AuthorizationRequestFilter│ OAuth2 authorization           │
│   10    │ UsernamePasswordAuthFilter      │ Form login                     │
│   11    │ BasicAuthenticationFilter       │ HTTP Basic auth                │
│   12    │ BearerTokenAuthFilter           │ JWT/Bearer token               │
│   13    │ RequestCacheAwareFilter         │ Cache requests                 │
│   14    │ SecurityContextHolderAwareFilter│ Servlet API integration        │
│   15    │ AnonymousAuthenticationFilter   │ Anonymous users                │
│   16    │ SessionManagementFilter         │ Session handling               │
│   17    │ ExceptionTranslationFilter      │ Handle security exceptions     │
│   18    │ AuthorizationFilter             │ Access control (final check)   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```java
// Custom Filter Implementation
@Component
public class CustomAuthenticationFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) 
            throws ServletException, IOException {
        
        String token = extractToken(request);
        
        if (token != null && validateToken(token)) {
            Authentication auth = createAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String extractToken(HttpServletRequest request) {
        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            return header.substring(7);
        }
        return null;
    }
}

// Add custom filter to chain
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .addFilterBefore(customAuthFilter, UsernamePasswordAuthenticationFilter.class)
            // or
            .addFilterAfter(customFilter, BasicAuthenticationFilter.class)
            // or
            .addFilterAt(customFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}
```

### Q5: What is DelegatingFilterProxy?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DELEGATING FILTER PROXY                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────┐          ┌──────────────────────────────────┐   │
│   │   Servlet Container │          │     Spring ApplicationContext    │   │
│   │   (Tomcat, etc.)    │          │                                  │   │
│   │                     │          │  ┌────────────────────────────┐ │   │
│   │  ┌───────────────┐  │ delegates│  │  FilterChainProxy          │ │   │
│   │  │Delegating     │──┼──────────┼─►│  (springSecurityFilterChain)│ │   │
│   │  │FilterProxy    │  │   to     │  │                            │ │   │
│   │  └───────────────┘  │          │  │  ┌────────────────────┐   │ │   │
│   │                     │          │  │  │ SecurityFilterChain│   │ │   │
│   │                     │          │  │  │ (multiple filters) │   │ │   │
│   │                     │          │  │  └────────────────────┘   │ │   │
│   │                     │          │  └────────────────────────────┘ │   │
│   └─────────────────────┘          └──────────────────────────────────┘   │
│                                                                             │
│   DelegatingFilterProxy bridges Servlet container and Spring context        │
│   FilterChainProxy is the actual entry point for Spring Security            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Authentication

### Q6: How does Spring Security Authentication work?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AUTHENTICATION FLOW                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   1. User submits credentials (username/password)                           │
│                    │                                                        │
│                    ▼                                                        │
│   2. AuthenticationFilter creates Authentication token                      │
│      (UsernamePasswordAuthenticationToken)                                  │
│                    │                                                        │
│                    ▼                                                        │
│   3. AuthenticationManager.authenticate()                                   │
│      └── ProviderManager (implementation)                                   │
│                    │                                                        │
│                    ▼                                                        │
│   4. AuthenticationProvider.authenticate()                                  │
│      └── DaoAuthenticationProvider                                          │
│                    │                                                        │
│                    ▼                                                        │
│   5. UserDetailsService.loadUserByUsername()                                │
│                    │                                                        │
│                    ▼                                                        │
│   6. PasswordEncoder.matches() - Verify password                            │
│                    │                                                        │
│          ┌────────┴────────┐                                               │
│          ▼                 ▼                                               │
│   7a. Success           7b. Failure                                         │
│       │                     │                                               │
│       ▼                     ▼                                               │
│   Return authenticated   Throw AuthenticationException                      │
│   Authentication object  (BadCredentialsException, etc.)                    │
│       │                                                                     │
│       ▼                                                                     │
│   8. Store in SecurityContextHolder                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```java
// UserDetailsService Implementation
@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
        
        return org.springframework.security.core.userdetails.User.builder()
            .username(user.getUsername())
            .password(user.getPassword())  // Must be encoded
            .roles(user.getRoles().toArray(new String[0]))
            .authorities(getAuthorities(user))
            .accountExpired(!user.isAccountNonExpired())
            .accountLocked(!user.isAccountNonLocked())
            .credentialsExpired(!user.isCredentialsNonExpired())
            .disabled(!user.isEnabled())
            .build();
    }
    
    private Collection<? extends GrantedAuthority> getAuthorities(User user) {
        return user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
            .collect(Collectors.toList());
    }
}

// Custom UserDetails
public class CustomUserDetails implements UserDetails {
    private final User user;
    
    public CustomUserDetails(User user) {
        this.user = user;
    }
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
            .collect(Collectors.toList());
    }
    
    @Override
    public String getPassword() { return user.getPassword(); }
    
    @Override
    public String getUsername() { return user.getUsername(); }
    
    @Override
    public boolean isAccountNonExpired() { return true; }
    
    @Override
    public boolean isAccountNonLocked() { return !user.isLocked(); }
    
    @Override
    public boolean isCredentialsNonExpired() { return true; }
    
    @Override
    public boolean isEnabled() { return user.isEnabled(); }
    
    // Custom method to access full user
    public User getUser() { return user; }
}
```

### Q7: What are the different AuthenticationProviders?

```java
// 1. DaoAuthenticationProvider - Database authentication
@Bean
public AuthenticationProvider daoAuthenticationProvider() {
    DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
    provider.setUserDetailsService(userDetailsService);
    provider.setPasswordEncoder(passwordEncoder());
    return provider;
}

// 2. LdapAuthenticationProvider - LDAP authentication
@Bean
public AuthenticationProvider ldapAuthenticationProvider() {
    LdapAuthenticationProvider provider = new LdapAuthenticationProvider(
        ldapAuthenticator, ldapAuthoritiesPopulator);
    return provider;
}

// 3. JwtAuthenticationProvider - JWT token authentication
// Custom implementation needed

// 4. OAuth2LoginAuthenticationProvider - OAuth2 login
// Auto-configured by Spring Security OAuth2

// 5. Custom AuthenticationProvider
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {
    
    @Override
    public Authentication authenticate(Authentication authentication) 
            throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();
        
        // Custom authentication logic
        if (isValidUser(username, password)) {
            List<GrantedAuthority> authorities = getAuthorities(username);
            return new UsernamePasswordAuthenticationToken(
                username, password, authorities);
        }
        
        throw new BadCredentialsException("Invalid credentials");
    }
    
    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
    }
}

// Configure multiple providers
@Bean
public AuthenticationManager authenticationManager(
        List<AuthenticationProvider> providers) {
    return new ProviderManager(providers);
}
```

---

## 4. Authorization

### Q8: How does Authorization work in Spring Security?

```java
// URL-based authorization (Spring Security 6.x)
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                // Public endpoints
                .requestMatchers("/", "/home", "/public/**").permitAll()
                .requestMatchers("/css/**", "/js/**", "/images/**").permitAll()
                
                // Role-based access
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
                
                // Authority-based access
                .requestMatchers("/api/write/**").hasAuthority("WRITE_PRIVILEGE")
                .requestMatchers("/api/read/**").hasAnyAuthority("READ_PRIVILEGE", "WRITE_PRIVILEGE")
                
                // HTTP method specific
                .requestMatchers(HttpMethod.POST, "/api/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.GET, "/api/**").hasAnyRole("USER", "ADMIN")
                .requestMatchers(HttpMethod.DELETE, "/api/**").hasRole("SUPER_ADMIN")
                
                // Custom matcher
                .requestMatchers(new AntPathRequestMatcher("/reports/**")).hasRole("ANALYST")
                
                // IP-based access
                .requestMatchers("/internal/**").access(
                    new WebExpressionAuthorizationManager("hasIpAddress('192.168.1.0/24')"))
                
                // Combined conditions (SpEL)
                .requestMatchers("/special/**").access(
                    new WebExpressionAuthorizationManager(
                        "hasRole('ADMIN') and hasIpAddress('10.0.0.0/8')"))
                
                // All other requests require authentication
                .anyRequest().authenticated()
            );
        
        return http.build();
    }
}
```

### Q9: What is the difference between hasRole() and hasAuthority()?

| Aspect | hasRole() | hasAuthority() |
|--------|-----------|----------------|
| Prefix | Automatically adds "ROLE_" | No prefix added |
| Usage | `hasRole("ADMIN")` | `hasAuthority("ROLE_ADMIN")` |
| Store as | ROLE_ADMIN | ROLE_ADMIN or ADMIN |
| Best for | Role-based access | Fine-grained permissions |

```java
// In UserDetailsService - Setting authorities
return User.builder()
    .username(user.getUsername())
    .password(user.getPassword())
    // These are equivalent:
    .roles("ADMIN", "USER")  // Stored as ROLE_ADMIN, ROLE_USER
    // OR
    .authorities("ROLE_ADMIN", "ROLE_USER", "READ_PRIVILEGE", "WRITE_PRIVILEGE")
    .build();

// In Security Config
.requestMatchers("/admin/**").hasRole("ADMIN")        // Checks ROLE_ADMIN
.requestMatchers("/admin/**").hasAuthority("ROLE_ADMIN")  // Same effect

// Custom permissions
.requestMatchers("/api/write").hasAuthority("WRITE_PRIVILEGE")  // No ROLE_ prefix
```

---

## 5. Security Configuration

### Q10: How to configure Spring Security? (Spring Security 6.x)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // For @PreAuthorize, @PostAuthorize
public class SecurityConfig {
    
    @Autowired
    private CustomUserDetailsService userDetailsService;
    
    @Autowired
    private JwtAuthenticationFilter jwtAuthFilter;
    
    // Security Filter Chain - Main configuration
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // Disable CSRF for REST APIs
            .csrf(csrf -> csrf.disable())
            
            // CORS configuration
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            
            // Session management
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            
            // Authorization rules
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            
            // Exception handling
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(authenticationEntryPoint())
                .accessDeniedHandler(accessDeniedHandler())
            )
            
            // Add JWT filter
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            
            // Authentication provider
            .authenticationProvider(authenticationProvider());
        
        return http.build();
    }
    
    // Password Encoder
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    // Authentication Provider
    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }
    
    // Authentication Manager
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) 
            throws Exception {
        return config.getAuthenticationManager();
    }
    
    // CORS Configuration
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("http://localhost:3000"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(Arrays.asList("*"));
        configuration.setAllowCredentials(true);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
    
    // Authentication Entry Point (401)
    @Bean
    public AuthenticationEntryPoint authenticationEntryPoint() {
        return (request, response, authException) -> {
            response.setContentType("application/json");
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("{\"error\": \"Unauthorized\", \"message\": \"" 
                + authException.getMessage() + "\"}");
        };
    }
    
    // Access Denied Handler (403)
    @Bean
    public AccessDeniedHandler accessDeniedHandler() {
        return (request, response, accessDeniedException) -> {
            response.setContentType("application/json");
            response.setStatus(HttpServletResponse.SC_FORBIDDEN);
            response.getWriter().write("{\"error\": \"Forbidden\", \"message\": \"" 
                + accessDeniedException.getMessage() + "\"}");
        };
    }
}
```

### Q11: How to configure multiple Security Filter Chains?

```java
@Configuration
@EnableWebSecurity
public class MultiSecurityConfig {
    
    // API Security - JWT based
    @Bean
    @Order(1)
    public SecurityFilterChain apiSecurityFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/**")
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    // Web Security - Form based
    @Bean
    @Order(2)
    public SecurityFilterChain webSecurityFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/**")
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/home", "/login").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutSuccessUrl("/login?logout")
                .permitAll()
            );
        
        return http.build();
    }
}
```

---

## 6. Password Encoding

### Q12: What are the password encoders in Spring Security?

```java
// 1. BCryptPasswordEncoder (Recommended)
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();  // Default strength: 10
    // return new BCryptPasswordEncoder(12);  // Custom strength (4-31)
}

// 2. Argon2PasswordEncoder (Most secure, but slower)
@Bean
public PasswordEncoder argon2PasswordEncoder() {
    return new Argon2PasswordEncoder(16, 32, 1, 65536, 4);
}

// 3. SCryptPasswordEncoder
@Bean
public PasswordEncoder scryptPasswordEncoder() {
    return new SCryptPasswordEncoder(16384, 8, 1, 32, 64);
}

// 4. Pbkdf2PasswordEncoder
@Bean
public PasswordEncoder pbkdf2PasswordEncoder() {
    return new Pbkdf2PasswordEncoder("secret", 16, 310000, 
        Pbkdf2PasswordEncoder.SecretKeyFactoryAlgorithm.PBKDF2WithHmacSHA256);
}

// 5. DelegatingPasswordEncoder (Supports multiple encodings)
@Bean
public PasswordEncoder delegatingPasswordEncoder() {
    String encodingId = "bcrypt";
    Map<String, PasswordEncoder> encoders = new HashMap<>();
    encoders.put("bcrypt", new BCryptPasswordEncoder());
    encoders.put("noop", NoOpPasswordEncoder.getInstance());  // Plain text (testing only!)
    encoders.put("sha256", new StandardPasswordEncoder());     // Deprecated
    encoders.put("argon2", new Argon2PasswordEncoder());
    
    return new DelegatingPasswordEncoder(encodingId, encoders);
}
// Stored format: {bcrypt}$2a$10$... or {argon2}$argon2id$...

// Usage
@Service
public class UserService {
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    public User createUser(String username, String rawPassword) {
        User user = new User();
        user.setUsername(username);
        user.setPassword(passwordEncoder.encode(rawPassword));
        return userRepository.save(user);
    }
    
    public boolean verifyPassword(String rawPassword, String encodedPassword) {
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }
}
```

### Q13: Why not use MD5 or SHA for passwords?

**Answer:** MD5 and SHA are **NOT suitable** for password hashing because:

| Issue | MD5/SHA | BCrypt/Argon2 |
|-------|---------|---------------|
| Speed | Very fast (vulnerability) | Intentionally slow |
| Salt | No built-in salt | Automatic unique salt |
| Rainbow tables | Vulnerable | Protected |
| GPU attacks | Easy to brute force | Resistant |
| Work factor | Fixed | Adjustable |

```java
// ❌ NEVER DO THIS
String hashedPassword = DigestUtils.md5Hex(password);  // Insecure!

// ✅ ALWAYS DO THIS
String hashedPassword = passwordEncoder.encode(password);  // Secure
```

---

## 7. Form-Based Authentication

### Q14: How to configure form-based login?

```java
@Configuration
@EnableWebSecurity
public class FormLoginConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/home", "/register").permitAll()
                .requestMatchers("/css/**", "/js/**").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")                    // Custom login page
                .loginProcessingUrl("/perform_login")   // Form action URL
                .usernameParameter("email")             // Custom username field
                .passwordParameter("pass")              // Custom password field
                .defaultSuccessUrl("/dashboard", true)  // Redirect after login
                .failureUrl("/login?error=true")        // Redirect on failure
                .successHandler(authenticationSuccessHandler())
                .failureHandler(authenticationFailureHandler())
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/perform_logout")
                .logoutSuccessUrl("/login?logout=true")
                .deleteCookies("JSESSIONID")
                .invalidateHttpSession(true)
                .clearAuthentication(true)
                .logoutSuccessHandler(logoutSuccessHandler())
                .permitAll()
            )
            .rememberMe(remember -> remember
                .key("uniqueAndSecretKey")
                .tokenValiditySeconds(86400)  // 24 hours
                .rememberMeParameter("remember-me")
                .userDetailsService(userDetailsService)
            );
        
        return http.build();
    }
    
    @Bean
    public AuthenticationSuccessHandler authenticationSuccessHandler() {
        return (request, response, authentication) -> {
            // Custom logic on successful authentication
            log.info("User {} logged in successfully", authentication.getName());
            
            // Redirect based on role
            if (authentication.getAuthorities().stream()
                    .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"))) {
                response.sendRedirect("/admin/dashboard");
            } else {
                response.sendRedirect("/user/dashboard");
            }
        };
    }
    
    @Bean
    public AuthenticationFailureHandler authenticationFailureHandler() {
        return (request, response, exception) -> {
            String errorMessage = "Invalid credentials";
            
            if (exception instanceof BadCredentialsException) {
                errorMessage = "Invalid username or password";
            } else if (exception instanceof DisabledException) {
                errorMessage = "Account is disabled";
            } else if (exception instanceof LockedException) {
                errorMessage = "Account is locked";
            }
            
            response.sendRedirect("/login?error=" + URLEncoder.encode(errorMessage, "UTF-8"));
        };
    }
}
```

```html
<!-- login.html (Thymeleaf) -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Login</title>
</head>
<body>
    <h2>Login</h2>
    
    <div th:if="${param.error}" class="error">
        Invalid username or password
    </div>
    
    <div th:if="${param.logout}" class="success">
        You have been logged out
    </div>
    
    <form th:action="@{/perform_login}" method="post">
        <div>
            <label>Email:</label>
            <input type="email" name="email" required/>
        </div>
        <div>
            <label>Password:</label>
            <input type="password" name="pass" required/>
        </div>
        <div>
            <input type="checkbox" name="remember-me"/> Remember me
        </div>
        <button type="submit">Login</button>
    </form>
</body>
</html>
```

---

## 8. JWT Authentication

### Q15: How to implement JWT authentication?

```java
// 1. JWT Utility Class
@Component
public class JwtUtil {
    
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.expiration}")
    private Long expiration;  // milliseconds
    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList()));
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }
    
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }
    
    public Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }
    
    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }
    
    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
    }
    
    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
    
    public boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }
    
    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }
}

// 2. JWT Authentication Filter
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    @Autowired
    private JwtUtil jwtUtil;
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) 
            throws ServletException, IOException {
        
        final String authHeader = request.getHeader("Authorization");
        final String jwt;
        final String username;
        
        // Check for Bearer token
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }
        
        jwt = authHeader.substring(7);
        
        try {
            username = jwtUtil.extractUsername(jwt);
            
            // If username exists and no authentication in context
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                
                if (jwtUtil.validateToken(jwt, userDetails)) {
                    UsernamePasswordAuthenticationToken authToken = 
                        new UsernamePasswordAuthenticationToken(
                            userDetails, null, userDetails.getAuthorities());
                    
                    authToken.setDetails(new WebAuthenticationDetailsSource()
                        .buildDetails(request));
                    
                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        } catch (ExpiredJwtException e) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("Token expired");
            return;
        } catch (Exception e) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("Invalid token");
            return;
        }
        
        filterChain.doFilter(request, response);
    }
}

// 3. Auth Controller
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    
    @Autowired
    private AuthenticationManager authenticationManager;
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Autowired
    private JwtUtil jwtUtil;
    
    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest request) {
        try {
            authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    request.getUsername(), request.getPassword()));
            
            UserDetails userDetails = userDetailsService
                .loadUserByUsername(request.getUsername());
            
            String token = jwtUtil.generateToken(userDetails);
            
            return ResponseEntity.ok(new AuthResponse(token));
            
        } catch (AuthenticationException e) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body(new ErrorResponse("Invalid credentials"));
        }
    }
    
    @PostMapping("/refresh")
    public ResponseEntity<?> refresh(@RequestHeader("Authorization") String authHeader) {
        String token = authHeader.substring(7);
        String username = jwtUtil.extractUsername(token);
        
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        String newToken = jwtUtil.generateToken(userDetails);
        
        return ResponseEntity.ok(new AuthResponse(newToken));
    }
}
```

### Q16: What is the difference between JWT and Session-based authentication?

| Aspect | JWT (Stateless) | Session (Stateful) |
|--------|-----------------|-------------------|
| Storage | Client (token) | Server (session store) |
| Scalability | Easy (no server state) | Harder (session replication) |
| Performance | No DB lookup per request | Session lookup needed |
| Revocation | Difficult (needs blacklist) | Easy (delete session) |
| Size | Larger (contains claims) | Small (just session ID) |
| Security | Vulnerable if secret leaked | Session ID can be stolen |
| Mobile | Better suited | Requires cookie handling |

---

## 9. OAuth2 & OpenID Connect

### Q17: How to configure OAuth2 login with Google/GitHub?

```yaml
# application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope: openid, profile, email
          
          github:
            client-id: ${GITHUB_CLIENT_ID}
            client-secret: ${GITHUB_CLIENT_SECRET}
            scope: read:user, user:email
            
          custom-provider:
            client-id: ${CUSTOM_CLIENT_ID}
            client-secret: ${CUSTOM_CLIENT_SECRET}
            authorization-grant-type: authorization_code
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
            scope: openid, profile, email
            
        provider:
          custom-provider:
            authorization-uri: https://auth.example.com/authorize
            token-uri: https://auth.example.com/token
            user-info-uri: https://auth.example.com/userinfo
            jwk-set-uri: https://auth.example.com/.well-known/jwks.json
            user-name-attribute: sub
```

```java
@Configuration
@EnableWebSecurity
public class OAuth2SecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login", "/error").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .userInfoEndpoint(userInfo -> userInfo
                    .userService(customOAuth2UserService())
                    .oidcUserService(customOidcUserService())
                )
                .successHandler(oAuth2AuthenticationSuccessHandler())
                .failureHandler(oAuth2AuthenticationFailureHandler())
            )
            .oauth2Client(Customizer.withDefaults());
        
        return http.build();
    }
    
    @Bean
    public OAuth2UserService<OAuth2UserRequest, OAuth2User> customOAuth2UserService() {
        return new CustomOAuth2UserService();
    }
    
    @Bean
    public OidcUserService customOidcUserService() {
        return new CustomOidcUserService();
    }
}

// Custom OAuth2 User Service
@Service
public class CustomOAuth2UserService extends DefaultOAuth2UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2User oAuth2User = super.loadUser(userRequest);
        
        String provider = userRequest.getClientRegistration().getRegistrationId();
        String providerId = oAuth2User.getAttribute("sub");
        String email = oAuth2User.getAttribute("email");
        String name = oAuth2User.getAttribute("name");
        
        // Find or create user
        User user = userRepository.findByProviderAndProviderId(provider, providerId)
            .orElseGet(() -> {
                User newUser = new User();
                newUser.setProvider(provider);
                newUser.setProviderId(providerId);
                newUser.setEmail(email);
                newUser.setName(name);
                return userRepository.save(newUser);
            });
        
        return new CustomOAuth2User(oAuth2User, user);
    }
}
```

### Q18: How to configure OAuth2 Resource Server?

```java
@Configuration
@EnableWebSecurity
public class ResourceServerConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .decoder(jwtDecoder())
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())
                )
                // OR use opaque tokens
                // .opaqueToken(opaque -> opaque
                //     .introspectionUri("https://auth.example.com/introspect")
                //     .introspectionClientCredentials("client-id", "client-secret")
                // )
            );
        
        return http.build();
    }
    
    @Bean
    public JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withJwkSetUri("https://auth.example.com/.well-known/jwks.json")
            .build();
    }
    
    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter grantedAuthoritiesConverter = 
            new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthoritiesClaimName("roles");
        grantedAuthoritiesConverter.setAuthorityPrefix("ROLE_");
        
        JwtAuthenticationConverter jwtAuthenticationConverter = new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(grantedAuthoritiesConverter);
        
        return jwtAuthenticationConverter;
    }
}
```

---

## 10. Method-Level Security

### Q19: How to use method-level security annotations?

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(
    prePostEnabled = true,   // @PreAuthorize, @PostAuthorize
    securedEnabled = true,   // @Secured
    jsr250Enabled = true     // @RolesAllowed (JSR-250)
)
public class MethodSecurityConfig {
}

@Service
public class UserService {
    
    // @PreAuthorize - Check BEFORE method execution
    @PreAuthorize("hasRole('ADMIN')")
    public void adminOnlyMethod() {
        // Only ADMIN can access
    }
    
    @PreAuthorize("hasAnyRole('ADMIN', 'MANAGER')")
    public void managementMethod() {
        // ADMIN or MANAGER can access
    }
    
    @PreAuthorize("hasAuthority('USER_CREATE')")
    public void createUser() {
        // Requires USER_CREATE permission
    }
    
    // Access method parameters
    @PreAuthorize("#userId == authentication.principal.id or hasRole('ADMIN')")
    public User getUser(Long userId) {
        // User can only access their own data, or ADMIN can access all
    }
    
    // SpEL expressions
    @PreAuthorize("@securityService.canAccess(#userId)")
    public User getUserWithCustomCheck(Long userId) {
        // Uses custom security service
    }
    
    // @PostAuthorize - Check AFTER method execution
    @PostAuthorize("returnObject.owner == authentication.name or hasRole('ADMIN')")
    public Document getDocument(Long docId) {
        // Check if returned document belongs to current user
        return documentRepository.findById(docId).orElseThrow();
    }
    
    // @PreFilter - Filter input collection
    @PreFilter("filterObject.owner == authentication.name")
    public void deleteDocuments(List<Document> documents) {
        // Only delete documents owned by current user
    }
    
    // @PostFilter - Filter returned collection
    @PostFilter("filterObject.owner == authentication.name or hasRole('ADMIN')")
    public List<Document> getAllDocuments() {
        // Returns only documents user can see
        return documentRepository.findAll();
    }
    
    // @Secured (older style)
    @Secured("ROLE_ADMIN")
    public void securedAdminMethod() {
    }
    
    @Secured({"ROLE_ADMIN", "ROLE_MANAGER"})
    public void securedMultiRoleMethod() {
    }
    
    // @RolesAllowed (JSR-250 standard)
    @RolesAllowed("ADMIN")
    public void jsr250AdminMethod() {
    }
}

// Custom Security Service for complex checks
@Service
public class SecurityService {
    
    @Autowired
    private UserRepository userRepository;
    
    public boolean canAccess(Long userId) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        User currentUser = userRepository.findByUsername(auth.getName()).orElse(null);
        
        if (currentUser == null) return false;
        
        // Custom logic
        return currentUser.getId().equals(userId) || 
               currentUser.getRoles().contains("ADMIN");
    }
    
    public boolean isOwner(Object entity) {
        // Check ownership logic
    }
}
```

---

## 11. CSRF Protection

### Q20: What is CSRF and how does Spring Security protect against it?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CSRF ATTACK SCENARIO                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   1. User logs into Bank website (authenticated session)                    │
│                                                                             │
│   2. User visits malicious website while still logged in                    │
│                                                                             │
│   3. Malicious site has hidden form:                                        │
│      <form action="https://bank.com/transfer" method="POST">               │
│        <input type="hidden" name="to" value="attacker"/>                   │
│        <input type="hidden" name="amount" value="10000"/>                  │
│      </form>                                                               │
│      <script>document.forms[0].submit();</script>                          │
│                                                                             │
│   4. Browser sends request WITH session cookie → Money transferred!         │
│                                                                             │
│   PROTECTION: CSRF Token                                                    │
│   - Server generates unique token per session                               │
│   - Token must be included in each state-changing request                   │
│   - Attacker can't know the token → Attack fails                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```java
// CSRF Configuration
@Configuration
@EnableWebSecurity
public class CsrfConfig {
    
    // Default: CSRF enabled for web apps
    @Bean
    public SecurityFilterChain webSecurityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                .csrfTokenRequestHandler(new CsrfTokenRequestAttributeHandler())
                .ignoringRequestMatchers("/api/webhook/**")  // Exclude specific paths
            );
        return http.build();
    }
    
    // Disable CSRF for REST APIs (stateless, JWT-based)
    @Bean
    public SecurityFilterChain apiSecurityFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/**")
            .csrf(csrf -> csrf.disable())  // Disable for stateless API
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS));
        return http.build();
    }
}
```

```html
<!-- Including CSRF token in forms (Thymeleaf) -->
<form th:action="@{/transfer}" method="post">
    <!-- CSRF token automatically included by Thymeleaf -->
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
    
    <input type="text" name="amount"/>
    <button type="submit">Transfer</button>
</form>

<!-- Or using th:action automatically includes it -->
<form th:action="@{/transfer}" method="post">
    <!-- Thymeleaf automatically adds CSRF token -->
</form>
```

```javascript
// CSRF token in JavaScript (Cookie-based)
// Read token from cookie
function getCsrfToken() {
    return document.cookie
        .split('; ')
        .find(row => row.startsWith('XSRF-TOKEN='))
        ?.split('=')[1];
}

// Include in AJAX requests
fetch('/api/data', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-XSRF-TOKEN': getCsrfToken()
    },
    body: JSON.stringify(data)
});

// Axios configuration
axios.defaults.xsrfCookieName = 'XSRF-TOKEN';
axios.defaults.xsrfHeaderName = 'X-XSRF-TOKEN';
```

---

## 12. CORS Configuration

### Q21: How to configure CORS in Spring Security?

```java
@Configuration
@EnableWebSecurity
public class CorsSecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // Method 1: Use CorsConfigurationSource bean
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            
            // OR Method 2: Use custom filter
            // .cors(cors -> cors.disable())
            // .addFilterBefore(corsFilter(), ChannelProcessingFilter.class)
            
            .authorizeHttpRequests(auth -> auth.anyRequest().authenticated());
        
        return http.build();
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        
        // Allowed origins
        configuration.setAllowedOrigins(Arrays.asList(
            "http://localhost:3000",
            "https://myapp.com"
        ));
        // OR allow all origins (not recommended for production)
        // configuration.setAllowedOriginPatterns(Arrays.asList("*"));
        
        // Allowed HTTP methods
        configuration.setAllowedMethods(Arrays.asList(
            "GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"
        ));
        
        // Allowed headers
        configuration.setAllowedHeaders(Arrays.asList(
            "Authorization",
            "Content-Type",
            "X-Requested-With",
            "Accept",
            "Origin",
            "Access-Control-Request-Method",
            "Access-Control-Request-Headers"
        ));
        
        // Exposed headers (client can read these)
        configuration.setExposedHeaders(Arrays.asList(
            "Access-Control-Allow-Origin",
            "Access-Control-Allow-Credentials",
            "Authorization"
        ));
        
        // Allow credentials (cookies, authorization headers)
        configuration.setAllowCredentials(true);
        
        // Cache preflight response
        configuration.setMaxAge(3600L);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        
        return source;
    }
}

// Method 3: Controller-level CORS
@RestController
@CrossOrigin(origins = "http://localhost:3000", maxAge = 3600)
@RequestMapping("/api/users")
public class UserController {
    
    @CrossOrigin(origins = "http://admin.example.com")  // Method-level override
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }
}

// Global CORS configuration (WebMvcConfigurer)
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("http://localhost:3000")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}
```

---

## 13. Session Management

### Q22: How to configure session management?

```java
@Configuration
@EnableWebSecurity
public class SessionConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .sessionManagement(session -> session
                // Session creation policy
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
                // Options: ALWAYS, IF_REQUIRED, NEVER, STATELESS
                
                // Session fixation protection
                .sessionFixation(fixation -> fixation
                    .migrateSession()  // Default: Create new session, copy attributes
                    // .newSession()   // Create new session, don't copy
                    // .changeSessionId()  // Just change ID (Servlet 3.1+)
                    // .none()         // No protection (not recommended)
                )
                
                // Maximum sessions per user
                .maximumSessions(1)
                    .maxSessionsPreventsLogin(false)  // false = kick out old session
                    // .maxSessionsPreventsLogin(true)  // true = prevent new login
                    .expiredUrl("/session-expired")
                    .sessionRegistry(sessionRegistry())
                
                // Invalid session handling
                .invalidSessionUrl("/invalid-session")
            );
        
        return http.build();
    }
    
    @Bean
    public SessionRegistry sessionRegistry() {
        return new SessionRegistryImpl();
    }
    
    // For concurrent session control to work with Spring Boot
    @Bean
    public HttpSessionEventPublisher httpSessionEventPublisher() {
        return new HttpSessionEventPublisher();
    }
}

// Session Timeout Configuration (application.properties)
// server.servlet.session.timeout=30m

// Programmatic session management
@RestController
public class SessionController {
    
    @Autowired
    private SessionRegistry sessionRegistry;
    
    // Get all active sessions for a user
    @GetMapping("/admin/sessions/{username}")
    @PreAuthorize("hasRole('ADMIN')")
    public List<SessionInformation> getUserSessions(@PathVariable String username) {
        return sessionRegistry.getAllSessions(
            new UsernamePasswordAuthenticationToken(username, null),
            false  // includeExpiredSessions
        );
    }
    
    // Invalidate a session
    @DeleteMapping("/admin/sessions/{sessionId}")
    @PreAuthorize("hasRole('ADMIN')")
    public void invalidateSession(@PathVariable String sessionId) {
        SessionInformation sessionInfo = sessionRegistry.getSessionInformation(sessionId);
        if (sessionInfo != null) {
            sessionInfo.expireNow();
        }
    }
    
    // Get current session info
    @GetMapping("/session/current")
    public Map<String, Object> getCurrentSession(HttpSession session) {
        Map<String, Object> info = new HashMap<>();
        info.put("sessionId", session.getId());
        info.put("creationTime", new Date(session.getCreationTime()));
        info.put("lastAccessedTime", new Date(session.getLastAccessedTime()));
        info.put("maxInactiveInterval", session.getMaxInactiveInterval());
        return info;
    }
}
```

---

## 14. Remember-Me

### Q23: How to implement Remember-Me functionality?

```java
@Configuration
@EnableWebSecurity
public class RememberMeConfig {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Autowired
    private DataSource dataSource;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            )
            .rememberMe(remember -> remember
                // Simple token-based (hash-based)
                .key("mySecretKey")
                .tokenValiditySeconds(86400 * 14)  // 14 days
                .rememberMeParameter("remember-me")
                .rememberMeCookieName("remember-me")
                .userDetailsService(userDetailsService)
                
                // OR Persistent token-based (more secure)
                // .tokenRepository(persistentTokenRepository())
                // .tokenValiditySeconds(86400 * 14)
                // .userDetailsService(userDetailsService)
            );
        
        return http.build();
    }
    
    // Persistent token repository (database-backed)
    @Bean
    public PersistentTokenRepository persistentTokenRepository() {
        JdbcTokenRepositoryImpl tokenRepository = new JdbcTokenRepositoryImpl();
        tokenRepository.setDataSource(dataSource);
        // tokenRepository.setCreateTableOnStartup(true);  // Create table if not exists
        return tokenRepository;
    }
}

// SQL for persistent_logins table
/*
CREATE TABLE persistent_logins (
    username VARCHAR(64) NOT NULL,
    series VARCHAR(64) PRIMARY KEY,
    token VARCHAR(64) NOT NULL,
    last_used TIMESTAMP NOT NULL
);
*/
```

| Type | Security | Token Storage |
|------|----------|---------------|
| Simple (Hash-based) | Less secure | Cookie only |
| Persistent | More secure | Database + Cookie |

---

## 15. Security Headers

### Q24: What security headers does Spring Security add?

```java
@Configuration
@EnableWebSecurity
public class HeadersConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .headers(headers -> headers
                // Cache Control
                .cacheControl(cache -> cache.disable())  // or .cacheControl()
                
                // Content Type Options
                .contentTypeOptions(Customizer.withDefaults())  // X-Content-Type-Options: nosniff
                
                // X-Frame-Options (Clickjacking protection)
                .frameOptions(frame -> frame
                    .deny()  // or .sameOrigin() or .disable()
                )
                
                // XSS Protection
                .xssProtection(xss -> xss
                    .headerValue(XXssProtectionHeaderWriter.HeaderValue.ENABLED_MODE_BLOCK)
                )
                
                // HTTP Strict Transport Security
                .httpStrictTransportSecurity(hsts -> hsts
                    .includeSubDomains(true)
                    .maxAgeInSeconds(31536000)  // 1 year
                    .preload(true)
                )
                
                // Content Security Policy
                .contentSecurityPolicy(csp -> csp
                    .policyDirectives("default-src 'self'; " +
                        "script-src 'self' 'unsafe-inline'; " +
                        "style-src 'self' 'unsafe-inline'; " +
                        "img-src 'self' data:; " +
                        "font-src 'self'; " +
                        "frame-ancestors 'none';")
                )
                
                // Referrer Policy
                .referrerPolicy(referrer -> referrer
                    .policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.STRICT_ORIGIN_WHEN_CROSS_ORIGIN)
                )
                
                // Permissions Policy (formerly Feature Policy)
                .permissionsPolicy(permissions -> permissions
                    .policy("camera=(), microphone=(), geolocation=()")
                )
            );
        
        return http.build();
    }
}
```

**Default Security Headers:**
| Header | Value | Purpose |
|--------|-------|---------|
| Cache-Control | no-cache, no-store... | Prevent caching |
| X-Content-Type-Options | nosniff | Prevent MIME sniffing |
| X-Frame-Options | DENY | Prevent clickjacking |
| X-XSS-Protection | 1; mode=block | XSS filter |
| Strict-Transport-Security | max-age=31536000 | Force HTTPS |

---

## 16. Custom Authentication

### Q25: How to implement custom authentication?

```java
// Custom Authentication Token
public class ApiKeyAuthenticationToken extends AbstractAuthenticationToken {
    
    private final String apiKey;
    private Object principal;
    
    // Unauthenticated token
    public ApiKeyAuthenticationToken(String apiKey) {
        super(null);
        this.apiKey = apiKey;
        setAuthenticated(false);
    }
    
    // Authenticated token
    public ApiKeyAuthenticationToken(Object principal, String apiKey, 
            Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.principal = principal;
        this.apiKey = apiKey;
        setAuthenticated(true);
    }
    
    @Override
    public Object getCredentials() {
        return apiKey;
    }
    
    @Override
    public Object getPrincipal() {
        return principal;
    }
}

// Custom Authentication Provider
@Component
public class ApiKeyAuthenticationProvider implements AuthenticationProvider {
    
    @Autowired
    private ApiKeyRepository apiKeyRepository;
    
    @Override
    public Authentication authenticate(Authentication authentication) 
            throws AuthenticationException {
        
        String apiKey = (String) authentication.getCredentials();
        
        ApiKeyEntity apiKeyEntity = apiKeyRepository.findByKey(apiKey)
            .orElseThrow(() -> new BadCredentialsException("Invalid API key"));
        
        if (!apiKeyEntity.isActive()) {
            throw new DisabledException("API key is disabled");
        }
        
        if (apiKeyEntity.isExpired()) {
            throw new CredentialsExpiredException("API key has expired");
        }
        
        List<GrantedAuthority> authorities = apiKeyEntity.getPermissions().stream()
            .map(SimpleGrantedAuthority::new)
            .collect(Collectors.toList());
        
        return new ApiKeyAuthenticationToken(
            apiKeyEntity.getOwner(),
            apiKey,
            authorities
        );
    }
    
    @Override
    public boolean supports(Class<?> authentication) {
        return ApiKeyAuthenticationToken.class.isAssignableFrom(authentication);
    }
}

// Custom Authentication Filter
@Component
public class ApiKeyAuthenticationFilter extends OncePerRequestFilter {
    
    private static final String API_KEY_HEADER = "X-API-Key";
    
    @Autowired
    private AuthenticationManager authenticationManager;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) 
            throws ServletException, IOException {
        
        String apiKey = request.getHeader(API_KEY_HEADER);
        
        if (apiKey != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            try {
                ApiKeyAuthenticationToken token = new ApiKeyAuthenticationToken(apiKey);
                Authentication auth = authenticationManager.authenticate(token);
                SecurityContextHolder.getContext().setAuthentication(auth);
            } catch (AuthenticationException e) {
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                response.getWriter().write("Invalid API key");
                return;
            }
        }
        
        filterChain.doFilter(request, response);
    }
    
    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) {
        // Only apply to /api/** endpoints
        return !request.getRequestURI().startsWith("/api/");
    }
}

// Security Configuration
@Configuration
@EnableWebSecurity
public class CustomAuthConfig {
    
    @Autowired
    private ApiKeyAuthenticationProvider apiKeyAuthProvider;
    
    @Autowired
    private ApiKeyAuthenticationFilter apiKeyAuthFilter;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authenticationProvider(apiKeyAuthProvider)
            .addFilterBefore(apiKeyAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/**").authenticated()
                .anyRequest().permitAll()
            );
        
        return http.build();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

---

## 17. Testing Security

### Q26: How to test Spring Security?

```java
// Add test dependencies
// testImplementation 'org.springframework.security:spring-security-test'

@SpringBootTest
@AutoConfigureMockMvc
class SecurityControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    // Test unauthenticated access
    @Test
    void whenUnauthenticated_thenUnauthorized() throws Exception {
        mockMvc.perform(get("/api/users"))
            .andExpect(status().isUnauthorized());
    }
    
    // Test with mock user
    @Test
    @WithMockUser(username = "user", roles = {"USER"})
    void whenUser_thenAccessUserEndpoint() throws Exception {
        mockMvc.perform(get("/api/users/profile"))
            .andExpect(status().isOk());
    }
    
    // Test with specific authorities
    @Test
    @WithMockUser(username = "admin", authorities = {"ADMIN", "USER_READ"})
    void whenAdmin_thenAccessAdminEndpoint() throws Exception {
        mockMvc.perform(get("/api/admin/users"))
            .andExpect(status().isOk());
    }
    
    // Test access denied
    @Test
    @WithMockUser(username = "user", roles = {"USER"})
    void whenUser_thenForbiddenOnAdminEndpoint() throws Exception {
        mockMvc.perform(get("/api/admin/users"))
            .andExpect(status().isForbidden());
    }
    
    // Test with custom UserDetails
    @Test
    @WithUserDetails(value = "john@example.com", userDetailsServiceBeanName = "customUserDetailsService")
    void whenCustomUser_thenAccessProfile() throws Exception {
        mockMvc.perform(get("/api/users/profile"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }
    
    // Test with JWT token
    @Test
    void whenValidJwt_thenAccessProtectedEndpoint() throws Exception {
        String jwt = jwtUtil.generateToken(testUserDetails);
        
        mockMvc.perform(get("/api/users/profile")
                .header("Authorization", "Bearer " + jwt))
            .andExpect(status().isOk());
    }
    
    // Test form login
    @Test
    void whenValidCredentials_thenLoginSuccessful() throws Exception {
        mockMvc.perform(post("/login")
                .with(csrf())
                .param("username", "user@example.com")
                .param("password", "password"))
            .andExpect(status().is3xxRedirection())
            .andExpect(redirectedUrl("/dashboard"));
    }
    
    // Test with SecurityMockMvcRequestPostProcessors
    @Test
    void whenBasicAuth_thenAccessEndpoint() throws Exception {
        mockMvc.perform(get("/api/users")
                .with(httpBasic("user", "password")))
            .andExpect(status().isOk());
    }
    
    // Test CSRF
    @Test
    void whenNoCsrf_thenForbidden() throws Exception {
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\": \"John\"}"))
            .andExpect(status().isForbidden());
    }
    
    @Test
    void whenWithCsrf_thenSuccess() throws Exception {
        mockMvc.perform(post("/api/users")
                .with(csrf())
                .with(user("admin").roles("ADMIN"))
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\": \"John\"}"))
            .andExpect(status().isCreated());
    }
}

// Custom annotation for testing
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithMockAdminSecurityContextFactory.class)
public @interface WithMockAdmin {
    String username() default "admin";
}

class WithMockAdminSecurityContextFactory implements WithSecurityContextFactory<WithMockAdmin> {
    @Override
    public SecurityContext createSecurityContext(WithMockAdmin annotation) {
        SecurityContext context = SecurityContextHolder.createEmptyContext();
        
        List<GrantedAuthority> authorities = Arrays.asList(
            new SimpleGrantedAuthority("ROLE_ADMIN"),
            new SimpleGrantedAuthority("ROLE_USER")
        );
        
        Authentication auth = new UsernamePasswordAuthenticationToken(
            annotation.username(), "password", authorities);
        
        context.setAuthentication(auth);
        return context;
    }
}
```

---

## 18. Common Vulnerabilities

### Q27: How does Spring Security protect against common attacks?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SECURITY VULNERABILITIES & PROTECTION                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Attack              │ Protection                   │ Spring Security      │
│   ────────────────────┼──────────────────────────────┼────────────────────  │
│   CSRF                │ CSRF tokens                  │ Enabled by default   │
│   XSS                 │ Content-Type header          │ X-XSS-Protection     │
│   Clickjacking        │ X-Frame-Options              │ DENY by default      │
│   Session Fixation    │ New session on login         │ migrateSession()     │
│   Brute Force         │ Rate limiting, lockout       │ Custom impl needed   │
│   MIME Sniffing       │ X-Content-Type-Options       │ nosniff              │
│   SQL Injection       │ Parameterized queries        │ JPA/Hibernate        │
│   Password Storage    │ BCrypt/Argon2                │ PasswordEncoder      │
│   Man-in-the-Middle   │ HTTPS, HSTS                  │ Channel security     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```java
// Brute Force Protection (Custom Implementation)
@Component
public class LoginAttemptService {
    
    private final LoadingCache<String, Integer> attemptsCache;
    private static final int MAX_ATTEMPTS = 5;
    
    public LoginAttemptService() {
        attemptsCache = CacheBuilder.newBuilder()
            .expireAfterWrite(15, TimeUnit.MINUTES)
            .build(new CacheLoader<String, Integer>() {
                @Override
                public Integer load(String key) {
                    return 0;
                }
            });
    }
    
    public void loginSucceeded(String key) {
        attemptsCache.invalidate(key);
    }
    
    public void loginFailed(String key) {
        int attempts = attemptsCache.getUnchecked(key);
        attemptsCache.put(key, attempts + 1);
    }
    
    public boolean isBlocked(String key) {
        return attemptsCache.getUnchecked(key) >= MAX_ATTEMPTS;
    }
}

// Event Listener for login attempts
@Component
public class AuthenticationEventListener {
    
    @Autowired
    private LoginAttemptService loginAttemptService;
    
    @Autowired
    private HttpServletRequest request;
    
    @EventListener
    public void onAuthenticationSuccess(AuthenticationSuccessEvent event) {
        String ip = getClientIP();
        loginAttemptService.loginSucceeded(ip);
    }
    
    @EventListener
    public void onAuthenticationFailure(AuthenticationFailureBadCredentialsEvent event) {
        String ip = getClientIP();
        loginAttemptService.loginFailed(ip);
    }
    
    private String getClientIP() {
        String xfHeader = request.getHeader("X-Forwarded-For");
        if (xfHeader == null) {
            return request.getRemoteAddr();
        }
        return xfHeader.split(",")[0];
    }
}
```

---

## 19. Best Practices

### Q28: Spring Security Best Practices

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SPRING SECURITY BEST PRACTICES                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  AUTHENTICATION                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ✅ Use BCrypt or Argon2 for password hashing                               │
│  ✅ Implement account lockout after failed attempts                         │
│  ✅ Use MFA (Multi-Factor Authentication) for sensitive operations          │
│  ✅ Validate password strength                                              │
│  ✅ Use secure session management                                           │
│  ✅ Implement proper logout (invalidate session, clear cookies)             │
│                                                                             │
│  AUTHORIZATION                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ✅ Use principle of least privilege                                        │
│  ✅ Prefer role-based + attribute-based access control                      │
│  ✅ Validate authorization on server-side (never trust client)              │
│  ✅ Use method-level security for business logic                            │
│  ✅ Audit security-sensitive operations                                     │
│                                                                             │
│  JWT / TOKENS                                                               │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ✅ Use short expiration times (15-60 minutes)                              │
│  ✅ Implement refresh token rotation                                        │
│  ✅ Store tokens securely (HttpOnly cookies or secure storage)              │
│  ✅ Use strong signing keys (256+ bits)                                     │
│  ✅ Validate all claims (issuer, audience, expiration)                      │
│                                                                             │
│  CONFIGURATION                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ✅ Enable HTTPS only in production                                         │
│  ✅ Configure proper CORS policies                                          │
│  ✅ Enable CSRF for stateful applications                                   │
│  ✅ Use security headers (CSP, HSTS, X-Frame-Options)                       │
│  ✅ Externalize secrets (use environment variables or vault)                │
│  ✅ Keep dependencies updated                                               │
│                                                                             │
│  AVOID                                                                      │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ❌ Don't disable CSRF without understanding implications                   │
│  ❌ Don't store passwords in plain text                                     │
│  ❌ Don't expose stack traces in error messages                             │
│  ❌ Don't use default credentials                                           │
│  ❌ Don't trust user input without validation                               │
│  ❌ Don't log sensitive data (passwords, tokens)                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📝 Quick Reference

### Security Annotations

| Annotation | Purpose |
|------------|---------|
| `@EnableWebSecurity` | Enable Spring Security |
| `@EnableMethodSecurity` | Enable method security |
| `@PreAuthorize` | Check before method |
| `@PostAuthorize` | Check after method |
| `@PreFilter` | Filter input collection |
| `@PostFilter` | Filter output collection |
| `@Secured` | Role-based access |
| `@RolesAllowed` | JSR-250 standard |

### Common SpEL Expressions

| Expression | Description |
|------------|-------------|
| `hasRole('ADMIN')` | Has ROLE_ADMIN authority |
| `hasAnyRole('ADMIN', 'USER')` | Has any of the roles |
| `hasAuthority('READ')` | Has specific authority |
| `isAuthenticated()` | Is authenticated |
| `isAnonymous()` | Is anonymous user |
| `isFullyAuthenticated()` | Not remember-me |
| `principal.username` | Current username |
| `#paramName` | Method parameter |
| `returnObject` | Method return value |

### HTTP Status Codes

| Code | Meaning | When Used |
|------|---------|-----------|
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Not authorized |
| 419 | Authentication Timeout | Session expired |

---

*Document generated: December 2024*  
*Spring Security Version: 6.x*

