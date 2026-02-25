# Tactical Design - Từ mô hình 3 lớp đến DDD

!!! tip "Cho sinh viên đang học Spring Boot"
    Bạn đã biết **Controller-Service-Repository**. Phần này sẽ giúp bạn nâng cấp lên **DDD** từng bước, với ví dụ so sánh cụ thể.

## Vấn đề của mô hình 3 lớp truyền thống

Khi làm project thực tế, bạn sẽ gặp tình huống này:

### ❌ Cách làm thông thường (Anemic Model)

```java
// Entity chỉ là data holder
@Entity
public class BankAccount {
    @Id private Long id;
    private BigDecimal balance;  // getter/setter public
    private String status;
    
    // Chỉ có getter/setter, KHÔNG CÓ business logic
    public BigDecimal getBalance() { return balance; }
    public void setBalance(BigDecimal balance) { 
        this.balance = balance;  // ❌ Ai cũng set được!
    }
}

// Service phình to với đống logic
@Service
public class BankAccountService {
    @Autowired BankAccountRepository repo;
    
    public void withdraw(Long accountId, BigDecimal amount) {
        BankAccount account = repo.findById(accountId).orElseThrow();
        
        // ❌ Tất cả validation ở đây - khó đọc, khó test
        if (account.getBalance().compareTo(amount) < 0) {
            throw new RuntimeException("Không đủ tiền");
        }
        if ("LOCKED".equals(account.getStatus())) {
            throw new RuntimeException("Tài khoản bị khóa");
        }
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new RuntimeException("Số tiền không hợp lệ");
        }
        
        // ❌ Có thể quên check một số rule
        account.setBalance(account.getBalance().subtract(amount));
        repo.save(account);
    }
}
```

**Vấn đề:**
- Service quá dài, khó đọc
- Entity không tự bảo vệ mình (ai cũng `setBalance()` được)
- Dễ quên check rule ở nơi khác
- Khó test từng rule riêng

### ✅ Cách làm DDD (Rich Domain Model)

```java
// Entity có business logic, tự bảo vệ mình
@Entity
public class BankAccount {
    @Id private AccountId id;
    private Money balance;  // ❌ KHÔNG CÓ setter public
    private AccountStatus status;
    
    // ✅ Constructor có validation
    public BankAccount(AccountId id, Money initialBalance) {
        this.id = Objects.requireNonNull(id);
        this.balance = initialBalance;
        this.status = AccountStatus.ACTIVE;
    }
    
    // ✅ Business logic ở trong Entity
    public void withdraw(Money amount) {
        // Tự check tất cả rules
        validateWithdrawal(amount);
        this.balance = balance.subtract(amount);
    }
    
    private void validateWithdrawal(Money amount) {
        if (status.isLocked()) {
            throw new AccountLockedException("Tài khoản bị khóa");
        }
        if (balance.isLessThan(amount)) {
            throw new InsufficientBalanceException("Không đủ tiền");
        }
        if (amount.isNegativeOrZero()) {
            throw new InvalidAmountException("Số tiền không hợp lệ");
        }
    }
    
    // ❌ KHÔNG CÓ setBalance() public - không ai bypass được rules
}

// Service mỏng, chỉ điều phối
@Service
public class BankAccountService {
    @Autowired BankAccountRepository repo;
    
    public void withdraw(AccountId accountId, Money amount) {
        BankAccount account = repo.findById(accountId);
        account.withdraw(amount);  // ✅ Rules tự động check
        repo.save(account);
    }
}
```

**Lợi ích:**
- ✅ Service mỏng, dễ đọc
- ✅ Entity tự bảo vệ mình
- ✅ Không thể bypass rules
- ✅ Dễ test từng rule riêng
- ✅ Code đọc như tài liệu nghiệp vụ

## Entity vs Value Object - Phân biệt thế nào?

Đây là 2 khái niệm CỐT LÕI bạn phải hiểu trong DDD.

### 1. Entity - Đối tượng có định danh

**Entity** = Đối tượng có **ID duy nhất**, tồn tại qua thời gian.

#### Ví dụ thực tế: Student

```java
// ✅ Student LÀ Entity vì có ID
public class Student {
    private StudentId id;  // ID duy nhất
    private String name;
    private Email email;
    
    // Constructor
    public Student(StudentId id, String name, Email email) {
        this.id = Objects.requireNonNull(id);
        this.name = name;
        this.email = email;
    }
    
    // Business logic
    public void changeEmail(Email newEmail) {
        // Có thể thêm validation ở đây
        this.email = newEmail;
    }
    
    // So sánh bằng ID
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Student)) return false;
        Student other = (Student) o;
        return this.id.equals(other.id);
    }
}
```

