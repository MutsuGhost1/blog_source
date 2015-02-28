title: ATT Spec
date: 2015-02-26 23:02:28
tags: [Bluetooth, Bluetooth Core Spec, Attribute Protocol]
---

## Protocol Overview ##
<!--more-->
<img src="/images/att spec/ATTOverview.PNG" width="50%" height="50%"/>

> This specification defines the Attribute Protocol.
> A protocol for discovering, reading, and writting attributes on a peer device.

### ATT ###
* ATT = Attribute Protocol
* ATT defines two roles:
    * Client uses ATT to access the server.
    * Server exposes a set of attributes to client.

### ATT @ Protocol Stack ###
<img src="/images/att spec/ATT@ProtocolStack.PNG" width="75%" height="75%"/>

* The above figure shows ATT @ the BLE Protocol Stack.
* Use different ATT bearer(LE or BR/EDR) will have different default ATT_MTU size.
* 注意一點, 通常 Header 都會包含 Length 資訊, 只有 ATT PDU 沒有 Header 喔!

### Attribute ###
<img src="/images/att spec/ATT@Attribute.PNG" width="75%" height="75%"/>
* An attribute is composed by the following four parts:
    <a name="Attribute handle"/>
    1. Attribute handle
        * A 16 bit non-zero value
        * From 0X0001 to 0XFFFF
        * Uniquely identifies an attribute on a server
    <a name="Attribute type"/>
    2. Attribute type, defined by a UUID (16-bit or 128-bit)
        * Specifies what the attribute represents 
        * UUID
            * Short for universally unique identifier
            * A UUID is 128-bit value
            * 16-bit UUID
                * It's used for efficiency.
                * 16-bit Attribute UUID * 2^96 + Bluetooth_Base_UUID = 128-bit UUID
                    * 0000xxxx-0000-1000-8000-00805F9B34FB (xxxx is the 16-bit value)
                    * 00000000-0000-1000-8000-00805F9B34FB also called as **Bluetooth_Base_UUID**
                * Use the same namespace as SDP 16-bit UUIDs.
                * Assigned by Bluetooth SIG and published in the Bluetooth Assigned Numbers Page.
    <a name="Attribute value"/>
    3. Attribute value
        * An octet array that may be either fixed or variable length.
        * Each attribute type has its own format.
    <a name="Attribute permission"/>
    4. A set of permission 
        * **These permisssion can't be accessed using the ATT**
        * Add additional security requirement
        * Defined by each higher layer spec that utilizes the attribute.

### Interaction between client and server using ATT ###
* A client may send attribute protocol requests to a server,
  and the server shall respond to all reuests that it receives.
* A device can implement both client and server roles,
  and both roles can function concurrently in the same device and between the same devices.
* **There shall be only one instance of a server on each Bluetooth device.
  This implies that the attribute handles shall be identical for all supported bearers**.
* A server can support multiple clients.
* ATT has notification and indication capabilities that
  provides an efficient way of sending attribute values 
  to a client without the need for them to be read.


----------
## Protocol Requirements ##

### Introduction ###
* Each attribute:
    1. Has an attribute type that identifies what the attribute represents.
    2. Has an attribute handle that is used for accessing the attribute on a server.
    3. Has a set of permissions that controls whether they can be read or written,
       or whether the attribute value shall be sent ovver an encrypted link 
* **Attributes that have the same attribute type may exist more than once in a server**.

### Basic Concepts ###

#### Attribute Handle ####
* See <a href="#Attribute handle">Attribute handle</a>
* An attribute handle shall not be reused while an ATT Bearer exists
  between a client and its server.
* Attributes can be added or removed while an ATT Bearer is active,
  however, an attribute that has been removed can't be replaced
  by another attribute with the same handle while an ATT Bearer is active.

#### Attribute Type ####
* See <a href="#Attribute type">Attribute type</a>

#### Attribute Value ####
* See <a href="#Attribute value">Attribute value</a>
* The values that are transmitted are opaque to the attribute protocol.
  The encoding of these octet arrays is defined by the attribute type.
    * 簡單說, 因為不同 attribute type 有不同的 format, 這是被更上層的 protocol 定義的,
      ATT 無法知道其內容格式, 更別說要知道其長度. 
