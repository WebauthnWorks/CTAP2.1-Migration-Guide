
# AuthenticatorBioEnrollment (0x09)



The authenticatorBioEnrollment commands provides the platform the ability to create, delete and list bio enrollments in the authenticator. 

Using this API, the platform is able to:

- Create/enroll a fingerprint
- Cancel fingerprint enrollments before they finish
- Find all fingerprints the have been added
- Give the fingerprint an alias (friendlyName)
- Delete a fingerprint
- Get info about fingerprint sensor

This command requires support of PinUvAuthProtocol 2

The platform must obtain pinUvAuthToken with `be` permission flag

The specified PinUvAuthToken will be used in all future examples:

```SessionPUAT = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad```



## Platform Request

Request keys ENUM:

```
modality             : 0x01 
subCommand           : 0x02 
subCommandParams     : 0x03 
pinUvAuthProtocol    : 0x04 
pinUvAuthParam       : 0x05 
getModality          : 0x06 
```

Modalities supported:
```
fingerprint          : 0x01
```

Sub Commands ENUM:

```
enrollBegin                : 0x01
enrollCaptureNextSample    : 0x02
cancelCurrentEnrollment    : 0x03
enumerateEnrollments       : 0x04
setFriendlyName            : 0x05
removeEnrollment           : 0x06
getFingerprintSensorInfo   : 0x07
```

Sub Command Param Keys:
```
templateId 		: 0x01
templateFriendlyName	: 0x02
timeoutMilliseconds 	: 0x03
```


Response keys ENUM:
```
modality                                : 0x01
fingerprintKind                         : 0x02
maxCaptureSamplesRequiredForEnroll      : 0x03
templateId                              : 0x04
lastEnrollSampleStatus                  : 0x05
remainingSamples                        : 0x06
templateInfos                           : 0x07
maxTemplateFriendlyName                 : 0x08
```
**modality:**
User verification modality.

**fingerprintKind:**
Type of fingerprint sensor. Touch type sensor = 1. Swipe type sensor = 2.

**maxCaptureSamplesRequiredForEnroll:**
Maximum good samples required for enrollment

**templateId:**
Template Identifier. 

