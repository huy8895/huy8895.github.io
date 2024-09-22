---
title: "Xây dựng service lập lịch với Spring Boot và Quartz Scheduler"
description: ""
author: huy8895
date: 2024-09-22 09:51:00 +0700
categories: [ springboot, quartz ]
tags: [ springboot, quartz ]
---

Xin chào các bạn! Hôm nay, tôi muốn chia sẻ với các bạn về một dự án thú vị mà tôi đang thực hiện -
một hệ thống lập lịch công việc sử dụng Spring Boot và Quartz Scheduler.__

## 1.Giới thiệu

Trong nhiều ứng dụng, việc lập lịch và thực thi các tác vụ tự động là một yêu cầu phổ
biến. Đặc biệt trong kiến trúc microservice việc có 1 servcie chuyên phục vụ cho việc lập lịch là
cần thiết.
Vì vậy hôm nay tôi sẽ chia sẻ đến các bạn cách sử dụng quartz và springboot để tạo ra 1 ứng dụng
chuyên phục vụ việc lập lịch và quản lý bằng database và quản lý bằng api.

### 1.1.Công nghệ sử dụng

- **Spring Boot**: Framework Java mạnh mẽ để xây dựng ứng dụng web.
- **Quartz Scheduler**: Thư viện lập lịch công việc mạnh mẽ và linh hoạt.

### 1.2 Các tính năng chính

1. **Tạo mới job** Tạo mới job.

2. **Chỉnh sửa**: Chỉnh sửa thông tin của job.

3. **Tạm dừng/Tiếp tục job**: Hỗ trợ cập nhật thông tin và lịch trình của các công việc hiện có.

4. **Xoá job**: Xoá các job.

5. **Tìm kiếm/Danh sách job** Tìm kiếm job theo jobName, jobGroup.

### 1.3 Điểm nổi bật của dự án

1. **Linh hoạt**: Hỗ trợ cả công việc định kỳ và công việc chạy một lần.

2. **Khả năng mở rộng**: Thiết kế modular cho phép dễ dàng thêm các loại công việc mới.

3. **Quản lý hiệu quả**: Cung cấp các API để quản lý toàn diện vòng đời của công việc.

4. **An toàn và đáng tin cậy**: Sử dụng Quartz Scheduler đảm bảo việc thực thi công việc đáng tin
   cậy và có khả năng phục hồi.

## 2. Triển khai

### 2.1 Chuẩn bị môi trường

- Database

> Do việc lưu dữ liệu vào DB nên chúng ta cần một database để lưu.Ở đây tôi sử dụng docker để cài
> đặt môi trường
`docker-compose.yml` có thể sử dụng các loại database khác nhau tuỳ dự án của bạn.

```yaml
version: '3.8'

services:
  mysql:
    image: mysql
    container_name: mysql_container
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: demo_springboot_quartz
      MYSQL_USER: springboot_quartz
      MYSQL_PASSWORD: springboot_quartz
    ports:
      - "3306:3306"
```

- Springboot dependencies

```xml

<dependencies>
  <!-- Dependency cho cơ sở dữ liệu, ví dụ MySQL -->
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
  </dependency>

  <!-- Dependency cho JPA nếu bạn sử dụng JPA cho quản lý các entity -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

</dependencies>
```

- Cấu hình file `application.properties`

