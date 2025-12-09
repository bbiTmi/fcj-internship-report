---
title: "Worklog Tuần 4"
date: ""
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---
### Mục tiêu tuần 4:

* Hiểu và thực hành triển khai lưu trữ web tĩnh bằng Amazon S3 kết hợp CloudFront.  
* Làm quen cách sử dụng AWS CLI thông qua VSCode để thao tác dịch vụ nhanh và chuyên nghiệp hơn.  
* Thực hành tạo và quản lý Amazon RDS (MySQL).

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---- | :---- | :---- | :---- | :---- |
| 2   | Nắm kiến thức cơ bản về S3 và CloudFront | 29/09/2025 | 29/09/2025 |   |
|  | Thực hành: \- Chuyển S3 bucket thành web tĩnh  \- Cấu hình truy cập cho trang web \- Tạo CloudFront distribution \- Thực hành với Bucket Versioning | 29/09/2025 | 30/09/2025 | [with Amazon S3](https://000057.awsstudygroup.com/)   |
| 3 | Connect AWS với VSCode và sử dụng CLI để thao tác với dịch vụ | 30/09/2025 | 30/09/2025 |   |
| 4 | Thực hành: Tạo các RDS instance và thao tác với MySQL trên AWS | 01/10/2025 | 01/10/2025 | [Amazon Relational Database Service (Amazon RDS)](https://000005.awsstudygroup.com/)   |

###  Kết quả đạt được tuần 4:

#### Hiểu cơ bản về S3 và cơ chế static website hosting

* S3 là object storage, thích hợp lưu trữ web tĩnh (HTML/CSS/JS).  
* Static Web Hosting yêu cầu bật Static website hosting và chỉ định index.html & error.html.  
* S3 bucket cần cấu hình public access hoặc dùng CloudFront để phân phối nội dung an toàn hơn.

#### Thực hành triển khai website tĩnh trên S3

* Upload source web vào S3  
* Bật static hosting  
* Thêm bucket policy để cho phép public read  
* Kiểm thử truy cập bằng URL endpoint

#### Tạo và cấu hình CloudFront Distribution

* Origin \= S3 bucket  
* Behavior: cache, TTL, HTTP/HTTPS  
* Distribution domain name  
* Invalidations khi cập nhật website

CloudFront giúp:

* Cải thiện tốc độ phân phối nội dung  
* Ẩn trực tiếp S3 khỏi người dùng  
* Hỗ trợ HTTPS qua ACM Certificate

#### Kết nối AWS với VSCode \+ AWS CLI

Thiết lập: 

* AWS Toolkit trong VSCode  
* Cấu hình credentials (aws configure)  
* Thao tác S3, EC2, RDS bằng lệnh CLI

#### Thực hành Amazon RDS (MySQL)

* Tạo RDS MySQL instance  
* Cấu hình subnet group, security group  
* Kết nối bằng MySQL Workbench  
* Tạo database & chạy lệnh SQL cơ bản