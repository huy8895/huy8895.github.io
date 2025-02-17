---
title: "Những thay đổi từ Java 8 đến Java 23: Từ Lý Thuyết đến Thực Hành"
description: ""
author: huy8895
date: 2025-02-09 09:22:00 +0700
categories: [Java,Programming]
tags: [Java, "Java 8"]
pin: false
math: false
mermaid: true
image:
  path: assets/img/posts/20250209/0.png
---


# Những thay đổi từ Java 8 đến Java 23: Từ Lý Thuyết đến Thực Hành

Trong bài viết này, chúng ta sẽ cùng tổng hợp những cải tiến và thay đổi quan trọng của Java từ phiên bản 8 đến phiên bản 23. Qua đó, bạn sẽ nắm được hành trình phát triển của Java qua các phiên bản, từ những cải tiến về ngôn ngữ, hiệu năng đến các tính năng hiện đại hỗ trợ công nghệ mới như AI/ML.

## Java 8

> **Lưu ý quan trọng**: Các tính năng Java 8 là nền tảng cho mọi phiên bản sau này. Hãy nắm vững Stream API và Lambda trước khi học các tính năng mới hơn.

- **Tính ổn định và phổ biến**: Java 8 là một trong những phiên bản được sử dụng rộng rãi.
- **Lambda Expressions** và **Stream API**: Mở ra kỷ nguyên lập trình hàm trong Java.
- **Optional**: Giúp xử lý giá trị null một cách an toàn.
- **Default Methods trong Interface**: Cho phép thêm phương thức mới vào interface mà không làm vỡ các lớp cũ
- **Date/Time API mới (java.time)**: Thay thế Date/Calendar cũ bằng API xử lý ngày giờ hiện đại và an toàn hơn
- **CompletableFuture**: Cải tiến lập trình bất đồng bộ với API Future mạnh mẽ hơn

**Lambda Expressions - Biểu thức hàm ngắn gọn**
```java
// Ví dụ Lambda với Comparator
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
Collections.sort(names, (a, b) -> a.length() - b.length());
```

> Dùng method reference khi có thể: `String::length` thay vì `s -> s.length()`
{: .prompt-tip }

**Stream API - Xử lý tập dữ liệu theo kiểu chuỗi**
```java
List<String> filtered = languages.stream()
                               .filter(s -> s.length() > 3)
                               .map(String::toUpperCase)
                               .collect(Collectors.toList());
```

**Optional - Tránh NullPointerException**
```java
Optional<String> optionalName = Optional.ofNullable(getName());
String result = optionalName.orElseGet(() -> "Guest");
optionalName.ifPresent(name -> System.out.println("Xin chào " + name));
```

**Default Methods - Mở rộng interface linh hoạt**
```java
// Ví dụ Default Methods
interface Calculator {
    default double sqrt(int a) {
        return Math.sqrt(a);
    }
}

class ScientificCalculator implements Calculator {
    // Kế thừa phương thức mặc định
}
```

**Date/Time API - Quản lý thời gian hiện đại**
```java
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
System.out.println("Thời gian hiện tại: " + now.format(formatter));
```

**CompletableFuture - Xử lý bất đồng bộ nâng cao**
```java
CompletableFuture.supplyAsync(() -> fetchDataFromAPI())
                .thenApply(data -> processData(data))
                .thenAccept(result -> saveToDatabase(result))
                .exceptionally(ex -> {
                    System.out.println("Lỗi: " + ex.getMessage());
                    return null;
                });
```

## Java 9 (2017)

> Project Jigsaw (module system) có thể gây breaking change với các thư viện cũ
{: .prompt-warning }
    
- **Module System (Project Jigsaw)**: Giúp chia nhỏ ứng dụng thành các mô-đun độc lập, dễ quản lý và bảo trì.

```java
// module-info.java
module com.example.myapp {
    requires java.base;
    requires java.sql;
    exports com.example.myapp.api;
}
```

