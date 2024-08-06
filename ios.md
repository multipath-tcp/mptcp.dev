---
layout: home
title: Implementation guide (iOS)
nav_order: 6
nav_titles: true
titles_max_depth: 2
---

This guide is for **application developers** working to add native MPTCP support
when running on iOS.

## MPTCP connection

iOS offers the possibility to establish new MPTCP connections by using its 
`URLSession` API -- prefixed with `NS` with Objective-C. Note that on iOS, only 
the client side MPTCP is supported. The path manager's behavior cannot be 
modified. However, the app developers can influence which MPTCP packet scheduler
is being used by selecting a type of service linked to a connection.

To create an MPTCP connection, simply request it when creating the 
`(NS)URLSession` object:

1. Create an `(NS)URLSessionConfiguration` object, typically by using the 
   `default` configuration. 
2. Select the type of service amongst:

    | Type of service  | Description                                                                                                                    | 
    |:-----------------|:-------------------------------------------------------------------------------------------------------------------------------|
    | None             | The default service type indicating that Multipath TCP should not be used.                                                     |
    | Handover         | A Multipath TCP service that provides seamless handover between Wi-Fi and cellular in order to preserve the connection.        |
    | Interactive      | A service whereby Multipath TCP attempts to use the lowest-latency interface.                                                  |
    | Aggregate        | A service that aggregates the capacities of other Multipath options in an attempt to increase throughput and minimize latency. |

    Official documentation: 
    [`URLSession`](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/2875967-multipathservicetype)
    [`NSURLSession`](https://developer.apple.com/documentation/foundation/nsurlsessionmultipathservicetype)

3. Associate the selected service type to the `multipathServiceType` property. 
   The value is `.<type>` for `URLSession`, or  `NSURLSessionMultipathServiceType<TYPE>`
   for `NSURLSession`, e.g. for the _Handover_ type, pass  `.handover` 
   (`URLSession`) or `NSURLSessionMultipathServiceTypeHandover` (`NSURLSession`)

4. Create the `(NS)URLSession` by passing in the configuration to the constructor. From now on, every request done with this session should use MPTCP, if the server supports it.
   
{: .warning }
> To be able to enable the _Aggregate_ service type, it is required to add the 
> [MPTCP entitlement](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/improving_network_reliability_using_multipath_tcp) 
> to the app not to have an error when picking it. For the other modes, no 
> additional steps are required.
> 
> The `(NS)URLSession` API is also available on macOS, but it is not possible to 
> set any `multipathServiceType`. There is thus not possible to enable MPTCP with 
> `(NS)URLSession` on macOS yet.

## Examples in different languages

For more detailed examples, please see this [Git
repository](https://github.com/mptcp-apps/mptcp-hello/). Do not hesitate to
contribute here and there!

<details markdown="block">
<summary>Example in Swift </summary>

`multipathServiceType` support has been added
in [iOS 11.0](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/2875967-multipathservicetype)

```swift
import Foundation

var conf = URLSessionConfiguration.default
// Handover mode: Other types can be selected
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
// Handover mode: Other types can be selected
conf.multipathServiceType = .handover

let session = Session(configuration: conf)
```
</details> {: .ctsm}

<details markdown="block">
<summary>Example in Objective-C </summary>

`multipathServiceType` support has been added
in [iOS 11.0](https://developer.apple.com/documentation/foundation/nsurlsessionconfiguration/2875967-multipathservicetype)

```objc
#import <Foundation/Foundation.h>

NSURLSessionConfiguration *conf = [NSURLSessionConfiguration defaultSessionConfiguration];
// Handover mode: Other types can be selected
conf.multipathServiceType = NSURLSessionMultipathServiceTypeHandover;

NSURLSession *session = [NSURLSession sessionWithConfiguration:conf];
```
</details> {: .ctsm}