**lastEnrollSampleStatus:**
Last enrollment sample status. Values between 0x00 and 0x0E.
[View authenticatorBioEnrollment specs](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#authenticatorBioEnrollment) for reference.

**remainingSamples:**
Number of more sample required for enrollment to complete

**templateInfos:**
Array of templateInfos. 

**templateInfo**
```
templateId              : 0x01 // required
templateFriendlyName    : 0x02 // optional
```

**maxTemplateFriendlyName:**
Indicates the maximum number of bytes the authenticator will accept as a templateFriendlyName.



<br/><br/>
## Computing PinUvAuthParam

Based on exchange as specified in PinProtocol, we use the below value as a PinUvAuthToken test vector for authenticatorBioEnrollment in **[Example 1](#example-1---enrolling-a-fingerprint-0x01)**

BioEnrollment specifies the following structure to compute PinUvAuthParam
```
message         = 01 || subCommand || subCommandParams
pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)
```
Example PinUvAuthParam for `enrollBegin (0x01)` subcommand
```
subCommand = 0x01
subCommandParams = {
	0x03: 10000 // 10000ms timeout as arbitrary example
}
ENCODED: a103192710 (TODO verify values)

message = 0101a103192710

pinUvAuthParam = 66e4e667922def036d0cc065fa6429f8d94910a7848a6853c01db49d50a4c48a
```


**IMPORTANT NOTE:**

For pinUvAuthParam, PinUvAuthProtocol 1 returns first 16 bytes of the HMAC output, whereas PinUvAuthProtocol 2 is returning the full 32 byte HMAC output (See #HMAC in PinProtocol 2)

**!! All examples below use exclusively PinUvAuthProtocol 2 !!**
<br/><br/>
## Enrolling a fingerprint - EnrollBegin (0x01)	EnrollCaptureNextSample (0x02)
**EnrollBegin (0x01)**
- To enroll a fingerprint, platform calls EnrollBegin (0x01).
- Upon success, authenticator returns response including number of remainingSamples to capture.
- Platform then will continue prompting user to capture remaining samples and send EnrollCaptureNextSample (0x02) command to Authenticator. Authenticator subtracts 1 from remainingSamples each response, until remainingSamples is 0.



```
subCommand: 0x01
subCommandParams = {
	0x03: 10000 // arbitrary example, ms
}
ENCODED: a103192710 (TODO verify values)

message = 0101a103192710

pinUvAuthParam = 66e4e667922def036d0cc065fa6429f8d94910a7848a6853c01db49d50a4c48a
```

- Platform generates **authenticatorBioEnrollment(0x09)** calling **enrollBegin(0x01)** request
```
payload = { modality, subCommand, subCommandParams, pinUvAuthProtocol=2, pinUvAuthParam }
payload = { 
	0x01: 0x01, 
	0x02: 0x01, 
	0x03: subCommandParams, 
	0x04: 0x02, 
	0x05: pinUvAuthParam 
}

PAYLOAD:
a50101020103a103192710040205784064313665333565613535336430633933613461376361633765663338303163653162613338366533386164353537666232396236336430626565386265373963

CMD: 0x09

REQUEST: 0x09a50101020103a103192710040205784064313665333565613535336430633933613461376361633765663338303163653162613338366533386164353537666232396236336430626565386265373963
```

- On success the authenticator will return `CTAP_SUCCESS(0x00)` with a response struct containing `templateId(0x04)`, `lastEnrollSampleStatus(0x05)` and `remainingSamples(0x06)`.


e.g.,
```
response = {
	0x04    :    0x01 // arbitrary example
	0x05    :    0x00 // good fingerprint capture
	0x06    :    0x01 // arbitrary example
}
ENCODED: a3040105000601
```

**EnrollCaptureNextSample (0x02)**
- Platform sends EnrollCaptureNextSample cmd to continue enrollment, capturing remaining samples:

```
modality: 0x01
subCommand: 0x02 // enrollCaptureNextSample

subCommandParams = {
	0x01: 0x01 // from above templateId response
	0x03: 10000 // arbitrary timeout
}
ENCODED = a2010103192710

msg = 0x01 || 0x02 || subCommandParam
message         = 01 || subCommand || subCommandParams
pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)
pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)
                = 0102a2010103192710
	 
payload = { 0x01: 0x01, 0x02: 0x02, 0x03: subCommandParams, 0x04: 0x02, 0x05: pinUvAuthParam }
encoded =
```
Authenticator response:
```
response = {
	0x05: 0x00 // good fingerprint capture
	0x06: 0x00 // RemainingSamples changed from 1 -> 0
}
ENCODED: a205000600
```


## Cancel Enrollment cancelCurrentEnrollment (0x03)

Platform sends cancelCurrentEnrollment command, stopping the process of storing the fingerprint on authenticator. No PinUvAuthParam required.
REQ:
```
PAYLOAD:
{
0x01 : 0x01,
0x02: 0x03
}
CMD: 0x09
ENC = 0x09 [...]
...
```
Authenticator cancels enrollment and returns `CTAP2_OK`

## Find All Fingerprints enumerateEnrollments (0x04)

Platform sends enumerateEnrollments command to find all the fingerprints stored on device.

REQ:
```
...
```
Authenticator responds with templateInfos stored 
```
...
```

## Rename Fingerprint With Alias setFriendlyName (0x05)

Platform sends setFriendlyName command to rename a fingerprint stored on authenticator.
```
...
```
Authenticator renames fingerprint and returns `CTAP2_OK`

## Delete Fingerprint removeEnrollment (0x06)

Platform sends removeEnrollment command to authenticator to delete a fingerprint that is stored on authenticator.
```
....
```
Authenticator deletes the fingerprint and returns `CTAP2_OK`

## Get Info About Fingerprint Sensor getFingerprintSensorInfo (0x07)

Platform sends GetFingerprintSensorInfo command to get information about the sensor such as the type of fingerprint (touch or swipe), including the maximum number of good samples required for enrollment, and the size of a template alias name.

1. REQ:
```
PAYLOAD:
{
0x01 : 0x01,
0x02: 0x07
}
CMD: 0x09
ENC = 0x09 [...]
...
```

Authenticator cancels enrollment and returns `CTAP2_OK`
2. Authenticator responds with struct containing `fingerprintKind` `maxCaptureSamplesRequiredForEnroll` `maxTemplateFriendlyName`
```
RES: 
{
0x02: 1, // touch
0x03: 3, // arbitrary example
0x08: 64, // arbitrary example
}
```


