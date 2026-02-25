# Java Best Practices

Tổng hợp các best practices khi lập trình Java để viết code sạch, dễ bảo trì và hiệu quả.

## Naming Conventions

### Quy tắc đặt tên

| Loại | Convention | Ví dụ |
|------|------------|-------|
| Class | PascalCase | `UserService`, `ProductController` |
| Interface | PascalCase | `Comparable`, `UserRepository` |
| Method | camelCase | `getUserById()`, `calculateTotal()` |
| Variable | camelCase | `firstName`, `totalAmount` |
| Constant | UPPER_SNAKE_CASE | `MAX_SIZE`, `DEFAULT_TIMEOUT` |
| Package | lowercase | `com.example.service` |

### Đặt tên có ý nghĩa

```java
// BAD
int d; // elapsed time in days
List<int[]> list1;
public void process(String s) { }

// GOOD
int elapsedTimeInDays;
List<int[]> flaggedCells;
public void processUserInput(String userInput) { }
```

---

## Code Style

### Độ dài và format

```java
// BAD - quá dài
public void processOrder(Long orderId, Long customerId, List<OrderItem> items, String shippingAddress, String billingAddress, PaymentMethod paymentMethod) { }

// GOOD - chia nhỏ parameters
public void processOrder(OrderRequest request) { }

// Hoặc format nhiều dòng
public void processOrder(
        Long orderId,
        Long customerId,
        List<OrderItem> items,
        ShippingInfo shippingInfo,
        PaymentMethod paymentMethod) {
    // ...
}
```

### Early return

```java
// BAD - nested if
public String getStatus(User user) {
    if (user != null) {
        if (user.isActive()) {
            if (user.hasSubscription()) {
                return "Premium";
            } else {
                return "Free";
            }
        } else {
            return "Inactive";
        }
    } else {
        return "Unknown";
    }
}

// GOOD - early return
public String getStatus(User user) {
    if (user == null) {
        return "Unknown";
    }
    if (!user.isActive()) {
        return "Inactive";
    }
    if (user.hasSubscription()) {
        return "Premium";
    }
    return "Free";
}
```

### Avoid magic numbers

```java
// BAD
if (user.getAge() >= 18) { }
Thread.sleep(86400000);

// GOOD
private static final int ADULT_AGE = 18;
private static final long ONE_DAY_IN_MILLIS = 24 * 60 * 60 * 1000L;

if (user.getAge() >= ADULT_AGE) { }
Thread.sleep(ONE_DAY_IN_MILLIS);
```

---

## Object-Oriented Design

### Favor composition over inheritance

```java
// BAD - inheritance
public class EmailNotification extends Notification {
    public void send() {
        // send email
    }
}

// GOOD - composition
public class NotificationService {
    private final EmailSender emailSender;
    private final SmsSender smsSender;
    
    public void notify(User user, Message message, NotificationType type) {
        switch (type) {
            case EMAIL -> emailSender.send(user.getEmail(), message);
            case SMS -> smsSender.send(user.getPhone(), message);
        }
    }
}
```

### Program to interface

```java
// BAD
ArrayList<String> names = new ArrayList<>();
HashMap<String, User> users = new HashMap<>();

// GOOD
List<String> names = new ArrayList<>();
Map<String, User> users = new HashMap<>();

// Service layer
public interface UserRepository {
    User findById(Long id);
    void save(User user);
}

public class JpaUserRepository implements UserRepository {
    // implementation
}
```

### Immutability

```java
// BAD - mutable
public class User {
    private String name;
    private List<String> roles;
    
    public List<String> getRoles() {
        return roles;  // Có thể bị modify từ bên ngoài
    }
}

// GOOD - immutable
public final class User {
    private final String name;
    private final List<String> roles;
    
    public User(String name, List<String> roles) {
        this.name = name;
        this.roles = List.copyOf(roles);  // Defensive copy
    }
    
    public List<String> getRoles() {
        return roles;  // Đã là unmodifiable
    }
}

// Java 16+ Record
public record User(String name, List<String> roles) {
    public User {
        roles = List.copyOf(roles);
    }
}
```

---

## Exception Handling

### Catch specific exceptions

```java
// BAD
try {
    processFile(file);
} catch (Exception e) {
    log.error("Error", e);
}

// GOOD
try {
    processFile(file);
} catch (FileNotFoundException e) {
    log.error("File not found: {}", file.getName(), e);
    throw new ResourceNotFoundException("File not found", e);
} catch (IOException e) {
    log.error("Error reading file: {}", file.getName(), e);
    throw new ProcessingException("Error processing file", e);
}
```

