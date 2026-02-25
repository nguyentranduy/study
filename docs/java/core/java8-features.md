# Java 8+ Features

Java 8 đánh dấu bước ngoặt lớn với nhiều tính năng mới như Lambda, Stream API, Optional. Các phiên bản sau tiếp tục bổ sung nhiều cải tiến.

## Lambda Expressions

### Cú pháp

```java
// Cú pháp đầy đủ
(parameters) -> { statements; }

// Cú pháp rút gọn
(parameters) -> expression

// Ví dụ
(int a, int b) -> { return a + b; }
(a, b) -> a + b
x -> x * 2
() -> System.out.println("Hello")
```

### Ví dụ sử dụng

```java
// Trước Java 8
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
});

// Java 8+ với Lambda
Collections.sort(names, (a, b) -> a.compareTo(b));

// Hoặc với method reference
Collections.sort(names, String::compareTo);
names.sort(String::compareTo);
```

### Functional Interfaces

Lambda chỉ dùng được với Functional Interface (interface có đúng 1 abstract method).

```java
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
}

// Sử dụng
Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

System.out.println(add.calculate(5, 3));      // 8
System.out.println(multiply.calculate(5, 3)); // 15
```

### Built-in Functional Interfaces

| Interface | Method | Mô tả |
|-----------|--------|-------|
| `Predicate<T>` | `boolean test(T t)` | Kiểm tra điều kiện |
| `Function<T,R>` | `R apply(T t)` | Chuyển đổi T thành R |
| `Consumer<T>` | `void accept(T t)` | Xử lý T, không trả về |
| `Supplier<T>` | `T get()` | Cung cấp giá trị T |
| `BiFunction<T,U,R>` | `R apply(T t, U u)` | 2 input, 1 output |

```java
// Predicate
Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4));  // true

// Function
Function<String, Integer> length = s -> s.length();
System.out.println(length.apply("Hello"));  // 5

// Consumer
Consumer<String> printer = s -> System.out.println(s);
printer.accept("Hello");  // Hello

// Supplier
Supplier<Double> random = () -> Math.random();
System.out.println(random.get());  // 0.xxx

// Chaining
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEvenAndPositive = isEven.and(isPositive);
```

---

## Method References

### Các loại Method Reference

```java
// 1. Static method reference
Function<String, Integer> parser = Integer::parseInt;
// Tương đương: s -> Integer.parseInt(s)

// 2. Instance method reference (bound)
String str = "Hello";
Supplier<Integer> lengthSupplier = str::length;
// Tương đương: () -> str.length()

// 3. Instance method reference (unbound)
Function<String, Integer> lengthFunc = String::length;
// Tương đương: s -> s.length()

// 4. Constructor reference
Supplier<ArrayList<String>> listSupplier = ArrayList::new;
// Tương đương: () -> new ArrayList<>()

Function<Integer, int[]> arrayCreator = int[]::new;
// Tương đương: size -> new int[size]
```

---

## Stream API

### Tạo Stream

```java
// Từ Collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();

// Từ Array
String[] array = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(array);

// Stream.of()
Stream<String> stream3 = Stream.of("a", "b", "c");

// Stream.generate() - infinite stream
Stream<Double> randoms = Stream.generate(Math::random).limit(5);

// Stream.iterate() - infinite stream
Stream<Integer> numbers = Stream.iterate(0, n -> n + 2).limit(10);

// IntStream, LongStream, DoubleStream
IntStream intStream = IntStream.range(1, 10);      // 1-9
IntStream intStream2 = IntStream.rangeClosed(1, 10); // 1-10
```

### Intermediate Operations

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Eve");

// filter - lọc theo điều kiện
names.stream()
    .filter(name -> name.length() > 3)
    .forEach(System.out::println);  // Alice, Charlie, David

// map - chuyển đổi
names.stream()
    .map(String::toUpperCase)
    .forEach(System.out::println);  // ALICE, BOB, ...

// flatMap - flatten nested collections
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4, 5)
);
nested.stream()
    .flatMap(List::stream)
    .forEach(System.out::println);  // 1, 2, 3, 4, 5

// sorted
names.stream()
    .sorted()
    .forEach(System.out::println);  // Alice, Bob, Charlie, David, Eve

names.stream()
    .sorted(Comparator.reverseOrder())
    .forEach(System.out::println);  // Eve, David, Charlie, Bob, Alice

// distinct - loại bỏ trùng lặp
Stream.of(1, 2, 2, 3, 3, 3)
    .distinct()
    .forEach(System.out::println);  // 1, 2, 3