**Đặc điểm:**
- Có ID duy nhất
- Có thể thay đổi thuộc tính (mutable)
- 2 student cùng tên nhưng khác ID = 2 người khác nhau
- So sánh bằng ID

### 2. Value Object - Đối tượng giá trị

**Value Object** = Đối tượng **KHÔNG có ID**, định nghĩa bởi thuộc tính.

#### Ví dụ 1: Money (Tiền)

```java
// ✅ Money LÀ Value Object
public final class Money {  // final = không thể kế thừa
    private final BigDecimal amount;  // final = không thể thay đổi
    private final String currency;
    
    public Money(BigDecimal amount, String currency) {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Tiền không được âm");
        }
        this.amount = amount;
        this.currency = currency;
    }
    
    // ✅ Không có setter - IMMUTABLE
    public Money add(Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("Khác loại tiền");
        }
        return new Money(amount.add(other.amount), currency);  // Tạo object mới
    }
    
    // So sánh bằng TẤT CẢ thuộc tính
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Money)) return false;
        Money other = (Money) o;
        return amount.equals(other.amount) && 
               currency.equals(other.currency);
    }
}
```

**Cách dùng:**
```java
Money m1 = new Money(BigDecimal.valueOf(100), "VND");
Money m2 = new Money(BigDecimal.valueOf(50), "VND");

Money total = m1.add(m2);  // Tạo object mới, không thay đổi m1, m2
// m1 vẫn là 100 VND
// total là 150 VND
```

#### Ví dụ 2: Address (Địa chỉ)

```java
// ✅ Address LÀ Value Object
public final class Address {
    private final String street;
    private final String city;
    private final String zipCode;
    
    public Address(String street, String city, String zipCode) {
        this.street = street;
        this.city = city;
        this.zipCode = zipCode;
    }
    
    // ❌ Không có setter
    
    // Muốn thay đổi = tạo object mới
    public Address changeCity(String newCity) {
        return new Address(street, newCity, zipCode);
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Address)) return false;
        Address other = (Address) o;
        return street.equals(other.street) &&
               city.equals(other.city) &&
               zipCode.equals(other.zipCode);
    }
}
```

### So sánh Entity vs Value Object

| Khía cạnh | Entity | Value Object |
|-----------|--------|--------------|
| **ID** | Có ID duy nhất | Không có ID |
| **Immutable** | Không (có thể thay đổi) | Có (không thay đổi được) |
| **So sánh** | Bằng ID | Bằng tất cả thuộc tính |
| **Ví dụ** | Student, Order, User | Money, Address, Email |
| **Setter** | Có thể có | ❌ Không có |
| **Chia sẻ** | Không (mỗi cái độc lập) | Được (vì immutable) |

### Bài tập: Phân loại các đối tượng sau

??? question "Bài tập 1"
    **Câu hỏi:** `Order` (Đơn hàng) là Entity hay Value Object?
    
    **Đáp án:** ✅ **Entity** - vì có OrderId, theo dõi trạng thái qua thời gian

??? question "Bài tập 2"
    **Câu hỏi:** `PhoneNumber` (Số điện thoại) là Entity hay Value Object?
    
    **Đáp án:** ✅ **Value Object** - không có ID, chỉ là giá trị, 2 số giống nhau = cùng một số

??? question "Bài tập 3"
    **Câu hỏi:** `Product` (Sản phẩm) là Entity hay Value Object?
    
    **Đáp án:** ✅ **Entity** - vì có ProductId, cùng tên nhưng khác ID = sản phẩm khác nhau

## 3. Aggregate - Nhóm các Entity lại

!!! tip "Khi nào cần Aggregate?"
    Khi bạn có **nhiều Entity liên quan**, cần đảm bảo **business rules** giữa chúng.

### Vấn đề: Order có nhiều OrderItem

```java
// ❌ Cách làm SAI - ai cũng add được OrderItem
@Entity
public class Order {
    private List<OrderItem> items = new ArrayList<>();
    
    public List<OrderItem> getItems() {
        return items;  // ❌ Trả về list gốc
    }
}

// Ở Service
order.getItems().add(new OrderItem(...));  // ❌ Bypass validation!
```

**Vấn đề:** Không kiểm soát được việc thêm OrderItem, có thể vi phạm rules (ví dụ: Order đã confirm thì không được thêm item).

### ✅ Giải pháp: Order là Aggregate Root

