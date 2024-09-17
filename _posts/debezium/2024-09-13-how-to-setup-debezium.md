---
title: "Triển khai debezium để CDC dữ liệu"
description: ""
author: huy8895
date: 2024-09-13 23:15:00 +0700
categories: [Debezium]
tags: [debezium, CDC, kafka]
pin: true
mermaid: true
image:
  path: assets/img/posts/20240913/0.png
---

> Trong quá trình làm việc tại ngân hàng MSB mình đã có cơ hội được trải nghiệm với công nghệ CDC sử
> dụng Debezium

## 1. Khái quát về Debezium

### 1.1. **What (Cái gì):**

Debezium là một công cụ mã nguồn mở dùng để theo dõi các thay đổi trong cơ sở dữ liệu (CDC -
Change Data Capture). Nó giúp tự động trích xuất các thay đổi từ cơ sở dữ liệu và chuyển chúng
vào các hệ thống xử lý dữ liệu thời gian thực như Kafka. Với Debezium, mọi thay đổi (thêm, sửa,
xóa) trên dữ liệu của bạn sẽ được ghi nhận và có thể được xử lý ngay lập tức.

### 1.2. **Why (Tại sao):**

Debezium được sử dụng để giải quyết các bài toán cần theo dõi và xử lý thay đổi dữ liệu liên tục,
như đồng bộ dữ liệu giữa các hệ thống, tạo ra nhật ký thay đổi dữ liệu, hoặc phát hiện sự kiện
trong hệ thống cơ sở dữ liệu. Nó giúp giảm tải công việc thủ công của việc kiểm tra dữ liệu và
cải thiện khả năng phản ứng nhanh chóng với sự thay đổi.

### 1.3. **Who (Ai):**

Debezium phù hợp với các nhà phát triển, quản trị viên cơ sở dữ liệu, và các doanh nghiệp cần
quản lý dữ liệu thời gian thực, đặc biệt là trong các hệ thống phân tán hoặc quy mô lớn. Nó cũng
rất hữu ích cho các tổ chức muốn xây dựng hệ thống event-driven hoặc các ứng dụng cần phản ứng
ngay lập tức khi dữ liệu thay đổi.

### 1.4. **Where (Ở đâu):**

Debezium thường được triển khai trong các hệ thống sử dụng cơ sở dữ liệu như MySQL, PostgreSQL,
MongoDB, SQL Server, và Oracle. Nó có thể được tích hợp với Apache Kafka để chuyển đổi các thay
đổi dữ liệu thành các bản tin Kafka. Debezium cũng có thể triển khai trên nền tảng đám mây hoặc
trong các môi trường on-premises.

### 1.5. **When (Khi nào):**

Debezium nên được sử dụng khi bạn cần theo dõi các thay đổi trong cơ sở dữ liệu theo thời gian
thực và cần phản ứng nhanh chóng với những thay đổi này. Điều này rất hữu ích trong các trường
hợp cần đồng bộ hóa dữ liệu giữa các hệ thống khác nhau, hoặc trong các ứng dụng event-driven.

### 1.6. **How (Như thế nào):**

Debezium hoạt động bằng cách cài đặt một connector vào cơ sở dữ liệu. Connector này sẽ theo dõi
các log bin (binary log) hoặc các nguồn sự kiện của cơ sở dữ liệu để phát hiện thay đổi. Những
thay đổi này sau đó sẽ được xuất ra dưới dạng các sự kiện và chuyển vào hệ thống xử lý như Apache
Kafka, nơi bạn có thể tiêu thụ và xử lý dữ liệu theo nhu cầu của mình.

> Debezium là một giải pháp mạnh mẽ cho việc xử lý dữ liệu theo thời gian thực, giúp giảm độ phức
> tạp
> và tăng tính hiệu quả cho hệ thống.

## 2. Cách triển khai debezium