```properties
spring.application.name=demo-springboot-quartz-schedule
# Cấu hình quartz
spring.quartz.job-store-type=jdbc
spring.quartz.jdbc.initialize-schema=never
# Cấu hình database
spring.datasource.url=jdbc:mysql://localhost:3306/demo_springboot_quartz
spring.datasource.username=springboot_quartz
spring.datasource.password=springboot_quartz
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

### 2.2 Database
>Khi Quartz được cấu hình để sử dụng với một cơ sở dữ liệu, nó tạo ra một số bảng để lưu trữ thông
tin về các job, trigger và lịch trình. Dưới đây là danh sách các bảng chính mà Quartz sử dụng cùng
với giải thích về từng bảng.

Diagram:

![quartz_diagram](assets/img/posts/20240922-quartz-schedule-with-springboot/QRTZ_BLOB_TRIGGERS.png){:width="972" height="589" }
_Quart diagram_

Bảng chính mà Quartz sử dụng cùng với giải thích về từng bảng.

1. **QRTZ_JOB_DETAILS**
- Mục đích: Lưu trữ thông tin chi tiết về các job được lên lịch.
- Các cột quan trọng:
  - **SCHED_NAME**: Tên của scheduler.
  - **JOB_NAME**, **JOB_GROUP**: Tên và nhóm của job, là các cột định danh duy nhất.
  - **JOB_CLASS_NAME**: Lớp Java thực thi job.
  - **IS_DURABLE**: Cho biết job có tồn tại nếu không có trigger nào liên kết hay không.
  - **JOB_DATA**: Chứa thông tin dữ liệu cần thiết cho job (nếu có).
2. **QRTZ_TRIGGERS**
- Mục đích: Lưu trữ thông tin chi tiết về các trigger (cách các job được kích hoạt).
- Các cột quan trọng:
  - **SCHED_NAME**: Tên của scheduler.
  - **TRIGGER_NAME**, **TRIGGER_GROUP**: Tên và nhóm của trigger.
  - **JOB_NAME**, **JOB_GROUP**: Tên và nhóm của job mà trigger liên kết.
  - **TRIGGER_STATE**: Trạng thái của trigger (ví dụ: WAITING, PAUSED, ACQUIRED, BLOCKED).
  - **START_TIME, END_TIME, NEXT_FIRE_TIME, PREV_FIRE_TIME**: Các thời điểm kích hoạt tiếp theo và lần cuối cùng job được kích hoạt.
  - **TRIGGER_TYPE**: Loại trigger (ví dụ: SIMPLE, CRON).
  - **MISFIRE_INSTR**: Cách xử lý khi trigger bị lỡ (misfire).
3. **QRTZ_SIMPLE_TRIGGERS**
- Mục đích: Lưu trữ thông tin chi tiết về các trigger kiểu SimpleTrigger.
- Các cột quan trọng:
  - **SCHED_NAME, TRIGGER_NAME, TRIGGER_GROUP**: Thông tin định danh trigger.
  - **REPEAT_COUNT**: Số lần lặp lại.
  - **REPEAT_INTERVAL**: Khoảng thời gian giữa các lần kích hoạt.
  - **TIMES_TRIGGERED**: Số lần trigger đã được kích hoạt.
  > Lưu ý: Trigger kiểu SimpleTrigger được sử dụng để lên lịch các job lặp lại theo chu kỳ đơn giản (
    như “mỗi 10 giây”).
4. **QRTZ_CRON_TRIGGERS**
- Mục đích: Lưu trữ thông tin về các trigger kiểu CronTrigger.
- Các cột quan trọng:
  - **SCHED_NAME, TRIGGER_NAME, TRIGGER_GROUP**: Thông tin định danh trigger.
  - **CRON_EXPRESSION**: Biểu thức cron xác định lịch trình của trigger.
  - **TIME_ZONE_ID**: Múi giờ mà biểu thức cron dựa vào.
  > Trigger kiểu CronTrigger được sử dụng để lên lịch job theo biểu thức cron phức tạp (ví dụ: “chạy
    vào 9:00 AM mỗi ngày”).
5. **QRTZ_BLOB_TRIGGERS**
- Mục đích: Lưu trữ các trigger có dữ liệu blob (khi bạn cần lưu trữ nhiều dữ liệu tùy chỉnh với trigger).
- Các cột quan trọng:
  - **SCHED_NAME, TRIGGER_NAME, TRIGGER_GROUP**: Thông tin định danh trigger.
  - **BLOB_DATA**: Dữ liệu kiểu blob, chứa dữ liệu lớn hoặc dữ liệu tùy chỉnh cho trigger.
6. **QRTZ_CALENDARS**
- Mục đích: Lưu trữ thông tin về các lịch (calendar) được định nghĩa trong Quartz.
- Các cột quan trọng:
  - **SCHED_NAME, CALENDAR_NAME**: Tên của scheduler và lịch.
  - **CALENDAR**: Thông tin về lịch (thời gian mà job không được kích hoạt hoặc bị bỏ qua).

     > Lịch (Calendar) trong Quartz được dùng để xác định các khoảng thời gian mà job không được phép
     chạy (ví dụ: không chạy vào ngày lễ).
7. **QRTZ_PAUSED_TRIGGER_GRPS**
- Mục đích: Lưu trữ các nhóm trigger đã bị tạm dừng (paused).
- Các cột quan trọng:
  - **SCHED_NAME, TRIGGER_GROUP**: Tên của scheduler và nhóm trigger bị tạm dừng.
8. **QRTZ_FIRED_TRIGGERS**
- Mục đích: Lưu trữ thông tin về các trigger đã được kích hoạt gần đây.
- Các cột quan trọng:
  - **SCHED_NAME, ENTRY_ID**: Định danh cho mỗi lần trigger kích hoạt.
  - **TRIGGER_NAME, TRIGGER_GROUP**: Thông tin định danh trigger.
  - **INSTANCE_NAME**: Tên của instance Quartz đang chạy trigger.
  - **FIRED_TIME, SCHED_TIME**: Thời điểm trigger được kích hoạt và được lên lịch.
  - **STATE**: Trạng thái của job liên quan đến trigger (ví dụ: ACQUIRED, EXECUTED).
9. **QRTZ_SCHEDULER_STATE**
- Mục đích: Lưu trữ trạng thái của các scheduler Quartz đang chạy trong hệ thống.
- Các cột quan trọng:
  - **SCHED_NAME**: Tên của scheduler.
  - **INSTANCE_NAME**: Tên của instance Quartz.
  - **LAST_CHECKIN_TIME**: Thời gian lần cuối scheduler kiểm tra.
  - **CHECKIN_INTERVAL**: Khoảng thời gian giữa các lần kiểm tra.
10. **QRTZ_LOCKS**
- Mục đích: Lưu trữ các khóa (locks) để đảm bảo rằng chỉ một instance Quartz có thể thực thi job cụ thể tại một thời điểm (để tránh các job trùng lặp).
- Các cột quan trọng:
  - **SCHED_NAME, LOCK_NAME**: Tên của scheduler và tên của khóa.
11. **QRTZ_SIMPROP_TRIGGERS**
- Mục đích: Lưu trữ các trigger kiểu Simple Property Trigger (trigger dựa trên các thuộc tính đơn giản).
- Các cột quan trọng:
  - **SCHED_NAME, TRIGGER_NAME, TRIGGER_GROUP**: Thông tin định danh trigger.
  - Các cột khác lưu trữ các thuộc tính đơn giản liên quan đến trigger.

**Tóm tắt:**
Quartz sử dụng các bảng này để lưu trữ thông tin về:
```text
Job: Chi tiết về các công việc cần thực hiện.
Trigger: Lịch trình và trạng thái của các job.
Lịch: Các khoảng thời gian job không được kích hoạt.
Trạng thái hệ thống: Trạng thái của các scheduler và trigger đã kích hoạt.
```
Mỗi bảng có một vai trò quan trọng để đảm bảo rằng Quartz có thể theo dõi các công việc, quản lý
lịch trình, và xử lý trạng thái một cách chính xác trong một môi trường lưu trữ bền vững (persistent
store).

### 2.3 Coding
#### 2.3.1 Triển khai job
Để lập lịch được chúng ta cần có class implement lại hàm `execute` của interface `org.quartz.Job`
hàm này sẽ được gọi khi job của chúng ta tạo được chạy.

```java
@Slf4j
@Component
public class SampleJob implements Job {
    @Autowired
    ObjectMapper objectMapper;

