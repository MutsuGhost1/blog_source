title: ATT Spec - PDU for Read Attributes
date: 2015-02-27 21:42:43
tags: [Bluetooth, Bluetooth Core Spec, Attribute Protocol]
---
## Read Attributes ##
<!--more-->
<a name="Read By Type Request"/>
### Read By Type Request ###
* The *Read By Type Request* is used to obtain the values of attributes where
  the attribute type is known but the handle is not known.
<img src="/images/att spec/ATT@ReadByTypeRequestFormat.PNG" width="75%" height="75%"/>
* Only attributes with attribute handles between and including the Starting Handle parameter
  and Ending Handle parameter will be returned.
* To read all attributers, the Starting Handle parameter shall be set to 0X0001,
  and Ending Handler parameter shall be set to 0xFFFF.
* All attribute types are effectively compared as 128-bit UUIDs,
  even if a 16-bit UUID is provided in this request or defined for an attribute.
* If a serer receives a *Read By Type Request* with the Starting Handle parameter 
  greater than the Ending Handle parameter or the Starting Handle parameter is 0X0000,
  an Error Response shall be sent with the < < Invalid Handle > > error code;
  the Attribute Handle In Error parameter shall be set to the Starting Handle parameter.
* If no attributes will be returned, an Error Response shall be sent with the
  < < Attribute Not Found > > error; the Attribute Handle In Error parameter shall be
  set to the Starting Handle parameter.
* Requested attributes = attributes returned with the lowest handles within the handle range.
* If the attributes with the requested type within the handle range have attribute values that
    * Have **the same length**, then these attributes can all be read in a single request.
        * The attribute server shall include as many attributes as possible in the response
          in order to minimize the number of PDUs required to read attributes of the same type.
    * Have **different length**, then multiple *Read By Type Requests* must be made.
* When multiple attributes match, then the rules below shall be applied to each in turn:
    1. Only attributes that can be read shall be returned in a Read By Type Response.
    2. If an attribute in the set of requested attributes would cause an Error Response
       then this attribute can't be included in a Read By Type Response 
       and the attributes before this attribute shall be returned.
    3. If the first attribute in the set of requested attributes would cause an Error Response
       then no other attributes in the requested attributes can be considered.
* If there are multiple attributes with the requested type within the handle range,
  and the client would like to get the next attribute with the requested type,
  it would have to issue another *Read By Type Request* with its starting handle updated.
* The client can be sure there are no more such attributes remaining once it gets an
  Error Response with the error code < < Attribute Not Found > >.
* The server shall respond with a Read By Type Response if the requested attributes
  have sufficient permissions to allow reading.
* Insufficient permission for a client may include:
    * Insufficient authorization to read the requested attribute then an Error Response
      shall be sent with the error code < < Insuffcient Authorization > >.
      The Attribute Handle In Error parameter shall be set to the handle of the attribute
      causing the error.
    * Insufficient security to read the requested attribute then an Error Response
      shall be sent with the error code < < Insuffcient Authentication > >.
      The Attribute Handle In Error parameter shall be set to the handle of the attribute
      causing the error.
    * Insufficient encryption key size to read the requested attribute then an Error Response
      shall be sent with the error code < < Insuffcient Encryption Key Size > >.
      The Attribute Handle In Error parameter shall be set to the handle of the attribute
      causing the error.
    * Not enabled encryption, and encryption is required to read the requested attribute then an Error Response
      shall be sent with the error code < < Insuffcient Encryption > >.
      The Attribute Handle In Error parameter shall be set to the handle of the attribute
      causing the error.
    * The requested attribute's value can't be read due to permissions then an Error Response
      shall be sent with the error code < < Read Not Permitted > >.
      The Attribute Handle In Error parameter shall be set to the handle of the attribute
      causing the error.
* 要把所有的 Attribute Handle + Attribute Value 讀完, 要下多次的 *Read By Type Request*.
  中間有可能遇到 Permission 的問題, 但要 Query 到結束, 必須取到 error code < < Attribute Not Found > > 為止.

### Read By Type Response ###
* The *Read By Type Response* is sent in reply to received *Read By Type Request* and
  contains the handles and values of the attributes that have been read.
