---
title: "Triển khai Debezium để CDC dữ liệu"
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

> **Trải nghiệm với Debezium tại công ty**  
> Trong quá trình làm việc, mình đã có cơ hội tiếp xúc với công nghệ Change Data Capture (CDC) thông qua Debezium. Đây là một công cụ hữu ích giúp theo dõi và xử lý dữ liệu thời gian thực một cách hiệu quả.

## 1. Giới thiệu về Debezium

### 1.1. Debezium là gì?

Debezium là một công cụ mã nguồn mở giúp theo dõi và ghi nhận các thay đổi trong cơ sở dữ liệu (Change Data Capture - CDC). Khi có bất kỳ thay đổi nào xảy ra trên dữ liệu (thêm, sửa, xóa), Debezium sẽ tự động ghi nhận và đưa chúng vào các hệ thống xử lý dữ liệu thời gian thực như Apache Kafka. Điều này giúp chúng ta quản lý và phản ứng với sự thay đổi ngay lập tức mà không cần can thiệp thủ công.

### 1.2. Tại sao nên sử dụng Debezium?

Debezium là giải pháp tuyệt vời cho việc theo dõi và xử lý các thay đổi dữ liệu liên tục. Nó giúp đồng bộ hóa dữ liệu giữa các hệ thống khác nhau, tạo ra nhật ký thay đổi, và phát hiện các sự kiện trong hệ thống cơ sở dữ liệu. Điều này giúp doanh nghiệp giảm công sức kiểm tra dữ liệu thủ công và tăng khả năng phản ứng với những thay đổi.

### 1.3. Ai nên sử dụng Debezium?

Debezium phù hợp cho các nhà phát triển, quản trị viên cơ sở dữ liệu, và doanh nghiệp cần quản lý dữ liệu thời gian thực. Đặc biệt, trong các hệ thống phân tán hoặc có quy mô lớn, Debezium là công cụ không thể thiếu. Ngoài ra, nó rất hữu ích cho các tổ chức muốn xây dựng hệ thống event-driven hoặc ứng dụng có yêu cầu phản ứng nhanh với thay đổi dữ liệu.

### 1.4. Debezium được sử dụng ở đâu?

Debezium thường được triển khai với các cơ sở dữ liệu như MySQL, PostgreSQL, MongoDB, SQL Server, và Oracle. Nó tích hợp dễ dàng với Apache Kafka để chuyển các thay đổi dữ liệu thành các sự kiện. Bạn có thể triển khai Debezium trên các nền tảng đám mây hoặc hệ thống on-premises.

### 1.5. Khi nào nên sử dụng Debezium?

Debezium nên được sử dụng khi cần theo dõi và xử lý các thay đổi trong cơ sở dữ liệu theo thời gian thực. Đặc biệt hữu ích khi cần đồng bộ hóa dữ liệu giữa các hệ thống khác nhau hoặc trong các ứng dụng dựa trên sự kiện (event-driven).

### 1.6. Debezium hoạt động như thế nào?

Debezium cài đặt một connector vào cơ sở dữ liệu để theo dõi các log bin (binary log) hoặc nguồn sự kiện của cơ sở dữ liệu. Khi phát hiện có thay đổi, các sự kiện sẽ được xuất ra và chuyển vào hệ thống xử lý như Apache Kafka. Từ đó, dữ liệu có thể được tiêu thụ và xử lý theo nhu cầu.

> **Kết luận:**  
> Debezium là một giải pháp mạnh mẽ cho việc xử lý dữ liệu thời gian thực, giúp giảm độ phức tạp và tăng hiệu quả trong quản lý dữ liệu.

## 2. Cách triển khai Debezium


### 2.1 DockerFile
> Chuẩn bị DockerFile dựa trên image gốc và tải các driver cần thiết phục vụ cho việc kết nối.

