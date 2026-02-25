# Spring Data JPA

Spring Data JPA giúp đơn giản hóa việc truy cập database bằng cách tự động tạo implementation cho các repository interface.

## Tổng quan

### JPA là gì?

**JPA (Java Persistence API)** là một specification định nghĩa cách ánh xạ Java objects với database tables. **Hibernate** là implementation phổ biến nhất của JPA.

### Spring Data JPA

Spring Data JPA xây dựng trên JPA, cung cấp:
- Repository abstraction
- Query methods tự động
- Pagination và Sorting
- Auditing

---

## Entity

### Basic Entity

```java
@Entity
@Table(name = "products")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(columnDefinition = "TEXT")
    private String description;
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal price;
    
    @Column(nullable = false)
    private Integer stock = 0;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

### Relationships

```java
// One-to-Many
@Entity
public class Category {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL)
    private List<Product> products = new ArrayList<>();
}

@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;
}

// Many-to-Many
@Entity
public class Product {
    @ManyToMany
    @JoinTable(
        name = "product_tags",
        joinColumns = @JoinColumn(name = "product_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    private Set<Tag> tags = new HashSet<>();
}

@Entity
public class Tag {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToMany(mappedBy = "tags")
    private Set<Product> products = new HashSet<>();
}

// One-to-One
@Entity
public class User {
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
}
```

---

## Repository

### JpaRepository

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    // Các method CRUD có sẵn:
    // save(entity), saveAll(entities)
    // findById(id), findAll()
    // existsById(id), count()
    // deleteById(id), delete(entity), deleteAll()
}
```

### Query Methods

Spring Data JPA tự động tạo query từ tên method:

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    // SELECT * FROM products WHERE name = ?
    List<Product> findByName(String name);
    
    // SELECT * FROM products WHERE name LIKE ?
    List<Product> findByNameContaining(String keyword);
    List<Product> findByNameStartingWith(String prefix);
    List<Product> findByNameEndingWith(String suffix);
    
    // SELECT * FROM products WHERE price > ?
    List<Product> findByPriceGreaterThan(BigDecimal price);
    List<Product> findByPriceLessThanEqual(BigDecimal price);
    List<Product> findByPriceBetween(BigDecimal min, BigDecimal max);
    
    // SELECT * FROM products WHERE category_id = ?
    List<Product> findByCategoryId(Long categoryId);
    List<Product> findByCategory(Category category);
    
    // SELECT * FROM products WHERE name = ? AND price > ?
    List<Product> findByNameAndPriceGreaterThan(String name, BigDecimal price);
    
    // SELECT * FROM products WHERE name = ? OR description LIKE ?
    List<Product> findByNameOrDescriptionContaining(String name, String keyword);
    
    // SELECT * FROM products WHERE stock IS NULL
    List<Product> findByStockIsNull();
    List<Product> findByStockIsNotNull();
    
    // SELECT * FROM products ORDER BY price DESC
    List<Product> findByOrderByPriceDesc();
    List<Product> findByCategoryIdOrderByPriceAsc(Long categoryId);
    
    // SELECT * FROM products WHERE active = true
    List<Product> findByActiveTrue();
    List<Product> findByActiveFalse();
    
    // SELECT TOP 5 * FROM products ORDER BY created_at DESC
    List<Product> findTop5ByOrderByCreatedAtDesc();
    Product findFirstByOrderByPriceDesc();
    
    // COUNT, EXISTS
    long countByCategoryId(Long categoryId);
    boolean existsByName(String name);
    
    // DELETE
    void deleteByName(String name);
    long deleteByCategoryId(Long categoryId);
}
```

### @Query Annotation

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    // JPQL
    @Query("SELECT p FROM Product p WHERE p.price > :minPrice")
    List<Product> findExpensiveProducts(@Param("minPrice") BigDecimal minPrice);
    
    @Query("SELECT p FROM Product p WHERE p.category.name = :categoryName")
    List<Product> findByCategoryName(@Param("categoryName") String categoryName);
    
    @Query("SELECT p FROM Product p JOIN p.tags t WHERE t.name = :tagName")
    List<Product> findByTagName(@Param("tagName") String tagName);
    
    // Native SQL
    @Query(value = "SELECT * FROM products WHERE MATCH(name, description) AGAINST(:keyword)", 
           nativeQuery = true)
    List<Product> fullTextSearch(@Param("keyword") String keyword);
    
    // Update query
    @Modifying
    @Query("UPDATE Product p SET p.price = p.price * :factor WHERE p.category.id = :categoryId")
    int updatePriceByCategory(@Param("categoryId") Long categoryId, 
                              @Param("factor") BigDecimal factor);
    
    // Delete query
    @Modifying
    @Query("DELETE FROM Product p WHERE p.stock = 0")
    int deleteOutOfStock();
}
```

---

## Pagination và Sorting

### Pageable

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    Page<Product> findByCategoryId(Long categoryId, Pageable pageable);
    
    List<Product> findByPriceGreaterThan(BigDecimal price, Sort sort);
}

// Service
@Service
public class ProductService {
    
    public Page<Product> getProducts(int page, int size, String sortBy, String direction) {
        Sort sort = Sort.by(Sort.Direction.fromString(direction), sortBy);
        Pageable pageable = PageRequest.of(page, size, sort);
        return productRepository.findAll(pageable);
    }
    
    public Page<Product> getProductsByCategory(Long categoryId, int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("name").ascending());
        return productRepository.findByCategoryId(categoryId, pageable);
    }
}

