---
title: "Worklog Tuần 9"
date: ""
weight: 1
chapter: false
pre: " <b> 1.9. </b> "
---
### Mục tiêu tuần 9:

* Thành thạo thao tác DynamoDB bằng CLI và hiểu rõ cấu trúc bảng NoSQL.  
* Tạo dữ liệu test bằng AI để kiểm thử hệ thống.  
* Thống nhất kiến trúc tổng thể của dự án, lựa chọn dịch vụ AWS phù hợp và tính chi phí triển khai.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---- | :---- | :---- | :---- | :---- |
| 2 | \- Thực hành viết CLI để tạo bảng trong DynamoDB \- Dùng AI gen data test thử hệ thống | 03/11/2025 | 03/11/2025 |   |
| 3 | \- Thảo luận và thống nhất các service dùng cho dự án, vẽ lại kiến trúc và tính toán chi phí với Price Calculator, phân chia công việc và vai trò cho từng thành viên  | 04/11/2025 | 04/11/2025 |   |
| 4 | \- Tạo mock data cho dự án, tinh chỉnh các feature cho phù hợp với AWS Personalize | 05/11/2025 | 07/11/2025 |   |

###  Kết quả đạt được tuần 9:

#### Thao tác DynamoDB bằng AWS CLI

* Cách tạo bảng DynamoDB bằng CLI, định nghĩa Partition Key \- Sort Key.  
* Cách sử dụng lệnh để thêm (put-item), đọc (get-item), truy vấn (query), scan dữ liệu.  
* Hiểu rõ JSON structure và cách mapping dữ liệu NoSQL.

#### Khó khăn gặp phải

* Cú pháp CLI của DynamoDB dài và dễ sai định dạng JSON.  
* Mock data ban đầu thiếu tính logic dẫn đến sai lệch khi huấn luyện mô hình Personalize.

#### Cách giải quyết & Bài học rút ra

* Sử dụng file JSON template để tránh khai báo thiếu hoặc sai định dạng.  
* Điều chỉnh mock data theo pattern hành vi người dùng thực tế