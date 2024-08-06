---
layout: home
title: Implementation guide for iOS (Devs)
nav_order: 6
nav_titles: true
titles_max_depth: 2
---
This guide is for **application developers** working to add native MPTCP support
when running on iOS.

## MPTCP connection

iOS offers the possibility to establish new MPTCP connections by using its `URLSession` API, or `NSURLSession` API if you are working with Objective-C. Note that on iOS, only client side MPTCP is supported, and that the path manager can't be manually configured. However, the developers can influence the scheduler of MPTCP by selecting a type of service needed by their app.

 To establish a MPTCP connection, you simply have to tweak a bit the configuration used to create the `(NS)URLSession` object. 

1. Create an `(NS)URLSessionConfiguration` object, typically by using the `default` configuration. 
2. Select the type of service you want. Available types are:

    | Type of service       | Description         | 
    |:-------------|:------------------|
    | None     |The default service type indicating that Multipath TCP should not be used.|
    | Handover |A Multipath TCP service that provides seamless handover between Wi-Fi and cellular in order to preserve the connection.|
    | Interactive |A service whereby Multipath TCP attempts to use the lowest-latency interface.|
    | Aggregate   |A service that aggregates the capacities of other Multipath options in an attempt to increase throughput and minimize latency.|

    {: .note }
    Source for available types of service: [here](https://developer.apple.com/documentation/foundation/nsurlsessionmultipathservicetype) for `NSURLSession` and [here](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/2875967-multipathservicetype) for `URLSession`

3. Set the `multipathServiceType` property of this configuration to the selected type of service. The value is simply `NSURLSessionMultipathServiceType${MODE_NAME}` for `NSURLSession`, and `.{MODE_NAME_LOWER}` for `URLSession`. For example, if we want to use the handover mode, we should pass `NSURLSessionMultipathServiceTypeHandover` to `NSURLSession`, and `.handover` to `URLSession`

4. Create the `(NS)URLSession` by passing in the configuration to the constructor. From now on, every request done with this session should use MPTCP, if the server supports it.
   
{: .info }
To enable the aggregate type of service, you'll probably also need to add the [MPTCP entitlement](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/improving_network_reliability_using_multipath_tcp) to your app. For the other modes, they seem to work fine, even without this additional step, but the aggregate one crashes if you don't set it.

{: .warning }
The `(NS)URLSession` API is also available on macOS, but you won't be able to set the `multipathServiceType` in the configuration. There is thus no possible way to enable MPTCP with `(NS)URLSession` on macOS yet (though other possibilities exist, see [the FAQ](/faq.html) for more details)

## Examples in different languages

For more detailed examples, please see this [Git
repository](https://github.com/mptcp-apps/mptcp-hello/). Do not hesitate to
contribute here and there!

<details markdown="block">
<summary>Example in Objective-C </summary>

`multipathServiceType` support has been added
since [iOS 11.0](https://developer.apple.com/documentation/foundation/nsurlsessionconfiguration/2875967-multipathservicetype)

```objc
#import <Foundation/Foundation.h>

NSURLSessionConfiguration *conf = [NSURLSessionConfiguration defaultSessionConfiguration];
// we can choose other modes here, select .handover by default
conf.multipathServiceType = NSURLSessionMultipathServiceTypeHandover;

NSURLSession *session = [NSURLSession sessionWithConfiguration:conf];
```
</details> {: .ctsm}


<details markdown="block">
<summary>Example in Swift </summary>

`multipathServiceType` support has been added
since [iOS 11.0](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/2875967-multipathservicetype)

```swift
import Foundation

var conf = URLSessionConfiguration.default
// we can choose other modes here, select .handover by default
conf.multipathServiceType = .handover

let session = URLSession(configuration: conf)
```
</details> {: .ctsm}

<details markdown="block">
<summary>Example in Swift using Alamofire </summary>

```swift
import Foundation
import Alamofire

var conf = URLSessionConfiguration.default
// we can choose other modes here, select .handover by default
conf.multipathServiceType = .handover

let session = Session(configuration: conf)
```
</details> {: .ctsm}