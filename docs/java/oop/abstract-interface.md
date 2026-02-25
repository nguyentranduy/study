# Abstract Class & Interface

## Abstract Class

### Khái niệm

**Abstract Class** là class không thể tạo instance trực tiếp, được sử dụng làm class cha cho các class khác kế thừa.

### Đặc điểm

- Được khai báo với từ khóa `abstract`
- Có thể chứa cả abstract method và concrete method
- Có thể có constructor
- Có thể có các thuộc tính với mọi access modifier
- Class con **bắt buộc** phải implement tất cả abstract method

### Ví dụ

```java
public abstract class Animal {
    // Thuộc tính
    protected String name;
    protected int age;
    
    // Constructor
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // Abstract method - không có body
    public abstract void makeSound();
    public abstract void move();
    
    // Concrete method - có body
    public void eat() {
        System.out.println(name + " đang ăn");
    }
    
    public void sleep() {
        System.out.println(name + " đang ngủ");
    }
    
    // Getter
    public String getName() {
        return name;
    }
}

// Class con bắt buộc implement abstract methods
public class Dog extends Animal {
    private String breed;
    
    public Dog(String name, int age, String breed) {
        super(name, age);
        this.breed = breed;
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " sủa: Gâu gâu!");
    }
    
    @Override
    public void move() {
        System.out.println(name + " đang chạy bằng 4 chân");
    }
}

public class Bird extends Animal {
    public Bird(String name, int age) {
        super(name, age);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " hót: Chíp chíp!");
    }
    
    @Override
    public void move() {
        System.out.println(name + " đang bay");
    }
}
```

---

## Interface

### Khái niệm

**Interface** là một "hợp đồng" định nghĩa các phương thức mà class implement nó phải cung cấp.

### Đặc điểm

- Được khai báo với từ khóa `interface`
- Tất cả method mặc định là `public abstract` (trước Java 8)
- Tất cả thuộc tính mặc định là `public static final`
- Một class có thể implement **nhiều** interface
- Java 8+: có thể có `default` và `static` method
- Java 9+: có thể có `private` method

### Ví dụ cơ bản

```java
public interface Flyable {
    // Constant (mặc định public static final)
    int MAX_ALTITUDE = 10000;
    
    // Abstract method (mặc định public abstract)
    void fly();
    void land();
}

public interface Swimmable {
    void swim();
    void dive();
}

// Class implement nhiều interface
public class Duck implements Flyable, Swimmable {
    private String name;
    
    public Duck(String name) {
        this.name = name;
    }
    
    @Override
    public void fly() {
        System.out.println(name + " đang bay");
    }
    
    @Override
    public void land() {
        System.out.println(name + " đang hạ cánh");
    }
    
    @Override
    public void swim() {
        System.out.println(name + " đang bơi");
    }
    
    @Override
    public void dive() {
        System.out.println(name + " đang lặn");
    }
}
```

### Java 8+ Features

```java
public interface Vehicle {
    // Abstract method
    void start();
    void stop();
    
    // Default method - có implementation mặc định
    default void horn() {
        System.out.println("Beep beep!");
    }
    
    // Static method
    static void printInfo() {
        System.out.println("This is Vehicle interface");
    }
}

public class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car starting...");
    }
    
    @Override
    public void stop() {
        System.out.println("Car stopping...");
    }
    
    // Có thể override default method
    @Override
    public void horn() {
        System.out.println("Car horn: Honk honk!");
    }
}

// Sử dụng
Car car = new Car();
car.start();  // "Car starting..."
car.horn();   // "Car horn: Honk honk!"
Vehicle.printInfo();  // "This is Vehicle interface"
```

---

## So sánh Abstract Class vs Interface

| Tiêu chí | Abstract Class | Interface |
|----------|----------------|-----------|
| **Từ khóa** | `abstract class` | `interface` |
| **Kế thừa** | `extends` (chỉ 1) | `implements` (nhiều) |
| **Constructor** | Có | Không |
| **Thuộc tính** | Mọi loại | Chỉ `public static final` |
| **Method** | Abstract + Concrete | Abstract + Default + Static |
| **Access modifier** | Mọi loại | Chỉ `public` |
| **Mục đích** | IS-A relationship | CAN-DO relationship |

### Khi nào dùng Abstract Class?

