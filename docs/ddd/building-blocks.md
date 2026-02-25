# Building Blocks - Các thành phần xây dựng DDD

Tài liệu này tổng hợp chi tiết các building blocks của DDD với ví dụ thực tế và best practices.

## 1. Domain Model Pattern

### Khái niệm

Domain Model là object model của domain, chứa cả behavior (hành vi) lẫn data. Khác với Anemic Domain Model (chỉ có data, logic ở Service layer).

### Rich Domain Model vs Anemic Domain Model

❌ **Anemic Domain Model (Anti-pattern):**
```java
// Chỉ có getter/setter - không có logic
public class Order {
    private Long id;
    private List<OrderLine> orderLines;
    private String status;
    private BigDecimal total;
    
    // Chỉ có getter/setter
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public List<OrderLine> getOrderLines() { return orderLines; }
    public void setOrderLines(List<OrderLine> orderLines) { 
        this.orderLines = orderLines; 
    }
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
    // ...
}

// Logic ở Service - tách rời khỏi data
public class OrderService {
    public void confirmOrder(Order order) {
        if (order.getOrderLines().isEmpty()) {
            throw new Exception("Empty order");
        }
        order.setStatus("CONFIRMED");
        
        BigDecimal total = BigDecimal.ZERO;
        for (OrderLine line : order.getOrderLines()) {
            total = total.add(line.getPrice().multiply(
                BigDecimal.valueOf(line.getQuantity())
            ));
        }
        order.setTotal(total);
    }
}
```

✅ **Rich Domain Model:**
```java
// Domain logic nằm trong Entity
public class Order {
    private OrderId id;
    private List<OrderLine> orderLines;
    private OrderStatus status;  // Enum, không phải String
    private Money total;  // Value Object
    
    // Constructor với validation
    private Order(OrderId id) {
        this.id = Objects.requireNonNull(id);
        this.orderLines = new ArrayList<>();
        this.status = OrderStatus.PENDING;
        this.total = Money.zero();
    }
    
    // Business method
    public void confirm() {
        if (orderLines.isEmpty()) {
            throw new EmptyOrderException("Cannot confirm empty order");
        }
        if (status != OrderStatus.PENDING) {
            throw new InvalidOrderStateException("Order already confirmed");
        }
        
        this.status = OrderStatus.CONFIRMED;
        recalculateTotal();
    }
    
    public void addItem(Product product, int quantity) {
        if (status != OrderStatus.PENDING) {
            throw new InvalidOrderStateException("Cannot modify confirmed order");
        }
        
        OrderLine line = new OrderLine(product, quantity);
        orderLines.add(line);
        recalculateTotal();
    }
    
    // Private helper
    private void recalculateTotal() {
        this.total = orderLines.stream()
            .map(OrderLine::getSubtotal)
            .reduce(Money.zero(), Money::add);
    }
    
    // Getters - no setters
    public Money getTotal() { return total; }
    public OrderStatus getStatus() { return status; }
}
```

## 2. Specifications Pattern

### Khái niệm

Specification pattern cho phép tách riêng business rules thành các object có thể tái sử dụng và kết hợp.

### Ví dụ

**Interface:**
```java
public interface Specification<T> {
    boolean isSatisfiedBy(T candidate);
    
    // Combinator methods
    default Specification<T> and(Specification<T> other) {
        return candidate -> this.isSatisfiedBy(candidate) 
                         && other.isSatisfiedBy(candidate);
    }
    
    default Specification<T> or(Specification<T> other) {
        return candidate -> this.isSatisfiedBy(candidate) 
                         || other.isSatisfiedBy(candidate);
    }
    
    default Specification<T> not() {
        return candidate -> !this.isSatisfiedBy(candidate);
    }
}
```

**Implementations:**
```java
// Specification: Customer có credit limit đủ
public class CreditLimitSpecification implements Specification<Customer> {
    private final Money minimumLimit;
    
    public CreditLimitSpecification(Money minimumLimit) {
        this.minimumLimit = minimumLimit;
    }
    
    @Override
    public boolean isSatisfiedBy(Customer customer) {
        return customer.getCreditLimit().isGreaterThanOrEqual(minimumLimit);
    }
}

// Specification: Customer ở region cụ thể
public class RegionSpecification implements Specification<Customer> {
    private final Region region;
    
    public RegionSpecification(Region region) {
        this.region = region;
    }
    
    @Override
    public boolean isSatisfiedBy(Customer customer) {
        return customer.getAddress().getRegion().equals(region);
    }
}

// Specification: Customer đủ điều kiện cho promotion
public class EligibleForPromotionSpecification implements Specification<Customer> {
    
    @Override
    public boolean isSatisfiedBy(Customer customer) {
        return customer.isActive() 
            && customer.hasNoOverdueInvoices()
            && customer.hasMinimumPurchaseHistory();
    }
}
```

