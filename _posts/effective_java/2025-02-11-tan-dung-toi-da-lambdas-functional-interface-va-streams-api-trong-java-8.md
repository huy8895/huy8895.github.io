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

## 1. Ưu tiên sử dụng lambdas thay vì anonymous classes

Trước Java 8, các interface với một phương thức trừu tượng (SAM) thường được triển khai thông qua anonymous classes. Cách tiếp cận này tạo ra nhiều boilerplate code và khó đọc. Lambda expressions ra đời giúp giải quyết những vấn đề này bằng cú pháp ngắn gọn và biểu cảm hơn.
### 1.1 Ví dụ so sánh

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

### 1.2 Ưu điểm chính

Ưu điểm chính của lambdas:
- Tự động suy luận kiểu dữ liệu (type inference)
- Giảm 90% boilerplate code
- Dễ dàng kết hợp với method references
- Hỗ trợ lập trình hàm hiệu quả

### 1.3 Trường hợp vẫn cần anonymous classes

**Trường hợp vẫn cần dùng anonymous classes:**
1. Khi làm việc với abstract class
2. Interface có nhiều phương thức trừu tượng
3. Cần truy cập instance của chính đối tượng (this)
4. Yêu cầu serialization (lambda và anonymous class đều hạn chế serialization)

### 1.4 Lưu ý quan trọng

**Lưu ý quan trọng:**
- Giữ lambda ngắn gọn (tối ưu trong 1-3 dòng)
- Sử dụng generic types đầy đủ để hỗ trợ type inference
- Với các thao tác phức tạp, vẫn ưu tiên dùng class truyền thống

### 1.5 Ứng dụng lambda trong enum

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

### 1.6 Giới hạn khi dùng lambda trong enum

```java
// Không thể truy cập instance members từ lambda trong constructor
Operation(String symbol, DoubleBinaryOperator op) {
    this.symbol = symbol;
    this.op = x -> x * this.symbol.length(); // LỖI COMPILE
}
```

## 2. Ưu tiên sử dụng method references thay cho lambdas

Khi lambda chỉ gọi một phương thức đã tồn tại, method reference giúp mã nguồn súc tích và biểu đạt rõ ý định hơn. Cú pháp `Class::method` loại bỏ boilerplate code trong khi vẫn giữ nguyên chức năng.

**Ví dụ điển hình với Map.merge:**
```java
// Lambda version
map.merge(key, 1, (count, incr) -> count + incr);

// Method reference version (ưu tiên)
map.merge(key, 1, Integer::sum);
```

**Phân loại method reference:**
Ưu tiên method references khi ngắn gọn và rõ ràng, nhưng vẫn sử dụng lambda khi cần thiết. Phần tóm tắt bảng so sánh sẽ củng cố điều này, giúp bạn có cái nhìn tổng quan và đưa ra quyết định phù hợp.

| Loại                  | Ví dụ                   | Tương đương lambda         | Ứng dụng điển hình       |
|-----------------------|-------------------------|----------------------------|--------------------------|
| Static method         | `Integer::parseInt`    | `str -> Integer.parseInt(str)` | Xử lý chuyển đổi kiểu dữ liệu |
| Bound instance method | `Instant.now()::isAfter` | `t -> Instant.now().isAfter(t)` | Kiểm tra trạng thái đối tượng cụ thể |
| Unbound instance method | `String::toLowerCase`  | `str -> str.toLowerCase()`   | Xử lý chuỗi trong stream |
| Class constructor     | `TreeMap::new`          | `() -> new TreeMap<>()`       | Tạo đối tượng factory    |
| Array constructor     | `int[]::new`            | `len -> new int[len]`         | Tạo mảng động            |

**Giải thích chi tiết từng loại:**

1. **Static method reference**  
   Dùng khi gọi phương thức static của class  
   ```java
   List<Integer> numbers = Arrays.asList("1", "2", "3").stream()
       .map(Integer::parseInt)  // Tương đương s -> Integer.parseInt(s)
       .collect(Collectors.toList());
   ```