<img src="/images/att spec/ATT@ReadByTypeResponseFormat.PNG" width="75%" height="75%"/>
* The Read By Type Response shall contain complete handle-value pairs.
  Such pairs shall not be split across response packets.
  The handle-value pairs shall be returned sequentially based on the attriute handle.
  (依照 Handle 值回傳, 由小到大)
* The maximum length of an attribute handle-value pair is 255 octets, bounded by
  the Length parameter that is one octet.
  Therefore, the maximum length of an attribute value in this resonse is (Length-2) = 253 octets.
* If the attribute value is longer than (ATT_MTU-4) or 253 octets, whichever is smaller,
  then the first (ATT_MTU-4) or 253 octets shall be included in this response.
  (這邊的 Length 為 1 byte, 因此表示成數值最大為 255, 扣掉 Attribute Handle 佔 2, 因此 Value 最大是 253)
* Use Read Blob Request to read the remaining octets of a long attribute value.

<a name="Read Request"/>
### Read Request ###
* The *Read Request* is used to request the server to read the value of an attribute
  and return its value in a *Read Response*.
<img src="/images/att spec/ATT@ReadRequestFormat.PNG" width="75%" height="75%"/>
* The server shall respond with a Read Response if the handle is valid 
  and the attribute has sufficient permission to allow reading.
* If the client has
    * Insufficient authorization to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authorization > >.
    * Insufficient security to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authentication > >.
    * Insufficient encryption key size to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption Key Size > >.
    * Not enabled encryption and encryption is required to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption > >.
* If the handle is invalid, then an Error Response shall be sent with the error code
  < < Invalid Handle > >.
* If the attribute value can't be read due to permissions then an
  Error Response ahll be sent with the error code < < Read Not Permitted > >.

### Read Response ###
* The read response is sent in reply to received *Read Request* and contains the
  value of the attribute that has been read.
<img src="/images/att spec/ATT@ReadResponseFormat.PNG" width="75%" height="75%"/>
* The attribute value shall be set to the value of the attribute identified 
  by the attribute handle in the request.
  If the attribute value is longer than (ATT_MTU-1) then the first (ATT_MTU-1)
  octets shall be included in this response.
* The *Read Blob Request* would be used to read the remaining octets of a long attribute value. 

<a name="Read Blob Request"/>
### Read Blob Request ###
* The *Read Blob Request* is used to request the server to read part of the value of an attribute
  at a given offset and return a specific part of the value in a *Read Blob Response*.
<img src="/images/att spec/ATT@ReadBlobRequestFormat.PNG" width="75%" height="75%"/>
* The value of offset is based from zero; the first value octet has an offset of zero,
  the second octet has a value offset of one, etc...
* The server shall respond with a Read Blob Response if the handle is valid and
  the attribute and value offset is not greater than the length of the attriute value and
  has sufficient permissions to allow reading.
* If the client has
    * Insufficient authorization to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authorization > >.
    * Insufficient security to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authentication > >.
    * Insufficient encryption key size to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption Key Size > >.
    * Not enabled encryption and encryption is required to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption > >.
* If the attribute value 
    * Can't be read due to permissions then an Error Response 
      shall be sent with the error code < < Read Not Permitted > >.
    * Has a fixed length that is less than or equal to (ATT_MTU-3) octets in length, then an *Error Response* can be sent
      with the error code < < Attribute Not Long > >.
* If the value offset of the Read Blob Request is
    * Greater than the length of the attribute value, and *Error Response*
      shall be sent with the error code < < Invalid Offset > >
    * Equal to the length of the attribute value, then the length of the part attribute value in the response shall be zero.
* If the handle is invalid, then an Error Response shall be sent with the error code
  < < Invalid Handle > >.
* If the attribute is longer than (ATT_MTU-1) octets, the 
  *Read Blob Request* is the only way to read the additional octets of a long attribute.
  The first (ATT_MTU-1) octets may be read using a Read Request, an Handle Value Notification or Handle Value Indication.
* Long attributes may or may not have their length specified by a higher specification.
  **If the long attribute has a variable length, the only way to get the end of it is to read it part by part until
  the value in the *Read Blob Response* has a length shorter than (ATT_MTU-1) or an *Error Response* 
  with the error code < < Invalid Offset > >**.