**Usage:**
```java
// Sử dụng
Specification<Customer> vipSpec = 
    new CreditLimitSpecification(Money.of(10000, "USD"))
        .and(new RegionSpecification(Region.NORTH_AMERICA))
        .and(new EligibleForPromotionSpecification());

if (vipSpec.isSatisfiedBy(customer)) {
    // Apply VIP discount
}

// Hoặc filter một list
List<Customer> vipCustomers = customers.stream()
    .filter(vipSpec::isSatisfiedBy)
    .collect(Collectors.toList());
```

## 3. Domain Events

### Khái niệm

**Domain Event** là một sự kiện đã xảy ra trong domain, có ý nghĩa với domain experts. Events giúp các Aggregate giao tiếp mà không cần reference trực tiếp.

### Ví dụ

**Event class:**
```java
public abstract class DomainEvent {
    private final LocalDateTime occurredOn;
    private final UUID eventId;
    
    protected DomainEvent() {
        this.occurredOn = LocalDateTime.now();
        this.eventId = UUID.randomUUID();
    }
    
    public LocalDateTime getOccurredOn() { return occurredOn; }
    public UUID getEventId() { return eventId; }
}

public class OrderConfirmed extends DomainEvent {
    private final OrderId orderId;
    private final CustomerId customerId;
    private final Money totalAmount;
    
    public OrderConfirmed(OrderId orderId, CustomerId customerId, Money totalAmount) {
        super();
        this.orderId = orderId;
        this.customerId = customerId;
        this.totalAmount = totalAmount;
    }
    
    public OrderId getOrderId() { return orderId; }
    public CustomerId getCustomerId() { return customerId; }
    public Money getTotalAmount() { return totalAmount; }
}

public class PaymentReceived extends DomainEvent {
    private final PaymentId paymentId;
    private final OrderId orderId;
    private final Money amount;
    
    public PaymentReceived(PaymentId paymentId, OrderId orderId, Money amount) {
        super();
        this.paymentId = paymentId;
        this.orderId = orderId;
        this.amount = amount;
    }
    
    // Getters...
}
```

**Publish events from Entity:**
```java
public class Order {
    private OrderId id;
    private OrderStatus status;
    
    // Danh sách events chờ publish
    @Transient
    private List<DomainEvent> domainEvents = new ArrayList<>();
    
    public void confirm() {
        // Business logic
        if (orderLines.isEmpty()) {
            throw new EmptyOrderException();
        }
        
        this.status = OrderStatus.CONFIRMED;
        
        // Raise domain event
        addDomainEvent(new OrderConfirmed(this.id, this.customerId, this.total));
    }
    
    protected void addDomainEvent(DomainEvent event) {
        this.domainEvents.add(event);
    }
    
    public List<DomainEvent> getDomainEvents() {
        return Collections.unmodifiableList(domainEvents);
    }
    
    public void clearDomainEvents() {
        this.domainEvents.clear();
    }
}
```

**Event Handler:**
```java
@Component
public class OrderEventHandler {
    
    private final EmailService emailService;
    private final InventoryService inventoryService;
    
    @EventListener
    public void handleOrderConfirmed(OrderConfirmed event) {
        // Send confirmation email
        emailService.sendOrderConfirmation(
            event.getCustomerId(), 
            event.getOrderId()
        );
        
        // Reserve inventory
        inventoryService.reserveItems(event.getOrderId());
    }
    
    @EventListener
    public void handlePaymentReceived(PaymentReceived event) {
        // Mark order as paid
        // Ship order
    }
}
```

## 4. Factories

### Khái niệm

**Factory** đảm nhiệm việc tạo các Aggregate phức tạp, đảm bảo tất cả invariants được thỏa mãn ngay từ đầu.

### Ví dụ

**Simple Factory Method:**
```java
public class Order {
    // Private constructor
    private Order(OrderId id, CustomerId customerId) {
        this.id = id;
        this.customerId = customerId;
        this.status = OrderStatus.PENDING;
        this.orderLines = new ArrayList<>();
    }
    
    // Factory method
    public static Order create(OrderId id, CustomerId customerId) {
        // Có thể thêm validation hoặc business logic
        return new Order(id, customerId);
    }
}
```

