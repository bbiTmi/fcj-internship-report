---
title: "Blog 1"
date: ""
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
# Building geolocation verification for iGaming and sports betting on AWS

Geolocation verification in sports betting and iGaming serves two primary purposes: compliance and fraud prevention. Online sports betting and iGaming operators may need to ensure that only players from certain regions of the world are able to view content due to in-market licensing restrictions. Or gaming regulations may require that access is restricted to persons within allowed jurisdictions while blocking unauthorized traffic at geopolitical boundaries.

In the United States, the [Federal Wire Act](https://www.law.cornell.edu/uscode/text/18/1084), which prohibits sports betting across state lines, begins with the following text:

Whoever being engaged in the business of betting or wagering knowingly uses a wire communication facility for the transmission in interstate or foreign commerce of bets or wagers or information assisting in the placing of bets or wagers on any sporting event or contest, or for the transmission of a wire communication which entitles the recipient to receive money or credit as a result of bets or wagers, or for information assisting in the placing of bets or wagers, shall be fined under this title or imprisoned not more than two years, or both.

IGT and the New Hampshire State lottery have [prevailed in a court case](https://www.bhfs.com/insight/federal-court-clarifies-sort-of-the-wire-act-s-application-beyond-sports-betting/) against the United States (US) Department of Justice regarding the non-applicability of the Wire Act to iGaming and lottery operations. However, some states still require that players and wagering infrastructure be located within the same state. In these states, operators must also put in place mechanisms to detect virtual private networks (VPNs), software, virtualization, and other methods capable of circumventing player location detection. Usually this means an SDK must be compiled into an app or other software ([a sidecar](https://medium.com/@justinricedev/what-is-a-software-sidecar-8f89feff09f9#:~:text=This%20creates%20a%20single%20point,Why%20Use%20Software%20Sidecars?)) must run alongside it.

In addition to allowing an operator to comply with regulations, geographic access controls deliver additional business benefits. Blocking traffic at the edge reduces infrastructure costs by eliminating unauthorized traffic before it reaches core systems. These controls can also be used to thwart fraud, for example, so-called *proxy betting* where a player in an allowed geographic zone places wagers for multiple people outside that zone.

The technology choice for geolocation verification depends on the specific use case of the company employing it. In the US, Brazil, and some other regions, sports betting and iGaming operators utilize compliance-tested, licensed geolocation systems, with SDKs accessing mobile OS directly to satisfy anti-spoofing requirements. However, there are alternatives which may be appropriate for content providers or operators in other regions.

These methods may be used alone or in conjunction with one another to provide a satisfactory solution, depending on compliance and business needs. Let’s explore five distinct approaches companies can use to build and deploy geolocation verification using Amazon Web Service (AWS).



## 1. Geolocation blocking using Amazon Route 53

The [geolocation routing](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-geo.html) capabilities of [Amazon Route 53](https://aws.amazon.com/route53/) can restrict access to content by country, continent, or US state domain name system server (DNS) level. This DNS-based approach processes location decisions before traffic reaches your infrastructure, reducing unnecessary load and associated costs. A key benefit of this solution is that it requires no modifications to existing server or client code bases. The implementation uses DNS records with geolocation routing policies, which determine a user’s location and respond according to the rules configured for that geography. Following is an example Route 53 geolocation traffic routing policy:


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

The example policy routes traffic to endpoints based on whether the country of origin is Sweden. In this example, traffic originating from Sweden will be sent to an application load balancer, while traffic originating from elsewhere will be directed to an error page hosted on [Amazon CloudFront](https://aws.amazon.com/cloudfront/). Route 53 geolocation routing integrates directly with other AWS services and includes fallback rules for unmatched locations.

DNS-based geolocation does have limitations. The solution relies on the accuracy of IP-to-location mapping, which can be circumvented by VPNs or proxy servers. DNS responses may be cached by intermediate resolvers, potentially allowing users to retain access from a blocked location after moving. Additionally, the solution cannot detect device tampering, location spoofing, or prevent proxy betting. It has not been approved for use by licensed US online gambling operators for the purpose of meeting their compliance needs.



## 2. Amazon Location Service with JavaScript

Customers can use [Amazon Location Service](https://aws.amazon.com/location/?did=ap_card&trk=ap_card) combined with a client-side JavaScript to build geolocation capabilities directly inside any app which uses JavaScript (web apps, React Native, Swift, and so on). This approach retrieves GPS coordinates from the user’s environment, potentially offering a more precise location determination than IP-based methods. Developers must modify their applications to include location request code, integrate the [AWS SDK for JavaScipt (version 3)](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_location_code_examples.html), and create API endpoints for location verification. For a basic implementation, following is an example JavaScript implementation with error handling:


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


Amazon Location Service provides a [JavaScript authentication helper](https://docs.aws.amazon.com/location/previous/developerguide/loc-sdk-auth.html) to streamline authentication when making API calls. This way you can avoid hardcoding credentials in your JavaScript. You can use either [Amazon Cognito](https://aws.amazon.com/cognito/) or API keys as your authentication method. This solution requires user interaction. Browsers will display a permission prompt requesting access to location services. Once granted, the code retrieves GPS coordinates and emits events when geofence borders are crossed.

If using the `ForecastGeofenceEvents` API call, Amazon Location Service can validate them against defined geographic boundaries. For Android WebView implementations, developers must enable JavaScript in their WebView settings by requesting `ACCESS_FINE_LOCATION` permission in the app manifest. They must additionally implement a custom `WebChromeClient` to handle geolocation permission requests. Android applications also require a local device cache for storing geolocation permissions and positions, configured through the WebView’s geolocation database path setting.

For iOS WebView implementations using `WKWebView`, geolocation is allowed by default, but developers must add the `NSLocationWhenInUseUsageDescription` key to their `Info.plist` file with a message explaining the location access requirement. The application must also request location authorization through the iOS location services framework.

This solution is only viable for apps utilizing JavaScript. The implementation introduces latency as the system waits for user permission and GPS acquisition. User experience can be impacted when location access is denied, or GPS signals are unavailable in certain environments. This method cannot detect location spoofing or device tampering. While this method provides more precise location data than IP-based solutions, it still falls short of satisfying more stringent requirements for regulated gaming operations, particularly when detecting location spoofing is required.



## 3. Blocking or allowing through Amazon CloudFront

The geographic restrictions Amazon CloudFront offers is content access control at AWS edge locations. This solution blocks or allows requests before they reach your origin servers, reducing both infrastructure costs and potential security risks.

Following is a CloudFront geographic restriction configuration used in the Distribution Settings:


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


It’s also possible for you to [configure CloudFront to add location headers](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/adding-cloudfront-headers.html) to the requests that CloudFront receives from your users and forward them to your app. In this way, you can create app logic that can take other actions besides allowing or denying users’ access. The geographic restrictions of CloudFront operate at the country level only—they cannot differentiate between US states or regions within countries.

For operators in markets requiring country-level blocking, CloudFront provides a cost-effective first line of defense. CloudFront geo-blocking requires minimal development effort. It can be configured through the AWS Console, command line interface (CLI), or Infrastructure as Code (IaC) tools and services without modifying application code. This makes it an attractive option for quick deployment of basic geographic controls.

Using the geographic restrictions of CloudFront does not incur additional charges beyond standard CloudFront usage fees. Blocked requests are denied at the edge location, resulting in minimal data transfer costs. When a request is blocked, CloudFront returns an HTTP 403 (Forbidden) error response, consuming only a few bytes of data transfer. This makes CloudFront geo-blocking particularly economical for applications receiving high volumes of traffic from restricted regions.

Blocking requests at CloudFront edge locations instead of your origin servers could save significant bandwidth costs and reduce the load on your backend infrastructure. Additionally, since distributed denial-of-service (DDoS) attacks often originate from specific geographic regions, country-level blocking can reduce your exposure to these attacks and associated costs.

However, CloudFront geo-blocking has limitations beyond its country-level granularity. The service relies on IP address mapping to determine location, which can be circumvented by VPNs or proxy servers. It relies on cache-control headers and invalidation strategies to verify data freshness for dynamic APIs—meaning location data may not always be up-to-date. Additionally, no device integrity checking or anti-spoofing measures are available through this method, which may invalidate it as an option for iGaming or sports betting operators with more stringent geolocation requirements.



## 4. AWS WAF geo match statements

[AWS WAF](https://aws.amazon.com/waf/) provides geographic access control through geo match statements, offering more granular control than CloudFront restrictions. The service can filter traffic based on both country and US state locations, making it useful for operators who need regional precision in their access controls.

Following is an AWS WAF rule with a geo match statement:


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


A key advantage of AWS WAF geo matching is its integration with [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) metrics, enabling detailed monitoring of blocked requests and traffic patterns. This visibility helps operators understand access patterns and adjust their blocking strategies accordingly. AWS WAF also handles IPv6 traffic natively and can process requests based on the `X-Forwarded-For` header, important for applications behind proxy servers or load balancers.

However, AWS WAF geo matching cannot detect sophisticated location spoofing attempts or verify device integrity. While it can [identify and block traffic from known proxy services and VPN endpoints](https://docs.aws.amazon.com/whitepapers/latest/aws-best-practices-ddos-resiliency/aws-waf-ip-reputation.html), determined users may still circumvent these controls.



## 5. Licensed geolocation verification suppliers

There are many jurisdictions with stringent requirements for geolocation verification systems (for example, US and Brazil). For regulated gaming operators, these markets generally require geolocation verification systems that can detect and prevent location spoofing by verifying integrity at the device level. Licensed geolocation verification suppliers such as [GeoComply](https://partners.amazonaws.com/partners/0010h00001doKdFAAU/GeoComply), [OpenBet](https://aws.amazon.com/marketplace/pp/prodview-wawct5skllemo?sr=0-1&ref_=beagle&applicationId=AWSMPContessa), and [Xpoint](https://xpoint.tech/) provide these capabilities.

These company’s solutions differ fundamentally from the previously described solutions in their approach to location verification. Rather than relying solely on IP addresses or GPS coordinates, they retrieve location information from the device itself (using Wi-Fi trilateration, GPS, and so on). They verify device integrity by using a device-specific SDK to examine the operating system for signs of tampering, VPNs, or location spoofing software.

Geofence management systems determine whether the device is within or near a geofence border, which may be a state border or other political boundary. Real-time monitoring detects sudden location changes that might indicate proxy betting or other nefarious action. Building these systems typically requires integration at multiple levels, including:
- Native SDK (iOS and Android) integration for mobile apps
- Browser plugin or sidecar application for web apps
- Amazon Location Services API integration
- Compliance reporting endpoints

Licensed geolocation suppliers are a good option when:
* Regulations require precise geofencing and location verification (such as the US Federal Wire Act or Brazil’s gaming laws)
* Jurisdictional requirements mandate anti-spoofing and device integrity verification
* Operators need to prevent proxy betting through sophisticated device-level checks


## Conclusion

The choice of geolocation method depends on various factors, including regulatory requirements for the target geography, risk profile, and cost. While many operators implement multiple layers of protection, it’s crucial to understand which solutions are appropriate, based on the risks and requirements at issue. If stringent criteria are not required, the less costly solutions outlined previously can provide helpful geolocation capabilities while maintaining performance and the user experience.

Contact an [AWS Representative](https://pages.awscloud.com/Amazon-Game-Tech-Contact-Us.html) to know how we can help accelerate your business.


### Further reading
* [Guidance for Building Geolocation Systems for the Betting & Gaming Industry on AWS](https://aws.amazon.com/solutions/guidance/building-geolocation-systems-for-the-betting-and-gaming-industry-on-aws/)
* [Amazon Location Service launches Enhanced Location Integrity features](https://aws.amazon.com/about-aws/whats-new/2024/06/amazon-location-service-enhanced-location-integrity-features/)
* [OpenBet Delivers Geolocation Integrity for Betting and Gaming Using Solutions on AWS](https://aws.amazon.com/solutions/case-studies/openbet-case-study/)
* [Winning the Cat-and-Mouse Race: Staying One Step Ahead of Streaming Geopiracy with GeoGuard and AWS](https://aws.amazon.com/blogs/apn/winning-the-cat-and-mouse-race-staying-one-step-ahead-of-streaming-geopiracy-with-geoguard-and-aws/)


**Dr. Mike Reaves** 
<img src="/images/3-Blog/3.1-Blog1/Mike-Reaves.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Dr. Mike Reaves is Principal Solutions Architect, Betting & Gaming at Amazon Web Services (AWS). He works with customers to develop technical strategy, and leverages AWS technologies to develop purpose-built solutions for customers. An industry veteran, Dr. Reaves is an AWS certified Solutions Architect with over 25 years of software development and IT experience. Dr. Reaves holds a Ph.D. in Physics from the University of Connecticut.
<div style="clear: both;"></div>

**Dr. Haowen You** 
<img src="/images/3-Blog/3.1-Blog1/Haowen-You.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Dr. Haowen You is Software Engineering Manager, Geospatial at Amazon Web Services (AWS). He leads a software development team for Amazon Location Service. With 12+ years of experience managing software teams and projects, Dr. You is currently focusing his efforts on AWS applied AI products. Dr. You holds a Ph.D. in Systems Engineering from the University of Virginia.