**Sử dụng module trong ứng dụng**:
```java
// Trong package được export
package com.example.myapp.api;

public class DataService {
    public static String fetchData() {
        return "Dữ liệu từ module";
    }
}

// Module khác sử dụng
module com.example.client {
    requires com.example.myapp;
}

// Sử dụng trong code
import com.example.myapp.api.DataService;

public class Main {
    public static void main(String[] args) {
        System.out.println(DataService.fetchData());
    }
}
```

> Lưu ý: Các package không được export sẽ không thể truy cập từ module khác
{: .prompt-warning }

- **JShell (REPL)**: Công cụ tương tác dòng lệnh để thử nghiệm các đoạn mã Java.

```java
jshell> int a = 10
jshell> int b = 20
jshell> System.out.println("Tổng: " + (a + b))
Tổng: 30
```

- **Cải tiến API Stream** và **Factory Methods cho Collections**: Đơn giản hóa việc khởi tạo danh sách, tập hợp và bản đồ.

```java
// Tạo List bất biến
List<String> colors = List.of("Đỏ", "Xanh", "Vàng");

// Tạo Map bất biến
Map<String, Integer> populations = Map.of(
    "Hà Nội", 8_000_000,
    "TPHCM", 9_000_000
);

// takeWhile trong Stream
List<Integer> numbers = List.of(2, 4, 6, 7, 8, 10);
List<Integer> evenNumbers = numbers.stream()
                                 .takeWhile(n -> n % 2 == 0)
                                 .collect(Collectors.toList()); // [2, 4, 6]
```

> Các collection tạo bởi `List.of()`, `Set.of()`, `Map.of()` là bất biến và không thể thêm/xóa phần tử
{: .prompt-tip }

## Java 10 (2018)

- **Local-Variable Type Inference (`var`)**: Cho phép khai báo biến mà không cần chỉ rõ kiểu dữ liệu, giúp code ngắn gọn và dễ đọc hơn.

```java
var list = List.of("Java", "Python", "C++");
var map = Map.of("Hà Nội", 8_000_000, "TPHCM", 9_000_000);

list.forEach((var item) -> System.out.println(item.toUpperCase()));
```

> Chỉ dùng `var` khi kiểu dữ liệu rõ ràng từ phía bên phải
{: .prompt-tip }

- **Cải tiến G1 Garbage Collector**: Tối ưu hóa thời gian dừng của quá trình thu gom rác.

```bash
# Kích hoạt G1 GC với các tham số tối ưu
java -XX:+UseG1GC -Xmx4g -XX:MaxGCPauseMillis=200 MyApp
```

## Java 11 (2018) - Phiên bản LTS

> Java 11 yêu cầu cập nhật các dependencies cũ không hỗ trợ module system
{: .prompt-warning }

- Phiên bản **LTS (Long-Term Support)**: Đảm bảo hỗ trợ và ổn định lâu dài.
- **HTTP Client API**: Hỗ trợ giao tiếp với HTTP/2 và WebSocket.

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://api.example.com/data"))
        .build();

client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
        .thenApply(HttpResponse::body)
        .thenAccept(System.out::println)
        .join();
```

- Khả năng chạy trực tiếp file `.java` mà không cần biên dịch trước:

```bash
java Main.java
```

- Cải tiến các tiện ích của **String API**:

```java
String text = "  Java 11  ";
System.out.println(text.strip());          // "Java 11"
System.out.println(text.repeat(3));        // "  Java 11   Java 11   Java 11  "
System.out.println(" ".isBlank());         // true
```

## Java 12 (2019)

- **Switch Expressions (Preview)**: Cung cấp cú pháp mới cho `switch` với arrow syntax và trả về giá trị.

```java
// Switch expression dạng arrow syntax
String dayType = switch (day) {
    case "MON", "TUE", "WED", "THU", "FRI" -> "Ngày làm việc";
    case "SAT", "SUN" -> "Ngày nghỉ";
    default -> throw new IllegalArgumentException("Ngày không hợp lệ: " + day);
};