    @Override
    @SneakyThrows
    public void execute(JobExecutionContext context) {
        log.info("[START] =====> execute() called with: [{}]", context.getJobDetail());
        // Lấy thông tin jobDetail từ context
        JobDetail jobDetail = context.getJobDetail();
        // Ghi thông tin jobDetail vào log
        log.info("Thông tin JobDetail sau khi lấy từ context: {}", jobDetail);
        // Lấy thông tin jobDataMap từ jobDetail
        JobDataMap jobDataMap = jobDetail.getJobDataMap();
        // Ghi thông tin jobDataMap vào log
        log.info("[END] Thông tin jobDataMap: {}", objectMapper.writeValueAsString(jobDataMap));
    }
}
```



#### 2.3.1 Tạo mới job
Để tạo mới 1 job ta cần chuẩn bị các thông tin và truyền vào request body
`AddJobDTO`. Ta có logic để thêm mới 1 job tại `JobService`

```java
@Transactional(rollbackFor = ObjectAlreadyExistsException.class)
public void addNewJob(AddJobDTO addJobDTO) throws SchedulerException {
  logger.info("Đang thêm job mới: {}", addJobDTO);

  Class<? extends Job> jobClass;
  try {
    jobClass = (Class<? extends Job>) Class.forName(addJobDTO.getJobClassName());
  } catch (ClassNotFoundException e) {
    throw new IllegalArgumentException("Không tìm thấy lớp job", e);
  }

  JobDetail jobDetail = JobBuilder.newJob(jobClass)
    .withIdentity(addJobDTO.getJobName(), addJobDTO.getGroupName())
    .withDescription(addJobDTO.getDescription())
    .setJobData(addJobDTO.getJobDataMap())
    .build();

  Trigger trigger = TriggerBuilder.newTrigger()
    .withIdentity(addJobDTO.getTriggerName(), addJobDTO.getGroupName())
    .startNow()
    .withSchedule(SimpleScheduleBuilder.simpleSchedule()
      .withIntervalInSeconds(addJobDTO.getIntervalInSeconds())
      .repeatForever())
    .build();

  scheduler.scheduleJob(jobDetail, trigger);
}
```

- gọi thử api Tạo mới:

```shell
curl --location 'localhost:8080/jobs/add' \
--header 'Content-Type: application/json' \
--data '{
    "jobName": "job-1,
    "groupName": "group-1",
    "triggerName": "trigger-1",
    "intervalInSeconds": 10,
    "jobClassName": "com.example.demospringbootquartzschedule.config.SampleJob",
    "jobData": {
        "data": "test"
    }
}'
```
> Job này sẽ được thực hiện bởi class SampleJob với khoảng thời gian giữa các lần chạy là 10 giây.

Class `SampleJob` khi được chạy sẽ in ra console các thông tin của job như:
```shell
2024-09-22 13:07:20.036  INFO 73164 --- [eduler_Worker-8] c.e.d.config.SampleJob : [START] =====> execute() called with: [JobDetail 'group-1.job-1'
2024-09-22 13:07:20.036  INFO 73164 --- [eduler_Worker-8] c.e.d.config.SampleJob : Thông tin JobDetail sau khi lấy từ context: JobDetail 'group-1.job-1':  
2024-09-22 13:07:20.036  INFO 73164 --- [eduler_Worker-8] c.e.d.config.SampleJob : [END] Thông tin jobDataMap: {"data": "test"}
```

## Kết luận

Dự án này không chỉ giúp tôi học hỏi thêm về Spring Boot và Quartz Scheduler mà còn tạo ra một giải
pháp thực tế cho việc lập lịch công việc trong các ứng dụng doanh nghiệp. Tôi hy vọng chia sẻ này sẽ
hữu ích cho những ai đang tìm hiểu về lập lịch công việc trong Java.

Nếu bạn quan tâm đến dự án này hoặc có bất kỳ câu hỏi nào, đừng ngần ngại để lại bình luận bên dưới.
Cảm ơn các bạn đã đọc!