// Controller
@GetMapping
public String list(@RequestParam(defaultValue = "0") int page,
                   @RequestParam(defaultValue = "10") int size,
                   @RequestParam(defaultValue = "id") String sortBy,
                   @RequestParam(defaultValue = "asc") String direction,
                   Model model) {
    
    Page<Product> productPage = productService.getProducts(page, size, sortBy, direction);
    
    model.addAttribute("products", productPage.getContent());
    model.addAttribute("currentPage", page);
    model.addAttribute("totalPages", productPage.getTotalPages());
    model.addAttribute("totalItems", productPage.getTotalElements());
    
    return "product/list";
}
```

### Thymeleaf Pagination

```html
<nav th:if="${totalPages > 1}">
    <ul class="pagination">
        <li th:classappend="${currentPage == 0} ? 'disabled'">
            <a th:href="@{/products(page=${currentPage - 1}, size=${size})}">Previous</a>
        </li>
        
        <li th:each="i : ${#numbers.sequence(0, totalPages - 1)}"
            th:classappend="${i == currentPage} ? 'active'">
            <a th:href="@{/products(page=${i}, size=${size})}" th:text="${i + 1}">1</a>
        </li>
        
        <li th:classappend="${currentPage == totalPages - 1} ? 'disabled'">
            <a th:href="@{/products(page=${currentPage + 1}, size=${size})}">Next</a>
        </li>
    </ul>
</nav>
```

---

## Specifications (Dynamic Queries)

```java
// Specification
public class ProductSpecifications {
    
    public static Specification<Product> hasName(String name) {
        return (root, query, cb) -> 
            name == null ? null : cb.like(root.get("name"), "%" + name + "%");
    }
    
    public static Specification<Product> hasCategory(Long categoryId) {
        return (root, query, cb) -> 
            categoryId == null ? null : cb.equal(root.get("category").get("id"), categoryId);
    }
    
    public static Specification<Product> hasPriceBetween(BigDecimal min, BigDecimal max) {
        return (root, query, cb) -> {
            if (min == null && max == null) return null;
            if (min == null) return cb.lessThanOrEqualTo(root.get("price"), max);
            if (max == null) return cb.greaterThanOrEqualTo(root.get("price"), min);
            return cb.between(root.get("price"), min, max);
        };
    }
}

// Repository
public interface ProductRepository extends JpaRepository<Product, Long>, 
                                          JpaSpecificationExecutor<Product> {
}

// Service
@Service
public class ProductService {
    
    public Page<Product> search(String name, Long categoryId, 
                                BigDecimal minPrice, BigDecimal maxPrice,
                                Pageable pageable) {
        
        Specification<Product> spec = Specification
            .where(ProductSpecifications.hasName(name))
            .and(ProductSpecifications.hasCategory(categoryId))
            .and(ProductSpecifications.hasPriceBetween(minPrice, maxPrice));
        
        return productRepository.findAll(spec, pageable);
    }
}
```

---

## Auditing

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext())
            .map(SecurityContext::getAuthentication)
            .map(Authentication::getName);
    }
}

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Data
public abstract class BaseEntity {
    
    @CreatedDate
    @Column(name = "created_at", updatable = false)
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
}

@Entity
public class Product extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    // ...
}
```

---

## Transactions

```java
@Service
@Transactional(readOnly = true)
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final ProductRepository productRepository;
    
    // Read-only transaction (inherited from class)
    public Order findById(Long id) {
        return orderRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Order", id));
    }
    
    // Write transaction
    @Transactional
    public Order createOrder(OrderRequest request) {
        Order order = new Order();
        order.setCustomerId(request.getCustomerId());
        
        for (OrderItemRequest item : request.getItems()) {
            Product product = productRepository.findById(item.getProductId())
                .orElseThrow(() -> new ResourceNotFoundException("Product", item.getProductId()));
            
            if (product.getStock() < item.getQuantity()) {
                throw new InsufficientStockException(product.getName());
            }
            
            product.setStock(product.getStock() - item.getQuantity());
            productRepository.save(product);
            
            order.addItem(product, item.getQuantity());
        }
        
        return orderRepository.save(order);
    }
    
    // Rollback on specific exception
    @Transactional(rollbackFor = Exception.class)
    public void processOrder(Long orderId) throws Exception {
        // ...
    }
}
```

---

## N+1 Problem và Solutions

### Problem

```java
// N+1 queries: 1 query for products + N queries for categories
List<Product> products = productRepository.findAll();
for (Product p : products) {
    System.out.println(p.getCategory().getName()); // Lazy load each category
}
```

### Solutions

```java
// 1. JOIN FETCH
@Query("SELECT p FROM Product p JOIN FETCH p.category")
List<Product> findAllWithCategory();

// 2. EntityGraph
@EntityGraph(attributePaths = {"category", "tags"})
List<Product> findAll();

// 3. Batch fetching (in application.properties)
// spring.jpa.properties.hibernate.default_batch_fetch_size=20
```

---

## Bài tập thực hành

!!! example "Bài tập"
    Xây dựng hệ thống quản lý đơn hàng:
    
    1. Entities: Order, OrderItem, Product, Customer
    2. Relationships: One-to-Many, Many-to-Many
    3. Repository với query methods và @Query
    4. Pagination và dynamic search
    5. Transaction management

## Tiếp theo

- [Spring Security](spring-security.md)
