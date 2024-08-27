---
title: "Chương 1: Giới thiệu"
description: "Giới thiệu tổng quan về mục tiêu, nội dung và cách tiếp cận của cuốn sách, đồng thời giải thích những khái niệm chính và cách cuốn sách tổ chức các thông tin."
author: huytvdev
date: 2024-08-26 11:05:00 +0800
categories: [ Effective Java ]
tags: [ java ]
pin: true
math: true
mermaid: true
#image:
#  path: /commons/devices-mockup.png
#  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
#  alt: Responsive rendering of Chirpy theme on multiple devices.
---

## Một số mục quan trọng trong cuốn sách

- Cuốn sách này được thiết kế để giúp bạn sử dụng hiệu quả ngôn ngữ lập trình Java và các thư viện
  cơ bản của nó như:
  `java.lang, java.util, java.io` và các phân nhánh như `java.util.concurrent`
  và `java.util.function`.
  Cuốn sách bao gồm 90 mục, mỗi mục là một quy tắc lập trình hữu ích, được các lập trình viên có
  kinh nghiệm đánh giá cao. Các mục này được sắp xếp thành 11 chương, mỗi chương tập trung vào một
  khía cạnh chính của thiết kế phần mềm.

- Cuốn sách không nhất thiết phải đọc theo thứ tự từ đầu đến cuối, vì mỗi mục độc lập và có liên kết
  chéo với nhau. Những điểm chính như lập trình với Lambda, Stream, Optional, hay các phương thức
  mặc định trong interface (từ Java 8 trở đi) đều được trình bày rõ ràng trong từng mục.

- Các ví dụ mã nguồn trong sách giúp minh họa nhiều mẫu thiết kế (design patterns) và các quy tắc
  lập trình tốt. Bên cạnh đó, những ví dụ phản diện (antipattern) cũng được chỉ ra với chú thích
  “Đừng bao giờ làm điều này!”. Các ví dụ này giúp người đọc hiểu vì sao phương pháp đó là không tốt
  và cách thay thế hợp lý.

- Cuốn sách không dành cho người mới bắt đầu học Java. Nó yêu cầu người đọc có hiểu biết căn bản về
  ngôn ngữ này. Nếu bạn chưa quen với Java, hãy tham khảo các sách nhập môn trước khi đến với
  Effective Java.

- Phần lớn các quy tắc trong sách được dựa trên những nguyên tắc cơ bản như: rõ ràng, đơn giản, mã
  dễ đọc và dễ bảo trì. Mã nên được tái sử dụng thay vì sao chép, và các lỗi cần được phát hiện càng
  sớm càng tốt, tốt nhất là trong quá trình biên dịch.

- Dù không phải quy tắc nào trong sách cũng áp dụng 100% thời gian, nhưng chúng phản ánh phần lớn
  các
  thực tiễn lập trình tốt. Hãy học cách tuân thủ các quy tắc trước, sau đó mới học cách “phá luật”
  một
  cách hợp lý.

- Cuốn sách tập trung vào việc viết mã rõ ràng, chính xác, linh hoạt, và dễ bảo trì hơn là tối ưu
  hóa
  hiệu suất. Những ví dụ về hiệu suất chỉ là phụ và không nên coi chúng là chuẩn mực tuyệt đối.
