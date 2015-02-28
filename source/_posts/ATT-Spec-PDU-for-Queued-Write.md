title: ATT Spec - PDU for Queued Write
date: 2015-02-28 13:52:12
tags: [Bluetooth, Bluetooth Core Spec, Attribute Protocol]
---
## Queued Write ##
<!--more-->

* The purpose of queued writes is to queue up writes of values 
  of multiple attributes in a first-in first-out queue and
  then execute the write on all of them in a single atomic operation. 

<a name="Prepare Write Request"/>
### Prepare Write Request ###
* The *Prepare Write Request* is used to request the server 
  to prepare to write the value of an attribute.
  The server will respond to this requet with a *Prepare Write Response*,
  so that the client can verify that the value was received correctly.
* A client may send more than one *Prepare Wrie Request* to a server,
  which will queue and send a response for each handle value pair.
* A server may limit the number of *Prepare Write Requests* that 
  it can accept. A higher layer specification should define this limit.
* After a Prepare Write Request has been issued, and the response received,
  any other attribute command or request can be issued from the same client 
  to same server.
  (完成一個完整的 Prepare Write Request/Response, Client 又可以穿插其它 Request/Command)
* Each client's queued value are separate; the execution of one queue shall not
  affect the preparation or execution of any other client's queued values.
  (每個 client 的 prepare write queue 都是獨立運行, 不會互相影響)
* Any actions on attributes that exist in the prepare queue shall proceed
  as if the prepare queue didn't exist, and the prepare queue 
  shall be unaffected by these actions.
  A subsequent execute write will write the values in the prepare queue
  even if the value of the attribute has changed 
  since the prepare writes were started.
  (在 prepare write 的執行過程中, 如果 attribute 被改變, 也並不會影響未完成的 prepare write queue)
* **Each *Prepare Write Request* will be queued even if the attribute handle is the same
  as a previous Prepare Write Request.
  These will then be executed in the order received, 
  causing multiple writes for this attribute to occur**.
* **If the link is lost while a number of prepared write requests have been queued,
  the queue will be cleared and no writes will be execute**.
  (執行到一半還沒完成 Execute Write 的 Prepare Write 遇到斷線, 則 queue 全清空, 當沒發生)
* The attribute protocol makes no determination on the validity 
  of the Part Attribute Value or the Vlue Offset.
  A higher layer specification determines the meaning of the data.
  (執行 Prepare Write Request/Response 只能 Check Permission, 至於值的正確性要在 Execute Write Request/Response 完成)
<img src="/images/att spec/ATT@PrepareWriteRequestFormat.PNG" width="75%" height="75%"/>
* The Value Offset parameter shall be set to the ofset of the first octet 
  where the Part Attribute Value parameter is to be written within the attribute value.
  The Value Offset parameter is based from zero;
  the first octet has an offset of zero, the second octet has an offset of one, etc.
* The server shall respond with a *Prepare Write Response* if the handle is valid,
  the attribute has sufficient permissions to allow writing at this time,
  and the prepare queue has suffcient space.
* **The Attribute Value validation is done when an *Execute Write Request* is received**. 
  Hence, any Invalid Offset or Invalid Attribute Value Length errors are generated
  when an Execute Write Request is received.
* If the client has
    * Insufficient authorization to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authorization > >.
    * Insufficient security to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authentication > >.
    * Insufficient encryption key size to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption Key Size > >.
    * Not enabled encryption and encryption is required to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption > >.
* If the server doesn't have sufficient space to queue this request then an *Error Response* shall be sent
  with the error code < < Prepare Queue Full > >.
* If the handle is invalid, then an *Error Response* shall be sent
  with the error code < < Invalid Handle > >.
* If the attribute value can't be written then an *Error Response* shall be sent
  with the error code < < Write Not Permitted > >.
* The server shall not change the value of the attribute until an *Execute Write Request* is received.
* If a Prepare Write Request was invalid, and therefore an Error Response has been issued,
  then this prepare write will be considered to have not been received.
  All existing prepared writes in the prepare queue shall 
  not be affected by this invalid request.


### Prepare Write Response ###
* The *Prepare Write Response* is sent in response to a received *Prepare Write Request*
  and **acknowledges that the value has been successfully received**
  and **placed in the prepare write queue**.
<img src="/images/att spec/ATT@PrepareWriteResponseFormat.PNG" width="75%" height="75%"/>
* The attribute handle shall be set to the same value as 
  in the corresponding *Prepare Write Request*.
* The Value Offset and Part Attribute Value shall be set to the same values as
  in the corresponding *Prepare Write Request*. 

<a name="Execute Write Request"/>
### Execute Write Request ###
* The Execute Write Request is used to request the server to write or cancel
  the write of all the prepared values currently held in the prepare queue
  from this client.
  This request shall be handled by the server as an atomic operation.
<img src="/images/att spec/ATT@ExecuteWriteRequestFormat.PNG" width="75%" height="75%"/>
* When the Flags parameter is set to 
    * 0X01, values that were queued by the previous prepare write requests 
      shall be written in the order they were received in the 
      corresponding Prepare Write Request.
      The queue shall then be cleared, and an *Execute Write Response* shall be sent.
    * 0X00, all pending prepare write values shall be discarded for this client.
      The queue shall then be cleared, and an *Execute Write Response* shall be sent.     
* If the prepared Attribute Value exceeds the maximum valid length of the attribute value
  then all pending prepare write values shall be discarded for this client,
  the queue shall then be cleared, and an *Error Response* shall be sent
  with the error code < < Invalid Attribute Value Length > >.
* If the prepare Value Offset is greater than the current length of the attribute value, 
  then all pending prepare write values shall be discarded for this client,
  the queue shall then be cleared, and an *Error Response* shall be sent
  with the error code < < Invalid Offset > >.
* If the prepare write requests can't be written, due to an application error,
  the queue shall be cleared and then an *Error Response* shall be sent with
  a higher layer specification defined error code.
  The Attribute Handle In Error parameter shall be set to the attribute handle
  of the attribute from the prepare queue that caused this application error.
  The state of the attributes that were to be written from the prepare queue
  is not defined in this case.
* Execute Write 時, 只要有一個 attribute value 發生問題, 就捨棄所有的.
  *Error Response* 中, 會紀錄發生問題的 attribute handle.

### Execute Write Response ###
* The *Execute Write Response* is sent in response to received *Execute Write Request*.
<img src="/images/att spec/ATT@ExecuteWriteResponseFormat.PNG" width="75%" height="75%"/>
* The Execute Write Resonse shall be sent after the attributes are written.
  In case an action is taken in response to the write,
  an indication may be used once the action is complete.