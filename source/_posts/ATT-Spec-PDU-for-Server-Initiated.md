title: ATT Spec - PDU for Server Initiated
date: 2015-02-28 13:32:42
tags: [Bluetooth, Bluetooth Core Spec, Attribute Protocol]
---
## Server Initiated ##
<!--more-->

<a name = "Handle Value Notification"/>
### Handle Value Notification ###
* A server can send a notification of an attribute's value at any time.
<img src="/images/att spec/ATT@HandleValueNotificationFormat.PNG" width="75%" height="75%"/>
* The attribute handle shall be set to a valid handle.
* The attribute value shall be set to the current value of attribute 
  identified by the attribute handle.
* If the attribue value is longer than (ATT_MTU-3) octets, 
  then only the first (ATT_MTU-3) octets of this attribute value
  can be sent in a notification.
* **For a client to get a long attribute, it would have to use the
  *Read Blob Request* following the receipt of this notification**.
* **If the attribute handle or the attribute value is invalid,
  then this notification shall be ignored upon reception**.

<a name = "Handle Value Indication"/>
### Handle Value Indication ###
* A server can send an indication of an attribute's value.
<img src="/images/att spec/ATT@HandleValueIndicationFormat.PNG" width="75%" height="75%"/>
* The attribute handle shall be set to a valid handle.
* The attribute value shall be set to the current value of attribute 
  identified by the attribute handle.
* If the attribue value is longer than (ATT_MTU-3) octets, 
  then only the first (ATT_MTU-3) octets of this attribute value
  can be sent in a indication.
* **For a client to get a long attribute, it would have to use the
  *Read Blob Request* following the receipt of this indication**.
* The client shall send a *Handle Value Confirmation* in response to 
  a *Handle Value Indication* 
* **No further indications to this client shall occur until
  the confirmation has been received by the server**.
* If the attribute handle or the attribute value is invalid,
  the client shall send a *Handle Value Confirmation* in response
  and shall discard the handle and value from the received indication.

### Handle Value Confirmation ###
* The *Handle Value Confirmation* is sent in response to a received
  *Handle Value Indication* and confirms that the client has receiveed
  an indication of the given attribute.
<img src="/images/att spec/ATT@HandleValueConfirmationFormat.PNG" width="75%" height="75%"/>