* The length of attribute value will different for each attribute type.
* The length of a variable length field in the PDU is implicit given
  by the length of the packet that carries this PDU. This implies:
    1. Only one attribute value can be placed in a single request, response, notification or indication
       unless the attribute values have lengths known by both the server and client,
       as defined by the attribute type.
    2. The attribute value will always be the only variable length field
       of a request, response, notification or indication.
    3. The bearer protocol (e.g. L2CAP) preserves datagram boundaries.
* **Some responses include multiple attributes, for example when client requests multiple attribute read**.
  **For the client to determine the attribute value boundaries**,
  **the attribute values must have a fixed size defined by the attribute type**.
* 上面一堆論述, 其實是在講, 由於不同的 attribute type 有不同的 attribute value, 其長度有可能是變動的.
  而這些長度的變動, 是被定義在 ATT 以上的應用.
  因此 ATT 的各種 PDU 裡面, 如果要能解讀 attribute value 的長度,
  要嘛就把這部分的變動, 都放在 PDU 資料的最尾端, 且只有一份.
  如果要放多份, 在這個 PDU 裡面, 就必須有資訊能夠告訴上層的應用, 怎麼去拆解出一個一個的 attribute value.
    * See Read Request (Response 只有一個 attribute value, 且放最尾端)
    * See Read By Type Request (Response 有多個 attribute values, 但 Response 資料中, 把一樣長的 attribute value 集中在同個 PDU, 且出 attribute value 的 length 資訊)

#### Attribute Permission ####
* See <a href="#Attribute permission">Attribute permission</a>
* 這邊沒搞懂 Authentication/Authorization/Encryption 的各種使用情境.

#### Attribute Handle Grouping ####
* Grouping is defined by a specific attribute placed 
  at the beginning of a range of other attributes 
  that are grouped with that attribute.
* Defined by a higher layer specification.
* Client can request the first and last handles associated with a group of attributes.
    * See Find By Type Value Request  
* 簡單講就是更上層的 Protocol 定義某個 Attribute 做為各個 Group 的第一個, 這樣就可以區分出不同 Group.
    * e.g. Service Definition 視為一個 Group.
    * e.g. Characteristic Definition 視為一個 Group.

#### Control-Point Attribute ####
* Attributes that can't be read, but can only be written, notified or indicated are called it.
* It's used by higher layers to enable device specific procedures.
    * For example, the writing of a command or the indication when a given procedure on a device has completed. 

#### Protocol Methods ####
* Use to find/read/write/notify or indicate attributes.
* Protocol methods can be categoried as:
* Some attribute protocol PDUs can also include an Authentication Signature,
  to allow authentication of the originator of this PDU without requiring encryption.
* The method and signed bit are known as the opcode. (透過 opcode 可以知道是哪個 protocol method, 且是否有 signed 功能)

#### Exchaning MTU Size ####
* ATT_MTU is defined as the max size of package sent between a client and a server.
* A higher layer specification defines the default ATT_MTU value. (根據應用可以自訂其 default ATT_MTU value)
* Client and server can optionally exchange the max size of ATT_MTU,
  but both devices then use the minimum of these exchanged values for further communication.
* A device that is acting as a server and client at the same time shall
  use the same value for Client Rx MTU and Server Rx MTU.
* **Different ATT Bearer can have different ATT_MTU size**.

#### Long Attribute Values ####
* The longest attribute that can be sent in a single packet is (ATT_MTU-1) octets in size.
  At a minimum, the Attribute Opcode is included in an Attribute PDU.
  (一個 single packet 最大只能傳 ATT_MTU-1. 最少就是只傳 Opcode)
* An attribute value may be defined to be larger than (ATT_MTU-1) octets in size.
  These attribute are called long attributes.
* For reading attribute value > ATT_MTU-1: (See Read Resonse)
    * Use Read Blob Request to read whole long attribute value.
    * Read Request only can read the first ATT_MTU-1 octets of a long attribute value.  
* For writing attribute value > ATT_MTU-3: (See Write Request)
    * Use Prepare Write/Execute Write to write whole long attribute value.
    * Write Request only can write the first ATT_MTU-3 octets of a long attribute value
