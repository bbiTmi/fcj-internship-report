---
title: "Worklog Tuần 2"
date: ""
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---
### Mục tiêu tuần 2:

* Hiểu rõ các khái niệm cốt lõi về Amazon VPC để xây dựng nền tảng mạng an toàn và chuẩn best practice.  
* Thực hành tạo VPC, Subnet, Route Table, Gateway trên AWS để nắm cơ chế hoạt động thực tế.  
* Biết cách vẽ kiến trúc mạng bằng Draw.io

### Các công việc cần triển khai trong tuần này:

 

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---- | :---- | :---- | :---- | :---- |
| 4 | Học khái niệm về VPC \- Tìm hiểu Public Cloud/Private Cloud \- Phân biệt Public Subnet, Private Subnet và VPN-only Subnet \- Route table, Destination/Target \- Internet Gateway vs NAT Gateway | 16/09/2025 | 17/09/2025 | [https://www.youtube.com/watch?v=O9Ac\_vGHquM\&list=PLahN4TLWtox2a3vElknwzU\_urND8hLn1i\&index=25](https://www.youtube.com/watch?v=O9Ac_vGHquM&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=25)   |
| 4 | Thực hành tạo VPC trên AWS | 17/09/2025 | 17/09/2025 | [Amazon VPC and AWS Site-to-Site VPN Workshop](https://000003.awsstudygroup.com/)   |
| 6 | Vẽ kiến trúc với Draw.io | 19/09/2025 | 19/09/2025 | [Hướng dẫn vẽ kiến trúc AWS trên draw.io](https://www.youtube.com/watch?v=l8isyDe-GwY&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=2)    |

###  Kết quả đạt được tuần 2:

#### VPC và sự khác nhau giữa Public Cloud \- Private Cloud

* Public Cloud chỉ nói về việc hạ tầng do AWS sở hữu; Private Cloud liên quan đến mạng cô lập và bảo mật theo yêu cầu doanh nghiệp.  
* VPC trong AWS là dạng Virtual Private Cloud \- private theo logic, nhưng vẫn dựa trên hạ tầng public của AWS.

#### Phân biệt Public Subnet \- Private Subnet \- VPN-only Subnet

* Public Subnet: có Route Table trỏ ra Internet Gateway (IGW) \-\> cho phép tài nguyên trong subnet truy cập internet trực tiếp.  
* Private Subnet: không kết nối IGW \-\> muốn ra internet phải đi qua NAT Gateway hoặc NAT Instance.  
* VPN-only Subnet: chỉ dùng để kết nối về on-premises qua VPN Gateway, không đi internet.

#### Route Table \- Destination \- Target

* Route Table định nghĩa đường đi của lưu lượng trong VPC.  
* Destination: địa chỉ mạng muốn gửi traffic đến  
* Target: cổng hoặc thành phần sẽ xử lý và chuyển tiếp traffic đến Destination.  
  Route Table cho Public Subnet  
* Ví dụ: 

Route table cho **Public Subnet**

| Destination | Target |
| :---- | :---- |
| 0.0.0.0/0 | igw-12345 |

**Giải thích:**  
 \-\> Mọi traffic ra ngoài internet (0.0.0.0/0) phải đi qua Internet Gateway.  
 Route Table cho **Private Subnet**

| Destination | Target |
| :---- | :---- |
| 0.0.0.0/0 | nat-67890 |

Giải thích:  
 \-\> Instance trong Private Subnet muốn ra internet phải đi qua NAT Gateway (không nhận inbound).

#### Internet Gateway vs NAT Gateway

| Thành phần | Công dụng | Dùng trong |
| :---- | :---- | :---- |
| Internet Gateway | Cho phép instance trong public subnet **truy cập** internet và **nhận traffic** vào từ internet | Public subnet |
| NAT Gateway | Cho phép instance trong private subnet chỉ được outbound ra internet nhưng **không nhận traffic** inbound từ internet | Private subnet |

####  Vẽ kiến trúc với Draw.io

* Biết cách diễn tả VPC, subnet, AZ, IGW, NAT Gateway, EC2 bằng icon.  
* Hiểu tiêu chuẩn vẽ kiến trúc AWS.  
* Sơ đồ dễ nhìn giúp truyền đạt kiến trúc rõ ràng hơn khi báo cáo hoặc trình bày.