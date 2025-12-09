---
title: "Worklog Tuần 1"
date: ""
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---
### Mục tiêu tuần 1:

* Làm quen với tài khoản AWS và thiết lập các cấu hình cơ bản để đảm bảo quản lý chi phí, bảo mật và khả năng hỗ trợ kỹ thuật khi gặp sự cố.  
* Bắt đầu tìm hiểu về các dịch vụ nền tảng: AWS Account, AWS Budget, IAM, AWS Support.

### Các công việc cần triển khai trong tuần này:


| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Tạo tài khoản AWS | 08/09/2025 | 08/09/2025 | [Create a new AWS account.](https://000001.awsstudygroup.com/) |
| 3 | Tạo Budget | 09/09/2025 | 09/09/2025 | [COST MANAGEMENT WITH AWS BUDGETS](https://000007.awsstudygroup.com/) |
| 4 | \- Tìm hiểu và tạo IAM \- Tạo tài khoản IAM Admin cho các tác vụ hằng ngày | 09/09/2025 | 10/09/2025 | [AWS Identity and Access Management (IAM) Access Control](https://000002.awsstudygroup.com/) |
| 5 | \- Tìm hiểu AWS Support và gửi Support Case | 10/09/2025 | 11/09/2025 | [AWS Support](https://000009.awsstudygroup.com/) |

###  Kết quả đạt được tuần 1:

####  Tạo tài khoản AWS

* Hiểu được cấu trúc một tài khoản AWS gồm: root user, IAM users, billing, security.  
* Root user chỉ dùng cho tác vụ nhạy cảm (như thiết lập Billing, kích hoạt MFA, hỗ trợ tài chính).  
* Nắm được tầm quan trọng của MFA theo nguyên tắc bảo mật của AWS (Least Privilege & Multi-Factor Authentication).

#### Tạo AWS Budget

* Biết cách thiết lập Cost Budget để theo dõi chi phí và nhận email cảnh báo khi vượt mức.  
* Hiểu và phân biệt khái niệm của Actual cost (chi phí thực tế) vs Forecasted cost (chi phí dự đoán đến cuối tháng)

#### Tìm hiểu IAM và tạo IAM Admin

* Hiểu cấu trúc IAM: User \- Group \- Role \- Policy.  
* Phân biệt khái niệm của IAM User (con người/ứng dụng) và IAM Role (quyền được assume bởi service hoặc user tạm thời)  
* Tạo một IAM Admin user thay cho root để thao tác hằng ngày.

#### Tìm hiểu AWS Support và gửi Support Case

* Biết được 4 cấp độ Support: Basic, Developer, Business, Enterprise.  
* Dưới Free Tier, chỉ có Basic Support, bao gồm:  
  * Tài liệu, forum, Trusted Advisor checks cơ bản  
  * Gửi Support Case về billing  
* Biết cách tạo một ticket để làm quen quy trình hỗ trợ kỹ thuật.

### Khó khăn gặp phải

* Khó hiểu cấu trúc Policy JSON của IAM (Action, Resource, Effect).

### Cách giải quyết & Bài học rút ra

* Xem mẫu Policy trong AWS Documentation và sử dụng giao diện Visual Editor để dễ tạo IAM Policy.