**Complex Factory:**
```java
@Component
public class OrderFactory {
    
    private final CustomerRepository customerRepository;
    private final ProductRepository productRepository;
    
    public Order createOrder(CustomerId customerId, List<OrderItemRequest> items) {
        // Validate customer exists
        Customer customer = customerRepository.findById(customerId)
            .orElseThrow(() -> new CustomerNotFoundException(customerId));
        
        // Create order
        OrderId orderId = OrderId.generate();
        Order order = Order.create(orderId, customerId);
        
        // Add items
        for (OrderItemRequest itemRequest : items) {
            Product product = productRepository.findById(itemRequest.getProductId())
                .orElseThrow(() -> new ProductNotFoundException(itemRequest.getProductId()));
            
            // Validate inventory
            if (!product.isAvailable(itemRequest.getQuantity())) {
                throw new InsufficientInventoryException(product.getId());
            }
            
            order.addItem(product, itemRequest.getQuantity());
        }
        
        return order;
    }
}
```

## 5. Application Service

### Khái niệm

**Application Service** (khác với Domain Service) là lớp điều phối (orchestration). Nó:

- Nhận request từ presentation layer
- Load aggregate từ repository
- Gọi domain logic
- Save aggregate
- Publish events

!!! warning "Lưu ý"
    Application Service KHÔNG chứa business logic. Business logic ở Domain Model.

### Ví dụ

```java
@Service
@Transactional
public class OrderApplicationService {
    
    private final OrderRepository orderRepository;
    private final CustomerRepository customerRepository;
    private final OrderFactory orderFactory;
    private final EventPublisher eventPublisher;
    
    // Use case: Create Order
    public OrderId createOrder(CreateOrderCommand command) {
        // 1. Load dependencies
        Customer customer = customerRepository.findById(command.getCustomerId())
            .orElseThrow(() -> new CustomerNotFoundException());
        
        // 2. Create aggregate using factory
        Order order = orderFactory.createOrder(
            command.getCustomerId(), 
            command.getItems()
        );
        
        // 3. Save aggregate
        orderRepository.save(order);
        
        // 4. Publish events
        order.getDomainEvents().forEach(eventPublisher::publish);
        order.clearDomainEvents();
        
        return order.getId();
    }
    
    // Use case: Confirm Order
    public void confirmOrder(ConfirmOrderCommand command) {
        // 1. Load aggregate
        Order order = orderRepository.findById(command.getOrderId())
            .orElseThrow(() -> new OrderNotFoundException());
        
        // 2. Execute domain logic (ở Entity, không phải ở đây!)
        order.confirm();
        
        // 3. Save
        orderRepository.save(order);
        
        // 4. Publish events
        order.getDomainEvents().forEach(eventPublisher::publish);
        order.clearDomainEvents();
    }
}
```

## 6. Anti-Patterns cần tránh

### ❌ Anemic Domain Model

Đã nói ở trên - Entity chỉ có getter/setter, không có logic.

### ❌ Smart UI

UI chứa business logic thay vì ở domain layer.

```java
// ❌ BAD: Logic ở UI
public class OrderController {
    @PostMapping("/orders/confirm")
    public void confirmOrder(@RequestParam Long orderId) {
        Order order = orderRepository.findById(orderId);
        
        // Business logic ở controller!
        if (order.getOrderLines().isEmpty()) {
            throw new RuntimeException("Empty order");
        }
        order.setStatus("CONFIRMED");
        
        BigDecimal total = BigDecimal.ZERO;
        for (OrderLine line : order.getOrderLines()) {
            total = total.add(line.getPrice());
        }
        order.setTotal(total);
        
        orderRepository.save(order);
    }
}
```

### ❌ God Object

Một class quá lớn, chứa quá nhiều responsibility.

```java
// ❌ BAD: Order làm quá nhiều việc
public class Order {
    // Order data
    private OrderId id;
    private List<OrderLine> orderLines;
    
    // Payment logic
    public void processPayment(CreditCard card) { }
    
    // Shipping logic
    public void ship(Address address) { }
    
    // Notification logic
    public void sendConfirmationEmail() { }
    
    // Inventory logic
    public void reserveInventory() { }
    
    // ... 50 methods khác
}
```

Giải pháp: Tách thành nhiều Aggregate và Service.

### ❌ Transaction Script

Mỗi use case là một procedure, không có domain model.

```java
// ❌ BAD: Procedural style
public class OrderService {
    public void processOrder(Long orderId) {
        // 100 lines of procedural code
        // if ... else ... 
        // SQL queries everywhere
    }
}
```

## 7. Best Practices

### ✅ Small Aggregates

Aggregate càng nhỏ càng tốt. Tránh "aggregate of everything".

