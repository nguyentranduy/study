# Strategic Design - Thiết kế chiến lược

Strategic Design là phần cốt lõi của DDD, tập trung vào cách tổ chức và quản lý các domain model lớn. Theo Eric Evans, việc xây dựng một model thống nhất cho toàn bộ hệ thống lớn là không khả thi và không hiệu quả về chi phí.

!!! quote "Eric Evans - Domain-Driven Design (2003)"
    "Total unification of the domain model for a large system will not be feasible or cost-effective."

## 1. Bounded Context (Ngữ cảnh giới hạn)

### Khái niệm

**Bounded Context** là ranh giới rõ ràng mà trong đó một domain model cụ thể được định nghĩa và áp dụng. Mỗi Bounded Context có:

- Model riêng biệt và nhất quán nội bộ
- Ubiquitous Language riêng
- Ranh giới rõ ràng với các context khác

### Tại sao cần Bounded Context?

Trong một tổ chức lớn, cùng một thuật ngữ có thể có ý nghĩa khác nhau ở các phòng ban khác nhau. Đây gọi là **polyseme** (đa nghĩa).

**Ví dụ thực tế:** Trong công ty điện lực, từ "meter" (đồng hồ đo) có thể nghĩa là:

- **Trong phòng kỹ thuật**: Thiết bị vật lý đo điện
- **Trong phòng khách hàng**: Điểm kết nối giữa lưới điện và khách hàng
- **Trong phòng tài chính**: Đơn vị tính phí

Thay vì cố gắng tạo một model thống nhất (gây nhầm lẫn), DDD chia nhỏ thành các Bounded Context riêng biệt.

### Ví dụ E-commerce

```
┌───────────────────────────┐     ┌───────────────────────────┐
│   Sales Context           │     │   Shipping Context        │
│                           │     │                           │
│   - Product (SKU, Price)  │     │   - Package (Weight, Size)│
│   - Customer (Email)      │     │   - Address (Full)        │
│   - Order                 │────▶│   - Shipment              │
└───────────────────────────┘     └───────────────────────────┘
             │
             │
             ▼
┌───────────────────────────┐
│   Catalog Context         │
│                           │
│   - Product (Detailed)    │
│   - Category              │
│   - Pricing History       │
└───────────────────────────┘
```

**Lưu ý:** "Product" xuất hiện ở cả Sales và Catalog context nhưng có model khác nhau:

- **Sales Context**: Chỉ quan tâm SKU, Price để xử lý đơn hàng
- **Catalog Context**: Có thông tin đầy đủ về sản phẩm, ảnh, mô tả

### Code ví dụ

**Sales Context:**
```java
// Sales bounded context
public class Product {
    private String sku;
    private Money price;
    private String name;
    
    public Money calculateTotal(int quantity) {
        return price.multiply(quantity);
    }
}
```

**Catalog Context:**
```java
// Catalog bounded context
public class Product {
    private String sku;
    private String name;
    private String description;
    private List<String> images;
    private Category category;
    private List<Review> reviews;
    
    public double getAverageRating() {
        return reviews.stream()
            .mapToDouble(Review::getRating)
            .average()
            .orElse(0.0);
    }
}
```

## 2. Context Map (Bản đồ ngữ cảnh)

**Context Map** là một bản đồ mô tả mối quan hệ giữa các Bounded Contexts và cách chúng tích hợp với nhau.

### Các mẫu quan hệ trong Context Map

#### a) Partnership (Đối tác)

Hai team phối hợp chặt chẽ để phát triển. Thay đổi ở một bên ảnh hưởng đến bên kia.

```
┌────────────────┐                    ┌────────────────┐
│   Context A    │◀──── Partnership ───▶│   Context B    │
│                │                    │                │
└────────────────┘                    └────────────────┘
```

#### b) Shared Kernel (Nhân chung)

Hai context chia sẻ một phần model nhỏ. Mọi thay đổi phải được cả hai team đồng ý.

```
┌────────────────┐                    ┌────────────────┐
│   Context A    │                    │   Context B    │
│                │                    │                │
│    ┌───────────┴────────────────────┴──────────┐    │
│    │          Shared Kernel                    │    │
│    └───────────┬────────────────────┬──────────┘    │
└────────────────┘                    └────────────────┘
```

#### c) Customer-Supplier (Khách hàng - Nhà cung cấp)

Downstream (khách hàng) phụ thuộc vào Upstream (nhà cung cấp). Upstream phải cung cấp API cho downstream.

```
┌────────────────┐                    ┌────────────────┐
│   Supplier     │──────── API ──────▶│   Customer     │
│   (Upstream)   │                    │  (Downstream)  │
└────────────────┘                    └────────────────┘
```

**Ví dụ:** Payment Service (upstream) cung cấp API cho Order Service (downstream).

#### d) Conformist (Tuân thủ)

Downstream phải tuân theo model của upstream mà không có quyền thương lượng.

```
┌────────────────┐                    ┌────────────────┐
│   Upstream     │───── Must use ────▶│  Downstream    │
│   (External)   │                    │   (Conform)    │
└────────────────┘                    └────────────────┘
```

**Ví dụ:** Ứng dụng phải tuân theo API của Google Maps, Facebook, v.v.

#### e) Anti-Corruption Layer (ACL - Lớp chống tham nhũng)

Downstream tạo một lớp dịch để bảo vệ model của mình khỏi model của upstream.

