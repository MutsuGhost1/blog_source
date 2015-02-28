title: ATT Spec - PDU for FindInformation
date: 2015-02-27 19:31:00
tags: [Bluetooth, Bluetooth Core Spec, Attribute Protocol]
---
## Find Information ##
<!--more-->
<a name="Find Information Request"/>
### Find Information Request ###
* The Find Information Request is **used to obtain the mapping of attribute handle**
  with their associated types on a server.
* This allows a client to discover the list of attributes and their types on a server.
<img src="/images/att spec/ATT@FindInformationRequestFormat.PNG" width="75%" height="75%"/>
* Only attributes with attribute handles between and including the Starting Handle parameter
  and Ending Handle parameter will be returned.
* To read all attributers, the Starting Handle parameter shall be set to 0X0001,
  and Ending Handler parameter shall be set to 0xFFFF.
* The Starting Handle parameter shall be less than or equal to the Ending Handle parameter.
* If a serer receives a *Find Information Request* with the Starting Handle parameter 
  greater than the Ending Handle parameter or the Starting Handle parameter is 0X0000,
  an Error Response shall be sent with the < < Invalid Handle > > error code;
  the Attribute Handle In Error parameter shall be set to the Starting Handle parameter.
* If one or more attributes will be returned, a *Find Information Respon*se PDU shall be sent.
* If no attributes will be returned, an Error Response shall be sent with the
  < < Attribute Not Found > > error; the Attribute Handle In Error parameter shall be
  set to the Starting Handle parameter.
* The server shall not respond to the Find Informatino Request with an Error Response
  with the < < Insufficient Authentication > >, < < Insufficient Authorization > >,
  < < Insufficient Encryption Key Size > > or < < Application Error > > error code.
  (因為 Find Information 系列的指令, 都只是查詢 Attribute Handle, 需要保護的通常是 Attribute Value)

### Find Information Response ###
* The *Find Information Response* is sent in reply to a received *Find Information Request*
  and contains information about this server.
<img src="/images/att spec/ATT@FindInformationResponseFormat.PNG" width="75%" height="75%"/>
<img src="/images/att spec/ATT@FindInformationResponseInformationFieldFormat.PNG" width="75%" height="75%"/>
* The Find Information Response shall have complete handle-UUID pairs.
  Such pairs shall not be split across response packets;
  this is also implies that a handle-UUID pair shall fit into a single response packet.
* The handle-UUID pairs shall be returned in ascending order of attribute handles.
* If sequential attributes have differing UUID size, 
  it may happen that a *Find Information Response* is not filled with the maximum possible
  amount of (handle, UUID) pairs.
  **In that case, the following attribute would have to be read 
  using another *Find Information Request* with its starting handle updated**.
  (若回傳的 pair list 中, 會有不同 size 的 attribute handle, 那 user 就必須分段讀取)

<a name="Find By Type Value Request"/>
### Find By Type Value Request ###
* The Find By Type Value Request is used to obtain the handles of attributes
  that have a 16-bit UUID attribute type and attribute value.
  (不一定要用 Attribute Type 為 Grouping 的, 一般非 Grouping 的 Attribute Type 也行)
* This allows the range of handles associated with a given attribute to be discovered
  when the attribute type determines the grouping of a set of attributes.
* It's not possible to use this request on an attribute that has a value
  longer than (ATT_MTU-7).
* Generic Attribute Protocol defines grouping of attributes by attribute type.
<img src="/images/att spec/ATT@FindByTypeValueRequestFormat.PNG" width="75%" height="75%"/>
* Only attributes with attribute handles between and 
  including the the Starting Handle parameter and 
  the Ending Handle parameter that **match the 
  requested attribute type and attribute value that 
  have sufficient permissions to allow reading wiil be returned**.
* Attribute value will be compared in terms of length and binary representation.
* To read all attributers, the Starting Handle parameter shall be set to 0X0001,
  and Ending Handler parameter shall be set to 0xFFFF.
* If one or more handles will be returned, 
  a Find By Type Value Response PDU shall be sent.
* If a serer receives a *Find By Type Value Request* with the Starting Handle parameter 
  greater than the Ending Handle parameter or the Starting Handle parameter is 0X0000,
  an Error Response shall be sent with the < < Invalid Handle > > error code;
  the Attribute Handle In Error parameter shall be set to the Starting Handle parameter.
* If no attributes will be returned, an Error Response shall be sent with the
  < < Attribute Not Found > > error; the Attribute Handle In Error parameter shall be
  set to the Starting Handle parameter.
* The server shall not respond to the *Find By Type Value Request* with an Error Response
  with the < < Insufficient Authentication > >, < < Insufficient Authorization > >,
  < < Insufficient Encryption Key Size > >, < < Insufficient Encryption > > or
  < < Application Error > > error code. (因為沒權限讀的 value 不會被 match)


### Find By Type Value Response ###
* The Find By Type Value Response is sent in reply to 
  a received Find By Type Value Request and contains information about this server.
* The Handles Information List field is a list of one or more Handle Informations.
<img src="/images/att spec/ATT@FindByTypeValueResponseFormat.PNG" width="75%" height="75%"/>
* The *Find By Type Value Response* shall contain one or more complete Handles Information.
  Such Handles Informatoin List is ordered sequentially based on the found attribute handles.
* For each handle that matches the attribute type and attribute value in the
  *Find By Type Value Request* a Handles Information shall be returned.
  The Found Attribute Handle shall be set to the handle of the attribute 
  that has the exact attribute type and attribute value from the *Find By Type Value Request*.
* **If the attribute type in the Find By Type Value Request is a grouping attribute as defined
  by a higher layer specification, the Group End Handle shall be defined 
  by that higher layer specification**.
* If on other attributes with the same attribute type exist after the Found Atribute Handle,
  the End Found Handle shall be set to 0XFFFF.
* **If the attribute type in the Find By Type Value Request is not a grouping attribute as defined
  by a higher layer specification, the Groupd End Handle shall be 
  equal to Found Attribute Attribute Handle**.
* The Group End Handle may be greater than the Ending Handle in the *Find By Type Value Request*.
* If a server receives a Find By Type Value Request, the server shall respond with the
  Find By Type Value Response containing as many handles for attributes that match
  the requested attribute type and attribute value that exist in the server that will
  fit into the maximum PDU size of (ATT_MTU-1).
  (Find 的時候只能填 ATT_MTU-7, 那為何可以 Match ATT_MTU-1, Size 似乎有點不一致??)
