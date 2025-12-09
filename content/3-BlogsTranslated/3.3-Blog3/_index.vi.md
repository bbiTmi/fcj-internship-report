---
title: "Blog 3"
date: ""
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---
# Thực thi chính sách gắn thẻ (tagging) trên toàn tổ chức cho Amazon S3 Buckets

Trong các môi trường đám mây phức tạp ngày nay, việc duy trì tính nhất quán trong resource tagging (gắn thẻ tài nguyên) là một thách thức quan trọng mà các tổ chức ở mọi quy mô đều phải đối mặt. Resource tagging đúng cách đóng vai trò thiết yếu trong phân bổ chi phí, tuân thủ bảo mật, quản lý vận hành, và duy trì quản trị ở quy mô lớn. Tuy nhiên, việc thực thi các tiêu chuẩn tagging trên các nhóm phân tán và số lượng lớn tài nguyên có thể rất khó khăn, đặc biệt khi phải xử lý chu kỳ triển khai nhanh và các yêu cầu đến từ nhiều bên liên quan khác nhau. Nhiều tổ chức thường gặp khó khăn trong việc duy trì các thực hành tagging nhất quán, dẫn đến khả năng quan sát tài nguyên kém, phân bổ chi phí không chính xác, và gặp trở ngại trong việc duy trì tuân thủ với các chính sách nội bộ.

