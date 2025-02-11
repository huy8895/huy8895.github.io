---
title: "Tận Dụng Tối Đa Lambdas, Functional Interface và Streams API trong Java 8"
description: "Cách sử dụng hiệu quả các tính năng mới trong Java 8 bao gồm lambdas, functional interface và Streams API."
author: huy8895
date: 2025-02-11 08:00:00 +0700
categories: [Effective Java]
tags: [java, "java 8", best-practices]
pin: false
math: false
mermaid: true
---

Java 8 đã mang đến những cải tiến lớn trong lập trình với các tính năng như lambdas, functional interface và Streams API. Bài viết này sẽ giúp bạn tận dụng tối đa các tính năng này thông qua những nguyên tắc và lời khuyên thực tế.

## Ưu tiên sử dụng lambdas thay vì anonymous classes

Trước Java 8, các interface với một phương thức trừu tượng (SAM) thường được triển khai thông qua anonymous classes. Cách tiếp cận này tạo ra nhiều boilerplate code và khó đọc. Lambda expressions ra đời giúp giải quyết những vấn đề này bằng cú pháp ngắn gọn và biểu cảm hơn.

Ví dụ so sánh khi sắp xếp danh sách:
```java
// Anonymous class (cũ)
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});

// Lambda expression (Java 8+)
Collections.sort(words, 
    (s1, s2) -> Integer.compare(s1.length(), s2.length()));

// Method reference (cách viết tối ưu)
words.sort(comparingInt(String::length));
```

Ưu điểm chính của lambdas:
- Tự động suy luận kiểu dữ liệu (type inference)
- Giảm 90% boilerplate code
- Dễ dàng kết hợp với method references
- Hỗ trợ lập trình hàm hiệu quả

**Trường hợp vẫn cần dùng anonymous classes:**
1. Khi làm việc với abstract class
2. Interface có nhiều phương thức trừu tượng
3. Cần truy cập instance của chính đối tượng (this)
4. Yêu cầu serialization (lambda và anonymous class đều hạn chế serialization)

**Lưu ý quan trọng:**
- Giữ lambda ngắn gọn (tối ưu trong 1-3 dòng)
- Sử dụng generic types đầy đủ để hỗ trợ type inference
- Với các thao tác phức tạp, vẫn ưu tiên dùng class truyền thống

**Ứng dụng lambda trong enum:**
Lambda cho phép đơn giản hóa các enum có hành vi đặc thù. So sánh 2 cách triển khai enum Operation:

```java
// Cách cũ dùng class body riêng
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    // ... các hằng số khác ...
}

// Cách mới dùng lambda
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

Ưu điểm khi dùng lambda với enum:
- Giảm 70% số dòng code
- Tập trung logic vào nơi khai báo hằng số
- Dễ dàng thêm/hủy các phép toán mới

**Giới hạn khi dùng lambda trong enum:**
```java
// Không thể truy cập instance members từ lambda trong constructor
Operation(String symbol, DoubleBinaryOperator op) {
    this.symbol = symbol;
    this.op = x -> x * this.symbol.length(); // LỖI COMPILE
}
```

## Ưu tiên sử dụng method references thay cho lambdas

Trong nhiều trường hợp, method references có thể thay thế lambdas để mã rõ ràng hơn. Nếu lambda chỉ gọi một phương thức có sẵn, bạn nên dùng method reference thay vì lambda để thể hiện ý định của mã một cách rõ ràng hơn.

Ví dụ:

```java
// Lambda
list.forEach(item -> System.out.println(item));

// Method reference
list.forEach(System.out::println);
```

Method references giúp mã sạch hơn, dễ đọc và giúp người khác nhanh chóng hiểu rõ bạn đang gọi đến phương thức nào.

## Ưu tiên sử dụng functional interface chuẩn

Java 8 cung cấp một số functional interface chuẩn như `Predicate`, `Function`, `Supplier`, và `Consumer`. Hạn chế việc tạo các functional interface tùy chỉnh khi có thể sử dụng các interface này. Điều này không chỉ giúp mã dễ hiểu mà còn tăng tính tương thích với các API khác của Java.

Ví dụ:

```java
Function<String, Integer> stringToLength = String::length;
Predicate<String> isEmpty = String::isEmpty;
```

## Sử dụng Streams API một cách hợp lý

Streams API cung cấp một cách mạnh mẽ để xử lý các tập hợp dữ liệu. Tuy nhiên, sử dụng streams quá mức có thể gây khó hiểu và phức tạp không cần thiết. Bạn nên sử dụng streams khi chúng giúp mã ngắn gọn và dễ đọc hơn, nhưng không nên lạm dụng chúng cho các trường hợp đơn giản mà một vòng lặp for thông thường sẽ dễ hiểu hơn.

Ví dụ:

```java
// Sử dụng hợp lý
List<String> filteredList = list.stream()
                                .filter(item -> !item.isEmpty())
                                .collect(Collectors.toList());

// Không cần thiết
for (String item : list) {
    if (!item.isEmpty()) {
        filteredList.add(item);
    }
}
```

## Ưu tiên các hàm không gây tác dụng phụ trong streams

Khi làm việc với streams, các hàm bạn sử dụng trong các thao tác như `map`, `filter` hay `forEach` nên tránh gây tác dụng phụ (side effects). Một hàm có tác dụng phụ sẽ thay đổi trạng thái bên ngoài hàm, điều này làm giảm tính dễ hiểu và khó kiểm soát hơn.

Hãy cố gắng giữ các hàm trong streams "trong sáng" (pure functions), chỉ nhận đầu vào và trả về kết quả mà không thay đổi trạng thái bên ngoài.

## Ưu tiên trả về Collection thay vì Stream

Streams là một công cụ mạnh mẽ, nhưng khi trả về dữ liệu từ một phương thức, bạn nên ưu tiên trả về `Collection` thay vì `Stream`. Điều này giúp người dùng phương thức có thể thao tác với dữ liệu trả về dễ dàng hơn, đặc biệt là khi họ không quen thuộc với Streams API.

Ví dụ:

```java
// Trả về List thay vì Stream
public List<String> getNames() {
    return people.stream()
                 .map(Person::getName)
                 .collect(Collectors.toList());
}
```

## Cẩn trọng khi sử dụng streams song song

Java cung cấp khả năng chạy streams song song (`parallelStream`) để tăng hiệu suất trong các tác vụ đa luồng. Tuy nhiên, việc sử dụng streams song song cần thận trọng vì nó có thể dẫn đến các vấn đề như điều kiện tranh chấp (race condition), và không phải lúc nào nó cũng mang lại hiệu suất tốt hơn. Chỉ nên sử dụng `parallelStream` khi bạn chắc chắn rằng tác vụ đó có thể tận dụng được lợi thế của việc xử lý song song.

Ví dụ:

```java
// Sử dụng parallelStream
list.parallelStream()
    .forEach(item -> processItem(item));
```

### Kết luận

Java 8 mang đến nhiều tính năng mới để cải thiện hiệu suất và độ dễ đọc của mã, nhưng việc sử dụng chúng một cách hiệu quả yêu cầu bạn phải hiểu rõ về lambdas, functional interface và Streams API. Bằng cách tuân theo các nguyên tắc trên, bạn có thể tận dụng tối đa những lợi ích mà chúng mang lại.