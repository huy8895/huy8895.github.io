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

- **Sealed Classes (Preview)**: Cho phép giới hạn các lớp có thể kế thừa, tăng khả năng kiểm soát cấu trúc hệ thống.
- **Text Blocks**: Chính thức được đưa vào sử dụng.
- **Hidden Classes**: Hỗ trợ các lớp ẩn cho các framework, giúp tăng cường bảo mật và tối ưu hóa reflection.

## Java 16 (2021)

- **Pattern Matching cho `instanceof`** trở nên chính thức, giúp viết điều kiện kiểm tra kiểu ngắn gọn hơn.
- **Records**: Chính thức trở thành tính năng của ngôn ngữ.
- **Vector API (Incubator)**: Hỗ trợ tối ưu hóa tính toán dựa trên phần cứng SIMD.

## Java 17 (2021) - Phiên bản LTS

> Nâng cấp lên Java 17 để nhận hỗ trợ dài hạn và các cải tiến hiệu năng
{: .prompt-important }

- **Sealed Classes** được đưa vào chính thức.
- **Foreign Function & Memory API (Incubator)**: Cho phép gọi hàm bên ngoài JVM và thao tác với bộ nhớ ngoài heap.
- Cải tiến bảo mật với **Context-Specific Deserialization Filters**.

## Java 18 (2022)

- **UTF-8** trở thành bộ ký tự mặc định cho JVM.
- **Simple Web Server (Incubator)**: Cung cấp một máy chủ web đơn giản phục vụ cho việc phát triển và thử nghiệm.

## Java 19 (2022)

- **Virtual Threads (Preview)**: Cho phép tạo hàng triệu luồng ảo, giúp xử lý đa luồng hiệu quả hơn.
- **Structured Concurrency (Incubator)**: Cải tiến cách quản lý các tác vụ bất đồng bộ.
- **Foreign Function & Memory API** được cải tiến thêm qua phiên bản incubator thứ hai.

## Java 20 (2023)

- **Scoped Values (Incubator)**: Hỗ trợ quản lý giá trị theo phạm vi trong xử lý đa luồng.
- **Virtual Threads (Second Preview)**: Tiếp tục hoàn thiện tính năng luồng ảo.
- **Record Patterns (Second Preview)**: Mở rộng khả năng pattern matching, đặc biệt khi làm việc với record.

## Java 21 (2023) - Phiên bản LTS

- Hoàn thiện các tính năng mới như virtual threads, pattern matching và structured concurrency.
- **String Templates (Preview)**: Cung cấp cú pháp mới để xử lý chuỗi động hiệu quả hơn.

## Java 22 (2024)

- **Generational ZGC**: Cải thiện hiệu suất thu gom rác nhờ việc tối ưu hóa theo thế hệ.
- **Pattern Matching Enhancements**: Mở rộng khả năng của pattern matching cho các kiểu dữ liệu phức tạp và tùy biến.

## Java 23 (Dự kiến 2026)

- **API cho AI và Machine Learning**: Hỗ trợ tích hợp các thư viện trí tuệ nhân tạo và học máy.
- **Cải tiến Module System**: Tối ưu hóa việc quản lý mô-đun và triển khai ứng dụng.
- **Reactive Programming**: Hỗ trợ lập trình bất đồng bộ mạnh mẽ hơn cho các ứng dụng hiện đại.

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