```java
// ✅ GOOD: Small aggregate
public class Order {
    private OrderId id;
    private CustomerId customerId;  // Reference by ID
    private List<OrderLine> orderLines;
    // ...
}

// ✅ Separate aggregate
public class Customer {
    private CustomerId id;
    private String name;
    // ...
}
```

### ✅ Ubiquitous Language trong code

```java
// ✅ GOOD: Dùng domain language
public class BankAccount {
    public void debit(Money amount) { }
    public void credit(Money amount) { }
}

// ❌ BAD: Dùng technical language
public class BankAccount {
    public void subtract(BigDecimal amt) { }
    public void add(BigDecimal amt) { }
}
```

### ✅ Encapsulation

```java
// ✅ GOOD: Encapsulated
public class Order {
    private List<OrderLine> orderLines = new ArrayList<>();
    
    public void addItem(Product product, int quantity) {
        OrderLine line = new OrderLine(product, quantity);
        orderLines.add(line);
        recalculateTotal();
    }
    
    // Return unmodifiable view
    public List<OrderLine> getOrderLines() {
        return Collections.unmodifiableList(orderLines);
    }
}

// ❌ BAD: Exposed collection
public class Order {
    private List<OrderLine> orderLines = new ArrayList<>();
    
    public List<OrderLine> getOrderLines() {
        return orderLines;  // Ai cũng có thể modify!
    }
}
```

## Tổng kết các Building Blocks

| Building Block | Mục đích | Ví dụ |
|----------------|----------|-------|
| **Entity** | Đối tượng có identity | Customer, Order, Product |
| **Value Object** | Đối tượng giá trị immutable | Money, Address, Email |
| **Aggregate** | Cụm objects với consistency boundary | Order + OrderLines |
| **Repository** | Persistence abstraction | OrderRepository |
| **Domain Service** | Logic không thuộc Entity | MoneyTransferService |
| **Domain Event** | Sự kiện đã xảy ra | OrderConfirmed |
| **Factory** | Tạo Aggregate phức tạp | OrderFactory |
| **Specification** | Business rules tái sử dụng | CreditLimitSpecification |

## Aggregate Design Guidelines (Vaughn Vernon)

Theo **Vaughn Vernon** trong "Implementing Domain-Driven Design", có 4 nguyên tắc thiết kế Aggregate:

### 1. Bảo vệ Business Invariants trong Aggregate Boundaries

Aggregate đảm bảo business rules luôn đúng. Mọi thay đổi phải qua Aggregate Root.

```java
// ✅ Aggregate Root kiểm soát tất cả thay đổi
public class Order {
    private List<OrderLine> orderLines;
    private Money orderLimit = Money.of(10000, "USD");
    
    public void addItem(Product product, int quantity) {
        OrderLine line = new OrderLine(product, quantity);
        
        // Validate invariant TRƯỚC KHI thêm
        Money newTotal = calculateTotalWith(line);
        if (newTotal.isGreaterThan(orderLimit)) {
            throw new OrderLimitExceededException();
        }
        
        orderLines.add(line);
        recalculateTotal();
    }
    
    // ❌ KHÔNG expose collection để modify trực tiếp
    // public List<OrderLine> getOrderLines() { return orderLines; }
}
```

### 2. Thiết kế Aggregate nhỏ (Small Aggregates)

Aggregate càng nhỏ càng tốt. Chỉ bao gồm những gì THỰC SỰ cần consistency ngay lập tức.

```java
// ❌ BAD: Aggregate quá lớn
public class Customer {
    private CustomerId id;
    private List<Order> orders;           // Không cần
    private List<Payment> payments;       // Không cần
    private List<Address> addresses;      // Không cần
    private ShoppingCart cart;            // Không cần
    // ... quá nhiều entities
}

// ✅ GOOD: Aggregate nhỏ gọn
public class Customer {
    private CustomerId id;
    private String name;
    private Email email;
    private CustomerStatus status;
    
    // Orders, Payments là Aggregate riêng
    // Reference bằng ID
}
```

### 3. Tham chiếu Aggregate khác bằng Identity

Aggregate chỉ nên giữ **ID** của Aggregate khác, không phải object.

```java
// ❌ BAD: Hold reference to other Aggregate
public class Order {
    private OrderId id;
    private Customer customer;  // ❌ Entire Customer object
    private List<OrderLine> orderLines;
}

// ✅ GOOD: Reference by Identity
public class Order {
    private OrderId id;
    private CustomerId customerId;  // ✅ Chỉ ID
    private List<OrderLine> orderLines;
    
    // Nếu cần Customer data, fetch qua Repository
    public void printInvoice(CustomerRepository customerRepository) {
        Customer customer = customerRepository.findById(customerId);
        // Use customer data...
    }
}
```