### Don't swallow exceptions

```java
// BAD
try {
    riskyOperation();
} catch (Exception e) {
    // Swallowed - không biết có lỗi
}

// BAD
try {
    riskyOperation();
} catch (Exception e) {
    e.printStackTrace();  // Không log properly
}

// GOOD
try {
    riskyOperation();
} catch (SpecificException e) {
    log.error("Operation failed: {}", e.getMessage(), e);
    throw new ServiceException("Operation failed", e);
}
```

### Use try-with-resources

```java
// BAD
FileInputStream fis = null;
try {
    fis = new FileInputStream(file);
    // process
} finally {
    if (fis != null) {
        try {
            fis.close();
        } catch (IOException e) {
            // ignore
        }
    }
}

// GOOD
try (FileInputStream fis = new FileInputStream(file);
     BufferedReader reader = new BufferedReader(new InputStreamReader(fis))) {
    // process
}
```

---

## Collections

### Choose the right collection

| Use case | Collection |
|----------|------------|
| Ordered, allow duplicates | ArrayList |
| Frequent insert/delete | LinkedList |
| No duplicates | HashSet |
| Sorted, no duplicates | TreeSet |
| Key-value pairs | HashMap |
| Sorted key-value | TreeMap |
| Thread-safe | ConcurrentHashMap |
| Queue | ArrayDeque |

### Initialize with capacity

```java
// BAD - resize nhiều lần
List<String> names = new ArrayList<>();
for (int i = 0; i < 10000; i++) {
    names.add("name" + i);
}

// GOOD - biết trước size
List<String> names = new ArrayList<>(10000);
for (int i = 0; i < 10000; i++) {
    names.add("name" + i);
}
```

### Use Stream API appropriately

```java
// BAD - quá phức tạp
list.stream()
    .filter(x -> x != null)
    .map(x -> x.toString())
    .filter(x -> !x.isEmpty())
    .map(x -> x.toUpperCase())
    .filter(x -> x.startsWith("A"))
    .collect(Collectors.toList());

// GOOD - tách ra hoặc dùng method reference
list.stream()
    .filter(Objects::nonNull)
    .map(Object::toString)
    .filter(s -> !s.isEmpty())
    .map(String::toUpperCase)
    .filter(s -> s.startsWith("A"))
    .toList();

// Hoặc tách logic phức tạp
list.stream()
    .filter(this::isValidItem)
    .map(this::transformItem)
    .toList();
```

---

## String Handling

### Use StringBuilder for concatenation

```java
// BAD - tạo nhiều String objects
String result = "";
for (String s : list) {
    result += s + ", ";
}

// GOOD
StringBuilder sb = new StringBuilder();
for (String s : list) {
    sb.append(s).append(", ");
}
String result = sb.toString();

// BETTER - Java 8+
String result = String.join(", ", list);
```

### String comparison

```java
// BAD - có thể NullPointerException
if (input.equals("expected")) { }

// GOOD - constant first
if ("expected".equals(input)) { }

// GOOD - null-safe
if (Objects.equals(input, "expected")) { }

// GOOD - Java 14+ pattern matching
if (input instanceof String s && s.equals("expected")) { }
```

---

## Null Handling

### Use Optional

```java
// BAD
public User findUser(Long id) {
    User user = repository.findById(id);
    if (user == null) {
        return null;  // Caller phải check null
    }
    return user;
}

// GOOD
public Optional<User> findUser(Long id) {
    return repository.findById(id);
}

// Usage
userService.findUser(id)
    .map(User::getName)
    .orElse("Unknown");

userService.findUser(id)
    .orElseThrow(() -> new UserNotFoundException(id));
```

### Avoid returning null collections

```java
// BAD
public List<Order> getOrders(Long userId) {
    List<Order> orders = repository.findByUserId(userId);
    return orders;  // Có thể null
}

// GOOD
public List<Order> getOrders(Long userId) {
    List<Order> orders = repository.findByUserId(userId);
    return orders != null ? orders : Collections.emptyList();
}

// BETTER
public List<Order> getOrders(Long userId) {
    return repository.findByUserId(userId);  // Repository trả về empty list
}
```

---

## Performance

### Avoid premature optimization

