---
title: "Blog 1"
date: ""
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
# Xây dựng hệ thống xác minh vị trí địa lý cho iGaming và cá cược thể thao trên AWS

Xác minh vị trí địa lý trong cá cược thể thao và iGaming phục vụ hai mục đích chính: tuân thủ quy định và phòng chống gian lận. Các nhà vận hành cá cược thể thao trực tuyến và iGaming cần đảm bảo rằng chỉ người chơi từ một số khu vực nhất định trên thế giới mới có thể xem nội dung do các hạn chế về cung cấp giấy phép trên thị trường. Hoặc các quy định về trò chơi có thể yêu cầu quyền truy cập bị giới hạn trong phạm vi pháp lý cho phép trong khi chặn lưu lượng trái phép tại biên giới địa lý.

Tại Hoa Kỳ, [Federal Wire Act](https://www.law.cornell.edu/uscode/text/18/1084), đạo luật cấm cá cược thể thao xuyên bang, bắt đầu với đoạn văn sau:

Bất kỳ ai tham gia vào hoạt động kinh doanh cá cược hoặc cá cược mà cố ý sử dụng một hệ thống truyền thông điện tử để truyền tải trong phạm vi thương mại liên bang hoặc quốc tế các kèo cược hoặc hỗ trợ thông tin trong việc đặt cược vào bất kỳ sự kiện hoặc cuộc thi thể thao nào, hoặc để truyền tải thông tin điện tử cho phép người nhận nhận được tiền hoặc tín dụng từ kết quả của cá cược, hoặc thông tin hỗ trợ việc đặt cược, thì sẽ bị phạt tiền theo quy định tại điều luật này hoặc bị phạt tù không quá hai năm, hoặc cả hai hình phạt.

IGT và Xổ số bang New Hampshire đã[ thắng kiện](https://www.bhfs.com/insight/federal-court-clarifies-sort-of-the-wire-act-s-application-beyond-sports-betting/) Bộ Tư pháp Hoa Kỳ về việc không áp dụng đạo luật Wire Act cho iGaming và vận hành xổ số. Tuy nhiên, một số bang vẫn yêu cầu người chơi và hạ tầng kinh doanh cá cược phải được đặt trong cùng một bang. Trong các bang này, nhà vận hành phải có cơ chế phát hiện VPN, phần mềm, ảo hóa và các phương pháp khác có thể được sử dụng để lách việc xác định vị trí của người chơi. Thông thường điều này có nghĩa là một SDK phải được tích hợp vào ứng dụng hoặc một phần mềm khác (*[sidecar](https://medium.com/@justinricedev/what-is-a-software-sidecar-8f89feff09f9#:~:text=This%20creates%20a%20single%20point,Why%20Use%20Software%20Sidecars?)*) chạy song song.

Ngoài việc cho phép nhà điều hành tuân thủ quy định, kiểm soát truy cập theo vị trí địa lý còn mang lại lợi ích kinh doanh. Chặn lưu lượng ngay tại biên giúp giảm chi phí hạ tầng bằng cách loại bỏ truy cập trái phép trước khi nó đến hệ thống lõi. Các kiểm soát này cũng có thể ngăn chặn gian lận, ví dụ, cái gọi là *proxy betting* là nơi một người chơi trong khu vực được phép đặt cược thay cho nhiều người bên ngoài khu vực.

Lựa chọn công nghệ xác minh vị trí địa lý phụ thuộc vào trường hợp sử dụng cụ thể. Ở Hoa Kỳ, Brazil và một số khu vực khác, các nhà vận hành sử dụng hệ thống xác minh vị trí địa lý đã được cấp phép và kiểm định tuân thủ, với SDK truy cập trực tiếp vào hệ điều hành di động để đáp ứng yêu cầu chống giả mạo. Tuy nhiên, cũng có những giải pháp thay thế phù hợp cho nhà cung cấp nội dung hoặc nhà vận hành ở các khu vực khác.

Các phương pháp này có thể được sử dụng riêng lẻ hoặc kết hợp để tạo ra giải pháp phù hợp, tùy vào yêu cầu tuân thủ và kinh doanh. Hãy khám phá năm phương pháp để xây dựng và triển khai xác minh vị trí địa lý bằng Amazon Web Services (AWS):

## Geolocation blocking sử dụng Amazon Route 53

Khả năng [định tuyến vị trí địa lý (Geolocation Routing)](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-geo.html) của [Amazon Route 53](https://aws.amazon.com/vi/route53/) có thể giới hạn quyền truy cập nội dung theo quốc gia, châu lục hoặc bang của Hoa Kỳ ngay tại lớp máy chủ hệ thống tên miền (DNS server). Cách tiếp cận dựa trên DNS này xử lý quyết định về vị trí trước khi lưu lượng truy cập đến hạ tầng của bạn, giúp giảm tải không cần thiết và chi phí liên quan. Một lợi ích chính của giải pháp này là không yêu cầu chỉnh sửa mã nguồn trên server hoặc client hiện có. Việc triển khai sử dụng DNS records với các chính sách định tuyến theo vị trí địa lý, từ đó xác định vị trí của người dùng và phản hồi dựa trên các quy tắc đã được cấu hình cho khu vực địa lý đó. Dưới đây là ví dụ về chính sách định tuyến lưu lượng theo vị trí địa lý trong Route 53:

```
{
"RuleType": "geo",
    "Locations": [
        {"EvaluateTargetHealth": true,
        "Country": "SE",
                       "EndpointReference": ENDPOINT_IF_TRAFFIC_ORINGINATES_FROM_SWEDEN
        },
        {
                       "Country": "*",
                       "IsDefault": true,
                       "EvaluateTargetHealth": true,
                       "EndpointReference": ENDPOINT_FOR_ALL_OTHER_TRAFFIC
                       }
                   ]
}
```
Ví dụ về chính sách định tuyến lưu lượng đến các endpoint dựa trên việc quốc gia gốc có phải là Thụy Điển hay không. Trong ví dụ này, lưu lượng bắt nguồn từ Thụy Điển sẽ được gửi đến bộ cân bằng tải, trong khi lưu lượng đến từ các khu vực khác sẽ được chuyển hướng đến một trang lỗi được hosted trên [Amazon CloudFront](https://aws.amazon.com/vi/cloudfront/). Định tuyến vị trí địa lý Route 53 tích hợp trực tiếp với các dịch vụ AWS khác và bao gồm cả các fallback rules cho những vị trí không khớp.

Giải pháp định tuyến theo vị trí dựa trên DNS có một số hạn chế. Giải pháp này phụ thuộc vào độ chính xác của việc ánh xạ địa chỉ IP sang vị trí địa lý, vốn có thể bị qua mặt bởi các VPN hoặc proxy servers. Các phản hồi DNS có thể được bộ phân giải trung gian lưu trong bộ nhớ đệm, điều này có thể cho phép người dùng vẫn duy trì quyền truy cập từ một vị trí đã bị chặn sau khi họ di chuyển. Ngoài ra, giải pháp này không thể phát hiện việc can thiệp thiết bị, giả mạo vị trí, hoặc ngăn chặn hành vi cá cược thông qua proxy betting. Giải pháp này cũng chưa được phê duyệt để sử dụng trong môi trường cá cược trực tuyến được quản lý tại Mỹ, do không đáp ứng đầy đủ yêu cầu tuân thủ..

## Amazon Location Service với JavaScript

Khách hàng có thể sử dụng [Amazon Location Service](https://aws.amazon.com/vi/location/?did=ap_card&trk=ap_card) kết hợp với JavaScript phía client để xây dựng khả năng xác định vị trí địa lý trực tiếp bên trong bất kỳ ứng dụng nào dùng JavaScript (web apps, React Native, Swift, v.v.). Phương pháp này thu thập tọa độ GPS trực tiếp từ thiết bị của người dùng, có thể cung cấp độ chính xác cao hơn so với các phương pháp dựa trên IP. Các lập trình viên phải chỉnh sửa ứng dụng của họ để thêm mã yêu cầu vị trí, tích hợp [AWS SDK cho JavaScript (version 3)](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_location_code_examples.html), và tạo các API endpoints cho việc xác minh vị trí. Dưới đây là một ví dụ triển khai JavaScript cơ bản với xử lý lỗi:

```
// use API Key id for credentials
	const authHelper = await withAPIKey("api-key-id"); 

// Initialize the Location client
const locationClient = new LocationClient({ 
    region: "us-east-1",
    ...authHelper.getLocationClientConfig()
});

async function updateDeviceLocation() {
    try {
        // Get current position from browser
        const position = await new Promise((resolve, reject) => {
            navigator.geolocation.getCurrentPosition(resolve, reject, {
                enableHighAccuracy: true,
                timeout: 10000
            });
        });

        // Prepare the update command
        const command = new BatchUpdateDevicePositionCommand({
            TrackerName: "MyDeviceTracker",
            Updates: [{
                DeviceId: "mobile-device-123",
                Position: [position.coords.longitude, position.coords.latitude],
                SampleTime: new Date()
            }]
        });

        // Send location update to Amazon Location Service
        const response = await locationClient.send(command);
        console.log("Location updated successfully:", response);
        
    } catch (error) {
        console.error("Failed to update location:", error);
    }
} 
```
Amazon Location Service cung cấp một trình [hỗ trợ xác thực bằng JavaScript](https://docs.aws.amazon.com/location/previous/developerguide/loc-sdk-auth.html)* *(JavaScript authentication helper) để đơn giản hóa quá trình xác thực khi thực hiện API call. Bằng cách này bạn có thể tránh việc hardcode thông tin xác thực trong JavaScript. Bạn có thể sử dụng [Amazon Cognito](https://aws.amazon.com/cognito/) hoặc API keys làm phương thức xác thực. Giải pháp này yêu cầu có sự tương tác của người dùng. Trình duyệt sẽ hiển thị một thông báo yêu cầu quyền truy cập dịch vụ vị trí. Khi được chấp thuận, đoạn code sẽ lấy tọa độ GPS và phát ra các sự kiện (events) khi người dùng vượt qua ranh giới địa lý đã định..

Nếu sử dụng `ForecastGeofenceEvents` API call, Amazon Location Service có thể xác minh dựa trên các biên giới địa lý. Đối với triển khai Android WebView, lập trình viên phải bật JavaScript trong cài đặt WebView bằng cách yêu cầu quyền `WKWebView` trong app manifest. Ngoài ra, họ cần triển khai một permission `ACCESS_FINE_LOCATION` tùy chỉnh để xử lý các yêu cầu quyền truy cập vị trí. Các ứng dụng Android cũng yêu cầu một bộ nhớ đệm để lưu trữ quyền truy cập vị trí, được cấu hình thông qua thiết lập đường dẫn cơ sở dữ liệu định vị WebView.

Đối với triển khai iOS WebView sử dụng `WKWebView`, vị trí địa lý được cho phép theo mặc định, nhưng lập trình viên phải thêm `NSLocationWhenInUseUsageDescription` key vào file `Info.plist` với một thông điệp giải thích lý do cần truy cập vị trí. Ứng dụng cũng phải yêu cầu quyền truy cập vị trí thông qua iOS location services framework.

Giải pháp này chỉ khả thi cho các ứng dụng sử dụng JavaScript. Cách triển khai này có thể gây độ trễ do cần chờ người dùng cấp quyền và thu nhận tín hiệu GPS. Trải nghiệm người dùng có thể bị ảnh hưởng khi quyền truy cập vị trí bị từ chối hoặc tín hiệu GPS không khả dụng trong một số môi trường nhất định. Phương pháp này không thể phát hiện giả mạo vị trí hoặc can thiệp thiết bị. Mặc dù phương pháp này cung cấp dữ liệu vị trí chính xác hơn so với các giải pháp dựa trên IP, nó vẫn chưa đáp ứng được các yêu cầu nghiêm ngặt hơn trong các hoạt động gaming được quản lý, đặc biệt là khi cần phát hiện giả mạo vị trí.

## Chặn hoặc cho phép qua Amazon CloudFront

Giới hạn địa lý mà Amazon CloudFront cung cấp là một cơ chế kiểm soát quyền truy cập nội dung tại các điểm biên (edge locations) của AWS. Giải pháp này chặn hoặc cho phép các yêu cầu truy cập trước khi chúng đến được server gốc, từ đó giảm cả chi phí hạ tầng và rủi ro bảo mật tiềm ẩn.

Dưới đây là một ví dụ cấu hình giới hạn truy cập theo vị trí địa lý trong CloudFront được sử dụng trong Distribution Settings:

```
{
    "GeoRestriction": {
        "RestrictionType": "allowlist",  // Alternative: "blocklist"
        "Locations": [
            "GB",  // United Kingdom
            "IE",  // Ireland
            "MT"   // Malta
        ]
    }
}
```

Bạn cũng có thể[ cấu hình CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/adding-cloudfront-headers.html) để thêm các tiêu đề vị trí vào các yêu cầu mà CloudFront nhận được từ người dùng và chuyển tiếp chúng đến ứng dụng của bạn. Bằng cách này, bạn có thể xây dựng logic ứng dụng để thực hiện các hành động khác ngoài việc chỉ cho phép hoặc từ chối quyền truy cập của người dùng. Tuy nhiên, các giới hạn địa lý của CloudFront chỉ hoạt động ở cấp độ quốc gia - không thể phân biệt giữa các tiểu bang của Hoa Kỳ hoặc các vùng lãnh thổ bên trong một quốc gia.

Đối với các nhà vận hành trong những thị trường yêu cầu chặn truy cập theo cấp quốc gia, Amazon CloudFront mang đến một lớp phòng vệ đầu tiên với chi phí tối ưu. Tính năng giới hạn địa lý (geo-blocking) của CloudFront chỉ yêu cầu ít công sức phát triển và có thể được cấu hình trực tiếp thông qua AWS Management Console, giao diện dòng lệnh (CLI) hoặc các công cụ và dịch vụ Infrastructure as Code (IaC) mà không cần chỉnh sửa mã ứng dụng. Nhờ vậy, đây là một lựa chọn hấp dẫn cho việc triển khai nhanh các cơ chế kiểm soát truy cập theo vị trí địa lý cơ bản.

Việc sử dụng tính năng giới hạn địa lý của Amazon CloudFront không phát sinh thêm chi phí ngoài phí sử dụng CloudFront tiêu chuẩn. Các yêu cầu bị chặn sẽ bị từ chối ngay tại điểm biên, do đó chi phí truyền dữ liệu gần như không đáng kể. Khi một yêu cầu bị chặn, CloudFront sẽ trả về mã lỗi HTTP 403, chỉ tiêu tốn vài byte dữ liệu truyền tải. Nhờ đó, chức năng geo-blocking của CloudFront đặc biệt tiết kiệm chi phí đối với các ứng dụng nhận khối lượng lớn lưu lượng truy cập từ những khu vực bị hạn chế.

Chặn các yêu cầu ngay tại các điểm biên của CloudFront thay vì để chúng đến máy chủ gốc có thể tiết kiệm đáng kể chi phí băng thông và giảm tải cho hạ tầng backend. Ngoài ra, vì các cuộc tấn công từ chối dịch vụ phân tán (DDoS) thường xuất phát từ một số khu vực địa lý nhất định, nên việc chặn truy cập ở cấp quốc gia có thể giảm thiểu mức độ phơi nhiễm của hệ thống trước các cuộc tấn công này, đồng thời giảm chi phí xử lý liên quan.

Tuy nhiên, chức năng geo-blocking của CloudFront có những hạn chế vượt ra ngoài mức độ phân giải theo quốc gia. Dịch vụ này xác định vị trí người dùng thông qua cơ chế ánh xạ địa chỉ IP để xác định vị trí người dùng, nên có thể bị qua mặt bởi VPN hoặc máy chủ proxy. CloudFront cũng phụ thuộc vào các header điều khiển bộ nhớ đệm và chiến lược làm mới dữ liệu để đảm bảo tính cập nhật của dữ liệu trong các API động - điều này có nghĩa là dữ liệu vị trí không phải lúc nào cũng được cập nhật theo thời gian thực. Ngoài ra, phương pháp này không hỗ trợ kiểm tra tính toàn vẹn của thiết bị hoặc các biện pháp chống giả mạo vị trí, do đó không phù hợp cho các nhà vận hành trong lĩnh vực iGaming hoặc cá cược thể thao với những yêu cầu nghiêm ngặt hơn về xác thực vị trí địa lý.

## AWS WAF với geo match statements

[AWS WAF](https://aws.amazon.com/waf/) cung cấp cơ chế kiểm soát truy cập theo vị trí địa lý thông qua các câu lệnh đối sánh địa lý, mang lại mức độ kiểm soát chi tiết hơn so với tính năng geo-blocking của CloudFront. Dịch vụ này có thể lọc lưu lượng truy cập dựa trên cả quốc gia và tiểu bang của Hoa Kỳ, giúp nó trở thành một công cụ hữu ích cho các nhà vận hành cần kiểm soát truy cập với độ chính xác theo vùng.

Dưới đây là một ví dụ về quy tắc AWS WAF với câu lệnh đối sánh địa lý:

```
{
  "Name": "RegionalAccessControl",
  "Statement": {
    "GeoMatchStatement": {
      "CountryCodes": ["SG"],
      "ForwardedIPConfig": {
        "HeaderName": "X-Forwarded-For",
        "FallbackBehavior": "MATCH"
      }
    }
  },
  "Action": {
    "Allow": {}
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "RegionalAccessMetric"
  }
}
```

Ưu điểm nổi bật của AWS WAF geo matching là khả năng tích hợp với chỉ số giám sát của [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), cho phép giám sát chi tiết các yêu cầu bị chặn và các mẫu lưu lượng. Khả năng quan sát này giúp các nhà vận hành hiểu rõ hơn về hành vi truy cập và điều chỉnh chiến lược chặn truy cập một cách phù hợp. AWS WAF cũng hỗ trợ xử lý lưu lượng IPv6 một cách nguyên bản và có thể xử lý các yêu cầu dựa trên header `X-Forwarded-For`, điều này đặc biệt quan trọng đối với các ứng dụng chạy phía sau proxy server hoặc load balancer.

Tuy nhiên, tính năng đối sánh địa lý của AWS WAF không thể phát hiện các hình thức giả mạo vị trí phức tạp hoặc xác minh tính toàn vẹn của thiết bị. Mặc dù dịch vụ này có thể[ nhận diện và chặn lưu lượng đến từ các dịch vụ proxy hoặc điểm cuối VPN](https://docs.aws.amazon.com/whitepapers/latest/aws-best-practices-ddos-resiliency/aws-waf-ip-reputation.html) đã biết, nhưng những người dùng cố tình vẫn có thể vượt qua các cơ chế kiểm soát này.

## Các nhà cung cấp xác minh vị trí địa lý được cấp phép

Có nhiều quốc gia và khu vực pháp lý đặt ra yêu cầu nghiêm ngặt đối với hệ thống xác minh vị trí địa lý (ví dụ, Hoa Kỳ và Brazil). Đối với các nhà vận hành trò chơi có quản lý, các thị trường này thường yêu cầu hệ thống xác minh vị trí có khả năng phát hiện và ngăn chặn hành vi giả mạo vị trí bằng cách xác minh tính toàn vẹn ở cấp độ thiết bị. Các nhà cung cấp dịch vụ xác minh vị trí được cấp phép như [GeoComply](https://partners.amazonaws.com/partners/0010h00001doKdFAAU/GeoComply), [OpenBet](https://aws.amazon.com/marketplace/pp/prodview-wawct5skllemo?sr=0-1&ref_=beagle&applicationId=AWSMPContessa), và [Xpoint](https://xpoint.tech/) hiện đang cung cấp các khả năng nêu trên.

Các giải pháp của những công ty này khác biệt một cách căn bản so với các giải pháp được mô tả trước đó trong cách tiếp cận xác minh vị trí. Thay vì chỉ dựa vào địa chỉ IP hoặc tọa độ GPS, chúng thu thập thông tin vị trí trực tiếp từ thiết bị (sử dụng phương pháp định vị tam giác Wi-Fi, tín hiệu GPS, v.v.). Các hệ thống này xác minh tính toàn vẹn của thiết bị thông qua bộ SDK chuyên biệt cho từng thiết bị, dùng để kiểm tra hệ điều hành nhằm phát hiện dấu hiệu can thiệp, sử dụng VPN hoặc phần mềm giả mạo vị trí.

Các hệ thống quản lý hàng rào địa lý xác định xem thiết bị có nằm trong hoặc gần ranh giới của một hàng rào địa lý hay không, có thể là ranh giới giữa các tiểu bang hoặc khu vực hành chính. Việc giám sát theo thời gian thực giúp phát hiện những thay đổi vị trí đột ngột có thể cho thấy hành vi cá cược qua proxy hoặc các hoạt động gian lận khác. Việc xây dựng các hệ thống này thường đòi hỏi sự tích hợp ở nhiều cấp độ, bao gồm:

- Native SDK (iOS và Android) tích hợp cho mobile apps

- Browser plugin hoặc sidecar application cho web apps

- Tích hợp API của Amazon Location Services

- Báo cáo tuân thủ endpoints

Các nhà cung cấp dịch vụ xác minh vị trí được cấp phép là một lựa chọn phù hợp trong các trường hợp sau:

- Quy định yêu cầu xác minh hàng rào địa lý và xác minh vị trí (chẳng hạn như US Federal Wire Act hoặc luật cá cược của Brazil)

- Yêu cầu pháp lý bắt buộc phải có biện pháp chống giả mạo và xác minh tính toàn vẹn thiết bị

- Nhà vận hành cần ngăn chặn proxy betting thông qua các biện pháp kiểm tra tinh vi ở cấp thiết bị

## Kết luận

Việc lựa chọn giải pháp xác minh vị trí địa lý phụ thuộc vào nhiều yếu tố, bao gồm yêu cầu pháp lý tại khu vực mục tiêu, hồ sơ rủi ro, và chi phí. Mặc dù nhiều nhà vận hành triển khai nhiều lớp bảo vệ, nhưng điều quan trọng là phải hiểu rõ giải pháp nào phù hợp, dựa trên những rủi ro và yêu cầu cụ thể. Nếu không cần các tiêu chuẩn nghiêm ngặt, những giải pháp ít tốn kém hơn đã được nêu ở trên vẫn có thể cung cấp khả năng xác minh vị trí địa lý hữu ích, đồng thời duy trì hiệu năng và trải nghiệm người dùng.

Liên hệ [AWS Representative](https://pages.awscloud.com/Amazon-Game-Tech-Contact-Us.html) để chúng tôi giúp tăng tốc doanh nghiệp của bạn.

### Đọc thêm

- [Guidance for Building Geolocation Systems for the Betting & Gaming Industry on AWS](https://aws.amazon.com/solutions/guidance/building-geolocation-systems-for-the-betting-and-gaming-industry-on-aws/)
- [Amazon Location Service launches Enhanced Location Integrity features](https://aws.amazon.com/about-aws/whats-new/2024/06/amazon-location-service-enhanced-location-integrity-features/)
- [OpenBet Delivers Geolocation Integrity for Betting and Gaming Using Solutions on AWS](https://aws.amazon.com/solutions/case-studies/openbet-case-study/)
- [Winning the Cat-and-Mouse Race: Staying One Step Ahead of Streaming Geopiracy with GeoGuard and AWS](https://aws.amazon.com/blogs/apn/winning-the-cat-and-mouse-race-staying-one-step-ahead-of-streaming-geopiracy-with-geoguard-and-aws/)

**Dr. Mike Reaves** 
<img src="/images/3-Blog/3.1-Blog1/Mike-Reaves.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Dr.Mike Reaves là Kiến trúc sư Giải pháp Cấp cao (Principal Solutions Architect) trong lĩnh vực Betting & Gaming tại Amazon Web Services (AWS). Ông làm việc với khách hàng để xây dựng chiến lược kỹ thuật, đồng thời ứng dụng các công nghệ của AWS nhằm phát triển những giải pháp chuyên biệt phù hợp với nhu cầu của từng doanh nghiệp. Là một chuyên gia kỳ cựu trong ngành, Dr.Reaves là kiến trúc sư giải pháp được chứng nhận bởi AWS, với hơn 25 năm kinh nghiệm trong phát triển phần mềm và công nghệ thông tin. Ông nhận bằng Tiến sĩ Vật lý tại Đại học Connecticut.
<div style="clear: both;"></div>

**Dr. Haowen You** 
<img src="/images/3-Blog/3.1-Blog1/Haowen-You.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Dr.Haowen You hiện là Quản lý Kỹ thuật Phần mềm mảng Không gian Địa lý (Software Engineering Manager, Geospatial) tại Amazon Web Services (AWS). Ông phụ trách đội ngũ phát triển phần mềm cho Amazon Location Service. Với hơn 12 năm kinh nghiệm trong việc quản lý các nhóm và dự án phần mềm, Tiến sĩ You hiện đang tập trung vào các sản phẩm Trí tuệ nhân tạo ứng dụng (Applied AI) của AWS. Ông nhận bằng Tiến sĩ Kỹ thuật Hệ thống tại Đại học Virginia.