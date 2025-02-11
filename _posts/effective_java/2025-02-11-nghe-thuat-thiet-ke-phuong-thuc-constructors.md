---
title: "Nghệ thuật thiết kế phương thức và constructors"
description: "Tối ưu hóa việc sử dụng constructors trong Java"
author: huy8895
date: 2025-02-11 14:57:00 +0700
categories: []
tags: []
pin: true
math: true
mermaid: true
#image:
#  path: assets/img/posts/20250211-nghe-thuat-thiet-ke-phuong-thuc-constructors/0.png
---

## 1. Luôn kiểm tra tham số đầu vào

### Nguyên tắc cốt lõi
- **Phát hiện lỗi sớm**: Kiểm tra tham số ngay từ đầu method giúp phát hiện lỗi nhanh chóng và rõ ràng
- **Bảo vệ tính toàn vẹn**: Ngăn chặn các lỗi tiềm ẩn ảnh hưởng đến trạng thái đối tượng
- **Tài liệu hóa rõ ràng**: Mô tả chính xác các điều kiện đầu vào và ngoại lệ có thể xảy ra

### Ví dụ thực tế
#### Anti-pattern: Không kiểm tra tham số

```java
public void processData(String input) {
    // Không kiểm tra input null
    int length = input.length(); // Ném NullPointerException tại đây
    // ... xử lý tiếp ...
}
```
**Hậu quả**: Lỗi xảy ra muộn, khó truy vết nguồn gốc, có thể làm hỏng trạng thái hệ thống

#### Pattern đúng: Kiểm tra đầu vào

```java
public void processData(String input) {
    if (input == null) {
        throw new IllegalArgumentException("Input không được null");
    }
    // ... xử lý an toàn ...
}
```
**Lợi ích**: Phát hiện lỗi ngay lập tức, thông báo rõ ràng về nguyên nhân

### Công cụ hỗ trợ quan trọng

| Công cụ               | Mục đích sử dụng                  | Phiên bản Java |
|-----------------------|-----------------------------------|----------------|
| requireNonNull        | Kiểm tra null                    | 7+             |
| checkIndex            | Kiểm tra chỉ số mảng/collection  | 9+             |
| Assert                | Kiểm tra nội bộ                  | 1.4+           |

### Best practices
1. **Document exceptions**:

```java
/**
 * @param divisor số chia phải khác 0
 * @throws ArithmeticException nếu divisor bằng 0
 */
public double divide(int dividend, int divisor) {
    // ... code ...
}
```

2. **Sử dụng annotation @Nullable**:

```java
public void updateProfile(@Nullable String nickname) {
    // Xử lý giá trị có thể null
}
```

3. **Kiểm tra tham số trì hoãn**:

```java
// Chỉ kiểm tra khi thực sự cần thiết
public void sort(List<?> list) {
    // Kiểm tra trong quá trình so sánh phần tử
}
```

### So sánh cách tiếp cận

| Tiêu chí            | Cách cũ                  | Cách mới                 |
|---------------------|--------------------------|--------------------------|
| Thời điểm phát hiện | Khi sử dụng tham số      | Ngay khi gọi method      |
| Debug               | Khó truy vết             | Dễ dàng xác định vị trí  |
| Hiệu suất           | Có thể tốt hơn           | Tăng độ tin cậy hệ thống |
| Bảo trì             | Khó theo dõi điều kiện   | Điều kiện rõ ràng        |

### Mẫu code chuẩn

```java
public class ValidationUtils {
    // Kiểm tra null với thông báo tùy chỉnh
    public void configure(Strategy strategy) {
        this.strategy = Objects.requireNonNull(strategy, "Chiến lược không được null");
    }

    // Kiểm tra chỉ số mảng
    public static void checkArrayIndex(long[] array, int offset, int length) {
        assert array != null : "Mảng không được null";
        assert offset >= 0 && offset <= array.length : "Offset không hợp lệ";
        assert length >= 0 && length <= array.length - offset : "Độ dài vượt quá giới hạn";
    }
    
    // Kiểm tra điều kiện nghiệp vụ
    public BigDecimal calculateDiscount(double rate) {
        if (rate < 0 || rate > 1) {
            throw new IllegalArgumentException("Tỷ lệ giảm giá phải trong khoảng 0-1");
        }
        // ... tính toán ...
    }
}
```

**Giải thích**:
- Sử dụng `requireNonNull` cho kiểm tra null tự động
- Assertion cho kiểm tra nội bộ
- Ném exception cụ thể cho lỗi nghiệp vụ
- Kết hợp kiểm tra nhiều điều kiện cùng lúc

> Việc kiểm tra tham số đầu vào đầy đủ giúp giảm 70% lỗi runtime theo thống kê từ các dự án mã nguồn mở. Luôn dành ít nhất 5% thời gian coding cho việc validate dữ liệu đầu vào để tiết kiệm 50% thời gian debug sau này.
{: .prompt-tip}

## 2. Tạo bản sao bảo vệ khi cần

## 3. Thiết kế phương thức khoa học

## 4. Dùng Overloading đúng lúc

## 5. Cẩn trọng với tham số biến đổi (varargs)

## 6. Ưu tiên trả về mảng/collection rỗng

## 7. Dùng Optional đúng cách

## 8. Viết tài liệu cho API public