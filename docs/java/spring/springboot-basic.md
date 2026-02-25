# Spring Boot cơ bản

Spring Boot là một framework giúp tạo ứng dụng Spring một cách nhanh chóng với cấu hình tối thiểu.

## Tổng quan

### Spring Boot là gì?

Spring Boot là một extension của Spring Framework, cung cấp:

- **Auto-configuration**: Tự động cấu hình dựa trên dependencies
- **Starter dependencies**: Các dependency được đóng gói sẵn
- **Embedded server**: Tomcat/Jetty tích hợp sẵn
- **Production-ready**: Actuator, metrics, health checks

### Tạo Project

**Cách 1: Spring Initializr**

Truy cập [start.spring.io](https://start.spring.io) và chọn:
- Project: Maven
- Language: Java
- Spring Boot: 3.x
- Dependencies: Spring Web, Spring Data JPA, MySQL Driver, Lombok

**Cách 2: IDE**

IntelliJ IDEA: File → New → Project → Spring Initializr

---

## Cấu trúc Project

```
my-app/
├── src/
│   ├── main/
│   │   ├── java/com/example/myapp/
│   │   │   ├── MyAppApplication.java
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── model/
│   │   │   └── config/
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── application.yml
│   │       ├── static/
│   │       └── templates/
│   └── test/
│       └── java/com/example/myapp/
├── pom.xml
└── README.md
```

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    
    <properties>
        <java.version>17</java.version>
    </properties>
    
    <dependencies>
        <!-- Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <!-- MySQL -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## Main Application

```java
package com.example.myapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyAppApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyAppApplication.class, args);
    }
}
```

`@SpringBootApplication` bao gồm:
- `@Configuration`: Đánh dấu class là source của bean definitions
- `@EnableAutoConfiguration`: Bật auto-configuration
- `@ComponentScan`: Scan các component trong package

---

## Configuration

### application.properties

```properties
# Server
server.port=8080
server.servlet.context-path=/api

# Database
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Logging
logging.level.root=INFO
logging.level.com.example=DEBUG
```

### application.yml

```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

logging:
  level:
    root: INFO
    com.example: DEBUG
```

### Profiles

```yaml
# application.yml (default)
spring:
  profiles:
    active: dev

---
# application-dev.yml
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:mysql://localhost:3306/mydb_dev

---
# application-prod.yml
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:mysql://prod-server:3306/mydb_prod
```

Chạy với profile:
```bash
java -jar app.jar --spring.profiles.active=prod
```

---

## Dependency Injection

### @Component và các biến thể

```java
@Component          // Generic component
@Service            // Business logic layer
@Repository         // Data access layer
@Controller         // Web controller (MVC)
@RestController     // REST API controller
@Configuration      // Configuration class
```

### Constructor Injection (Khuyến nghị)

```java
@Service
public class ProductService {
    
    private final ProductRepository productRepository;
    private final CategoryRepository categoryRepository;
    
    // Constructor injection - Spring tự động inject
    public ProductService(ProductRepository productRepository,
                          CategoryRepository categoryRepository) {
        this.productRepository = productRepository;
        this.categoryRepository = categoryRepository;
    }
}

// Với Lombok
@Service
@RequiredArgsConstructor
public class ProductService {
    private final ProductRepository productRepository;
    private final CategoryRepository categoryRepository;
}
```

### @Autowired

```java
@Service
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    // Hoặc setter injection
    @Autowired
    public void setProductRepository(ProductRepository repo) {
        this.productRepository = repo;
    }
}
```

### @Bean

```java
@Configuration
public class AppConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule());
        return mapper;
    }
}
```

---

## REST Controller

### Basic Controller

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    // GET /api/products
    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll();
    }
    
    // GET /api/products/1
    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.findById(id);
    }
    
    // POST /api/products
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Product createProduct(@RequestBody Product product) {
        return productService.save(product);
    }
    
    // PUT /api/products/1
    @PutMapping("/{id}")
    public Product updateProduct(@PathVariable Long id, 
                                 @RequestBody Product product) {
        return productService.update(id, product);
    }
    
    // DELETE /api/products/1
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteProduct(@PathVariable Long id) {
        productService.delete(id);
    }
}
```

### Request Parameters

```java
@GetMapping("/search")
public List<Product> searchProducts(
        @RequestParam String keyword,
        @RequestParam(required = false) Long categoryId,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) {
    return productService.search(keyword, categoryId, page, size);
}

// GET /api/products/search?keyword=phone&categoryId=1&page=0&size=10
```

### Request Headers

```java
@GetMapping("/secure")
public String secureEndpoint(
        @RequestHeader("Authorization") String authHeader,
        @RequestHeader(value = "X-Custom-Header", required = false) String customHeader) {
    return "OK";
}
```

### ResponseEntity

```java
@GetMapping("/{id}")
public ResponseEntity<Product> getProduct(@PathVariable Long id) {
    Product product = productService.findById(id);
    
    if (product == null) {
        return ResponseEntity.notFound().build();
    }
    
    return ResponseEntity.ok(product);
}

@PostMapping
public ResponseEntity<Product> createProduct(@RequestBody Product product) {
    Product saved = productService.save(product);
    
    URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(saved.getId())
            .toUri();
    
    return ResponseEntity.created(location).body(saved);
}
```

---

## Validation

### Entity Validation

```java
@Entity
@Data
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be 2-100 characters")
    private String name;
    
    @NotNull(message = "Price is required")
    @Positive(message = "Price must be positive")
    private BigDecimal price;
    
    @Min(value = 0, message = "Stock cannot be negative")
    private Integer stock;
    
    @Email(message = "Invalid email format")
    private String contactEmail;
}
```

### Controller Validation

```java
@PostMapping
public ResponseEntity<Product> createProduct(
        @Valid @RequestBody Product product,
        BindingResult result) {
    
    if (result.hasErrors()) {
        // Handle validation errors
        throw new ValidationException(result);
    }
    
    return ResponseEntity.ok(productService.save(product));
}
```

### Custom Validator

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

@Component
public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        return email != null && !userRepository.existsByEmail(email);
    }
}
```

---

## Exception Handling

### Global Exception Handler

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
        Map<String, String> errors = new HashMap<>();
        
        ex.getBindingResult().getFieldErrors().forEach(error -> 
            errors.put(error.getField(), error.getDefaultMessage())
        );
        
        return new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            errors,
            LocalDateTime.now()
        );
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        return new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal server error",
            LocalDateTime.now()
        );
    }
}

@Data
@AllArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private Object errors;
    private LocalDateTime timestamp;
    
    public ErrorResponse(int status, String message, LocalDateTime timestamp) {
        this(status, message, null, timestamp);
    }
}
```

### Custom Exception

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    
    public ResourceNotFoundException(String message) {
        super(message);
    }
    
    public ResourceNotFoundException(String resource, Long id) {
        super(String.format("%s not found with id: %d", resource, id));
    }
}
```

---

## Chạy ứng dụng

### Maven

```bash
# Development
mvn spring-boot:run

# Build JAR
mvn clean package

# Run JAR
java -jar target/my-app-1.0.0.jar
```

### IDE

Run `MyAppApplication.main()` method

---

## Bài tập thực hành

!!! example "Bài tập"
    Tạo REST API quản lý sách với:
    
    1. Entity: Book (id, title, author, isbn, price)
    2. CRUD endpoints
    3. Validation
    4. Exception handling
    5. Pagination

## Tiếp theo

- [Spring MVC](spring-mvc.md)