2. **Bound instance method reference**  
   Đối tượng được xác định trước khi gọi phương thức  
   ```java
   Instant cutoff = Instant.now();
   List<Instant> pastEvents = events.stream()
       .filter(cutoff::isAfter)  // event -> cutoff.isAfter(event)
       .collect(Collectors.toList());
   ```

3. **Unbound instance method reference**  
   Đối tượng được truyền như tham số đầu tiên  
   ```java
   List<String> upperNames = names.stream()
       .map(String::toUpperCase)  // name -> name.toUpperCase()
       .collect(Collectors.toList());
   ```

4. **Class constructor reference**  
   Thay thế Supplier để tạo đối tượng  
   ```java
   Supplier<List<String>> listMaker = ArrayList::new;  // () -> new ArrayList<>()
   List<String> tempList = listMaker.get();
   ```

5. **Array constructor reference**  
   Tạo mảng với kích thước động  
   ```java
   IntFunction<int[]> arrayCreator = int[]::new;  // size -> new int[size]
   int[] numbers = arrayCreator.apply(5);
   ```

**Nguyên tắc áp dụng:**
- Ưu tiên method reference khi làm mã sạch và dễ hiểu hơn
- Vẫn dùng lambda nếu:
    - Cần mô tả logic phức tạp
    - Tham số lambda cung cấp tên biến có ý nghĩa
    - Method reference cùng class dài dòng (`GoshThisClassNameIsHumongous::action` → `() -> action()`)
  
**Trường hợp đặc biệt:**
```java
// Tránh dùng Function.identity()
Stream.of(values).map(x -> x)... 

// Thay vì
Stream.of(values).map(Function.identity())...
```

**Lưu ý khi refactor:**
- IDE thường gợi ý chuyển lambda → method reference, nhưng cần kiểm tra tính rõ ràng
- Với logic phức tạp, tách thành phương thức riêng và dùng method reference
- Ưu tiên cách viết nào tự nhiên và dễ đọc hơn
- Method reference chỉ thực sự hữu ích khi làm code sáng nghĩa
- Khi nghi ngờ, hãy viết cả 2 cách và so sánh độ rõ ràng

## 3. Ưu tiên sử dụng functional interface chuẩn

Khi thiết kế API nhận function object, hãy ưu tiên các interface có sẵn trong `java.util.function`. Ví dụ điển hình với LinkedHashMap:

```java
// Cách cũ: Override method
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > 100;
}

// Cách mới: Sử dụng BiPredicate
BiPredicate<Map<K,V>, Map.Entry<K,V>> removalPredicate = (map, entry) -> map.size() > 100;
```

**Lý do nên dùng interface chuẩn:**
1. Giảm số lượng interface cần học
2. Tận dụng các phương thức mặc định (ví dụ: `and()`, `or()` trong Predicate)
3. Dễ dàng kết hợp với các API khác

**6 functional interface cốt lõi:**

| Interface          | Phương thức     | Ví dụ                  | Ứng dụng điển hình     |
|--------------------|-----------------|------------------------|------------------------|
| `Predicate<T>`     | test(T) → bool  | Collection::isEmpty    | Lọc phần tử trong stream |
| `Function<T,R>`    | apply(T) → R    | String::length         | Chuyển đổi dữ liệu     |
| `Consumer<T>`      | accept(T) → void| System.out::println    | Thao tác phụ          |
| `Supplier<T>`      | get() → T       | Instant::now           | Khởi tạo lazy         |
| `UnaryOperator<T>` | apply(T) → T    | String::toLowerCase    | Thao tác trên 1 toán hạng |
| `BinaryOperator<T>`| apply(T,T) → T  | BigInteger::add        | Thao tác trên 2 toán hạng |

**Biến thể cho primitive types:**
- Tránh boxing/unboxing với `IntPredicate`, `LongConsumer`,...
- Ví dụ hiệu năng cao:
```java
IntPredicate evenNumber = n -> n % 2 == 0; // Tốt hơn Predicate<Integer>
DoubleFunction<String> converter = d -> String.format("%.2f", d); 
```

**Lưu ý khi thiết kế API:**
- Đừng tự tạo interface mới nếu đã có sẵn
- Với tham số double/ int/ long, ưu tiên dùng primitive variants
- Khi cần 2 tham số, dùng `BiPredicate`, `BiFunction` thay vì tự định nghĩa

