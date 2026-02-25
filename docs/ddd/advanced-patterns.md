# Advanced DDD Patterns

Các pattern nâng cao trong Domain-Driven Design giúp xử lý các tình huống phức tạp trong enterprise applications.

## 1. Event Sourcing

### Khái niệm

**Event Sourcing** là pattern lưu trữ **tất cả các thay đổi** của state dưới dạng một chuỗi các **domain events**, thay vì chỉ lưu state hiện tại.

!!! quote "Vaughn Vernon"
    "Event Sourcing is a way to persist the state of an application as a sequence of events. Rather than storing just the current state, we store all events that lead to the current state."

### Lợi ích

✅ **Audit trail hoàn chỉnh**: Có thể xem lại lịch sử đầy đủ  
✅ **Temporal queries**: Có thể xem state ở bất kỳ thời điểm nào  
✅ **Event replay**: Rebuild state từ events  
✅ **Debugging dễ dàng**: Biết chính xác điều gì đã xảy ra  

### Ví dụ: Bank Account với Event Sourcing

```java
// Domain Events
public interface DomainEvent {
    LocalDateTime occurredOn();
    Long aggregateId();
}

public class AccountOpened implements DomainEvent {
    private final Long accountId;
    private final String accountNumber;
    private final LocalDateTime occurredOn;
    
    public AccountOpened(Long accountId, String accountNumber) {
        this.accountId = accountId;
        this.accountNumber = accountNumber;
        this.occurredOn = LocalDateTime.now();
    }
    
    // Getters...
}

public class MoneyDeposited implements DomainEvent {
    private final Long accountId;
    private final BigDecimal amount;
    private final LocalDateTime occurredOn;
    
    // Constructor, getters...
}

public class MoneyWithdrawn implements DomainEvent {
    private final Long accountId;
    private final BigDecimal amount;
    private final LocalDateTime occurredOn;
    
    // Constructor, getters...
}
```

```java
// Aggregate với Event Sourcing
public class BankAccount {
    private Long id;
    private String accountNumber;
    private BigDecimal balance;
    private List<DomainEvent> changes = new ArrayList<>();
    
    // Constructor cho new aggregate
    public BankAccount(String accountNumber) {
        apply(new AccountOpened(null, accountNumber));
    }
    
    // Constructor để rebuild từ events
    public static BankAccount fromEvents(List<DomainEvent> events) {
        BankAccount account = new BankAccount();
        for (DomainEvent event : events) {
            account.applyChange(event, false);
        }
        return account;
    }
    
    private BankAccount() {
        this.balance = BigDecimal.ZERO;
    }
    
    // Business methods
    public void deposit(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        apply(new MoneyDeposited(id, amount));
    }
    
    public void withdraw(BigDecimal amount) {
        if (amount.compareTo(balance) > 0) {
            throw new InsufficientFundsException();
        }
        apply(new MoneyWithdrawn(id, amount));
    }
    
    // Apply event (cho new events)
    private void apply(DomainEvent event) {
        applyChange(event, true);
    }
    
    // Apply change và track nếu là event mới
    private void applyChange(DomainEvent event, boolean isNew) {
        if (event instanceof AccountOpened) {
            when((AccountOpened) event);
        } else if (event instanceof MoneyDeposited) {
            when((MoneyDeposited) event);
        } else if (event instanceof MoneyWithdrawn) {
            when((MoneyWithdrawn) event);
        }
        
        if (isNew) {
            changes.add(event);
        }
    }
    
    // Event handlers - cập nhật state
    private void when(AccountOpened event) {
        this.id = event.aggregateId();
        this.accountNumber = event.getAccountNumber();
    }
    
    private void when(MoneyDeposited event) {
        this.balance = this.balance.add(event.getAmount());
    }
    
    private void when(MoneyWithdrawn event) {
        this.balance = this.balance.subtract(event.getAmount());
    }
    
    // Get uncommitted changes
    public List<DomainEvent> getUncommittedChanges() {
        return new ArrayList<>(changes);
    }
    
    // Mark changes as committed
    public void markChangesAsCommitted() {
        changes.clear();
    }
}
```

```java
// Event Store Interface
public interface EventStore {
    void saveEvents(Long aggregateId, List<DomainEvent> events, int expectedVersion);
    List<DomainEvent> getEventsForAggregate(Long aggregateId);
}
```

### Khi nào nên dùng Event Sourcing?