// Switch trả về giá trị
int numLetters = switch (fruit) {
    case "apple" -> 5;
    case "banana" -> 6;
    case "orange" -> 6;
    default -> throw new IllegalStateException();
};
```

- **JVM Constants API**: Hỗ trợ thao tác với các constant trong JVM qua lớp `ConstantDesc`.

```java
// Ví dụ sử dụng Constant API
import java.lang.constant.*;

ClassDesc stringClass = ConstantDescs.CD_String;
MethodTypeDesc concatDesc = MethodTypeDesc.of(stringClass, stringClass);
```

> Switch expressions trong Java 12 là tính năng preview - cần kích hoạt với `--enable-preview`
{: .prompt-info }

## Java 13 (2019)

- **Text Blocks (Preview)**: Giúp viết chuỗi nhiều dòng dễ đọc hơn.

```java
// JSON example
String json = """
    {
        "name": "Nguyễn Văn A",
        "age": 30,
        "address": {
            "city": "Hà Nội",
            "district": "Cầu Giấy"
        }
    }
    """;

// HTML template
String html = """
    <html>
        <body>
            <h1>Chào mừng %s</h1>
            <p>Bạn có %d thông báo mới</p>
        </body>
    </html>
    """.formatted("Huy", 5);
```

- **Dynamic CDS Archives**: Cải thiện thời gian khởi động ứng dụng qua việc tạo archive tự động.

```bash
# Tạo Class Data Archive
java -XX:ArchiveClassesAtExit=app.jsa -jar myapp.jar

# Sử dụng archive đã tạo
java -XX:SharedArchiveFile=app.jsa -jar myapp.jar
```

> Text Blocks giúp code dễ đọc hơn khi làm việc với JSON/XML/SQL
{: .prompt-tip }

## Java 14 (2020)

- **Records (Preview)**: Cung cấp cú pháp ngắn gọn để định nghĩa các lớp chỉ chứa dữ liệu.

```java
// Định nghĩa record cho thông tin người dùng
public record User(
    String username,
    String email,
    LocalDate registeredDate
) {}

// Sử dụng record
User newUser = new User("huy8895", "huy@example.com", LocalDate.now());
System.out.println(newUser.username()); // huy8895
```

- **Pattern Matching cho `instanceof` (Preview)**: Cải thiện cú pháp kiểm tra kiểu đối tượng.

```java
// Kiểm tra và ép kiểu trong 1 biểu thức
Object obj = "Hello Java 14";
if (obj instanceof String s && s.length() > 5) {
    System.out.println("Độ dài chuỗi: " + s.length());
}

// Sử dụng với switch expression
String formatted = switch (obj) {
    case Integer i -> String.format("Số nguyên: %d", i);
    case Double d -> String.format("Số thực: %.2f", d);
    case String s -> String.format("Chuỗi: %s", s);
    default -> "Kiểu không xác định";
};
```

> Cần kích hoạt preview features khi compile:
> `javac --enable-preview --release 14 Main.java`
{: .prompt-info }

## Java 15 (2020)

- **Sealed Classes (Preview)**: Cho phép kiểm soát chặt chẽ các lớp được phép kế thừa, giúp xây dựng hệ thống phân cấp lớp an toàn hơn. Đặc biệt hữu ích khi xây dựng các domain model phức tạp.

```java
// Sealed class định nghĩa các loại tài khoản ngân hàng
public sealed abstract class BankAccount 
    permits SavingsAccount, CheckingAccount, LoanAccount {
    
    protected double balance;
    
    public abstract void deposit(double amount);
    public abstract void withdraw(double amount);
}

// Tài khoản tiết kiệm - final class
public final class SavingsAccount extends BankAccount {
    private double interestRate;
    