```
┌────────────────┐          ┌─────────────┐          ┌────────────────┐
│   External     │─────────▶│     ACL     │─────────▶│  Our Context   │
│    System      │          │ Translation │          │                │
└────────────────┘          └─────────────┘          └────────────────┘
```

**Code ví dụ:**
```java
// External system có model phức tạp
public class ExternalCustomer {
    private String custId;
    private String fName;
    private String lName;
    private String addr1;
    private String addr2;
    // ... 50 fields khác
}

// ACL - dịch sang model của chúng ta
public class CustomerAdapter {
    public Customer toCustomer(ExternalCustomer external) {
        return new Customer(
            new CustomerId(external.getCustId()),
            external.getFName() + " " + external.getLName(),
            new Address(external.getAddr1(), external.getAddr2())
        );
    }
}

// Model của chúng ta - sạch sẽ, chỉ có thông tin cần thiết
public class Customer {
    private CustomerId id;
    private String fullName;
    private Address address;
}
```

#### f) Open Host Service (OHS)

Định nghĩa một protocol/API công khai cho phép các context khác dễ dàng tích hợp.

```
        ┌────────────────┐
        │   Open Host    │
        │    Service     │
        │   (REST API)   │◀────── Context A
        │                │◀────── Context B
        │                │◀────── Context C
        └────────────────┘
```

**Ví dụ:** Xây dựng REST API để các service khác consume.

#### g) Separate Ways (Đường riêng)

Hai context hoàn toàn độc lập, không có tích hợp.

```
┌────────────────┐                    ┌────────────────┐
│   Context A    │         X          │   Context B    │
│                │    (No relation)   │                │
│                │                    │                │
└────────────────┘                    └────────────────┘
```

## 3. Ubiquitous Language (Ngôn ngữ chung)

### Khái niệm

**Ubiquitous Language** là ngôn ngữ được dùng chung bởi developers, domain experts, và stakeholders. Ngôn ngữ này phải:

- Được nhúng vào code (tên class, method, biến)
- Được dùng trong meetings, documents
- Được cập nhật liên tục khi hiểu biết về domain tăng

### Tại sao quan trọng?

❌ **Không có Ubiquitous Language:**
```java
// Developer tự nghĩ ra tên
public class AccountInfo {
    public void doTransfer(int amt, String to) {
        // ...
    }
}
```

✅ **Có Ubiquitous Language:**
```java
// Dùng ngôn ngữ của domain experts
public class BankAccount {
    public void transferFunds(Money amount, BankAccount beneficiary) {
        this.debit(amount);
        beneficiary.credit(amount);
    }
    
    private void debit(Money amount) { /* ... */ }
    private void credit(Money amount) { /* ... */ }
}
```

### Cách xây dựng Ubiquitous Language

1. **Họp với domain experts** - Lắng nghe cách họ nói về nghiệp vụ
2. **Ghi chép thuật ngữ** - Tạo glossary các thuật ngữ quan trọng
3. **Dùng trong code** - Class, method phải dùng đúng thuật ngữ
4. **Cập nhật liên tục** - Khi hiểu biết thay đổi, refactor code

### Ví dụ trong Healthcare Domain

**Thuật ngữ từ domain experts:**

- **Patient**: Người bệnh
- **Admission**: Nhập viện
- **Discharge**: Xuất viện
- **Medication**: Thuốc
- **Prescription**: Đơn thuốc
- **Administer**: Tiêm/cho uống thuốc

**Áp dụng vào code:**
```java
public class Patient {
    private PatientId id;
    private List<Admission> admissions;
    
    public Admission admit(Ward ward, LocalDateTime admissionTime) {
        Admission admission = new Admission(this, ward, admissionTime);
        admissions.add(admission);
        return admission;
    }
}

public class Admission {
    private Patient patient;
    private Ward ward;
    private LocalDateTime admissionTime;
    private LocalDateTime dischargeTime;
    
    public void discharge(LocalDateTime time) {
        if (dischargeTime != null) {
            throw new AlreadyDischargedException();
        }
        this.dischargeTime = time;
    }
}

public class Prescription {
    private Patient patient;
    private Medication medication;
    private Dosage dosage;
    
    public void administer(Nurse nurse, LocalDateTime time) {
        // Record medication administration
    }
}
```

## Lợi ích của Strategic Design

1. **Quản lý complexity**: Chia nhỏ hệ thống lớn thành các phần dễ quản lý
2. **Team autonomy**: Mỗi team có thể làm việc độc lập trên context của mình
3. **Clear boundaries**: Ranh giới rõ ràng giữa các subsystem
4. **Flexible integration**: Nhiều cách tích hợp khác nhau tùy tình huống

## Nguồn tham khảo

!!! info "Tài liệu tham khảo"
    - **Eric Evans** - [Domain-Driven Design](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) - Chapter 14, 15: Context and Distillation
    - **Vaughn Vernon** - [Implementing Domain-Driven Design](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577) - Chapter 2, 3: Bounded Context and Context Mapping
    - **Martin Fowler** - [Bounded Context](https://martinfowler.com/bliki/BoundedContext.html)
    - **Martin Fowler** - [Ubiquitous Language](https://martinfowler.com/bliki/UbiquitousLanguage.html)

---

**Tiếp theo:** [Tactical Design →](tactical-design.md)
