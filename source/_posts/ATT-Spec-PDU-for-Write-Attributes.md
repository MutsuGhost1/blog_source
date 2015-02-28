title: ATT Spec - PDU for Write Attributes
date: 2015-02-28 13:04:35
tags: [Bluetooth, Bluetooth Core Spec, Attribute Protocol]
---
## Write Attributes ##
<!--more-->

<a name = "Write Request"/>
### Write Request ###
* The *Write Request* is used to request the server to write the value of an attribute
  and acknowledge that this has been achieved in a *Write Response*.
<img src="/images/att spec/ATT@WriteRequestFormat.PNG" width="75%" height="75%"/>
* If the attribute value has a 
    * Variable length, then the attribute value shall be truncated or lengthened
      to match the length of the Attribute Value parameter.
        * If an attribute value has a variable length and 
          if the Attribute Value parameter is of zero length,
          the attribute value will be fully truncated.
    * Fixed length and the Attribute Value parameter length is 
      less than or equal to the length of the attribute value, 
      the octets up the attribute value parameter ength shall be written;
      all other octets in this atttribute value shal be unchanged.
* The server shall respond with a *Write Response* if the handle is valid, 
  the attribute has sufficient permissions to allow writing, 
  and the attribute has a valid size and format,
  and it's successful in writing the attribute.
* If the attribute has a
    * Variable length and the Attribute Value parameter length 
      exceeds the maximum valid length of the attribute value
      then the server shall respond with an *Error Response* with the
      error code < < Invalid Attribute Value Length > >.
    * Fixed length  and the Attribute Value parameter length 
      exceeds the maximum valid length of the attribute value
      then the server shall respond with an *Error Response* with the
      error code < < Invalid Attribute Value Length > >.
* If the client has
    * Insufficient authorization to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authorization > >.
    * Insufficient security to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Authentication > >.
    * Insufficient encryption key size to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption Key Size > >.
    * Not enabled encryption and encryption is required to read the requested attribute then an *Error Response*
      shall be sent with the error code < < Insufficient Encryption > >.
* If the handle is invalid, then an *Error Response* shall be sent with the error code < < Invalid Handle > >.
* If the attribute value can't be written due to permissions then an *Error Response*
  shall be sent with the error code < < Write Not Permitted > >.
* If the attribute value can't be written due to an application error then an *Error Response*
  shall be sent with an error code defined by a higher layer specification.

### Write Response ###
* The *Write Response* is sent in reply to received *Write Request* and
  acknowledges that the attribute has been successfully written.
<img src="/images/att spec/ATT@WriteResponseFormat.PNG" width="75%" height="75%"/>
* The *Write Response* shall be sent after the attribute value is written.

<a name = "Write Command"/>
### Write Command ###
* The Write Command is used to request the server to write the value of an attribute,
  **typically into a control-point attribute**.
<img src="/images/att spec/ATT@WriteCommandFormat.PNG" width="75%" height="75%"/>
* The attribute handle parameter shall be set to a valid handle.
* The attribute value parameter shall be set to the new value of the attribute.
* If the attribute value has a 
    * Variable length, then the attribute value shall be truncated or lengthened
      to match the length of the Attribute Value parameter.
        * If an attribute value has a variable length and 
          if the Attribute Value parameter is of zero length,
          the attribute value will be fully truncated.
    * Fixed length and the Attribute Value parameter length is 
      less than or equal to the length of the attribute value, 
      the octets up the attribute value parameter ength shall be written;
      all other octets in this atttribute value shal be unchanged.
* If the attribute has a
    * Variable length and the Attribute Value parameter length 
      exceeds the maximum valid length of the attribute value
      then the server shall ignore the command.
    * Fixed length  and the Attribute Value parameter length 
      exceeds the maximum valid length of the attribute value
      then the server shall ignore the command.
* **No *Error Response* or *Write Resonse* shall be sent in response to this comand**.
* If the server can't write this attribute for any reason the command shall be ignored.

<a name = "Signed Write Command"/>
### Signed Write Command ###
* The *Signed Write Command* is used to request the server to write the value of an attribute
  **with an authentication signature, typically into a control-point attribute**.
<img src="/images/att spec/ATT@SignedWriteCommandFormat.PNG" width="75%" height="75%"/>
* The attribute handle parameter shall be set to a valid handle.
* The attribute value parameter shall be set to the new value of the attribute.
* If the attribute value has a 
    * Variable length, then the attribute value shall be truncated or lengthened
      to match the length of the Attribute Value parameter.
        * If an attribute value has a variable length and 
          if the Attribute Value parameter is of zero length,
          the attribute value will be fully truncated.
    * Fixed length and the Attribute Value parameter length is 
      less than or equal to the length of the attribute value, 
      the octets up the attribute value parameter ength shall be written;
      all other octets in this atttribute value shal be unchanged.
* If the attribute has a
    * Variable length and the Attribute Value parameter length 
      exceeds the maximum valid length of the attribute value
      then the server shall ignore the command.
    * Fixed length  and the Attribute Value parameter length 
      exceeds the maximum valid length of the attribute value
      then the server shall ignore the command.
* If the authentication signature verification fails, 
  then the server shall ignore the command.
* **No *Error Response* or *Write Resonse* shall be sent in response to this comand**.
* If the server can't write this attribute for any reason the command shall be ignored.