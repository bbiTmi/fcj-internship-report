---
title: "Các bài blogs đã dịch"
date: ""
weight: 3
chapter: false
pre: " <b> 3. </b> "
---
###  [Blog 1 - Building geolocation verification for iGaming and sports betting on AWS](3.1-Blog1/)
Blog này trình bày lý do ngành iGaming cần geolocation verification để đáp ứng yêu cầu pháp lý và ngăn chặn gian lận như VPN spoofing hoặc proxy betting. Bài viết giới thiệu các phương pháp xác minh vị trí bằng Route 53, Amazon Location Service và CloudFront, nêu rõ ưu – nhược điểm của từng kỹ thuật. Bạn sẽ biết cách chọn giải pháp phù hợp để chặn truy cập trái phép ngay tại biên, cải thiện bảo mật và tối ưu chi phí hạ tầng.

###  [Blog 2 - How to run AI model inference with GPUs on Amazon EKS Auto Mode](3.2-Blog2/)
Blog này giải thích cách EKS Auto Mode giúp đơn giản hóa việc vận hành GPU cho AI inference bằng cách tự động xử lý node, driver, scaling và các bản vá bảo mật. Bài viết hướng dẫn cách tạo GPU NodePool, triển khai mô hình LLM với vLLM và tận dụng Karpenter để scale tài nguyên theo nhu cầu. Bạn sẽ hiểu vì sao Auto Mode cho phép các nhóm kỹ thuật tập trung vào tối ưu mô hình AI thay vì phòng thủ vận hành Kubernetes.

###  [Blog 3 - Enforcing organization-wide Amazon S3 bucket-tagging policies](3.3-Blog3/)
Blog này giới thiệu cách các tổ chức tự động hóa việc kiểm soát và chuẩn hóa resource tagging cho Amazon S3 để phục vụ quản trị, phân bổ chi phí và tuân thủ bảo mật. Bài viết giải thích cách kết hợp AWS Config, EventBridge và Lambda để tự động chặn hoặc cho phép upload tùy theo trạng thái tuân thủ tag. Bạn cũng sẽ học mô hình triển khai theo kiểu hub-and-spoke cùng CloudFormation StackSets để áp dụng chính sách tag trên toàn bộ tài khoản AWS trong tổ chức.
