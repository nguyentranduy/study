# Tổng quan OOP - Lập trình hướng đối tượng

## OOP là gì?

**OOP (Object-Oriented Programming)** - Lập trình hướng đối tượng là một phương pháp lập trình dựa trên khái niệm "đối tượng" (object), trong đó dữ liệu và các phương thức xử lý dữ liệu được đóng gói lại với nhau.

## Tại sao cần OOP?

### Vấn đề với lập trình thủ tục

```java
// Lập trình thủ tục - khó quản lý khi code lớn
public class Main {
    public static void main(String[] args) {
        String studentName = "Nguyen Van A";
        int studentAge = 20;
        double studentGPA = 3.5;
        
        // Xử lý logic rải rác
        if (studentGPA >= 3.2) {
            System.out.println(studentName + " đạt học bổng");
        }
    }
}
```

### Giải pháp với OOP

```java
// OOP - code có tổ chức, dễ bảo trì
public class Student {
    private String name;
    private int age;
    private double gpa;
    
    public Student(String name, int age, double gpa) {
        this.name = name;
        this.age = age;
        this.gpa = gpa;
    }
    
    public boolean isEligibleForScholarship() {
        return this.gpa >= 3.2;
    }
    
    public void displayInfo() {
        System.out.println("Tên: " + name);
        System.out.println("Tuổi: " + age);
        System.out.println("GPA: " + gpa);
    }
}
```

## Các khái niệm cơ bản

### 1. Class (Lớp)

**Class** là bản thiết kế (blueprint) để tạo ra các đối tượng. Class định nghĩa các thuộc tính và phương thức mà đối tượng sẽ có.

```java
public class Car {
    // Thuộc tính (attributes/fields)
    private String brand;
    private String color;
    private int speed;
    
    // Constructor
    public Car(String brand, String color) {
        this.brand = brand;
        this.color = color;
        this.speed = 0;
    }
    
    // Phương thức (methods)
    public void accelerate() {
        speed += 10;
    }
    
    public void brake() {
        if (speed >= 10) {
            speed -= 10;
        }
    }
}
```

### 2. Object (Đối tượng)

**Object** là một thể hiện (instance) cụ thể của class. Mỗi object có trạng thái riêng.

```java
// Tạo các object từ class Car
Car myCar = new Car("Toyota", "Red");
Car yourCar = new Car("Honda", "Blue");

myCar.accelerate();  // myCar.speed = 10
yourCar.accelerate(); // yourCar.speed = 10
myCar.accelerate();  // myCar.speed = 20
// yourCar.speed vẫn = 10
```

### 3. Thuộc tính (Attributes/Fields)

Thuộc tính là các biến được định nghĩa trong class, đại diện cho trạng thái của object.

```java
public class Person {
    // Instance variables (thuộc tính của từng object)
    private String name;
    private int age;
    
    // Class variable (thuộc tính chung cho tất cả objects)
    private static int count = 0;
}
```

### 4. Phương thức (Methods)

Phương thức là các hàm được định nghĩa trong class, đại diện cho hành vi của object.

```java
public class Calculator {
    // Instance method
    public int add(int a, int b) {
        return a + b;
    }
    
    // Static method
    public static int multiply(int a, int b) {
        return a * b;
    }
}
```

## 4 Tính chất của OOP

| Tính chất | Mô tả | Ví dụ |
|-----------|-------|-------|
| **Encapsulation** | Đóng gói dữ liệu và phương thức | private fields + getter/setter |
| **Inheritance** | Kế thừa từ class cha | class Dog extends Animal |
| **Polymorphism** | Đa hình - nhiều hình thái | method overriding/overloading |
| **Abstraction** | Trừu tượng hóa | abstract class, interface |

!!! info "Xem chi tiết"
    Tìm hiểu chi tiết về [4 tính chất OOP](4-tinh-chat.md)

## So sánh OOP vs Procedural

| Tiêu chí | OOP | Procedural |
|----------|-----|------------|
| Cấu trúc | Dựa trên objects | Dựa trên functions |
| Dữ liệu | Được bảo vệ (encapsulation) | Có thể truy cập tự do |
| Tái sử dụng | Cao (inheritance) | Thấp |
| Bảo trì | Dễ dàng | Khó khăn khi code lớn |
| Phù hợp | Dự án lớn, phức tạp | Script nhỏ, đơn giản |

## Bài tập thực hành

!!! example "Bài tập 1"
    Tạo class `BankAccount` với:
    
    - Thuộc tính: accountNumber, ownerName, balance
    - Phương thức: deposit(), withdraw(), getBalance()
    - Áp dụng encapsulation (private fields + getter/setter)

??? success "Gợi ý lời giải"
    ```java
    public class BankAccount {
        private String accountNumber;
        private String ownerName;
        private double balance;
        
        public BankAccount(String accountNumber, String ownerName) {
            this.accountNumber = accountNumber;
            this.ownerName = ownerName;
            this.balance = 0;
        }
        
        public void deposit(double amount) {
            if (amount > 0) {
                balance += amount;
            }
        }
        
        public boolean withdraw(double amount) {
            if (amount > 0 && balance >= amount) {
                balance -= amount;
                return true;
            }
            return false;
        }
        
        public double getBalance() {
            return balance;
        }
    }
    ```

## Tiếp theo

- [4 tính chất OOP chi tiết](4-tinh-chat.md)
- [Abstract Class & Interface](abstract-interface.md)
- [SOLID Principles](solid.md)