**Khi nào cần tự tạo functional interface?**
1. Khi cần đặt tên có ý nghĩa nghiệp vụ (ví dụ: `Comparator` thay vì `ToIntBiFunction`)
2. Khi cần thêm các phương thức mặc định đặc thù
3. Khi có các ràng buộc contract nghiêm ngặt
4. Khi cần xử lý exception checked
5. Khi cần tham số đặc biệt (ví dụ: 3 tham số)

**Ví dụ interface tùy chỉnh:**
```java
@FunctionalInterface
public interface TriPredicate<T, U, V> {
    boolean test(T t, U u, V v);
    
    default TriPredicate<T, U, V> and(TriPredicate<? super T, ? super U, ? super V> other) {
        return (t, u, v) -> test(t, u, v) && other.test(t, u, v);
    }
}
```

**Nguyên tắc thiết kế:**
- Luôn đánh dấu bằng `@FunctionalInterface`
- Tránh overload method nhận các functional interface khác nhau ở cùng vị trí tham số
- Ưu tiên primitive functional interfaces (`IntPredicate`) thay vì dùng boxed types (`Predicate<Integer>`)
- Đặt tên theo mẫu `<Hậu tố>Function` (ví dụ: `ToIntBiFunction`)

**Cảnh báo hiệu năng:**
```java
// Tránh dùng
Predicate<Integer> evenPredicate = n -> n % 2 == 0; // Boxing

// Nên dùng
IntPredicate evenIntPredicate = n -> n % 2 == 0; // Không boxing
```

**Xử lý exception:**
```java
@FunctionalInterface
public interface ThrowingFunction<T, R> {
    R apply(T t) throws Exception;
    
    static <T, R> Function<T, R> wrap(ThrowingFunction<T, R> fn) {
        return t -> {
            try {
                return fn.apply(t);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        };
    }
}
```

## 4. Sử dụng Streams API một cách hợp lý

Streams API cung cấp một cách mạnh mẽ để xử lý các tập hợp dữ liệu. Tuy nhiên, sử dụng streams quá mức có thể gây khó hiểu và phức tạp không cần thiết. Bạn nên sử dụng streams khi chúng giúp mã ngắn gọn và dễ đọc hơn, nhưng không nên lạm dụng chúng cho các trường hợp đơn giản mà một vòng lặp for thông thường sẽ dễ hiểu hơn.

**Khi nào nên dùng Streams:**
- Xử lý chuỗi phép biến đổi dữ liệu tuần tự (map, filter, reduce)
- Thao tác trên tập dữ liệu lớn hoặc vô hạn
- Tận dụng lazy evaluation và parallel processing
- Kết hợp các thao tác phức tạp thành pipeline rõ ràng

**Ví dụ điển hình:**
```java
// Tìm 20 số nguyên tố Mersenne đầu tiên
primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
        .filter(mersenne -> mersenne.isProbablePrime(50))
        .limit(20)
        .forEach(System.out::println);
```

**Khi nên tránh Streams:**
- Cần truy cập nhiều biến trung gian trong pipeline
- Xử lý char/byte (do Java không hỗ trợ CharStream)
- Cần sử dụng break/continue/return trong logic
- Thao tác I/O phức tạp hoặc xử lý exception checked

**Nguyên tắc vàng:**
1. Giữ stream pipeline ngắn gọn và tập trung
2. Tách logic phức tạp thành các helper method
3. Đặt tên biến stream có ý nghĩa (ví dụ: `words`, `primes`)
4. Kết hợp với code truyền thống khi cần thiết
5. Tránh nested stream (flatMap chỉ khi thực sự cần)

**Ví dụ kết hợp stream và code thường:**
```java
// Đếm các nhóm anagram
try (Stream<String> words = Files.lines(dictionary)) {
    words.collect(groupingBy(word -> alphabetize(word)))
         .values().stream()
         .filter(group -> group.size() >= minGroupSize)
         .forEach(g -> System.out.println(g.size() + ": " + g));
}
```