✅ **NÊN dùng:**
- Cần audit trail đầy đủ (banking, healthcare, legal)
- Cần phân tích lịch sử dữ liệu
- Domain có nhiều state transitions phức tạp

❌ **KHÔNG NÊN:**
- CRUD đơn giản
- Queries phức tạp (dùng CQRS kết hợp)
- Team chưa có kinh nghiệm

---

## 2. CQRS (Command Query Responsibility Segregation)

### Khái niệm

**CQRS** là pattern tách biệt **Command** (thay đổi state) và **Query** (đọc data) thành hai model riêng biệt.

!!! tip "Greg Young"
    "CQRS is simply the creation of two objects where there was previously only one."

### Kiến trúc CQRS

```
┌─────────────┐          ┌──────────────────┐
│   Client    │          │  Write Model     │
│             │────────▶│  (Command Side)  │
│             │ Command  │  - Domain Logic  │
└─────────────┘          │  - Aggregates    │
      │                  └────────┬─────────┘
      │                           │
      │                      Save Events
      │                           │
      │                           ▼
      │                  ┌────────────────┐
      │                  │  Event Store   │
      │                  └────────┬───────┘
      │                           │
      │                     Publish Events
      │                           │
      │                           ▼
      │                  ┌────────────────────┐
      │                  │  Event Processor   │
      │                  │  (Projection)      │
      │                  └────────┬───────────┘
      │                           │
      │                      Update
      │                           │
      │                           ▼
      │  Query            ┌─────────────────────┐
      └─────────────────▶│    Read Model       │
                          │   (Query Side)      │
                          │   - Optimized Views │
                          │   - Denormalized    │
                          └─────────────────────┘
```

### Ví dụ: Banking System với CQRS

#### Command Side (Write Model)

```java
// Commands
public class CreateAccountCommand {
    private final String accountNumber;
    private final String customerId;
    private final String currency;
    
    // Constructor, getters...
}

public class DepositMoneyCommand {
    private final Long accountId;
    private final BigDecimal amount;
    
    // Constructor, getters...
}
```

```java
// Command Handlers
@Service
public class AccountCommandHandler {
    private final AccountRepository accountRepository;
    private final EventPublisher eventPublisher;
    
    @Transactional
    public void handle(CreateAccountCommand command) {
        BankAccount account = new BankAccount(command.getAccountNumber());
        accountRepository.save(account);
        
        // Publish events
        account.getUncommittedChanges().forEach(eventPublisher::publish);
        account.markChangesAsCommitted();
    }
    
    @Transactional
    public void handle(DepositMoneyCommand command) {
        BankAccount account = accountRepository.findById(command.getAccountId())
            .orElseThrow(() -> new AccountNotFoundException());
        
        account.deposit(command.getAmount());
        accountRepository.save(account);
        
        // Publish events
        account.getUncommittedChanges().forEach(eventPublisher::publish);
        account.markChangesAsCommitted();
    }
}
```

#### Query Side (Read Model)

```java
// Read Model - Optimized cho queries
@Entity
@Table(name = "account_summary")
public class AccountSummaryView {
    @Id
    private Long accountId;
    private String accountNumber;
    private String customerName;
    private BigDecimal currentBalance;
    private String currency;
    private LocalDateTime lastTransactionDate;
    private Integer transactionCount;
    
    // Getters, setters...
}
```

```java
// Query Service
@Service
public class AccountQueryService {
    private final AccountSummaryRepository summaryRepository;
    
    public AccountSummaryView findAccountSummary(Long accountId) {
        return summaryRepository.findById(accountId)
            .orElseThrow(() -> new AccountNotFoundException());
    }
    
    public List<AccountSummaryView> findAccountsByCustomer(String customerId) {
        return summaryRepository.findByCustomerId(customerId);
    }
    
    public Page<AccountSummaryView> findAccountsWithHighBalance(
            BigDecimal minBalance, Pageable pageable) {
        return summaryRepository.findByCurrentBalanceGreaterThan(minBalance, pageable);
    }
}
```