* It is not possible to determine if an attribute value is longer than (ATT_MTU-3) octets
  using this protocol. A higher layer specification will state that a given attribute can
  have a max length larger than (ATT_MTU-3) octets. 
  (由於 ATT 無法知道 attribute value 的長度, 而 ATT 以上的 Spec 通常會定義 Max ATT_MTU & Attribute Value Length, 因此該由 ATT 以上的 Spec 來說明有這件事情, 而非 ATT 這邊來說明)
* **The maximum length of an attribute value shall be 512 octects**.
* **The protection of an attribute value changing when reading the value
  using multiple attribute protocol PDUs is the responsibility of the higher layer**.


#### Atomic Operations ####
* The server shall treat each request or command as an atomic operation 
  that can't be affected by another client sending a request or command at the same time.
* If a link is disconnected for any reason, the value of any modified attribute is
  the responsibility of the higher specification.
* Long attributes can't be read or written in a single atomic operation.

### Attribute PDU ###
* Attribute PDUs are one of six method types:

|  Method Type  |                     Description                          |
| ------------- |:--------------------------------------------------------:|
| Requsts       | Client --> Server  (Request and Response are pair)       |
| Responses     | Server --> Client  (Request and Response are pair)       |
| Commands      | Client --> Server  (No ack)                              |
| Indications   | Server --> Client  (Indication and Confirmation are pair)|
| Confirmations | Client --> Server  (Indication and Confirmation are pair)|
| Notifications | Server --> Client  (No ack)                              |

* If client sends a request, then the client shall support all possible responses
  PDUs for that request.
* A server shall be able to receive and properly response to the following request:
    1. Find Information Request
    2. Read Request
* Support for all other PDU types in a server can be specified in
  a higher layer specification.
* If server
    * Receives a request that it doesn't support, then the server shall respond with
      the Error Response with the Error Code < < Request Not Supported > > ,
      with the Attribute Handle In Error set to 0X0000.
    * Receives a command that it doesn't support, then the server shall ignore the command.
    * Receives an invalid request, then the server shall respond with
      the Error Response with the Error Code < < Invalid PDU > >,
      with the Attribute Handle In Error set to 0X0000.
    * Doesn't have sufficient resources to process a request, then the server shall respond with
      the Error Response with the Error Code < < Insufficient Resources > >,
      with the Attribute Handle In Error set to 0X0000.
    * Can't process a request because an error was encountered during the processing of this request,
      then the server shall respond with
      the Error Response with the Error Code < < Unlikely Error > >,
      with the Attribute Handle In Error set to 0X0000.

#### Attribute PDU Format ####
* A general form for Request/Response/Command/Indication/Confirmation/Notification.

<img src="/images/att spec/ATT@PDUFormat.PNG" width="75%" height="75%"/>

* Multi-octet fields within the attribute protocol shall be sent
  least significant octet first (little endian) with the exception of the Attribute Value field.
  The endianness of the Attribute Value field is defined by a higher layer specification.  
* The Authentication Signature field is calculated as defined in Security Manager.
* An Attribute PDU that includes an Authentication Signature should not be send on an encrypted link.
  An encrypted link already includes authentication data on every packet and
  therefore adding more authentication data is not required.
* **Only the Write Command may include an Authentication Signature**.

#### Sequential Protocol ####
* Many attribute protocol PDUs use a sequential request-response protocol.
    * For Request/Response
        * It means once a client sends a request to a server,
          that client shall send no other request to the same server until
          a response PDU has been received.
    * For Indication/Confirmation
        * No other indications shall be sent to the same client from this server until
          a confirmation PDU has been received.
        * However, the client is free to send commands and requests prior to sending a confirmation.
          (如果 server 正要做 indication, client 其實也不知道, 因此這時候還是可以做 send commands and requests) 
    * For notifications
        * Which don't have a response PDU, there's no flow control
          and a notification can be sent at any time.
        * Received but can't be processed, due to buffer overflows or other reasons, shall be discarded.
        * It's unreliable.
    * For commands 
        * Don't require a reponse don't have any flow control.
        * A server can be flooded with commands, 
          and a higher layer specification can define how to prevent this from occurring.
        * Received but can't be processed, due to buffer overflows or other reasons, shall be discarded.
        * It's unreliable.