**Cảnh báo hiệu năng:**
- Parallel stream chỉ hiệu quả với dữ liệu lớn (hàng triệu phần tử)
- Tránh dùng boxed types trong stream (ưu tiên IntStream, LongStream)
- Đo lường hiệu năng trước khi tối ưu

**Xử lý Cartesian Product:**
```java
// Cách tiếp cận truyền thống
List<Card> deck = new ArrayList<>();
for (Suit suit : Suit.values()) 
    for (Rank rank : Rank.values()) 
        deck.add(new Card(suit, rank));

// Cách dùng stream (chỉ khi team đã quen)
List<Card> deck = Stream.of(Suit.values())
                        .flatMap(suit -> Stream.of(Rank.values())
                                              .map(rank -> new Card(suit, rank)))
                        .collect(toList());
```

**Best Practices:**
- Ưu tiên readability hơn cleverness
- Comment giải thích các thao tác phức tạp
- Refactor code thường xuyên để cân bằng giữa stream và loop
- Test cả hai phiên bản stream và non-stream để so sánh

## 5. Ưu tiên các hàm không gây tác dụng phụ trong streams

**Nguyên tắc vàng:** Stream pipelines nên được xây dựng bằng các hàm thuần khiết (pure functions) - không phụ thuộc vào trạng thái bên ngoài và không thay đổi trạng thái hệ thống.

### Ví dụ điển hình về anti-pattern
```java
// Cách tiếp cận sai: Sử dụng forEach để thay đổi state bên ngoài
Map<String, Long> freq = new HashMap<>();
words.forEach(word -> {
    freq.merge(word.toLowerCase(), 1L, Long::sum); // Side-effect!
});
```

**Vấn đề:** 
- Khó parallel hóa 
- Code khó theo dõi
- Vi phạm nguyên tắc immutable trong stream

### Cách tiếp cận đúng với Collector
```java
// Sử dụng groupingBy + counting collector
Map<String, Long> freq = words.collect(
    groupingBy(String::toLowerCase, counting())
);
```

**Lợi ích:**
- Tự động xử lý song song
- Code ngắn gọn và biểu đạt rõ ý định
- Dễ dàng tối ưu hiệu năng

### 5 Collector quan trọng cần nắm:
1. **toList()/toSet()** - Thu thập phần tử vào collection
```java
List<String> topNames = filteredStream.collect(toList());
```

2. **toMap()** - Tạo map từ stream
```java
Map<String, Employee> employeeMap = employees.collect(
    toMap(Employee::getId, Function.identity())
);
```

3. **groupingBy()** - Nhóm phần tử theo key
```java
Map<Department, List<Employee>> byDept = employees.collect(
    groupingBy(Employee::getDepartment)
);
```

4. **joining()** - Nối chuỗi
```java
String csv = strings.collect(joining(", "));
```

5. **filtering/mapping** - Xử lý downstream
```java
Map<City, Set<String>> namesByCity = people.collect(
    groupingBy(Person::getCity, 
        mapping(Person::getName, toSet()))
);
```

### Xử lý collision trong toMap
```java
// Merge function khi key trùng
Map<Artist, Album> topHits = albums.collect(
    toMap(Album::artist, a -> a, 
        (existing, replacement) -> existing.sales() > replacement.sales() 
            ? existing 
            : replacement)
);
```

### Khi nào được phép dùng forEach?
- Chỉ để consume kết quả cuối cùng
- Không thực hiện tính toán/phụ thuộc vào thứ tự

```java
// Đúng: In kết quả sau khi xử lý
resultStream.forEach(System.out::println);

// Sai: Thực hiện logic nghiệp vụ trong forEach
orders.forEach(order -> processPayment(order)); 
```

**Best Practices:**
- Luôn ưu tiên dùng Collector thay vì thao tác thủ công
- Tách logic phức tạp thành các downstream collector
- Sử dụng static import Collectors.* để code sạch hơn
- Kết hợp với Comparator và Function khi cần thiết

## 6. Ưu tiên trả về Collection thay vì Stream

**Nguyên tắc cốt lõi:** Khi thiết kế API trả về chuỗi phần tử, cần hỗ trợ cả 2 trường hợp sử dụng: xử lý stream và vòng lặp for-each.