```java
// Event Projector - Cập nhật Read Model
@Service
public class AccountSummaryProjector {
    private final AccountSummaryRepository summaryRepository;
    
    @EventListener
    public void on(AccountOpened event) {
        AccountSummaryView view = new AccountSummaryView();
        view.setAccountId(event.getAccountId());
        view.setAccountNumber(event.getAccountNumber());
        view.setCurrentBalance(BigDecimal.ZERO);
        view.setTransactionCount(0);
        summaryRepository.save(view);
    }
    
    @EventListener
    public void on(MoneyDeposited event) {
        AccountSummaryView view = summaryRepository.findById(event.getAccountId())
            .orElseThrow();
        view.setCurrentBalance(view.getCurrentBalance().add(event.getAmount()));
        view.setLastTransactionDate(event.occurredOn());
        view.setTransactionCount(view.getTransactionCount() + 1);
        summaryRepository.save(view);
    }
    
    @EventListener
    public void on(MoneyWithdrawn event) {
        AccountSummaryView view = summaryRepository.findById(event.getAccountId())
            .orElseThrow();
        view.setCurrentBalance(view.getCurrentBalance().subtract(event.getAmount()));
        view.setLastTransactionDate(event.occurredOn());
        view.setTransactionCount(view.getTransactionCount() + 1);
        summaryRepository.save(view);
    }
}
```

### Lợi ích của CQRS

✅ **Performance**: Tối ưu riêng cho read và write  
✅ **Scalability**: Scale read và write độc lập  
✅ **Flexibility**: Nhiều read models khác nhau cho các use cases  
✅ **Separation of Concerns**: Logic đọc và ghi tách biệt  

### CQRS + Event Sourcing

CQRS thường được kết hợp với Event Sourcing:

```
Commands → Aggregates → Events → Event Store
                          │
                          ├─→ Projection 1 (SQL)
                          ├─→ Projection 2 (MongoDB)
                          └─→ Projection 3 (Elasticsearch)
```

---

## 3. Saga Pattern

### Khái niệm

**Saga** là pattern để quản lý **distributed transactions** trong microservices, đảm bảo data consistency qua nhiều services mà không dùng 2PC (Two-Phase Commit).

### Choreography-based Saga

Mỗi service thực hiện transaction local và publish event. Services khác lắng nghe events và react.

```java
// Order Service
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final EventPublisher eventPublisher;
    
    @Transactional
    public void createOrder(CreateOrderCommand command) {
        Order order = new Order(command);
        order.setStatus(OrderStatus.PENDING);
        orderRepository.save(order);
        
        // Publish event
        eventPublisher.publish(new OrderCreated(
            order.getId(),
            order.getCustomerId(),
            order.getTotalAmount()
        ));
    }
    
    // Compensating transaction
    @EventListener
    public void on(PaymentFailed event) {
        Order order = orderRepository.findById(event.getOrderId()).orElseThrow();
        order.cancel();
        orderRepository.save(order);
        
        eventPublisher.publish(new OrderCancelled(order.getId()));
    }
}
```

```java
// Payment Service
@Service
public class PaymentService {
    private final PaymentRepository paymentRepository;
    private final EventPublisher eventPublisher;
    
    @EventListener
    @Transactional
    public void on(OrderCreated event) {
        try {
            Payment payment = new Payment(event.getOrderId(), event.getAmount());
            payment.process();
            paymentRepository.save(payment);
            
            eventPublisher.publish(new PaymentCompleted(
                event.getOrderId(),
                payment.getId()
            ));
        } catch (InsufficientFundsException e) {
            eventPublisher.publish(new PaymentFailed(
                event.getOrderId(),
                e.getMessage()
            ));
        }
    }
}
```

```java
// Inventory Service
@Service
public class InventoryService {
    
    @EventListener
    @Transactional
    public void on(PaymentCompleted event) {
        // Reserve inventory
        inventoryRepository.reserve(event.getOrderId());
        eventPublisher.publish(new InventoryReserved(event.getOrderId()));
    }
    
    @EventListener
    @Transactional
    public void on(OrderCancelled event) {
        // Release inventory (compensating action)
        inventoryRepository.release(event.getOrderId());
    }
}
```

### Orchestration-based Saga

Một Saga Orchestrator điều phối các bước.

```java
// Saga Orchestrator
@Service
public class OrderSagaOrchestrator {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final ShippingService shippingService;
    
    public void executeOrderSaga(Order order) {
        try {
            // Step 1: Reserve inventory
            inventoryService.reserve(order.getItems());
            
            // Step 2: Process payment
            paymentService.charge(order.getCustomerId(), order.getTotalAmount());
            
            // Step 3: Arrange shipping
            shippingService.ship(order.getId(), order.getShippingAddress());
            
            order.complete();
        } catch (Exception e) {
            // Compensate in reverse order
            compensate(order);
        }
    }
    
    private void compensate(Order order) {
        try {
            shippingService.cancelShipping(order.getId());
        } catch (Exception e) { /* log */ }
        
        try {
            paymentService.refund(order.getCustomerId(), order.getTotalAmount());
        } catch (Exception e) { /* log */ }
        
        try {
            inventoryService.release(order.getItems());
        } catch (Exception e) { /* log */ }
        
        order.cancel();
    }
}
```