    @Override
    public void deposit(double amount) {
        balance += amount * (1 + interestRate);
    }
    
    // ... các phương thức khác ...
}

// Tài khoản vay - non-sealed class
public non-sealed class LoanAccount extends BankAccount {
    private double interestRate;
    private double loanLimit;
    
    @Override
    public void withdraw(double amount) {
        if (amount <= loanLimit) {
            balance -= amount;
        }
    }
    // ... có thể được mở rộng thêm ...
}

// Lớp con của LoanAccount
public class MortgageAccount extends LoanAccount {
    private int termYears;
    // ... triển khai cụ thể ...
}
```

**Cơ chế hoạt động**:
1. Lớp cha khai báo `sealed` và liệt kê các lớp con được phép kế thừa qua từ khóa `permits`
2. Lớp con phải là:
   - `final`: Không cho phép kế thừa tiếp
   - `sealed`: Cho phép kế thừa nhưng phải khai báo lớp con tiếp theo
   - `non-sealed`: Cho phép kế thừa tự do

**Lợi ích**:
- Kiểm soát được các lớp con trong hệ thống phân cấp
- Tăng tính bảo mật và ổn định của code
- Hỗ trợ tốt cho pattern matching trong các phiên bản Java mới

> Khi sử dụng Sealed Classes, cần khai báo rõ ràng các lớp con được phép kế thừa trong cùng package hoặc module
{: .prompt-warning }

## Java 16 (2021)

- **Pattern Matching cho `instanceof`** trở nên chính thức, giúp viết điều kiện kiểm tra kiểu ngắn gọn hơn.

```java
// Ví dụ 1: Xử lý đối tượng
public void process(Object input) {
    if (input instanceof String s) {
        System.out.println("Độ dài chuỗi: " + s.length());
    } else if (input instanceof List<?> list && !list.isEmpty()) {
        System.out.println("Phần tử đầu: " + list.get(0));
    }
}

// Ví dụ 2: Kết hợp với biểu thức
String format(Object o) {
    return o instanceof Integer i ? "Số nguyên: " + i 
         : o instanceof Double d ? "Số thực: " + d 
         : "Kiểu không hỗ trợ";
}
```

- **Records** chính thức trở thành tính năng của ngôn ngữ.

```java
// Record cho thông tin địa chỉ
public record Address(
    String street,
    String city,
    String postalCode
) {
    // Thêm phương thức validate
    public boolean isValid() {
        return postalCode.matches("\\d{6}");
    }
}

// Sử dụng record
Address addr = new Address("123 Lê Lợi", "Hà Nội", "100000");
System.out.println(addr.city()); // Hà Nội
System.out.println("Valid: " + addr.isValid()); // true
```

- **Vector API (Incubator)**: Hỗ trợ tối ưu hóa tính toán dựa trên phần cứng SIMD.

```java
// Ví dụ tính tổng 2 vector
int[] a = {1, 2, 3, 4};
int[] b = {5, 6, 7, 8};

IntVector vectorA = IntVector.fromArray(IntVector.SPECIES_128, a, 0);
IntVector vectorB = IntVector.fromArray(IntVector.SPECIES_128, b, 0);
IntVector result = vectorA.add(vectorB);

int[] resultArray = new int[4];
result.intoArray(resultArray, 0); // [6, 8, 10, 12]
```

> Vector API giúp tăng tốc các phép toán số học song song lên đến 4-8 lần
{: .prompt-tip }

## Java 17 (2021) - Phiên bản LTS

- **Sealed Classes** được đưa vào chính thức.
- **Foreign Function & Memory API (Incubator)**: Cho phép gọi hàm bên ngoài JVM và thao tác với bộ nhớ ngoài heap.
- Cải tiến bảo mật với **Context-Specific Deserialization Filters**.
- **Pattern Matching cho switch (Preview)**:

```java
String formatValue(Object value) {
    return switch (value) {
        case null -> "Giá trị null";
        case Integer i -> String.format("Số nguyên: %d", i);
        case Double d -> String.format("Số thực: %.2f", d);
        case String s -> "Chuỗi: " + s;
        default -> "Kiểu không xác định";
    };
}
```

## Java 18 (2022)

- **UTF-8 mặc định**:

```java
// Ghi file với encoding mặc định UTF-8
Path filePath = Path.of("test.txt");
Files.writeString(filePath, "Tiếng Việt có dấu: Đẹp quá!");

