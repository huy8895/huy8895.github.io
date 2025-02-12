---
title: "Java Nâng Cao: Tối Ưu Hóa Code Với Native Methods, Reflection & Quy Tắc Đặt Tên"
description: "Tìm hiểu cách sử dụng Native Methods, Reflection và Quy Tắc Đặt Tên để tối ưu hóa code trong Java."
author: huytv7
date: 2025-02-12 13:26:00 +0700
categories: [Effective Java]
tags: [java, "java 8", best-practices]
pin: false
math: false
mermaid: true
# image:
#   path: assets/img/posts/20250212-java-nang-cao-toi-uu-code-native-reflection-quy-tac-dat-ten/0.png
---

## 1.Giới Hạn Phạm Vi Biến Cục Bộ

### Nguyên Tắc Cơ Bản
- Khai báo biến ở vị trí gần nhất với nơi sử dụng đầu tiên
- Luôn khởi tạo giá trị ngay khi khai báo
- Ưu tiên sử dụng vòng lặp `for` thay vì `while`
- Tách phương thức thành các phần nhỏ, tập trung

### Ví Điển Hình Thường Gặp
#### ❌ Anti-pattern: Khai báo biến quá sớm

```java
Iterator<String> iterator = list.iterator(); // Khai báo ngoài phạm vi cần thiết
while (iterator.hasNext()) {
    processItem(iterator.next());
}

// ... 50 dòng code sau ...

Iterator<String> iterator2 = list2.iterator();
while (iterator.hasNext()) { // Lỗi khó phát hiện do dùng nhầm iterator
    processItem(iterator2.next());
}
```
**Vấn đề:** Biến `iterator` vẫn tồn tại sau khi hết phạm vi sử dụng, dễ gây lỗi khi copy-paste code

#### ✅ Pattern Đúng: Giới hạn phạm vi với for-loop

```java
// Sử dụng for-loop thay vì while
for (Iterator<String> i = list.iterator(); i.hasNext(); ) {
    String item = i.next();
    processItem(item);
}

// Vòng lặp độc lập an toàn
for (Iterator<String> i = list2.iterator(); i.hasNext(); ) {
    String item = i.next();
    processItem(item);
}
```
**Ưu điểm:** Mỗi biến iterator chỉ tồn tại trong phạm vi vòng lặp của nó, tránh lỗi dùng nhầm

### Best Practices
**Quy tắc khai báo muộn:** 

   ```java
   // ❌ Khai báo trước khi cần
   int result;
   // ... code không liên quan ...
   result = calculate();
   
   // ✅ Khai báo khi cần dùng
   int result = calculate();
   ```

**Xử lý ngoại lệ đúng cách:**
   ```java
   try {
       FileInputStream fis = new FileInputStream("data.txt"); // Khai báo trong try
       // Xử lý file
   } catch (FileNotFoundException e) {
       // Xử lý lỗi
   }
   ```

**Tối ưu vòng lặp:**
   ```java
   for (int i = 0, n = getMaxIterations(); i < n; i++) { 
       // Dùng biến n lưu giá trị tính toán tốn kém
       processItem(i);
   }
   ```

### Bảng So Sánh Cách Tiếp Cận

| Đặc Điểm               | Cách Cũ (while-loop) | Cách Mới (for-loop) |
|------------------------|----------------------|---------------------|
| Phạm vi biến           | Rộng, dễ leak        | Giới hạn chặt       |
| Nguy cơ lỗi copy-paste | Cao                  | Gần như không       |
| Khả năng đọc code      | Khó theo dõi         | Dễ hiểu             |
| Hiệu năng              | Có thể tính toán lại | Tối ưu hóa được     |

**Lưu ý quan trọng:** Luôn ưu tiên sử dụng for-each loop khi không cần thao tác với iterator (áp dụng từ Java 5 trở lên):
```java
for (String item : stringList) {
    System.out.println(item);
}
```