### 4. Sử dụng Eventual Consistency giữa các Aggregates

Nếu cần cập nhật nhiều Aggregate, dùng **Domain Events** thay vì transaction.

```java
// Aggregate 1: Order
public class Order {
    public void confirm() {
        this.status = OrderStatus.CONFIRMED;
        
        // Raise event thay vì gọi trực tiếp Inventory Aggregate
        domainEvents.add(new OrderConfirmed(this.id, this.orderLines));
    }
}

// Event Handler cập nhật Aggregate 2
@EventListener
public class InventoryEventHandler {
    private final InventoryRepository inventoryRepository;
    
    public void on(OrderConfirmed event) {
        // Eventual consistency - không cần immediate
        for (OrderLine line : event.getOrderLines()) {
            Inventory inventory = inventoryRepository.findByProductId(
                line.getProductId()
            );
            inventory.reserve(line.getQuantity());
            inventoryRepository.save(inventory);
        }
    }
}
```

### 5. Ví dụ: Aggregate Design Evolution

#### Version 1 - Large Aggregate (Bad)
```java
// ❌ Quá lớn, khó scale
public class Order {
    private OrderId id;
    private Customer customer;              // Aggregate khác
    private List<OrderLine> orderLines;
    private ShippingAddress shippingAddress;
    private BillingAddress billingAddress;
    private Payment payment;                 // Aggregate khác
    private Shipment shipment;               // Aggregate khác
    
    // Quá nhiều dependencies, performance issues
}
```

#### Version 2 - Small Aggregates (Good)
```java
// ✅ Order Aggregate - nhỏ gọn
public class Order {
    private OrderId id;
    private CustomerId customerId;          // Reference by ID
    private List<OrderLine> orderLines;     // Part of Order
    private ShippingAddressId shippingId;   // Reference by ID
    private OrderStatus status;
    
    public void confirm() {
        validateCanConfirm();
        this.status = OrderStatus.CONFIRMED;
        domainEvents.add(new OrderConfirmed(this.id));
    }
}

// ✅ Payment Aggregate - riêng biệt
public class Payment {
    private PaymentId id;
    private OrderId orderId;
    private Money amount;
    private PaymentStatus status;
    
    public void process() {
        // Payment logic
    }
}

// ✅ Shipment Aggregate - riêng biệt
public class Shipment {
    private ShipmentId id;
    private OrderId orderId;
    private ShippingAddress address;
    private ShipmentStatus status;
    
    public void ship() {
        // Shipping logic
    }
}
```

#### Orchestration với Domain Events
```java
// Application Service điều phối
@Service
public class OrderApplicationService {
    
    @Transactional
    public void placeOrder(PlaceOrderCommand command) {
        // 1. Create Order Aggregate
        Order order = new Order(command);
        orderRepository.save(order);
        
        // 2. Raise event
        eventPublisher.publish(new OrderPlaced(order.getId()));
    }
}

// Event Handlers xử lý các Aggregates khác
@EventListener
public void onOrderPlaced(OrderPlaced event) {
    // Create Payment (separate transaction)
    Payment payment = new Payment(event.getOrderId());
    paymentRepository.save(payment);
}

@EventListener
public void onPaymentCompleted(PaymentCompleted event) {
    // Create Shipment (separate transaction)
    Shipment shipment = new Shipment(event.getOrderId());
    shipmentRepository.save(shipment);
}
```

### Lợi ích của Small Aggregates

✅ **Performance**: Fewer conflicts, less locking  
✅ **Scalability**: Easier to scale horizontally  
✅ **Maintainability**: Simpler to understand and test  
✅ **Flexibility**: Easier to evolve domain model  

---

## Nguồn tham khảo

!!! info "Tài liệu tham khảo"
    - **Eric Evans** - [Domain-Driven Design](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) - Part II: The Building Blocks
    - **Vaughn Vernon** - [Implementing Domain-Driven Design](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577) - Chapter 10: Aggregates
    - **Vaughn Vernon** - [Effective Aggregate Design](https://www.dddcommunity.org/library/vernon_2011/) - 3-part series
    - **Martin Fowler** - [Anemic Domain Model](https://martinfowler.com/bliki/AnemicDomainModel.html)
    - **Martin Fowler** - [Domain Event](https://martinfowler.com/eaaDev/DomainEvent.html)
    - **Eric Evans & Martin Fowler** - [Specifications](https://www.martinfowler.com/apsupp/spec.pdf)

---

[← Quay lại Tactical Design](tactical-design.md) | [Advanced Patterns →](advanced-patterns.md)