```java
// Đừng optimize quá sớm
// Viết code đúng và dễ đọc trước
// Profile để tìm bottleneck thực sự
// Chỉ optimize khi cần thiết
```

### Use lazy initialization

```java
// Eager - luôn tạo
public class Service {
    private final ExpensiveResource resource = new ExpensiveResource();
}

// Lazy - chỉ tạo khi cần
public class Service {
    private ExpensiveResource resource;
    
    public ExpensiveResource getResource() {
        if (resource == null) {
            resource = new ExpensiveResource();
        }
        return resource;
    }
}

// Thread-safe lazy
public class Service {
    private volatile ExpensiveResource resource;
    
    public ExpensiveResource getResource() {
        if (resource == null) {
            synchronized (this) {
                if (resource == null) {
                    resource = new ExpensiveResource();
                }
            }
        }
        return resource;
    }
}
```

### Cache expensive operations

```java
@Service
public class ProductService {
    
    @Cacheable("products")
    public Product getProduct(Long id) {
        return repository.findById(id).orElseThrow();
    }
    
    @CacheEvict(value = "products", key = "#product.id")
    public void updateProduct(Product product) {
        repository.save(product);
    }
}
```

---

## Testing

### Write testable code

```java
// BAD - hard to test
public class OrderService {
    public void processOrder(Order order) {
        // Gọi trực tiếp static method
        EmailUtil.sendEmail(order.getCustomerEmail(), "Order confirmed");
        
        // Tạo dependency trong method
        PaymentGateway gateway = new StripePaymentGateway();
        gateway.charge(order.getTotal());
    }
}

// GOOD - dependency injection
public class OrderService {
    private final EmailService emailService;
    private final PaymentGateway paymentGateway;
    
    public OrderService(EmailService emailService, PaymentGateway paymentGateway) {
        this.emailService = emailService;
        this.paymentGateway = paymentGateway;
    }
    
    public void processOrder(Order order) {
        emailService.send(order.getCustomerEmail(), "Order confirmed");
        paymentGateway.charge(order.getTotal());
    }
}
```

### Unit test structure

```java
@Test
void shouldCalculateTotalWithDiscount() {
    // Arrange (Given)
    Order order = new Order();
    order.addItem(new OrderItem("Product A", 100, 2));
    order.setDiscountPercent(10);
    
    // Act (When)
    BigDecimal total = order.calculateTotal();
    
    // Assert (Then)
    assertThat(total).isEqualTo(new BigDecimal("180.00"));
}
```

---

## Logging

### Use appropriate log levels

```java
// TRACE - very detailed, for debugging
log.trace("Entering method with params: {}", params);

// DEBUG - debugging information
log.debug("Processing item: {}", item);

// INFO - important business events
log.info("Order {} created for customer {}", orderId, customerId);

// WARN - potential problems
log.warn("Retry attempt {} for operation {}", attempt, operation);

// ERROR - errors that need attention
log.error("Failed to process payment for order {}", orderId, exception);
```

### Use parameterized logging

```java
// BAD - string concatenation always happens
log.debug("Processing order: " + order.getId() + " for customer: " + customer.getName());

// GOOD - only evaluated if log level is enabled
log.debug("Processing order: {} for customer: {}", order.getId(), customer.getName());
```

---

## Security

### Never log sensitive data

```java
// BAD
log.info("User login: username={}, password={}", username, password);
log.debug("Credit card: {}", creditCardNumber);

// GOOD
log.info("User login: username={}", username);
log.debug("Payment processed for card ending in {}", last4Digits);
```

### Validate input

```java
public void updateUser(Long id, UserUpdateRequest request) {
    // Validate
    if (id == null || id <= 0) {
        throw new IllegalArgumentException("Invalid user ID");
    }
    
    Objects.requireNonNull(request, "Request cannot be null");
    
    // Sanitize
    String name = StringUtils.trimToNull(request.getName());
    if (name == null || name.length() > 100) {
        throw new ValidationException("Invalid name");
    }
    
    // Process
    // ...
}
```

---

## Checklist

Trước khi commit code, kiểm tra:

- [ ] Tên biến/method có ý nghĩa
- [ ] Không có magic numbers
- [ ] Exception được handle đúng cách
- [ ] Không có null pointer risks
- [ ] Code được format đúng
- [ ] Có unit tests
- [ ] Không log sensitive data
- [ ] Input được validate

## Tiếp theo

- [SQL cơ bản](../sql/index.md)
