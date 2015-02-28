title: ATT Spec - PDU for Error Handling
date: 2015-02-27 17:20:06
tags: [Bluetooth, Bluetooth Core Spec, Attribute Protocol]
---
## Error Handling ##
<!--more-->
### Error Response ###
* The Error Resonse is used to state that a given request can't be performed,
  and to provide the resason.
* The Write Command doesn't generate a Error Response.
<img src="/images/att spec/ATT@ErrorResponseFormat.PNG" width="75%" height="75%"/>
* The Error Code parameter shall be set to one of the following values:
<img src="/images/att spec/ATT@ErrorCode.PNG" width="75%" height="75%"/>
* If an error code is received in the Error Response that is not understood by the client,
  for example an error code that was reserved for future use that is now
  being used in a future version of this speficication, then the
  Error Response shall still be considered to state that the given request
  can't be performed for an unknown reason.