* Flow control for each client and server is independent.
* About client behavior:
    * It's possible for a client to receive an indication from a server and
      then send a request or command to that server before sending the confirmation
      of the original indication.  
* About server behavior:
    * It's possible for a server to receive a request, send one or more notifications,
      and then the response to the original request.
    * It's possible for a server to receive a request and then a command
      before responding to the original request.
    * It's possible for a notification from a server to be sent 
      after an indication has been sent but the confirmation has not been received.  

#### Transaction ####
* A single transaction:
    * Pair of request/response.
    * Pair of indication/confirmation.
* A transaction shall always be performanced on one ATT Bearer,
  and shall not be split over multiple ATT Bearer.
* For request/response:
    * On a client (Initiator), a transaction shall start when the request is sent by the client.
      A transaction shall complete when the response is received by the client.
    * On a server (Responder), a transaction shall start a request is received by the server.
      A transaction shall complete when the response is sent by the server. 
* For indication/confirmation:
    * On a server (Initiator), a transaction shall start when an indication is sent by the client.
      A transaction shall complete when the confirmation is received by the server.
    * On a client (Responder), a transaction shall start when an indication is received by the client.
      A transaction shall complete when the confirmation is sent by the client.
* **A transaction shall time out if it's not complete within 30 seconds**.
    * Such a transaction shall be considered to have failed and 
      the local higher layers shall be informed of this failure.
    * No more attribute protocol requests, commands, indications or notifications
      shall be sent to the target device on this ATT earer.    
        * To send another attribute protocol PDU, a new ATT Bearer must be established between these devices.
          The existing ATT Bearer may need to be disconnected 
          or the bearer terminated before the new ATT Bearer is established.
* If the ATT Bearer is disconnected during a transaction:
    * The transaction shall be considered to be closed,
      and any values that were being modified on the server will be in an undetermined state
    * Any queue that was prepared by the client using this ATT Bearer shall be cleared.
* Each Prepare Write Request is a separate request and is therefore a separate transaction.
* Each Read Blob Request is a separate request and is therefore a separate transaction.

### Attribute Protocol PDUs ###