```java
// ✅ Order là Aggregate Root, điều khiển OrderItem
@Entity
public class Order {
    @Id private OrderId id;
    private OrderStatus status;
    
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
    
    // ❌ KHÔNG expose list ra ngoài
    // public List<OrderItem> getItems() { ... }
    
    // ✅ Chỉ cho phép thêm qua method này
    public void addItem(Product product, int quantity) {
        if (status == OrderStatus.CONFIRMED) {
            throw new OrderAlreadyConfirmedException();
        }
        if (quantity <= 0) {
            throw new InvalidQuantityException();
        }
        
        OrderItem item = new OrderItem(product, quantity);
        items.add(item);
    }
    
    public void confirm() {
        if (items.isEmpty()) {
            throw new EmptyOrderException();
        }
        this.status = OrderStatus.CONFIRMED;
    }
    
    // ✅ Chỉ trả về read-only list
    public List<OrderItem> getItems() {
        return Collections.unmodifiableList(items);
    }
}

@Entity
class OrderItem {  // package-private, chỉ Order truy cập được
    @Id private Long id;
    private Product product;
    private int quantity;
    
    // Constructor package-private
    OrderItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }
}
```

**Nguyên tắc Aggregate:**
- **Aggregate Root** (Order) là điểm vào duy nhất
- Các Entity con (OrderItem) chỉ truy cập qua Root
- Business rules được bảo vệ trong Aggregate

## 4. Repository - Bạn đã biết rồi!

!!! success "Tin tốt"
    Bạn đã dùng Repository từ Spring Data JPA. DDD chỉ thêm một số best practices.

### Bạn đã biết (Spring Data JPA)

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByName(String name);
    Optional<User> findByEmail(String email);
}
```

### DDD Best Practices

```java
// ✅ Repository chỉ làm việc với Aggregate Root
public interface OrderRepository {
    Order findById(OrderId id);
    void save(Order order);  // Lưu cả Order và OrderItem
    List<Order> findByCustomerId(CustomerId customerId);
}

// ❌ KHÔNG có OrderItemRepository riêng
// Vì OrderItem chỉ truy cập qua Order
```

**Nguyên tắc:**
- 1 Aggregate Root = 1 Repository
- Không tạo Repository cho Entity con trong Aggregate

## 5. Domain Service - Khi logic không thuộc về một Entity

### Khi nào cần Domain Service?

✅ **Dùng Domain Service khi:**
- Logic liên quan đến **nhiều Entity/Aggregate**
- Logic không thuộc về một Entity cụ thể nào

### Ví dụ: Transfer Money giữa 2 Account

```java
// ❌ Không nên để trong Account vì liên quan đến 2 accounts
public class Account {
    public void transferTo(Account other, Money amount) {
        this.withdraw(amount);
        other.deposit(amount);
        // ❌ Phải pass Account khác vào - không tự nhiên
    }
}

// ✅ Dùng Domain Service
@Service
public class TransferService {  // Domain Service
    
    public void transfer(Account from, Account to, Money amount) {
        // Validate
        if (from.equals(to)) {
            throw new SameAccountException();
        }
        
        // Thực hiện transfer
        from.withdraw(amount);  // Business logic trong Account
        to.deposit(amount);
        
        // Save qua Repository
        accountRepository.save(from);
        accountRepository.save(to);
    }
}
```

### So sánh Application Service vs Domain Service

| Khía cạnh | Application Service | Domain Service |
|-----------|---------------------|----------------|
| **Nằm ở layer** | Application Layer | Domain Layer |
| **Nhiệm vụ** | Điều phối, giao tiếp | Business logic |
| **Ví dụ** | `OrderAppService` | `TransferService` |
| **Phụ thuộc** | Repo, Domain Service | Chỉ Domain objects |

```java
// Application Service - Điều phối
@Service
public class OrderApplicationService {
    @Autowired OrderRepository orderRepo;
    @Autowired ProductRepository productRepo;
    @Autowired OrderConfirmationService domainService;  // Domain Service
    
    @Transactional
    public void placeOrder(PlaceOrderRequest request) {
        // 1. Lấy data
        Customer customer = customerRepo.findById(request.getCustomerId());
        Product product = productRepo.findById(request.getProductId());
        
        // 2. Tạo Order
        Order order = new Order(customer, product, request.getQuantity());
        
        // 3. Gọi Domain Service
        domainService.confirm(order);
        
        // 4. Save
        orderRepo.save(order);
        
        // 5. Notification (infrastructure)
        emailService.sendConfirmation(customer.getEmail());
    }
}

// Domain Service - Business logic
@DomainService  // Custom annotation
public class OrderConfirmationService {
    
