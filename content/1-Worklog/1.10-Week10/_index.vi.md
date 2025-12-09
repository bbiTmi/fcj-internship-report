---
title: "Worklog Tuần 10"
date: ""
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---
### Mục tiêu tuần 10:

* Hoàn thiện bộ dữ liệu dự án, bổ sung các thuộc tính cần thiết để phục vụ quá trình huấn luyện và xác thực người dùng thông qua AWS Cognito.  
* Sử dụng LocalStack để mô phỏng AWS nhằm thử nghiệm chức năng một cách an toàn, tiết kiệm chi phí và tăng tốc phát triển.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Tiếp tục tinh chỉnh dữ liệu, bổ sung thêm các feature cho AWS Cognito | 10/11/2025 | 12/11/2025 |   |
| 5 | Thử nghiệm dùng LocalStack mô phỏng môi trường AWS để test các chức năng | 13/11/2025 | 14/11/2025 |   |

###  Kết quả đạt được tuần 10:

#### Thử nghiệm AWS bằng LocalStack

LocalStack giúp mô phỏng môi trường AWS tại máy local mà không tốn chi phí, hỗ trợ nhiều dịch   
**Cài đặt và cấu hình LocalStack**

* Sử dụng Docker để chạy LocalStack.  
* Cấu hình AWS CLI để trỏ tới endpoint LocalStack thay vì AWS thật.

**Thử nghiệm các chức năng hệ thống**

* Tạo bảng DynamoDB local  
* Tạo bucket S3 thử nghiệm  
* Kiểm thử API backend mà không cần deploy lên AWS  
* Giảm rủi ro, tiết kiệm thời gian deploy và chi phí thử nghiệm