| Function Category |                   Related Operation                      |      Description             |
| ----------------- |:--------------------------------------------------------:|:-----------------------------:
| MTU Exchange      | [Exchange MTU Request/Response](/2015/02/27/ATT-Spec-PDU-for-MTU-Exchange/) |  Exchange Max ATT_MTU Size   |
| Find Information  | [Find Information Request/Response](/2015/02/27/ATT-Spec-PDU-for-FindInformation/#Find_Information_Request)| 給定 Handle Range, 取得 Range 內的 Handle 與 UUID|
|                   | [Find By Type Value Request/Response](/2015/02/27/ATT-Spec-PDU-for-FindInformation/#Find_By_Type_Value_Request)| 給定 Handle Range/Type/Value, 取得 Range 內符合條件的 Handle 與 Group End Handle 之 Pair List|
| Read Attributes   | [Read By Type Request/Response](/2015/02/27/ATT-Spec-PDU-for-Read-Attributes/#Read_By_Type_Request)| 給定 Handle Range/Type, 取得符合條件的 Handle 與 Value 之 Pair List|
|                   | [Read Request/Response](/2015/02/27/ATT-Spec-PDU-for-Read-Attributes/#Read_Request)| 給定 Handle, 取得指定 Handle 的 Value (first ATT_MTU-1 bytes)|
|                   | [Read Blob Request/Response](/2015/02/27/ATT-Spec-PDU-for-Read-Attributes/#Read_Blob_Request)| 給定 Handle/Value Offset, 取得指定 Handle 的 Value (start from offset)|
|                   | [Read Multiple Request/Response](/2015/02/27/ATT-Spec-PDU-for-Read-Attributes/#Read_Multiple_Request)| 給定 Set Of Handles, 依序取得 Set Of Values (Client 自己要會解讀)|
|                   | [Read by Group Type Request/Response](/2015/02/27/ATT-Spec-PDU-for-Read-Attributes/#Read_By_Group_Type_Request)| 給定 Handle Range/Group Type, 取得符合條件的 Handle/End Group Handle 與 Value 之 Pair List|
| Write Attributes  | [Write Request/Response](/2015/02/28/ATT-Spec-PDU-for-Write-Attributes/#Write_Request)| 給定 Handle/Value, 將指定的 Value 寫入 Server (最多寫 first ATT_MTU-3 bytes)|
|                   | [Write Command](/2015/02/28/ATT-Spec-PDU-for-Write-Attributes/#Write_Command)| 給定 Handle/Value, 將指定的 Value 寫入 Server (最多寫 first ATT_MTU-3 bytes)|
|                   | [Signed Write Command](/2015/02/28/ATT-Spec-PDU-for-Write-Attributes/#Signed_Write_Command)| 給定 Handle/Value, 將指定的 Value 寫入 Server (最多寫 first ATT_MTU-13 bytes)|
| Queue Writes      | [Prepare Write Request/Response](/2015/02/28/ATT-Spec-PDU-for-Queued-Write/#Prepare_Write_Request)| 給定 Handle/Value Offset/Value, 將指定的 Value 從 Offset 暫存在 Server (最多寫 ATT_MTU-5 bytes) |
|                   | [Execute Write Request/Response](/2015/02/28/ATT-Spec-PDU-for-Queued-Write/#Execute_Write_Request)| 將暫存在 Server 的 Prepare Write 全部寫入|
| Server Initiated  | [Handle Value Notification](/2015/02/28/ATT-Spec-PDU-for-Server-Initiated/#Handle_Value_Notification)| 將 Handle Value 主動通知給 Client|
|                   | [Handle Value Indication/Confirmation ](/2015/02/28/ATT-Spec-PDU-for-Server-Initiated/#Handle_Value_Indication)| 將 Handle Value 主動通知給 Client|
| Error Handling    | [Error Response](/2015/02/27/ATT-Spec-PDU-for-Error-Handling) | All the error response format are the same |

## Security Consideration ##
* The attribute protocol can be used to access information that may 
  require the following permission before an attribute can be read or written:
    * Authorization
    * Authenticated physical link
    * Encrypted physical link
* If such a request is issued when
    * The client has not been authorized to access this information,
      the server shall send an Error Response with the error code < < Insufficient Authorization > >.
      Each device implementation will determine how aughorization occurs.
      Authorization procedures are defined in GAP, and may be further refined in a higher layer specification.
    * The physical link is unauthenticated, the server shall send an Error Response with the error code set to
      < < Insufficient Authentication > >.
      A client wanting to read or write this attribute can then request that the physical link be authenticated,
      and once this has been completed, send the request again.
* The attribute protocol can be used to notify or indicate the value of an attribute that may require an authenticated
  and encrypted physical link before an attribute notification or indication is performed.
  A server wanting to notify or indicate this attribute can then request that the physical link be authenticated,
  and once this has been completed send the notification or indication.
* The list of attributes that a device supports is not considered private or confidential information,
  and therefore the *Find Information Reques*t shall always be permited.
  This implies that an < < Insufficient Authorization > > or < < Insufficient Authentication > > 
  error code shall not be used in an Error Response for a *Find Information Request*.
* When a client accesses an attribute, the order of checks that are performed on
  the server will have security implications.
  A server shall check authentication and authorization requirements 
  before any other check is performed.
  * If the authentication and authorization requirement checks are not
    performed first then the size of an attribute could be determined by
    performing repeated read blob requests on an attribute that a client doesn't
    have access to, because either an < < Invalid Offset > > error code or
    < < Insufficient Authentication > > error codes would be returned.
   (這是因為合理的 Offset 會造成 Insuffficient Authentication, 而不合理的 Offset 會造成 Invalid Offset, 這就是一個檢查順序造成的漏洞)

## Questions ##
1. Authorization/Authentication/Encryption 的 Use Cases ?
2. 為何只有 Write Command 可以用 Signed ?

## Reference ##
1. [ATT Spec 中文說明](http://www.cnblogs.com/hzl6255/p/4141505.html "ATT Spec 中文說明")