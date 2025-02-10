---
title: "Cách một senior sử dụng Enum trong Java"
description: "Những kỹ thuật sử dụng Enum hiệu quả trong Java với ví dụ thực tế"
author: huy8895
date: 2025-02-10 21:50:00 +0700
categories: [Effective Java]
tags: [java, enum, best-practices]
pin: false
math: false
mermaid: true
#image:
#  path: assets/img/posts/20250210/0.png
---

## 1. Dùng Enum thay thế hằng số số nguyên
**Ví dụ:** Thay vì dùng các hằng số int cho trạng thái
```java
// Cách cũ ❌
public static final int STATUS_PENDING = 0;
public static final int STATUS_APPROVED = 1;

// Cách mới ✅
public enum Status {
    PENDING, 
    APPROVED, 
    REJECTED
}
```
**Lợi ích:** Kiểm tra kiểu mạnh mẽ, tránh lỗi truyền sai giá trị

## 2. Dùng trường instance thay thế ordinal()
**Ví dụ:** Quản lý các loại hình học với thuộc tính riêng
```java
public enum Shape {
    CIRCLE(1.0), 
    SQUARE(4.0), 
    TRIANGLE(3.0);

    private final double sides;

    Shape(double sides) {
        this.sides = sides;
    }

    public double getSides() {
        return sides;
    }
}
```
**Lưu ý:** Tránh dùng `ordinal()` để lấy vị trí vì dễ gây lỗi khi thay đổi thứ tự

## 3. Sử dụng EnumSet thay bit fields
**Ví dụ:** Quản lý các ngày làm việc trong tuần
```java
EnumSet<DayOfWeek> weekend = EnumSet.of(DayOfWeek.SATURDAY, DayOfWeek.SUNDAY);
EnumSet<DayOfWeek> workDays = EnumSet.complementOf(weekend);

// Kiểm tra nhanh
if (workDays.contains(LocalDate.now().getDayOfWeek())) {
    System.out.println("Hôm nay đi làm!");
}
```
**Hiệu quả:** Tối ưu bộ nhớ, hỗ trợ các phép toán tập hợp

## 4. Dùng EnumMap thay chỉ mục ordinal
**Ví dụ:** Quản lý danh sách sản phẩm theo loại
```java
EnumMap<ProductType, List<Product>> productMap = new EnumMap<>(ProductType.class);

productMap.put(ProductType.ELECTRONIC, new ArrayList<>());
productMap.put(ProductType.BOOK, new ArrayList<>());

// Thêm sản phẩm
productMap.get(ProductType.ELECTRONIC).add(new Product("MacBook Pro"));
```
**Ưu điểm:** An toàn kiểu dữ liệu, hiệu năng cao

## 5. Mở rộng Enum bằng interface
**Ví dụ:** Thiết kế hệ thống phân quyền
```java
public interface Role {
    String getPermissions();
}

public enum AdminRole implements Role {
    SUPER_ADMIN {
        @Override
        public String getPermissions() {
            return "ALL";
        }
    }
}

public enum UserRole implements Role {
    EDITOR {
        @Override
        public String getPermissions() {
            return "READ/WRITE";
        }
    }
}
```
**Ứng dụng:** Linh hoạt thêm chức năng mà không phá vỡ enum gốc

## Case Study thực tế
**Bài toán:** Xử lý các trạng thái đơn hàng
```java
public enum OrderStatus {
    NEW {
        @Override
        public OrderStatus next() {
            return PROCESSING;
        }
    },
    PROCESSING {
        @Override
        public OrderStatus next() {
            return SHIPPED;
        }
    },
    SHIPPED {
        @Override
        public OrderStatus next() {
            return DELIVERED;
        }
    };

    public abstract OrderStatus next();
}

// Sử dụng
OrderStatus current = OrderStatus.NEW;
OrderStatus nextStatus = current.next();
```

**Lời khuyên chuyên gia:**
- Dùng enum cho các tập giá trị hữu hạn, biết trước
- Kết hợp với pattern matching (Java 14+)
- Tránh lạm dụng cho các dữ liệu động
- Kết hợp với các design pattern như Strategy

> **Pro tip:** Sử dụng `Enum::values` để xử lý các trường hợp cần lặp qua tất cả giá trị enum, kết hợp với Annotation Processing để tạo code tự động.
{: .prompt-tip }
```