### Ví dụ anti-pattern
```java
// API chỉ trả về Stream
public Stream<ProcessHandle> getAllProcesses() {
    return ProcessHandle.allProcesses();
}

// Client code phải dùng adapter
for (ProcessHandle ph : (Iterable<ProcessHandle>) 
    ProcessHandle.allProcesses()::iterator) { // Code rườm rà
    // Xử lý
}
```

**Vấn đề:**
- Client code phải tự xử lý chuyển đổi
- Giảm hiệu năng khi ép kiểu
- Khó kết hợp với code cũ

### Pattern đúng: Trả về Collection
```java
public Collection<ProcessHandle> getAllProcesses() {
    return new ArrayList<>(ProcessHandle.allProcesses().collect(toList()));
}

// Client code đơn giản
for (ProcessHandle ph : getAllProcesses()) {
    // Xử lý
}
```

**Lợi ích:**
- Tương thích ngược với code cũ
- Cho phép sử dụng cả stream và for-each
- Dễ dàng mở rộng

### Các trường hợp đặc biệt
**1. Dữ liệu lớn không lưu trữ được:**
```java
// Trả về Stream khi dữ liệu quá lớn
public Stream<LogEntry> readLogEntries(Path file) throws IOException {
    return Files.lines(file).map(LogEntry::parse);
}
```

**2. Custom Collection cho dữ liệu đặc biệt:**
```java
// PowerSet implementation
public class PowerSet {
    public static <E> Collection<Set<E>> of(Set<E> s) {
        return new AbstractList<Set<E>>() {
            @Override public int size() { return 1 << src.size(); }
            @Override public Set<E> get(int index) { 
                // Tạo subset từ index dạng bit vector
            }
        };
    }
}
```

### Best Practices

| Trường hợp               | Kiểu trả về ưu tiên  | Lý do                     |
|--------------------------|----------------------|---------------------------|
| Dữ liệu nhỏ               | `Collection`         | Tương thích cả 2 cách dùng |
| Dữ liệu lớn/không lưu trữ | `Stream`             | Tiết kiệm bộ nhớ          |
| Cấu trúc dữ liệu đặc biệt | Custom `Collection`  | Tối ưu hiệu năng          |

**Adapter pattern khi cần:**
```java
// Chuyển Iterable sang Stream
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}

// Chuyển Stream sang Iterable
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}
```

**Ví dụ xử lý sublist:**

```java
public class SubLists {
    public static <E> Stream<List<E>> of(List<E> list) {
        return IntStream.range(0, list.size())
            .mapToObj(start -> 
                IntStream.rangeClosed(start + 1, list.size())
                    .mapToObj(end -> list.subList(start, end)))
            .flatMap(x -> x);
    }
}
// Sử dụng: SubLists.of(list).forEach(System.out::println);
```

**Lưu ý hiệu năng:**
- Stream-to-Iterable adapter giảm tốc độ ~2.3 lần
- Custom Collection tăng tốc ~1.4 lần so với Stream
- Cân nhắc khi dữ liệu lớn hơn 2^31 phần tử (giới hạn `Collection.size()`)

**Kết luận:**
- Luôn ưu tiên `Collection` cho public API
- Chỉ trả về `Stream` khi xử lý dữ liệu lớn/real-time
- Triển khai custom collection cho cấu trúc dữ liệu đặc biệt
- Cung cấp adapter methods nếu cần hỗ trợ cả 2 cách dùng

## 7. Cẩn trọng khi sử dụng streams parallel

**Nguyên tắc cốt lõi:** Parallel stream không phải là giải pháp vạn năng. Sử dụng không đúng cách có thể dẫn đến:
- Sự cố hiệu năng (chậm hơn phiên bản tuần tự)
- Kết quả không chính xác
- Treo hệ thống (liveness failure)

### Ví dụ anti-pattern

```java
// Parallel stream gây treo hệ thống
public class MersennePrimes {
    public static void main(String[] args) {
        primes().parallel() // Thêm parallel() gây hại
               .map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
               .filter(mersenne -> mersenne.isProbablePrime(50))
               .limit(20)
               .forEach(System.out::println);
    }
    
    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
}
```