```dockerfile
ARG DEBEZIUM_VERSION
FROM quay.io/debezium/connect:2.7
ENV KAFKA_CONNECT_JDBC_DIR=$KAFKA_CONNECT_PLUGINS_DIR/kafka-connect-jdbc

ARG POSTGRES_VERSION=42.5.1
ARG KAFKA_JDBC_VERSION=5.3.2

# Deploy PostgreSQL JDBC Driver
RUN cd /kafka/libs && curl -sO https://jdbc.postgresql.org/download/postgresql-$POSTGRES_VERSION.jar

# Deploy Oracle JDBC Driver
RUN cd /kafka/libs && curl -L -O https://download.oracle.com/otn-pub/otn_software/jdbc/1921/ojdbc10.jar

# Deploy Kafka Connect JDBC
RUN mkdir $KAFKA_CONNECT_JDBC_DIR && cd $KAFKA_CONNECT_JDBC_DIR &&\
	curl -sO https://packages.confluent.io/maven/io/confluent/kafka-connect-jdbc/$KAFKA_JDBC_VERSION/kafka-connect-jdbc-$KAFKA_JDBC_VERSION.jar

RUN cd $KAFKA_CONNECT_JDBC_DIR &&\
    curl -L -O https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-j-8.2.0.tar.gz &&  \
    tar -xzvf mysql-connector-j-8.2.0.tar.gz
```

### 2.2 Tạo file Docker Compose

> Để triển khai Debezium với Docker Compose, bạn có thể tạo một file `docker-compose.yml` để dễ dàng quản lý và khởi chạy các container liên quan. Dưới đây là ví dụ về file Docker Compose sử dụng image vừa build:

```yaml
version: '3'
services:
  kafka:
    image: 'bitnami/kafka:3.5.1'
    container_name: kafka
    restart: always
    ports:
      - '9094:9094'
    environment:
      - KAFKA_CFG_NODE_ID=0
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka:9093
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094
  kafka-ui:
    container_name: kafka-ui
    image: provectuslabs/kafka-ui
    ports:
      - 8895:8080
    environment:
      - DYNAMIC_CONFIG_ENABLED=true

  mysql:
    container_name: mysql
    image: quay.io/debezium/example-mysql:2.7
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=debezium
      - MYSQL_USER=mysqluser
      - MYSQL_PASSWORD=mysqlpw
  connect:
    container_name: connect
    image: debezium/debezium-connect:2.7
    build:
      context: .
      args:
        DEBEZIUM_VERSION: 2.7
    ports:
      - 8083:8083
      - 5005:5005
    environment:
      - BOOTSTRAP_SERVERS=kafka:9092
      - GROUP_ID=1
      - CONFIG_STORAGE_TOPIC=my_connect_configs
      - OFFSET_STORAGE_TOPIC=my_connect_offsets
      - STATUS_STORAGE_TOPIC=my_source_connect_statuses
  debezium-ui:
    image: quay.io/debezium/debezium-ui:2.7
    container_name: debezium-ui
    ports:
      - "8080:8080"
    platform: linux/amd64
    environment:
      - KAFKA_CONNECT_URIS=http://connect:8083
    depends_on:
      - connect
```

- chạy lệnh 

```bash
docker-compose up -d
```

- ở đây ta có các container :
1. Kafka: Chạy Kafka phiên bản 3.5.1. Sử dụng cổng 9094 để kết nối với bên ngoài.
Tích hợp các vai trò controller và broker trên cùng một node.

2. Kafka UI: Cung cấp giao diện người dùng cho Kafka, giúp theo dõi và quản lý các topic, consumer group, và message của Kafka.

Expose trên cổng 8895 để truy cập giao diện web.
3. MySQL: Sử dụng image MySQL có sẵn từ Debezium để thử nghiệm.

Được cấu hình với các biến môi trường cho tài khoản root và user bình thường.
Debezium Connect: Đây là service Debezium, nơi các connector được triển khai để theo dõi các thay đổi trong cơ sở dữ liệu.

Kết nối với Kafka qua BOOTSTRAP_SERVERS.
Expose các cổng 8083 (Kafka Connect REST API) và 5005 để gỡ lỗi.

4. Debezium UI: Giao diện người dùng Debezium giúp bạn quản lý các connector và theo dõi trạng thái của chúng.

Kết nối với service Debezium Connect thông qua KAFKA_CONNECT_URIS để lấy dữ liệu và trạng thái.
Expose trên cổng 8080 để truy cập giao diện web Debezium UI.

