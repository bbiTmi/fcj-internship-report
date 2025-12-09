---
title: "Worklog Tuần 3"
date: ""
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---
### Mục tiêu tuần 3:

* Hiểu khái niệm Amazon EC2 và nắm được sự khác nhau giữa các loại instance.  
* Thiết lập bảo mật mạng cơ bản bằng Security Group để đảm bảo kết nối an toàn.  
* Thực hành triển khai EC2 trên Linux & Windows, kết nối qua SSH và RDP.  
* Làm quen với việc cài đặt Web Server (LAMP/XAMPP) trên EC2 để chuẩn bị cho các bài học về kiến trúc ứng dụng.

### Các công việc cần triển khai trong tuần này:

 

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Học khái niệm về EC2, các loại instances | 22/09/2025 | 22/09/2025 | [Module 02-Lab03-04.1 - Create EC2 Instances in Subnets](https://www.youtube.com/watch?v=duJEdF_g1To&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=41) |
|  | Thiết lập Security Group cho VPC \- Cấu hình inbound/outbound phù hợp để kết nối SSH | 22/09/2025 | 22/09/2025 |   |
|  | Thực hành: Khởi tạo, chạy và kết nối với EC2 instance trên AWS \- Các thao tác cơ bản với EC2 | 22/09/2025 | 23/09/2025 | [Introduction to Amazon EC2](https://000004.awsstudygroup.com/)   |
| 4 | Triển khai EC2 instance trên Linux: \- Kết nối thông qua SSH \- Cách khôi phục quyền truy cập khi mất Key Pair \- Cài đặt LAMP Web Server | 24/08/2025 | 24/08/2025 |  |
| 5 | Triển khai EC2 instance trên Window: \- Kết nối thông qua Remote Desktop Protocol (RDP) \- Cài đặt XAMPP trên Windows instance | 24/08/2025 | 25/08/2025 |  |

###  Kết quả đạt được tuần 3:

#### Khái niệm và kiến trúc EC2

* EC2 là dịch vụ compute theo mô hình IaaS, cho phép tạo máy ảo (instances) theo nhu cầu.  
* Hệ sinh thái EC2 gồm:  
  * Instance Types  
  * AMI  
  * EBS Volume  
  * Key Pair  
  * Security Group  
  * Elastic IP  
  * User Data (script khởi tạo máy)

#### Security Group \- lớp tường lửa đầu tiên

* Security Group là stateful firewall  
* Inbound rule: kiểm soát traffic đi vào instance  
* Outbound rule: kiểm soát traffic đi ra ngoài

#### Triển khai EC2 Linux

1. Kết nối SSH  
2. Khôi phục khi mất key pair:  
   1. Tạo key pair mới  
   2. Dùng Puttygen để truy xuất và sao chép toàn bộ giá trị Public Key của Key Pair mới vừa tạo  
   3. Dừng instance  
   4. Chỉnh sửa User Data của instance, dán toàn bộ câu lệnh cloud-config, bao gồm Public Key đã sao chép, vào mục ssh-authorized-keys cho tên người dùng tương ứng (ví dụ: ec2-user). Public Key phải bắt đầu bằng chuỗi ssh-rsa  
   5. Khởi động lại instance  
3. Cài đặt LAMP stack (Apache, MariaDB. PHP)

#### Triển khai EC2 Windows

* Kết nối qua RDP (mstsc)  
* Lấy password Administrator thông qua key pair  
* Cài và cấu hình XAMPP  
* Truy cập web server từ trình duyệt

#### Khó khăn gặp phải

* Ban đầu kết nối SSH bị timeout do Security Group chưa mở đúng port  
* Khi cài LAMP, Apache chưa chạy được do Từ phiên bản Amazon 2023 có các thay đổi từ "amazon-linux-extras" sang "distro-stream repositories"

#### Cách giải quyết & Bài học rút ra

* Kiểm tra lại Security Group đã mở đủ và đúng cổng chưa  
* Tắt instance khi không sử dụng để tránh phát sinh chi phí.  
* Kiểm tra phiên bản Amazon và chạy đúng lệnh.