**Hậu quả:**
- Không in kết quả, CPU tăng 90% vô thời hạn
- Nguyên nhân: Stream.iterate + limit khó chia nhỏ tác vụ

### Pattern đúng khi dùng parallel

```java
// Ví dụ tính số nguyên tố <= n với parallel hiệu quả
public class PrimeCounter {
    public static long pi(long n) {
        return LongStream.rangeClosed(2, n)
                        .parallel() // Hiệu quả với rangeClosed
                        .mapToObj(BigInteger::valueOf)
                        .filter(i -> i.isProbablePrime(50))
                        .count();
    }
}
```

**Kết quả:**
- Tốc độ tăng ~3.7 lần trên CPU 4 nhân
- Nguyên nhân: Dữ liệu nguồn (LongStream.range) dễ chia nhỏ

### Các yếu tố ảnh hưởng đến hiệu quả

| Yếu tố                  | Tốt cho parallel | Ví dụ                  |
|-------------------------|------------------|------------------------|
| Nguồn dữ liệu           | ArrayList, mảng  | new ArrayList().stream().parallel() |
| Kiểu dữ liệu nguyên thủy| Có              | IntStream.range(1,100).parallel() |
| Thao tác terminal       | Reduce, count    | sum(), reduce(0, Integer::sum) |
| Kích thước dữ liệu      | Lớn (>100k)     | List với 1 triệu phần tử |
| Độ phức tạp tính toán   | Cao             | Xử lý ảnh, mã hóa dữ liệu |

### Best practices
1. **Chọn nguồn dữ liệu phù hợp:**

```java
// Tốt: Mảng, ArrayList, HashMap
int[] numbers = new int[10_000];
Arrays.stream(numbers).parallel().sum();

// Xấu: Stream.iterate, HashSet nhỏ
Stream.iterate(0, i -> i+1).parallel().limit(1000); 
```

2. **Tránh stateful operations:**

```java
// Sai: Sử dụng biến ngoài trong parallel stream
AtomicInteger count = new AtomicInteger();
list.parallelStream().forEach(e -> count.incrementAndGet());

// Đúng: Dùng reduction thay thế
long correctCount = list.parallelStream().count();
```

3. **Chú ý thứ tự xử lý:**

```java
// In kết quả không theo thứ tự
list.parallelStream().forEach(System.out::println);

// Giữ nguyên thứ tự (giảm hiệu năng)
list.parallelStream().forEachOrdered(System.out::println);
```

4. **Kiểm tra hiệu năng:**

```java
// Đo thời gian trước/sau khi parallel
long start = System.nanoTime();
result = stream.count();
long duration = (System.nanoTime() - start) / 1_000_000;

// Chỉ parallel nếu cải thiện đáng kể
if (durationParallel < durationSequential * 0.7) {
    // Giữ parallel
}
```

5. **Sử dụng SplittableRandom:**

```java
// Tạo số ngẫu nhiên cho parallel
SplittableRandom random = new SplittableRandom();
random.ints(100_000).parallel().filter(i -> i % 2 == 0).count();
```

**Cảnh báo:**
- Không dùng `parallel()` cho I/O operations
- Tránh dùng với các thao tác blocking
- Test kỹ trên môi trường production-like
- Parallel stream là con dao hai lưỡi
- Chỉ sử dụng khi:
  - Dữ liệu đủ lớn (>100k phần tử)
  - Nguồn stream dễ chia nhỏ (array, ArrayList)
  - Thao tác đủ phức tạp (tính toán CPU-intensive)
  - Đã test hiệu năng thực tế
- Ưu tiên sử dụng các framework parallel chuyên dụng (Fork/Join) cho tác vụ phức tạp

## 8. Kết luận

Java 8 mang đến nhiều tính năng mới để cải thiện hiệu suất và độ dễ đọc của mã, nhưng việc sử dụng chúng một cách hiệu quả yêu cầu bạn phải hiểu rõ về lambdas, functional interface và Streams API. Bằng cách tuân theo các nguyên tắc trên, bạn có thể tận dụng tối đa những lợi ích mà chúng mang lại.
