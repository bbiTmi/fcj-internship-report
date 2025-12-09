---
title: "Worklog Tuần 11"
date: ""
weight: 2
chapter: false
pre: " <b> 1.11. </b> "
---
### Mục tiêu tuần 11:

* Tinh chỉnh bộ dữ liệu hoàn chỉnh cho dự án nhằm chuẩn hóa đầu vào cho các service backend.  
* Tìm hiểu luồng dữ liệu từ Cognito vào database và đánh giá lựa chọn lưu trữ phù hợp (S3 vs RDS vs DynamoDB).

###  Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---- | :---- | :---- | :---- | :---- |
| 3 | Tinh chỉnh và tạo bộ data hoàn chỉnh cho dự án | 18/11/2025 | 20/11/2025 |   |
| 6 | \- Hiểu cách Cognito ghi dữ liệu vào database theo thời gian thực \- Cân nhắc chuyển đổi sang lưu dữ liệu trên S3 hoặc RDS | 21/11/2025 | 21/11/2025 |   |

###  Kết quả đạt được tuần 9:

#### Cognito User Pool không tự động ghi dữ liệu vào database

\-\> Nhưng có thể kích hoạt dữ liệu theo thời gian thực bằng:

* Post Confirmation Trigger \-\> Lambda \-\> ghi vào DynamoDB / RDS.  
* Pre Token Generation Trigger \-\> thêm custom claims.  
* Sync thông tin user thông qua Cognito Identity Pool (nếu dùng).

#### So sánh S3/DynamoDB/RDS

| Dịch vụ | Ưu điểm | Nhược điểm | Phù hợp khi |
| :---- | :---- | :---- | :---- |
| S3 | Rẻ, lưu file/dataset lớn, backup | Không truy vấn nhanh, không transactional | Lưu lịch sử thao tác, log, dataset |
| RDS | SQL mạnh, truy vấn phức tạp | Chi phí cao hơn | Lưu thông tin quan hệ, báo cáo |
| DynamoDB | Nhanh, scale lớn, event-driven | Không hỗ trợ join, schema phẳng | Realtime user profiles, session data |

      