AWS cung cấp nhiều tính năng giúp các tổ chức triển khai và duy trì quản lý tagging hiệu quả cho [Amazon S3](https://aws.amazon.com/s3/), dịch vụ lưu trữ đám mây đóng vai trò nền tảng cho vô số tác vụ trong các trường hợp sử dụng như phát triển, phân tích dữ liệu, sao lưu, phân phối nội dung và nhiều hơn nữa. Khi các triển khai lưu trữ mở rộng và trở nên phân tán hơn giữa các nhóm và dự án, chúng thường trở thành ví dụ điển hình cho các thách thức về resource tagging. Thông qua sự kết hợp giữa các biện pháp kiểm soát chủ động và phản ứng (proactive and reactive controls), các tổ chức có thể thiết lập các cơ chế tự động để đảm bảo rằng tài nguyên đáp ứng đầy đủ các yêu cầu về tagging. Các biện pháp kiểm soát này có thể được áp dụng trên toàn bộ AWS [Organization](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html), giúp cung cấp khả năng giám sát tập trung đồng thời vẫn duy trì tính linh hoạt cần thiết cho các đơn vị kinh doanh và tác vụ khác nhau.

Trong bài viết này, tôi sẽ minh họa cách triển khai một giải pháp quản lý tagging tự động dạng phản ứng (reactive) cho S3 buckets bằng cách sử dụng [AWS Config](https://aws.amazon.com/config/), [AWS Lambda](https://aws.amazon.com/lambda/), và [Amazon EventBridge](https://aws.amazon.com/eventbridge). Bạn sẽ học cách triển khai một giải pháp có khả năng tự động hạn chế việc tải lên đối tượng vào các bucket không tuân thủ, và gỡ bỏ các hạn chế này khi các tag cần thiết được áp dụng. Cách tiếp cận này cho phép các tổ chức thực thi tiêu chuẩn tagging mà không cần can thiệp thủ công, đảm bảo phân loại tài nguyên nhất quán và phân bổ chi phí chính xác — đồng thời giảm thiểu tác động đến các nhóm phát triển và quy trình làm việc hiện có.

## Giới thiệu về giải pháp

Giải pháp này sử dụng cách tiếp cận phản ứng (reactive approach) trong quản lý gắn thẻ tài nguyên (tagging), dựa trên dịch vụ AWS Config với quy tắc tuân thủ có sẵn “required-tags”. Quy tắc này giám sát và xác định các tài nguyên thiếu tag cần thiết. Sau đó, quy tắc sẽ áp dụng chính sách tài nguyên cho các bucket không tuân thủ nhằm chặn việc tải lên đối tượng mới. Khi chủ sở hữu bucket thêm các tag cần thiết, giải pháp sẽ tự động gỡ bỏ chính sách, hủy bỏ hạn chế tải lên đối tượng. Với một số điều chỉnh nhỏ trong cấu hình, giải pháp này có thể được mở rộng để thực thi tagging cho các loại tài nguyên AWS khác.

Giải pháp được triển khai theo mô hình hub-and-spoke, trong đó việc giám sát tuân thủ được thực hiện ở cấp tài khoản và [AWS Region](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/). Sau đó, các thay đổi về trạng thái tuân thủ sẽ được chuyển tiếp đến một tài khoản trung tâm, tài khoản này chịu trách nhiệm thực hiện hành động khi một S3 bucket chuyển từ trạng thái tuân thủ (compliance) sang không tuân thủ (non-compliant), hoặc ngược lại. Giải pháp này sử dụng AWS Config để đánh giá trạng thái tuân thủ và phát hiện thay đổi, EventBridge để truyền thông tin thay đổi, và Lambda để thực hiện hành động khắc phục tự động.

Sơ đồ sau minh họa các thành phần được triển khai như một phần của giải pháp này:

![S3 Tagging Governance Solution Diagram](/images/3-Blog/3.3-Blog3/S3-Tagging-Governance-Solution-Diagram.png)

*Hình 1: Sơ đồ kiến trúc của các tài nguyên AWS được triển khai bởi các CloudFormation template vào tài khoản quản lý và tài khoản được liên kết để hỗ trợ cho giải pháp này.*

Mỗi tài khoản được triển khai giải pháp sẽ có một quy tắc AWS Config (AWS Config rule) dùng để giám sát việc tuân thủ quy định gắn thẻ (tagging compliance) của các S3 bucket. Mỗi tài khoản/Region được liên kết cũng có một EventBridge rule để chuyển tiếp thông báo về thay đổi trạng thái tuân thủ đến tài khoản quản lý giải pháp (hoặc tài khoản điều phối được chỉ định).

Tài khoản quản lý giải pháp có EventBridge bus tập trung, một rule, và một Lambda function chịu trách nhiệm cập nhật chính sách của S3 bucket dựa trên trạng thái tuân thủ tagging. Nếu bạn muốn thực thi tagging ngay trên tài khoản quản lý (hoặc tài khoản điều phối được chỉ định) đồng thời sử dụng nó để điều phối, hãy đảm bảo rằng cả hai mẫu CFN đều được triển khai trong tài khoản đó. Trong trường hợp này, tất cả các thành phần được minh họa trong hình trước đều sẽ được triển khai trong cùng tài khoản.

Giải pháp này cũng khuyến nghị bạn sử dụng khả năng quản lý chủ động với [AWS Organization tag policies ](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_tag-policies.html)(tùy chọn). Tag policies cho phép bạn đảm bảo tính nhất quán trong cách đặt tên các tag key và giới hạn các giá trị được phép gán. Các [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template được cung cấp cùng với bài viết này không triển khai tag policies, vì vậy bạn cần cấu hình chúng riêng biệt.

1. **Giám sát:** Một AWS Config rule trong mỗi tài khoản và AWS Region thuộc tổ chức của bạn sẽ giám sát các S3 bucket nhằm đảm bảo chúng tuân thủ chính sách tagging được định nghĩa trong quy tắc. Khi trạng thái tuân thủ thay đổi, quy tắc sẽ gửi một sự kiện đến Default EventBridge bus trong tài khoản đó.

2. **Chuyển tiếp sự kiện:** EventBridge rule trong từng tài khoản riêng lẻ sẽ nhận sự kiện này và chuyển tiếp nó đến custom EventBridge bus tập trung, được lưu trữ trong tài khoản quản lý (hoặc tài khoản điều phối được chỉ định).

3. **Xử lý:** Một rule được thiết lập trên custom EventBridge bus trong tài khoản quản lý (hoặc tài khoản điều phối được chỉ định) sẽ nhận sự kiện, sau đó gửi đến Lambda function để xử lý. Quy tắc này cũng ghi lại sự kiện trong [Amazon CloudWatch Logs](https://aws.amazon.com/cloudwatch/) nhằm lưu trữ nhật ký.

4. **Áp dụng chính sách:** Lambda function sẽ kiểm tra xem trạng thái tuân thủ của S3 bucket có phải là “Non-compliant” hay không, và cập nhật chính sách tài nguyên của bucket để ngăn việc tải lên bất kỳ đối tượng nào bằng chính sách tài nguyên (resource policy) sau đây:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "BlockFileUploadToBucket",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::<bucket name>/*"
        }
    ]
}
```

Hàm Lambda trong tài khoản quản lý (hoặc tài khoản điều phối được chỉ định) sẽ cập nhật chính sách của bucket trong tài khoản được liên kết. Quá trình này được thực hiện bằng cách giả định một vai trò được triển khai thông qua StackSets đến mỗi tài khoản mục tiêu.

5. **Khắc phục:** Khi bucket trở lại trạng thái “Compliant”, tức là khi các tag cần thiết đã được áp dụng cho S3 bucket, Lambda function sẽ gỡ bỏ hạn chế tải lên đối tượng bằng cách xóa chính sách tài nguyên của bucket.

## Quy trình triển khai

Để triển khai giải pháp này, hãy sử dụng hai [CloudFormation ](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)template được cung cấp [trong dự án GitHub](https://github.com/aws-samples/Creating-s3-tagging-governance-across-aws-organization) kèm theo. Một template được thiết kế để triển khai trong tài khoản quản lý hoặc tài khoản được chỉ định làm tài khoản trung tâm cho giải pháp. Mẫu CFN thứ hai sẽ được triển khai trong từng tài khoản và AWS Region nơi cần thực thi quy tắc tagging. Việc triển khai này có thể được thực hiện trực tiếp bằng CloudFormation Stacks hoặc thông qua CloudFormation StackSets từ tài khoản quản lý của Organization.

Mẫu CloudFormation **s3-tagging-governance-mngt.yaml** triển khai các thành phần được thiết kế để chạy trong tài khoản quản lý (hoặc tài khoản điều phối được chỉ định) của bạn.

- Bạn có thể triển khai mẫu này trong tài khoản quản lý của mình hoặc trong tài khoản khác đóng vai trò tài khoản điều phối được chỉ định, chịu trách nhiệm quản lý thông báo không tuân thủ và thực hiện các hành động khắc phục.
- Hãy triển khai tại một Region duy nhất, nơi phần lớn tài nguyên của bạn được triển khai.
- Sử dụng CloudFormation Stacks để triển khai mẫu này.
- Trong quá trình triển khai, bạn cần cung cấp OrgID của Organization. Để lấy OrgID: vào Organizations Console → **Settings**, mục **Organization Detail** sẽ hiển thị Organization ID.
- Khi quá trình triển khai hoàn tất thành công, truy cập tab **Outputs** của CFN Stack vừa triển khai và lưu lại các giá trị sau (bạn sẽ cần sử dụng chúng khi triển khai mẫu CFN s3-tagging-governance-linked.yaml):

    a. **CentralEventBusArn:** chứa [Amazon Resource Name (ARN)](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html) của Event Bus trung tâm đã được triển khai.

    b. **LambdaAssumeRole:** chứa tên của role mà Lambda sẽ assume trong quá trình thực thi để triển khai cập nhật chính sách tài nguyên trong từng tài khoản được liên kết.

    c. **LambdaMngtExecutionRoleArn:** chứa ARN của Lambda execution role được triển khai trong tài khoản quản lý (hoặc tài khoản điều phối được chỉ định).

Mẫu CloudFormation **s3-tagging-governance-linked.yaml** sẽ được triển khai vào các tài khoản và Region nơi bạn muốn thực hiện quản lý tagging.

- Triển khai bằng CloudFormation StackSets hoặc CFN Stacks thông thường.
- Hãy triển khai trong từng tài khoản và AWS Region nơi bạn muốn thực thi quy tắc tagging.
- Khi triển khai mẫu này, hãy sử dụng các giá trị mà bạn đã ghi lại khi triển khai mẫu CFN s3-tagging-governance-mngt.yaml:

    a. **ExecLambdaRoleName**: sử dụng giá trị đầu ra (output parameter) chứa tên của role mà Lambda sẽ assume trong quá trình thực thi để cập nhật chính sách tài nguyên trong từng tài khoản được liên kết.

    b. **MngtEventBusArn**: sử dụng giá trị đầu ra chứa ARN của Event Bus được triển khai trong tài khoản quản lý (hoặc tài khoản điều phối được chỉ định).

    c. **MngtLambdaRoleArn**: sử dụng giá trị đầu ra chứa ARN của Lambda execution role được triển khai trong tài khoản quản lý (hoặc tài khoản điều phối được chỉ định).

- Hãy đảm bảo triển khai mẫu này trong tài khoản quản lý giải pháp (hoặc tài khoản điều phối được chỉ định) cùng với s3-tagging-governance-mngt.yaml, nếu bạn cũng muốn thực thi quy tắc tuân thủ tagging cho S3 buckets trong tài khoản đó.

Các CloudFormation template không triển khai tag policies, tuy nhiên bạn có thể tự định nghĩa chúng bằng cách làm theo hướng dẫn tại đây: [AWS Organizations – Tag Policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_tag-policies-getting-started.html).

## Một vài cân nhắc quan trọng

- Giải pháp được đóng gói hiện tại sử dụng tag để xác định phạm vi cho AWS Config rule. Tag mà Config rule tìm kiếm có key là “TagsRequired-PLEASE-UPDATE” và value là “Yes”. Trước khi triển khai giải pháp, hãy cập nhật các CloudFormation template để đổi tag thành một tag được dùng để xác định bucket nào sẽ được quản lý bởi chính sách này, hoặc sử dụng kỹ thuật khác để[ định nghĩa phạm vi cho AWS Config Rule](https://docs.aws.amazon.com/config/latest/APIReference/API_Scope.html) (ví dụ resource).

- **Lưu ý quan trọng:** Khi triển khai giải pháp này, hãy cân nhắc kỹ những S3 bucket nào sẽ được nhắm mục tiêu (tham khảo bullet trước đó) và tác động của việc này đến các tác vụ hiện có đang sử dụng các bucket đó. Khi giải pháp áp dụng resource policy lên các bucket không tuân thủ, các tác vụ sẽ không còn có thể tải lên tệp vào những bucket này nữa, điều này có thể ảnh hưởng tiêu cực đến tác vụ của bạn.

- Nếu S3 bucket của bạn đã có bucket resource policy, và một bucket không tuân thủ (tức là chưa có các tag cần thiết) đã có chính sách hiện có, thì chính sách này sẽ được cập nhật để bổ sung một statement nhằm chặn việc tải lên tệp, theo quy tắc của giải pháp này. Các statement hiện có của bạn sẽ không bị ảnh hưởng. Lambda function sẽ kiểm tra tất cả các statement bằng Statement ID (SID) để xác định xem statement có ID “BlockFileUploadToBucket” đã tồn tại hay chưa, và thêm vào nếu chưa có. Khi bucket trở lại trạng thái tuân thủ (Compliant), Lambda sẽ sử dụng cùng SID đó để xóa statement này khỏi policy.

- Để ngăn người dùng tự chỉnh sửa chính sách của S3 bucket, hãy sử dụng Service Control Policies (SCPs) hoặc [chính sách AWS Identity and Access Management (IAM) ](https://aws.amazon.com/iam/)để giới hạn quyền của người dùng, bằng cách từ chối rõ ràng (explicit deny) các hành động như “s3:PutBucketPolicy” và “s3:DeleteBucketPolicy”.

## Khách hàng tiêu biểu: Marriott International

Nhóm Cloud FinOps của[ Marriott International](https://www.marriott.com/) dựa vào resource tag để phục vụ cho báo cáo tài chính. Các tag này được sử dụng để xác định chủ sở hữu tài nguyên và phân bổ chi phí cho các tác vụ và đơn vị kinh doanh tương ứng. Mặc dù họ đã có chính sách yêu cầu các nhóm phải sử dụng tagging, nhưng tỷ lệ tuân thủ rất thấp, và một phần lớn chi phí bị báo cáo là không được phân loại. Để thực thi chính sách này, nhóm FinOps của Marriott đã khởi động dự án xây dựng quản lý tagging trên phần lớn các dịch vụ AWS được sử dụng trong toàn bộ hệ thống của họ.

Họ đã thành công khi sử dụng Service Control Policies (SCPs) để chặn việc tạo bất kỳ [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) instance hoặc [Amazon Elastic Block Store (Amazon EBS)](https://aws.amazon.com/ebs/) volume nào không có các tag cần thiết. Tuy nhiên, khi họ cố áp dụng cùng phương pháp này cho S3 bucket, thì nảy sinh một số thách thức. Trước hết, họ gặp phải giới hạn về kích thước ký tự như đã mô tả trước đó trong bài viết, và họ phát hiện rằng điều kiện tag (tag condition) không được hỗ trợ trong SCP policies cho Amazon S3.

Tại thời điểm đó, nhóm FinOps của Marriott đã hợp tác với AWS Technical Account Manager và Solution Architect để phát triển giải pháp được mô tả trong bài viết này. Sau khi triển khai giải pháp này trên toàn bộ Organization, họ đã giảm được 80% số lượng S3 bucket chưa được gắn tag. Nhờ đó, họ có thể báo cáo chính xác chi phí liên quan đến việc sử dụng Amazon S3 cho chủ sở hữu tài nguyên tương ứng cũng như ban lãnh đạo.

## Kết luận

Giải pháp này minh họa một phương pháp toàn diện để thực thi quy tắc tuân thủ gắn thẻ (tagging compliance) trên Amazon S3 Buckets, sử dụng AWS Config, Amazon EventBridge, và AWS Lambda trong mô hình hub-and-spoke. Giải pháp giám sát các S3 bucket trong toàn bộ AWS Organization, tự động hạn chế việc tải lên đối tượng vào các bucket không tuân thủ thông qua resource policies, và gỡ bỏ các hạn chế này khi các tag cần thiết được áp dụng. Các thành phần chính của giải pháp bao gồm AWS Config rules cho việc giám sát, EventBridge để định tuyến sự kiện, và Lambda functions để thực thi chính sách tự động. Tất cả những thành phần này đều có thể được triển khai thông qua các AWS CloudFormation templates được cung cấp.

Lợi ích của việc triển khai giải pháp này vượt ra ngoài phạm vi thực thi tagging. Các tổ chức có thể đạt được phân bổ chi phí và quản lý tài nguyên hiệu quả hơn thông qua việc gắn tag nhất quán, như minh chứng từ Marriott với mức giảm 80% số lượng S3 bucket chưa được gắn tag. Cách tiếp cận này khắc phục các giới hạn của SCPs, mang đến một phương pháp có khả năng mở rộng để thực thi quản lý tagging mà không gặp phải giới hạn ký tự của SCP hoặc các ràng buộc đặc thù của dịch vụ. Hơn nữa, khả năng khắc phục tự động của giải pháp giúp giảm thiểu khối lượng công việc quản trị, đồng thời đảm bảo tính tuân thủ liên tục.

Để bắt đầu triển khai giải pháp quản lý tagging này, hãy tải xuống các CloudFormation templates được cung cấp từ GitHub repository và làm theo hướng dẫn triển khai chi tiết. Hãy cân nhắc kỹ chiến lược lựa chọn bucket mục tiêu và các yêu cầu của tác vụ hiện có trước khi triển khai. Để có thêm hướng dẫn chuyên sâu hoặc nhu cầu tùy chỉnh, hãy tham khảo tài liệu AWS hoặc liên hệ với nhóm tài khoản AWS của bạn để đảm bảo quá trình triển khai thành công, phù hợp với các yêu cầu cụ thể của tổ chức.

TAGS: [Amazon EventBridge,](https://aws.amazon.com/blogs/storage/tag/amazon-eventbridge/) [Amazon Simple Storage Service (Amazon S3),](https://aws.amazon.com/blogs/storage/tag/amazon-simple-storage-service-amazon-s3/) [AWS Cloud Storage, ](https://aws.amazon.com/blogs/storage/tag/aws-cloud-storage/)[AWS Config, ](https://aws.amazon.com/blogs/storage/tag/aws-config/)[AWS Lambda](https://aws.amazon.com/blogs/storage/tag/aws-lambda/)


**Pavel Rabinovich**
<img src="/images/3-Blog/3.3-Blog3/Pavel-Rabinovich-Profile-Image-1.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Pavel Rabinovich là Quản lý Tài khoản Kỹ thuật Cấp cao (Senior Technical Account Manager) tại AWS, làm việc tại Washington, D.C., với hơn 25 năm kinh nghiệm trong lĩnh vực IT và kỹ thuật phần mềm. Ông tư vấn cho các khách hàng doanh nghiệp trong ngành khách sạn, y tế và bảo hiểm về chiến lược điện toán đám mây và bảo mật. Là thành viên của AWS Security Technical Field Community, ông đặc biệt tập trung vào bảo mật danh tính và ứng dụng.
<div style="clear: both;"></div>

**Avinash Pala**
<img src="/images/3-Blog/3.3-Blog3/Avinash-Pala-Profile-Image.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Avinash Pala là Kiến trúc sư Giải pháp Điện toán Đám mây (Cloud Solution Architect) và Kiến trúc sư Dữ liệu (Data Architect). Anh là chuyên gia được chứng nhận bởi AWS (AWS Certified).
<div style="clear: both;"></div>

**Reema Bali**
<img src="/images/3-Blog/3.3-Blog3/Reema-Bali-Profile-Image.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Reema Bali là Giám đốc – Tài chính Doanh nghiệp trên Đám mây (Director – Enterprise FinOps) tại Marriott International, chịu trách nhiệm kết nối giữa bộ phận Kỹ thuật và Tài chính, đảm bảo chi tiêu cho điện toán đám mây được tối ưu hóa mà không ảnh hưởng đến tốc độ hay đổi mới sáng tạo. Cô là một nhà lãnh đạo IT dày dặn kinh nghiệm, với hơn 25 năm làm việc trong ngành và thành tích xuất sắc trong việc triển khai các chương trình quy mô lớn, có tác động cao, phù hợp với mục tiêu chiến lược của doanh nghiệp.
<div style="clear: both;"></div>

**Ruby Siddiqui**
<img src="/images/3-Blog/3.3-Blog3/Ruby-Siddiqui-Profile-Image-1.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Ruby Siddiqui là Giám đốc Cấp cao mảng Kỹ thuật FinOps và Chiến lược Công nghệ (Senior Director of FinOps Engineering and Technology Strategy) tại Marriott International, nơi cô dẫn dắt các sáng kiến về vận hành tài chính và tối ưu hóa chi phí điện toán đám mây trên toàn doanh nghiệp. Với chuyên môn sâu về kinh tế học đám mây (cloud economics), phát triển ứng dụng, quan sát hệ thống và quản trị, cô xây dựng và triển khai các thực hành FinOps có khả năng mở rộng, giúp mang lại tính minh bạch về chi phí và giá trị kinh doanh trong môi trường đa đám mây (multi-cloud).
<div style="clear: both;"></div>

**Sri Gudavalli**
<img src="/images/3-Blog/3.3-Blog3/srgudava.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Sri Gudavalli là Kiến trúc sư Giải pháp (Solutions Architect) tại AWS, nơi ông hỗ trợ các khách hàng doanh nghiệp trong hành trình di chuyển và hiện đại hóa hệ thống lên đám mây. Ông làm việc với các doanh nghiệp tại khu vực miền Đông Hoa Kỳ (US-East Region) để xây dựng các ứng dụng và dịch vụ điện toán đám mây tiên tiến trên nền tảng AWS.
<div style="clear: both;"></div>