// Đọc file không cần chỉ định encoding
String content = Files.readString(filePath);
System.out.println(content); 
```

- **Simple Web Server (Incubator)**: Cung cấp một máy chủ web đơn giản phục vụ cho việc phát triển và thử nghiệm.
  
```java
// Khởi động web server đơn giản
var server = SimpleFileServer.createFileServer(
    new InetSocketAddress(8080), 
    Path.of("."), 
    SimpleFileServer.OutputLevel.VERBOSE
);
server.start();

// Truy cập http://localhost:8080 để xem nội dung thư mục
```

- **Code Snippets trong JavaDoc**:

```java
/**
 * Tính tổng hai số
 * {@snippet :
 * int a = 5;
 * int b = 10;
 * int sum = a + b;  // Kết quả là 15
 * }
 */
public int add(int a, int b) {
    return a + b;
}
```

> Web Server chỉ dành cho mục đích phát triển và demo
{: .prompt-warning }

## Java 19 (2022)

- **Virtual Threads (Preview)**: Cho phép tạo hàng triệu luồng ảo, giúp xử lý đa luồng hiệu quả hơn.

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10_000).forEach(i -> {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));
            System.out.println("Xử lý task " + i);
            return i;
        });
    });
}
```

- **Structured Concurrency (Incubator)**: Cải tiến cách quản lý các tác vụ bất đồng bộ.

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> userFuture = scope.fork(() -> fetchUser());
    Future<String> orderFuture = scope.fork(() -> fetchOrders());
    
    scope.join();
    System.out.println("Kết quả: " + userFuture.get() + " - " + orderFuture.get());
}
```

## Java 20 (2023)

- **Scoped Values (Incubator)**: Cơ chế mới để chia sẻ dữ liệu an toàn giữa các thread con trong cùng phạm vi.

```java
final ScopedValue<Connection> DB_CONNECTION = ScopedValue.newInstance();

void handleRequest() {
    Connection conn = getDatabaseConnection();
    ScopedValue.where(DB_CONNECTION, conn)
               .run(() -> {
                   processData();
                   generateReport();
               });
}

void processData() {
    Connection currentConn = DB_CONNECTION.get();
    // Sử dụng connection đã được bind
}
```

- **Record Patterns (Second Preview)**: Phân tích cấu trúc record lồng nhau.

```java
record Coordinate(double lat, double lng) {}
record Location(String name, Coordinate coord) {}

void printGPS(Object obj) {
    if (obj instanceof Location(var name, Coordinate(var lat, var lng))) {
        System.out.printf("%s: %.4f, %.4f%n", name, lat, lng);
    }
}
```

- **Virtual Threads Improvements**:

```java
// Tạo virtual thread với timeout
Thread virtualThread = Thread.ofVirtual()
    .name("data-processor")
    .start(() -> {
        try {
            TimeUnit.SECONDS.sleep(5);
            System.out.println("Xử lý hoàn tất");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    });

virtualThread.join(Duration.ofSeconds(10));
```

> Record Patterns giúp xử lý dữ liệu phức tạp dễ dàng hơn với pattern matching
{: .prompt-tip }

## Java 21 (2023) - Phiên bản LTS

> Java 21 là phiên bản LTS với nhiều cải tiến đáng chú ý về hiệu năng và cú pháp
{: .prompt-info }

- **Virtual Threads** (Chính thức): Cho phép tạo hàng triệu luồng ảo với chi phí thấp.

```java
// Tạo và quản lý virtual threads
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10_000; i++) {
        executor.submit(() -> {
            Thread.sleep(Duration.ofMillis(100));
            return processRequest(i);
        });
    }
}