- Khi các class con có **quan hệ IS-A** rõ ràng
- Khi cần chia sẻ code giữa các class liên quan
- Khi cần định nghĩa các thuộc tính non-static, non-final
- Khi cần access modifier khác public

```java
// Dog IS-A Animal
public abstract class Animal { }
public class Dog extends Animal { }
```

### Khi nào dùng Interface?

- Khi các class không liên quan cần có chung hành vi
- Khi cần đa kế thừa (multiple inheritance)
- Khi muốn định nghĩa "hợp đồng" mà không quan tâm implementation

```java
// Duck CAN fly, CAN swim
public interface Flyable { }
public interface Swimmable { }
public class Duck implements Flyable, Swimmable { }
```

---

## Ví dụ thực tế: Hệ thống thanh toán

```java
// Interface định nghĩa hành vi thanh toán
public interface Payable {
    boolean processPayment(double amount);
    void refund(double amount);
}

// Interface cho việc gửi thông báo
public interface Notifiable {
    void sendNotification(String message);
}

// Abstract class cho các phương thức thanh toán
public abstract class PaymentMethod implements Payable, Notifiable {
    protected String accountId;
    protected double balance;
    
    public PaymentMethod(String accountId, double balance) {
        this.accountId = accountId;
        this.balance = balance;
    }
    
    // Template method
    public final void pay(double amount) {
        if (validatePayment(amount)) {
            if (processPayment(amount)) {
                sendNotification("Thanh toán thành công: " + amount);
            }
        }
    }
    
    // Abstract method - mỗi loại thanh toán validate khác nhau
    protected abstract boolean validatePayment(double amount);
    
    public double getBalance() {
        return balance;
    }
}

// Concrete class: Thanh toán bằng thẻ tín dụng
public class CreditCard extends PaymentMethod {
    private String cardNumber;
    private double creditLimit;
    
    public CreditCard(String accountId, String cardNumber, double creditLimit) {
        super(accountId, 0);
        this.cardNumber = cardNumber;
        this.creditLimit = creditLimit;
    }
    
    @Override
    protected boolean validatePayment(double amount) {
        return amount > 0 && amount <= creditLimit;
    }
    
    @Override
    public boolean processPayment(double amount) {
        if (amount <= creditLimit) {
            creditLimit -= amount;
            System.out.println("Thanh toán bằng thẻ tín dụng: " + amount);
            return true;
        }
        return false;
    }
    
    @Override
    public void refund(double amount) {
        creditLimit += amount;
        System.out.println("Hoàn tiền vào thẻ: " + amount);
    }
    
    @Override
    public void sendNotification(String message) {
        System.out.println("[SMS] " + message);
    }
}

// Concrete class: Thanh toán bằng ví điện tử
public class EWallet extends PaymentMethod {
    public EWallet(String accountId, double balance) {
        super(accountId, balance);
    }
    
    @Override
    protected boolean validatePayment(double amount) {
        return amount > 0 && amount <= balance;
    }
    
    @Override
    public boolean processPayment(double amount) {
        if (amount <= balance) {
            balance -= amount;
            System.out.println("Thanh toán bằng ví điện tử: " + amount);
            return true;
        }
        return false;
    }
    
    @Override
    public void refund(double amount) {
        balance += amount;
        System.out.println("Hoàn tiền vào ví: " + amount);
    }
    
    @Override
    public void sendNotification(String message) {
        System.out.println("[Push Notification] " + message);
    }
}
```

### Sử dụng

```java
public class Main {
    public static void main(String[] args) {
        // Polymorphism với interface
        List<Payable> paymentMethods = new ArrayList<>();
        paymentMethods.add(new CreditCard("ACC001", "1234-5678", 5000));
        paymentMethods.add(new EWallet("ACC002", 1000));
        
        for (Payable payment : paymentMethods) {
            payment.processPayment(100);
        }
    }
}
```

---

## Bài tập thực hành

!!! example "Bài tập"
    Thiết kế hệ thống quản lý hình học:
    
    1. Interface `Drawable` với method `draw()`
    2. Interface `Resizable` với method `resize(double factor)`
    3. Abstract class `Shape` với thuộc tính `color` và abstract method `calculateArea()`
    4. Class `Circle`, `Rectangle` kế thừa `Shape` và implement các interface

## Tiếp theo

- [SOLID Principles](solid.md)