// limit & skip
names.stream()
    .skip(2)
    .limit(2)
    .forEach(System.out::println);  // Charlie, David

// peek - debug
names.stream()
    .peek(n -> System.out.println("Processing: " + n))
    .map(String::toUpperCase)
    .forEach(System.out::println);
```

### Terminal Operations

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// forEach
numbers.stream().forEach(System.out::println);

// collect - thu thập kết quả
List<Integer> evenList = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

Set<Integer> evenSet = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toSet());

// toArray
Integer[] array = numbers.stream().toArray(Integer[]::new);

// reduce
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);

Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);

// count
long count = numbers.stream()
    .filter(n -> n > 2)
    .count();  // 3

// min, max
Optional<Integer> min = numbers.stream().min(Integer::compareTo);
Optional<Integer> maxVal = numbers.stream().max(Integer::compareTo);

// findFirst, findAny
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 2)
    .findFirst();  // 3

// anyMatch, allMatch, noneMatch
boolean anyEven = numbers.stream().anyMatch(n -> n % 2 == 0);  // true
boolean allPositive = numbers.stream().allMatch(n -> n > 0);   // true
boolean noneNegative = numbers.stream().noneMatch(n -> n < 0); // true
```

### Collectors

```java
List<Person> people = Arrays.asList(
    new Person("Alice", 25, "IT"),
    new Person("Bob", 30, "HR"),
    new Person("Charlie", 25, "IT"),
    new Person("David", 35, "HR")
);

// toList, toSet
List<String> names = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());

// joining
String joined = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining(", "));  // "Alice, Bob, Charlie, David"

// groupingBy
Map<String, List<Person>> byDept = people.stream()
    .collect(Collectors.groupingBy(Person::getDepartment));
// {IT=[Alice, Charlie], HR=[Bob, David]}

// groupingBy với downstream collector
Map<String, Long> countByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDepartment,
        Collectors.counting()
    ));
// {IT=2, HR=2}

// partitioningBy
Map<Boolean, List<Person>> partitioned = people.stream()
    .collect(Collectors.partitioningBy(p -> p.getAge() > 25));
// {false=[Alice, Charlie], true=[Bob, David]}

// summarizingInt
IntSummaryStatistics stats = people.stream()
    .collect(Collectors.summarizingInt(Person::getAge));
// count=4, sum=115, min=25, average=28.75, max=35

// toMap
Map<String, Integer> nameToAge = people.stream()
    .collect(Collectors.toMap(
        Person::getName,
        Person::getAge
    ));
```

### Parallel Streams

```java
// Parallel stream
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

int sum = numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .mapToInt(Integer::intValue)
    .sum();

// Convert to parallel
numbers.stream()
    .parallel()
    .forEach(System.out::println);
```

---

## Optional

### Tạo Optional

```java
// Có giá trị
Optional<String> opt1 = Optional.of("Hello");

// Có thể null
Optional<String> opt2 = Optional.ofNullable(null);

// Empty
Optional<String> opt3 = Optional.empty();
```

### Sử dụng Optional

```java
Optional<String> opt = Optional.of("Hello");

// isPresent, isEmpty (Java 11+)
if (opt.isPresent()) {
    System.out.println(opt.get());
}

// ifPresent
opt.ifPresent(System.out::println);

// ifPresentOrElse (Java 9+)
opt.ifPresentOrElse(
    System.out::println,
    () -> System.out.println("Empty")
);

// orElse - giá trị mặc định
String value = opt.orElse("Default");

// orElseGet - lazy evaluation
String value2 = opt.orElseGet(() -> computeDefault());

// orElseThrow
String value3 = opt.orElseThrow(() -> new RuntimeException("Not found"));

// map
Optional<Integer> length = opt.map(String::length);

// flatMap
Optional<String> result = opt.flatMap(s -> Optional.of(s.toUpperCase()));

// filter
Optional<String> filtered = opt.filter(s -> s.length() > 3);

// or (Java 9+)
Optional<String> result2 = opt.or(() -> Optional.of("Alternative"));
```

### Best Practices

```java
// GOOD: Dùng Optional làm return type
public Optional<User> findUserById(Long id) {
    User user = userRepository.find(id);
    return Optional.ofNullable(user);
}

// BAD: Không dùng Optional làm parameter
public void processUser(Optional<User> user) { }  // Không nên!

// BAD: Không dùng Optional cho fields
public class User {
    private Optional<String> nickname;  // Không nên!
}

// GOOD: Chain operations
String city = user
    .flatMap(User::getAddress)
    .flatMap(Address::getCity)
    .map(String::toUpperCase)
    .orElse("UNKNOWN");
```