    public void confirm(Order order) {
        // Business logic: Check inventory, pricing, discount...
        if (!hasInventory(order)) {
            throw new OutOfStockException();
        }
        
        order.confirm();  // Entity method
    }
    
    private boolean hasInventory(Order order) {
        // Domain logic
        return true;
    }
}
```

## Tóm tắt: Các khái niệm đã học

| Khái niệm | Định nghĩa ngắn gọn | Ví dụ |
|-----------|---------------------|-------|
| **Entity** | Đối tượng có ID duy nhất | Student, Order, Account |
| **Value Object** | Đối tượng không ID, immutable | Money, Address, Email |
| **Aggregate** | Nhóm Entity có Aggregate Root | Order (root) + OrderItem |
| **Repository** | Lấy/lưu Aggregate Root | OrderRepository |
| **Domain Service** | Logic liên quan nhiều Entity | TransferService |

## Checklist khi viết code DDD

### ✅ Entity
- [ ] Có ID duy nhất
- [ ] Constructor có validation
- [ ] Có business methods (không chỉ getter/setter)
- [ ] `equals()` dựa trên ID
- [ ] Không có setter cho ID

### ✅ Value Object
- [ ] Không có ID
- [ ] Tất cả fields là `final`
- [ ] Không có setter
- [ ] `equals()` so sánh tất cả fields
- [ ] Immutable (methods return new object)

### ✅ Aggregate
- [ ] Có Aggregate Root rõ ràng
- [ ] Entity con chỉ truy cập qua Root
- [ ] Không expose collection trực tiếp
- [ ] Business rules được bảo vệ

### ✅ Service
- [ ] Logic liên quan nhiều Entity
- [ ] Không chứa data
- [ ] Stateless (không lưu trạng thái)

## Bài tập thực hành

??? example "Bài tập 1: Library System"
    **Yêu cầu:** Thiết kế hệ thống thư viện với:
    - Book (có ISBN, title, author)
    - Member (có memberId, name, email)
    - Loan (mượn sách - có ngày mượn, ngày trả)
    
    **Câu hỏi:**
    1. Đâu là Entity, đâu là Value Object?
    2. Đâu là Aggregate Root?
    3. Có cần Domain Service không?
    
    ??? success "Đáp án"
        - **Entity**: Book, Member, Loan
        - **Value Object**: ISBN, Email, LoanPeriod
        - **Aggregate Root**: Loan (chứa Book + Member references)
        - **Domain Service**: `LoanService` (check member có thể mượn không, sách còn không)

??? example "Bài tập 2: E-commerce"
    **Yêu cầu:** Thiết kế shopping cart với:
    - Cart chứa nhiều CartItem
    - Mỗi CartItem có Product và quantity
    - Tính tổng tiền
    
    **Câu hỏi:**
    1. Cart là Entity hay Value Object?
    2. CartItem nằm trong Aggregate nào?
    3. Viết code Cart với DDD
    
    ??? success "Đáp án"
        ```java
        @Entity
        public class Cart {  // Aggregate Root
            @Id private CartId id;
            
            @OneToMany(cascade = ALL)
            private List<CartItem> items = new ArrayList<>();
            
            public void addItem(Product product, int quantity) {
                // Business logic: check duplicate, merge...
                CartItem existing = findItem(product.getId());
                if (existing != null) {
                    existing.increaseQuantity(quantity);
                } else {
                    items.add(new CartItem(product, quantity));
                }
            }
            
            public Money calculateTotal() {
                return items.stream()
                    .map(CartItem::getSubtotal)
                    .reduce(Money.ZERO, Money::add);
            }
        }
        ```

## Tiếp theo

Bạn đã hiểu được các khái niệm cơ bản nhất trong DDD! Giờ có thể:

1. ✅ **Xem ví dụ thực tế**: [Example Project - Banking System](example-project.md)
2. ✅ **Học sâu hơn**: [Building Blocks](building-blocks.md) - Domain Events, Specifications
3. ⏭️ **Tùy chọn**: [Strategic Design](strategic-design.md) - Khi project lớn

## Nguồn tham khảo

!!! info "Tài liệu tham khảo"
    - **Eric Evans** - [Domain-Driven Design](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) - Part II: The Building Blocks of a Model-Driven Design
    - **Vaughn Vernon** - [Implementing Domain-Driven Design](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577) - Chapter 5-8: Entities, Value Objects, Services, Aggregates
    - **Martin Fowler** - [Evans Classification](https://martinfowler.com/bliki/EvansClassification.html)

---

**Tiếp theo:** [Building Blocks →](building-blocks.md)
