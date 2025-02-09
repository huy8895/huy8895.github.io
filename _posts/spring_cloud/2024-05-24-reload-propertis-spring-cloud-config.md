---
title: "Tích hợp Reloading Properties trong spring cloud"
description: "Cập nhật giá trị mới cho config mà không cần restart service"
author: huy8895
date: 2024-05-24 16:15:00 +0800
categories: [Spring Cloud]
tags: [java, springcloud, springboot, refresh_config]
pin: false
math: true
mermaid: true
---

## 1. Cấu hình
### 1.1 Thêm dependency actuator trong file pom.xml
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 1.2 Enable refresh endpoint trong file application.yml

```properties
management.endpoints.web.exposure.include=refresh
```

### 1.3 Thêm các giá trị vào file resources/bootstrap.yml
thay giá trị `SERVICE_NAME` bằng giá trị `spring.application.name` trong file `application.yml`
```yaml
spring:
  cloud:
    config:
      uri: ${CONFIG_SERVER:http://<springcloud-server-host>/}
      name: ${SERVICE_NAME:customer-service}
      username: config-server-user
      password: config-server-password
```

### 1.4 Thêm annotation `@RefreshScope` trên các class config properties hoặc có sử dụng `@Value`

```java
@Data
@Component
@ConfigurationProperties(prefix = "clients.configuration")
@RefreshScope
public class ConfigurationClientProperties {

  @JsonProperty("base_url")
  private String baseUrl;

  @JsonProperty("endpoint")
  private Map<String, String> endpoint;
}
```

```java
@RequestMapping("/test")
@RestController
@RefreshScope
public class TestController {
  
  @Value("${clients.config.baseurl}")
  private String baseurl;
}
```

# 2. Sử dụng api để refresh config
- sau khi config mới được cấu hình trên service config thì dùng api để refresh config mà không restart service

```bash
curl --location --request POST '<domain>/actuator/refresh' \
--header 'Authorization: Bearer <token>'
```

> Trên đây là ứng dụng của tôi tại dự án sử dụng spring cloud config, các bạn có thể tham khảo thêm tại [**Baedung**][baedung]
{: .prompt-info }
[baedung]: https://www.baeldung.com/spring-reloading-properties