---

## Date/Time API (java.time)

### Các class chính

```java
import java.time.*;

// LocalDate - chỉ ngày
LocalDate today = LocalDate.now();
LocalDate birthday = LocalDate.of(1990, 5, 15);
LocalDate parsed = LocalDate.parse("2024-01-15");

// LocalTime - chỉ giờ
LocalTime now = LocalTime.now();
LocalTime meeting = LocalTime.of(14, 30, 0);

// LocalDateTime - ngày + giờ
LocalDateTime dateTime = LocalDateTime.now();
LocalDateTime specific = LocalDateTime.of(2024, 1, 15, 14, 30);

// ZonedDateTime - có timezone
ZonedDateTime zonedNow = ZonedDateTime.now();
ZonedDateTime tokyo = ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));

// Instant - timestamp
Instant instant = Instant.now();
long epochMilli = instant.toEpochMilli();

// Duration - khoảng thời gian (giờ, phút, giây)
Duration duration = Duration.between(time1, time2);
Duration twoHours = Duration.ofHours(2);

// Period - khoảng thời gian (năm, tháng, ngày)
Period period = Period.between(date1, date2);
Period oneMonth = Period.ofMonths(1);
```

### Thao tác với Date/Time

```java
LocalDate today = LocalDate.now();

// Cộng/trừ
LocalDate nextWeek = today.plusWeeks(1);
LocalDate lastMonth = today.minusMonths(1);
LocalDate nextYear = today.plusYears(1);

// Lấy thông tin
int year = today.getYear();
Month month = today.getMonth();
int day = today.getDayOfMonth();
DayOfWeek dayOfWeek = today.getDayOfWeek();

// So sánh
boolean isBefore = date1.isBefore(date2);
boolean isAfter = date1.isAfter(date2);

// Format
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
String formatted = today.format(formatter);  // "15/01/2024"

// Parse
LocalDate parsed = LocalDate.parse("15/01/2024", formatter);
```

---

## Các tính năng Java 9+

### Java 9

```java
// Collection factory methods
List<String> list = List.of("a", "b", "c");
Set<String> set = Set.of("a", "b", "c");
Map<String, Integer> map = Map.of("a", 1, "b", 2);

// Optional improvements
Optional.of("value").or(() -> Optional.of("default"));
Optional.of("value").ifPresentOrElse(
    System.out::println,
    () -> System.out.println("empty")
);

// Stream improvements
Stream.of(1, 2, 3).takeWhile(n -> n < 3);  // 1, 2
Stream.of(1, 2, 3).dropWhile(n -> n < 3);  // 3
Stream.ofNullable(null);  // empty stream
```

### Java 10

```java
// var - local variable type inference
var list = new ArrayList<String>();
var map = new HashMap<String, Integer>();

for (var entry : map.entrySet()) {
    var key = entry.getKey();
    var value = entry.getValue();
}
```

### Java 11

```java
// String methods
"  hello  ".strip();        // "hello"
"  hello  ".stripLeading(); // "hello  "
"  hello  ".stripTrailing();// "  hello"
"".isBlank();               // true
"hello\nworld".lines();     // Stream<String>
"abc".repeat(3);            // "abcabcabc"

// var in lambda
list.stream()
    .map((var s) -> s.toUpperCase())
    .collect(Collectors.toList());
```

### Java 14+

```java
// Switch expressions
String result = switch (day) {
    case MONDAY, FRIDAY -> "Working";
    case SATURDAY, SUNDAY -> "Weekend";
    default -> "Midweek";
};

// Text blocks (Java 15+)
String json = """
    {
        "name": "John",
        "age": 30
    }
    """;

// Records (Java 16+)
public record Person(String name, int age) {}

Person p = new Person("Alice", 25);
System.out.println(p.name());  // Alice
System.out.println(p.age());   // 25

// Pattern matching for instanceof (Java 16+)
if (obj instanceof String s) {
    System.out.println(s.length());
}

// Sealed classes (Java 17+)
public sealed class Shape permits Circle, Rectangle, Square {}
```

---

## Bài tập thực hành

!!! example "Bài tập"
    Cho danh sách nhân viên, sử dụng Stream API để:
    
    1. Lọc nhân viên có lương > 5000
    2. Nhóm theo phòng ban
    3. Tính tổng lương theo phòng ban
    4. Tìm nhân viên có lương cao nhất mỗi phòng ban

## Tiếp theo

- [Servlet cơ bản](../web/servlet.md)