// Tạo virtual thread đơn lẻ
Thread.ofVirtual()
    .name("data-loader")
    .start(() -> loadDataFromFile("data.csv"));
```

- **String Templates (Preview)**: Cú pháp mới cho phép nhúng biến vào chuỗi an toàn.

```java
String user = "Huy";
LocalDateTime now = LocalDateTime.now();
String message = STR."""
    Xin chào \{user},
    Thời gian hiện tại: \{now.format(DateTimeFormatter.ISO_DATE_TIME)}
    Số lượng: \{Math.random() * 100:%.2f}
    """;
```

- **Pattern Matching for switch** (Chính thức):

```java
String format(Object o) {
    return switch (o) {
        case null -> "null";
        case Integer i -> "int: " + i;
        case Long l -> "long: " + l;
        case String s -> "String: " + s;
        default -> "Unknown";
    };
}
```

- **Sequenced Collections**:

```java
// Các phương thức mới cho collections
List<Integer> list = new ArrayList<>(List.of(3, 1, 4));
list.addFirst(0);  // [0, 3, 1, 4]
list.addLast(5);   // [0, 3, 1, 4, 5]

int first = list.getFirst(); // 0
int last = list.getLast();   // 5
```

> Virtual Threads giúp xử lý đồng thời hiệu quả cho các ứng dụng I/O intensive
{: .prompt-tip }

## Java 22 (2024)

> **Java 22** mang đến các cải tiến tập trung vào hiệu năng, bảo mật và trải nghiệm developer
{: .prompt-info }

- **Scoped Values (Incubator)**: Quản lý dữ liệu an toàn giữa các luồng

```java
final ScopedValue<String> CURRENT_USER = ScopedValue.newInstance();

void processRequest() {
    ScopedValue.where(CURRENT_USER, "Nguyễn Văn A")
               .run(() -> {
                   System.out.println("User: " + CURRENT_USER.get());
                   // Xử lý logic nghiệp vụ
               });
}
```

- **Stream Gatherers (Preview)**: Thêm bộ thu thập tùy biến cho Stream

```java
List<String> filtered = Stream.of("a", "b", null, "c")
    .gather(Gatherers.filter(Objects::nonNull))
    .toList(); // [a, b, c]
```

- **Structured Concurrency (Incubator)**: Quản lý tác vụ đồng thời

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user = scope.fork(this::fetchUser);
    Future<String> order = scope.fork(this::fetchOrder);
    
    scope.join();
    System.out.println(user.resultNow() + " - " + order.resultNow());
}
```

- **Statements Before Super**: Khởi tạo trước khi gọi constructor cha

```java
public class Circle extends Shape {
    private final double radius;
    
    public Circle(double radius) {
        this.radius = radius; // Khởi tạo trước super()
        super("Circle");
    }
}
```

- **Class-File API (Preview)**: Thao tác với file .class

```java
ClassFile cf = ClassFile.of();
ClassModel model = cf.parse(Paths.get("MyClass.class"));
model.methods().forEach(m -> System.out.println(m.methodName()));
```

- **Region Pinning cho G1 GC**: Tối ưu thu gom rác

```bash
# Kích hoạt tính năng
java -XX:+UseG1GC -XX:+G1RegionPinning MyApp
```

- **String Templates (Second Preview)**: Chuỗi template nâng cao

```java
String name = "Huy";
int age = 25;
String info = STR."Tên: \{name}, Tuổi: \{age}";
```

- **Unnamed Variables & Patterns**: Bỏ qua biến không dùng

```java
// Bỏ qua exception trong try-with-resources
try (var _ = new FileInputStream("data.txt")) {
    // Xử lý file
}

// Pattern matching không cần tên biến
if (obj instanceof Point(int x, _)) {
    System.out.println("x = " + x);
}
```