---

## 4. Hexagonal Architecture (Ports & Adapters)

### Khái niệm

**Hexagonal Architecture** (còn gọi là Ports and Adapters) là pattern tách biệt business logic khỏi infrastructure concerns.

### Cấu trúc

```
┌────────────────────────────────────────────────────┐
│               Adapters (Input)                     │
│  REST │ GraphQL │ CLI │ Message Queue              │
└─────────────────┬──────────────────────────────────┘
                  │
          ┌───────▼───────┐
          │  Input Ports  │ (Interfaces)
          │  (Use Cases)  │
          └───────┬───────┘
                  │
        ┌─────────▼─────────┐
        │   Domain Layer    │
        │  - Entities       │
        │  - Value Objects  │
        │  - Domain Logic   │
        └─────────┬─────────┘
                  │
          ┌───────▼────────┐
          │  Output Ports  │ (Interfaces)
          │ (Repositories) │
          └───────┬────────┘
                  │
┌─────────────────▼──────────────────────────────────┐
│               Adapters (Output)                    │
│  PostgreSQL │ MongoDB │ Redis │ External APIs      │
└────────────────────────────────────────────────────┘
```

### Ví dụ

```java
// Domain Layer
package com.bank.domain;

public class BankAccount {
    private AccountId id;
    private Money balance;
    
    public void deposit(Money amount) {
        // Business logic
        this.balance = this.balance.add(amount);
    }
}
```

```java
// Output Port (Interface trong Domain)
package com.bank.domain.port;

public interface AccountRepository {
    BankAccount findById(AccountId id);
    void save(BankAccount account);
}
```

```java
// Input Port (Use Case Interface)
package com.bank.application.port;

public interface DepositMoneyUseCase {
    void deposit(AccountId accountId, Money amount);
}
```

```java
// Application Service implements Input Port
package com.bank.application;

@Service
public class AccountApplicationService implements DepositMoneyUseCase {
    private final AccountRepository accountRepository;
    
    @Override
    public void deposit(AccountId accountId, Money amount) {
        BankAccount account = accountRepository.findById(accountId);
        account.deposit(amount);
        accountRepository.save(account);
    }
}
```

```java
// Output Adapter (Implementation)
package com.bank.infrastructure.persistence;

@Repository
public class JpaAccountRepository implements AccountRepository {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public BankAccount findById(AccountId id) {
        AccountEntity entity = entityManager.find(AccountEntity.class, id.getValue());
        return AccountMapper.toDomain(entity);
    }
    
    @Override
    public void save(BankAccount account) {
        AccountEntity entity = AccountMapper.toEntity(account);
        entityManager.merge(entity);
    }
}
```

```java
// Input Adapter (REST Controller)
package com.bank.interfaces.rest;

@RestController
@RequestMapping("/api/accounts")
public class AccountController {
    private final DepositMoneyUseCase depositMoneyUseCase;
    
    @PostMapping("/{id}/deposit")
    public ResponseEntity<Void> deposit(
            @PathVariable String id,
            @RequestBody DepositRequest request) {
        
        depositMoneyUseCase.deposit(
            new AccountId(id),
            Money.of(request.getAmount(), request.getCurrency())
        );
        
        return ResponseEntity.ok().build();
    }
}
```

### Lợi ích

✅ **Testability**: Dễ test domain logic  
✅ **Flexibility**: Dễ thay đổi infrastructure  
✅ **Independence**: Domain không phụ thuộc frameworks  

---

## Nguồn tham khảo

- **Vaughn Vernon** - [Implementing Domain-Driven Design](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577) - Chapters 8-10
- **Martin Fowler** - [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- **Greg Young** - [CQRS Documents](https://cqrs.files.wordpress.com/2010/11/cqrs_documents.pdf)
- **Chris Richardson** - [Microservices Patterns: Saga](https://microservices.io/patterns/data/saga.html)
- **Alistair Cockburn** - [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)

---

**Tiếp theo:** [Example Project - Banking System →](example-project.md)

