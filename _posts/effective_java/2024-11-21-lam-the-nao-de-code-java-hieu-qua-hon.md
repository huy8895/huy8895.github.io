---
title: "Làm thế nào để code Java hiệu quả hơn"
description: "Trong quá trình làm việc với ngôn ngữ java đây là những điều mà tôi đã học được để code hiệu quả hơn."
author: huy8895
date: 2024-11-21 22:05:00 +0800
categories: [ Effective Java ]
tags: [ java ]
pin: true
math: true
mermaid: true
#image:
#  path: /commons/devices-mockup.png
#  alt: Responsive rendering of Chirpy theme on multiple devices.
---

# 1. Giới thiệu

# 2. Tạo và hủy đối tượng
- chương này nói về cách tạo và hủy đối tượng để code hiệu quả hơn.

## 1. Cân nhắc sử dụng phương thức static factory thay vì các hàm khởi tạo.

## 2. Cân nhắc sử dụng Builder khi gặp constructor nhiều tham số.

## 3. Thực thi thuộc tính singleton bằng cách sử dụng private constructor hoặc enum.

## 4. Thực thi không thể khởi tạo bằng cách sử dụng hàm khởi tạo riêng tư.

## 5. Ưu tiên sử dụng dependency injection thay vì gắn chặt tài nguyên một cách cố định.

## 6. Tránh tạo các đối tượng không cần thiết.

## 7. Loại bỏ các tham chiếu đối tượng không còn sử dụng.

## 8. Tránh sử dụng finalizers và cleaners.

## 9. Ưu tiên sử dụng try-with-resources thay vì try-finally.

# 3. Các phương thức chung cho tất cả các đối tượng.

## 10. Tuân thủ quy ước chung khi ghi đè phương thức equals.

## 11. Luôn ghi đè phương thức hashCode khi bạn ghi đè phương thức equals.

## 12. Luôn ghi đè phương thức toString.

## 13. Ghi đè phương thức clone một cách cẩn thận.

## 14. Cân nhắc việc triển khai interface Comparable.

# 4. Classes và Interfaces

## 15. Giảm thiểu mức độ truy cập của các lớp và thành viên.

## 16. Trong các lớp public, sử dụng các phương thức truy cập thay vì các trường public.

## 17. Giảm thiểu khả năng thay đổi (tính biến đổi).

## 18. Ưu tiên sử dụng thành phần (composition) hơn kế thừa (inheritance).

## 19. Hãy thiết kế và viết tài liệu rõ ràng cho việc kế thừa, hoặc nếu không thì ngăn cấm việc kế thừa.

## 20. Nên ưu tiên sử dụng interface hơn là abstract class.

## 21. Hãy thiết kế các interface để sử dụng lâu dài.

## 22. Chỉ sử dụng interface để định nghĩa kiểu.



