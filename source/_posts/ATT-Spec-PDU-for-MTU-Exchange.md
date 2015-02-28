title: ATT Spec - PDU for MTU Exchange
date: 2015-02-27 17:39:13
tags: [Bluetooth, Bluetooth Core Spec, Attribute Protocol]
---
## MTU Exchange ##
<!--more-->
### Exchange MTU Request ##
* The Exchange MTU Request is used by the client to inform the server
  of the client's maximum receive MTU size and request the server
  to respond with its maximum receive MTU size.
<img src="/images/att spec/ATT@ExchangeMTURequestFormat.PNG" width="75%" height="75%"/>
* The client Rx MTU shall be greater than or equal to the default ATT_MTU.
### Exchange MTU Response ##
* The Exchange MTU Response is sent in reply to a received Exchange MTU Request.
<img src="/images/att spec/ATT@ExchangeMTUResponseFormat.PNG" width="75%" height="75%"/>
* The server Rx MTU shall be greater than or equal to the default ATT_MTU.
* The server and client shall set ATT_MTU to the minimum of the client Rx MTU
  and the server Rx MTU.
  **The size is the same to ensure that a client can correctly detect the final
  packet of a long attribute read**.
* This ATT_MTU value shall be applied: 
    * In the server after this response has been
      sent and before any other attribute protocol PDU is sent.
    * In the client after this response has been 
      received and before any other attribute protocol PDU is sent.
* If a device is both a client and a server, the following rules shall apply:
    1. A device's Exchange MTU Request shall contain the same MTU as the
       device's Exchange MTU Response. (i.e. the MTU shall be symmetric)
    2. If MTU is exchanged in one direction, that is sufficient for both directions.
    3. It's permitted, (but not necessary - see 2) to exchange MTU in both
       directions, but the MTUs shall be the same in each direction. (see 1)
    4. If an Attribute Protocol Request is received after the MTU Exchange Request
       is sent and before the MTU Exchange Response is received,
       the associated Attribute Protocol Resonse shall use the default MTU.
    5. Once the MTU Exchange Request has been sent, the initiating device shall
       not send an Attribute Protocol Indication or Notification until after the
       MTU Exchange Response has been received.
       (This stops the risk of crossover condition where MTU size is unknown 
       for the Indication or Notification)