> Các tính năng incubator/preview cần kích hoạt bằng flag `--enable-preview --add-modules jdk.incubator.concurrent`
{: .prompt-warning }

## Java 23 (2024)

> **Java 23** mang đến những cải tiến đột phá về hiệu năng và trải nghiệm lập trình
{: .prompt-info }

- **Generational ZGC**: Nâng cấp trình dọn rác ZGC với cơ chế phân tách theo thế hệ

```bash
# Kích hoạt Generational ZGC
java -XX:+UseZGC -Xmx16g -XX:+ZGenerational MyApp
```

- **Pattern Matching mở rộng**: Hỗ trợ pattern matching cho kiểu nguyên thủy và mảng

```java
// Kiểm tra mảng số nguyên
if (obj instanceof int[] arr && arr.length > 0) {
    System.out.println("Phần tử đầu: " + arr[0]);
}
```

- **Stream Gatherers (Preview)**: Thêm bộ thu thập dữ liệu tùy biến cho Stream API

```java
List<String> result = Stream.of("a", "b", "c", "d")
    .gather(s -> s.map(String::toUpperCase))
    .toList(); // [A, B, C, D]
```

- **Class-File API (Preview)**: API mới cho thao tác với file class

```java
ClassFile classFile = ClassFile.read(Paths.get("MyClass.class"));
classFile.methods().forEach(m -> System.out.println(m.name()));
```

- **Vector API (Incubator)**: Tối ưu hóa tính toán vector cho CPU hiện đại

```java
FloatVector vectorA = FloatVector.fromArray(SPECIES, arr, 0);
FloatVector result = vectorA.mul(2).add(vectorB);
```

- **JavaDoc Markdown**: Viết tài liệu bằng cú pháp Markdown

```java
/**
 * # Ví dụ tính tổng
 * 
 * Phương thức này thực hiện **phép cộng** hai số nguyên
 * 
 * ```java
 * int sum = add(2, 3); // = 5
 * ```
 */
public int add(int a, int b) { return a + b; }
```

> Các tính năng preview cần kích hoạt bằng flag `--enable-preview`
{: .prompt-warning }

## Code mẫu trong Java

Dưới đây là một ví dụ minh họa cách sử dụng một số tính năng mới của Java như `var`, `record` và pattern matching cho `instanceof`:

```java
public class Main {
    public static void main(String[] args) {
        // Sử dụng 'var' để khai báo biến (Java 10+)
        var person = new Person("Alice", 30);
        System.out.println("Hello, " + person.name());
        
        // Pattern matching với 'instanceof' (Java 16+)
        Object obj = person;
        if (obj instanceof Person p) {
            System.out.println("Age: " + p.age());
        }
    }
    
    // Định nghĩa một record để lưu trữ thông tin cá nhân (Java 14 Preview, chính thức từ Java 16)
    public record Person(String name, int age) {}
}
```

## Kết luận
Qua hành trình phát triển từ Java 8 đến Java 23, chúng ta thấy rằng ngôn ngữ Java không ngừng cải tiến để đáp ứng nhu cầu của lập trình viên và xu hướng công nghệ hiện đại. Các tính năng mới như var, records, pattern matching, virtual threads, và các cải tiến về bộ thu gom rác, mô-đun hóa cũng như bảo mật đã làm cho Java trở nên linh hoạt và mạnh mẽ hơn rất nhiều.

Việc nâng cấp và làm quen với những tính năng mới này sẽ giúp bạn viết code ngắn gọn, dễ đọc và dễ bảo trì hơn, cũng như tận dụng tối đa hiệu năng của ứng dụng. Hy vọng bài viết sẽ cung cấp cho bạn cái nhìn tổng quan về các phiên bản Java và những cải tiến quan trọng qua từng giai đoạn phát triển.