* **The value of long attribute may change betwen on Read Blob Request and the next Read Blob Request.
  A higher layer specification should be aware of this and define appropriate behavior**.

### Read Blob Response ###
* The *Read Blob Response* is sent in reply to a received* Read Blob Request* and contains
  part of the value of the attribute that has been read.
<img src="/images/att spec/ATT@ReadBlobResponseFormat.PNG" width="75%" height="75%"/>
* If the attribute value is longer than (Value Offset + ATT_MTU-1), then (ATT_MTU-1) octets
  from Value Offset shall be included in this response.

<a name="Read Multiple Request"/>
### Read Multiple Request ###
* The *Read Multiple Request* is used to request the server to read two or more 
  values of a set of attributes and return their values in a *Read Multiple Response*.
  **Only values that have a known fixed size can be read**, 
  with the exception of the last value that can have a variable length.
  *The knowledge of whether attributes have a known fixed size is defined in a higher specification*.
<img src="/images/att spec/ATT@ReadMultipleRequestFormat.PNG" width="75%" height="75%"/>
* The server shall respond with a Read Multiple Response if all the handles are valid
  and all attribute have sufficient permissions to allow reading. (全都要有效,有權限)
* The attribute values for the attributes in the Set Of Handles parameters 
  don't have to all be the same size.
* **The attribute handles in the Set Of Handles parameter don't have to be in attribute handle order;
  they are in the order that have the values are required in the response**.
  (attribute handles order 可以是使用者自己給, 而 response 的結果會依照使用者給的 order)
* If the client has
    * Insufficient authorization to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authorization > >.
    * Insufficient security to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authentication > >.
    * Insufficient encryption key size to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption Key Size > >.
    * Not enabled encryption and encryption is required to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption > >.
* If **any of the handles** are invalid, then an *Error Response*
  shall be sent with the error code < < Invalid Handle > >.
* If **any of the attribute values** can't be read due to permissions then an *Error Response*
  shall be sent with the error code < < Read Not Permitted > >.
* If an Error Response is sent, the Attribute Handle In Error parameter shall be 
  **set to the handle of the first attribut**e causing the error. (設定錯誤的 handle 為第一個遇到錯誤的)
 
### Read Multiple Response ###
* The *Read Multiple Response* is sent in reply to a received* Read Multiple Request* and contains
  part of the value of the attribute that has been read.
<img src="/images/att spec/ATT@ReadMultipleResponseFormat.PNG" width="75%" height="75%"/>
* The Set Of Values parameter shall be a concatenation of attribute values 
  for each of the attribute handles in the request in the order that they were requested.
* If the Set Of Values parameter is longer than (ATT_MTU-1),
  then only the first (ATT_MTU-1) octets shall be included in this response.
* A client should not use this request for attributes when the Set Of Values parameter
  could be (ATT_MTU-1) as it will not be possible to determine if the last attribute is complete, 
  or if it overflowed. 
  (不建議 client 在無法判斷每一個 attribute value　的狀況下使用這方式, 因為有可能在讀回資料長度為 ATT_MTU-1 的狀況下無法判斷是否最後一筆資料有讀完)

<a name="Read By Group Type Request"/>
### Read By Group Type Request ###
* The *Read By Group Type Request* is used to obtain the values of attributes where the attribute type is known, 
  the type of grouping attribute as defined by a higher layer specification,
  but the handle is not known.
<img src="/images/att spec/ATT@ReadByGroupTypeRequestFormat.PNG" width="75%" height="75%"/>
* Only the attributes with attribute handles between and including the Starting Handle and the Ending Handle
  with the attribute type that is the same as the Attribute Group Type given will be returned.
* To search through all attributes, the starting handle shall be set to 0X0001 
  and the ending handle shall be to 0XFFFF.
* All attribute types are effectively compared as 128-bit UUIDs, 
  even if a 16-bit UUID is provided in this request or defined for an attribute.
* The attributes returned shall be the attribues with the lowest handles within the handle range.
  These are known as the requested attributes.
