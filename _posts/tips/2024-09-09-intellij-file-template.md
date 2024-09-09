---
title: "Sử dụng file template trong Intellij để tạo nhanh file theo mẫu"
description: ""
author: huy8895
date: 2024-09-07 09:15:00 +0700
categories: [ Tool, Intellij ]
tags: [ Tips, Intellij, 'File Template' ]
pin: true
math: true
mermaid: true
image:
  path: assets/img/posts/20240907/0.png
---

> Configure: **Settings \| Editor \| File and Code Templates**
> {: .prompt-info }

## 1. File templates

File templates là các mẫu nội dung mặc định khi tạo tệp mới trong IntelliJ IDEA. Chúng cung cấp mã
nguồn và định dạng ban đầu theo tiêu chuẩn của từng loại tệp. Các loại tệp được đề xuất tùy thuộc
vào module, cấu hình, và vị trí của bạn trong project. Bạn có thể chọn từ danh sách các tệp có sẵn
với template tương ứng trong cài đặt.

### 1.1. Tạo mới template

1. Mở cài đặt, sau đó chọn **Editor \| File and Code Templates**.
2. Chọn phạm vi áp dụng template qua danh sách Scheme:

- Default: Áp dụng cho tất cả các project trong IDE hiện tại, lưu tại thư mục cấu hình của IDE.
- Project: Áp dụng riêng cho từng project và được lưu trong thư mục .idea/fileTemplates.

3. Tại tab Files, nhấn Create Template để tạo template mới, đặt tên, định dạng file và nội dung cho
   template. Sau đó áp dụng thay đổi và đóng cửa sổ cài đặt.

### 1.2 Có thể copy từ 1 file template có sẵn

### 1.3 Để lưu một tệp dưới dạng template, mở tệp trong trình chỉnh sửa.

1. Trên menu chính, chọn File \| Save File as Template.

2. Trong hộp thoại Save File as Template, đặt tên mới, định dạng và chỉnh sửa nội dung nếu cần. Bạn
   có thể định sẵn tên tệp hoặc sử dụng các biến để tự động tạo tên. Ví dụ:

- **File name**: Nếu cần, bạn có thể chỉ định tên cho tệp được tạo từ template. Mặc định, IntelliJ
  IDEA sẽ yêu cầu người dùng nhập tên khi tạo tệp. Bạn có thể cố định một tên cụ thể để bỏ qua
  bước này, hoặc sử dụng các biến có sẵn để tạo tên tự động. Ví dụ, bạn có thể đặt tệp một thư mục
  trên bằng cách sử dụng ../${NAME}.

- **Reformat according to style**: Nội dung tạo ra từ template sẽ được định dạng theo quy tắc code
  style đã thiết lập cho loại tệp đó.

- **Enable Live Templates**: Chèn các live templates vào trong template tệp. Sử dụng cú pháp
  escape
  của [Velocity](https://velocity.apache.org/engine/devel/user-guide.html) để bao gồm các biến
  live
  template trong file template, ví dụ: #[[ $MY_VARIABLE$ $END$ ]]#.

3. Chọn **Reformat according to style** để định dạng theo quy tắc code style của loại tệp đó.

4. Bật **Enable Live Templates** để chèn live templates vào template tệp. Sau đó, áp dụng thay đổi
   và đóng hộp thoại.


- Truy cập
  trang [velocity](https://velocity.apache.org/engine/devel/user-guide.html#Velocity_Template_Language_VTL:_An_Introduction)
  để tham khảo cú pháp

[//]: # (  ![Jetbra]&#40;assets/img/posts/20240907/1.png&#41;{: width="972" height="589" })
_Màn hình thêm mới template_

## 2. Syntax

- Văn bản thuần được giữ nguyên.
- Các biến được thay thế bằng giá trị tương ứng, ví dụ: `${NAME}` chèn tên do người dùng cung cấp
  khi tạo tệp.
- Các chỉ thị khác nhau như `#parse`, `#set`, `#if`, và các chỉ thị khác.

## 3. Ví dụ

### 3.1 Tạo mới file template để tạo 1 jekyll new post

1. Mong muốn: 1 file sẽ có dạng: `YYYY-MM-DD-(file-name).md` và Nội dung file sẽ có dạng:

  ```markdown
---
title: "Sử dụng file template trong Intellij để tạo nhanh file theo mẫu"
description: "mô tả"
author: huy8895
date: 2024-09-07 09:15:00 +0700
categories: [ Tool, Intellij ]
tags: [ Tips, Intellij, 'File Template' ]
pin: true
math: true
mermaid: true
image:
path: assets/img/posts/20240907/0.png
---
  ```

2. Cách thực hiện:

- Tạị màn hình **File and Code Templates** chọn thêm mới template **+**
- **name**: `Jekyll post` tên của file template
- **Extensions**: `md` extension của file muốn tạo.
- **File name**:

```text
${YEAR}-${MONTH}-${DAY}-${NAME}.md
```

- Nội dung file template:

```markdown
---
title: ""
description: ""
author: ${USER}
date: ${YEAR}-${MONTH}-${DAY} ${HOUR}:${MINUTE}:00 +0700
categories: []
tags: []
pin: true
math: true
mermaid: true
image:
  path: assets/img/posts/20240907/0.png
---
```

> Trên đây là cách mà mình sử dụng file template trong Intellij để tắng hiệu quả cho công việc, rất
> mong giúp ích được cho các bạn.

Tham khảo thêm tại: [File Template](https://www.jetbrains.com/help/idea/using-file-and-code-templates.html)

