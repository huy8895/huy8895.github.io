---
title: "Cài đặt keycloak bằng Docker"
description: "Cài đặt keycloak trên Docker"
author: huy8895
date: 2025-01-09 21:35:00 +0800
categories: [Keycloak, Docker]
tags: [keycloak, Docker]
pin: true
math: false
mermaid: false
---

## Cài đặt Keycloak bằng Docker Compose

Để cài đặt Keycloak bằng Docker Compose, bạn cần tạo một file `docker-compose.yml`. Dưới đây là ví dụ về cấu hình:
```yaml
version: '3.8'

services:

  keycloak:
    image: bitnami/keycloak:25.0.6
    container_name: keycloak
    restart: always
    environment:
      KEYCLOAK_HTTP_PORT: 8080
      KEYCLOAK_HTTPS_PORT: 8443
      KEYCLOAK_DATABASE_HOST: "db"
      KEYCLOAK_DATABASE_PORT: 5432
      KEYCLOAK_DATABASE_NAME: keycloak
      KEYCLOAK_DATABASE_USER: keycloak
      KEYCLOAK_DATABASE_PASSWORD: keycloak

      KEYCLOAK_ADMIN: "admin"
      KEYCLOAK_ADMIN_PASSWORD: "12345678@Ab"
    ports:
      - "8070:8080"
    depends_on:
      - db
    networks:
      - keycloak
    volumes:
      - './mynewtheme:/opt/bitnami/keycloak/themes/mynewtheme'

  db:
    image: postgres:14
    container_name: keycloak_db
    restart: always
    volumes:
      - keycloak_postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: keycloak
    networks:
      - keycloak

volumes:
  keycloak_postgres_data:
    name: keycloak_postgres_data
    driver: local

networks:
  keycloak:
    name: keycloak
    driver: bridge
```

### Hướng dẫn sử dụng

1. Tạo file `docker-compose.yml` trong thư mục dự án của bạn.
2. Chạy lệnh sau để khởi động Keycloak:

   ```bash
   docker-compose up -d
   ```

3. Truy cập Keycloak tại địa chỉ `http://localhost:8070`.

4. Sử dụng tài khoản `admin` và mật khẩu `12345678@Ab` để đăng nhập.