* If the attribute with the requested type within the handle range have attribute values
  that have the same length, then these attributes can all be read in a single request.
  (相同長度的 attribute value　會被放在同一個 PDU 裡面, 不過必須要是連續的)
* The attribute server shall include as many attributes as possible in the response
  in order to minimize the number of PDUs required to read atttributes of the same type.
  (Sever 必須盡量將 PDU 中的內容填滿,　減少同一個 Type Request 需要回應的 PDUs 次數)
* If the attributes with the requested type within the handle range have attribute values
  with different lengths, then multiple *Read By Group Type Request*s must be made.
  (若指定 Group Type 的 attribue value 有不同長度的, 則必須透過多次的 Read By Group Type Request 完成讀取)
* When multiple attributes match, then the rules below shall e applied to each in turn:
    1. Only attributes that can be read shall be returned in a *Read By Group Response*.
    2. If an attribute in the set of requested attributes would cause an *Error Response*
       then this attribute can't be included in a Read By Group Type Response and
       the attributes before this attribute shall be returned.
    3. If the first attribute in the set of requested attributes would cause an *Error Response*
       then no other attributes in the requested attributes can be considered.
* If there are multiple atributes with the requested type within the handle range, 
  and the client would like to get the next attribute with the requested type,
  it would have to issue another Read By Groupd Type Request with its starting handle updated.
  The client can be sure there are no more such attributes remaining 
  once it gets an *Error Response* with the error code < < Attribute Not Found > >.
* If a server receives a *Read By Group Type Request* with the Starting Handle parameter
  greater than the Ending Handle parameter or the Starting Handle parameter is 0X0000,
  an *Error Response* shall be sent with the < < Invalid Handle > > error code;
  The Attribute Handle In Error parameter shall be set to the Starting Handle parameter.
* If the Attribute Group Type is not a supported grouping attribute
  as defined by a higher layer speficifaction then an Error Response shall be sent 
  with the error code < < Unsupported Group Type > >.
  The Attribute Handle In Error parameter shall be set to the Starting Handle parameter.
* If no attribute with the given type exists within the handle range, 
  then no attribute handle and value will be returned, and an *Error Response* shall be sent
  with the error code < < Attribute Not Found > >.
  The Attribute Handle In Error parameter shall be set to the Starting Handle parameter.
* If the client has
    * Insufficient authorization to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authorization > >.
    * Insufficient security to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authentication > >.
    * Insufficient encryption key size to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption Key Size > >.
    * Not enabled encryption and encryption is required to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption > >.
* If the attribute value can't be read due to permissions then an Error Response 
  shall be sent with the error code < < Read Not Permitted > >.

### Read By Group Type Response ###
* The *Read By Group Type Response* is sent in reply to a received *Read By Group Type Request* and contains
  part of the value of the attribute that has been read.
<img src="/images/att spec/ATT@ReadByGroupTypeResponseFormat.PNG" width="75%" height="75%"/>
* The *Read By Group Type Response* shall contain complete Attribute Data.
  An Attribute Data shall not be split across response packets.
* The Attribue Data List is ordered sequentially based on the attriute handles.
  (依照 handle 大小, 由小 -> 大)
* The maximum length of an Attribute Data is 255 octets, bounded by the Length parameter
  that is one octet.
  Therefore, the maximum length of an attribute value returned in this response is
  (Length-4) = 251 octets.
* The Attribute Data List shall be set to the value of the attribute 
  identified by the attribute type within the handle range within the request.
  If the attribute value is longer than (ATT_MTU-6) or 251 octets, whichever is smaller,
  then the first (ATT_MTU-6) or 251 octets shall be included in this response.
* The *Read Blob Request* would be used to read the remaining octets of a long attribute value.

### Question ###
1. 比較 Read By (Group) Type Request 與 Read Multiple Request,
   為何 Read By (Group) Type Request Request 的 Response 有帶 Length,
   而 Read Multiple Response 沒有帶 Length 呢？
   Ans: 
   因為後者在使用之前, 就必須知道指定的 Handle 是 Fixed Length.
   否則無法解讀出讀取物的內容.
   而前者是使用 (Group) Type 來查詢, 針對 Match 到的 Handle Value 長度,
   也許可以再反查, 但直接給定會比較有效率.