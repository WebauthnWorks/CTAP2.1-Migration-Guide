
# AuthenticatorBioEnrollment (0x09)



The authenticatorBioEnrollment commands provides the platform the ability to create, delete and list bio enrollments in the authenticator. 

Using this API, the platform is able to:

- Get info about fingerprint sensor
- Create/enroll a fingerprint
- Cancel fingerprint enrollments before they finish
- Delete a fingerprint
- Find all fingerprints the have been added
- Give the fingerprint an alias

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
templateId 		        : 0x01
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
```
puat = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad
```

The result of calling 
```
message         = subCommand || subCommandParams
pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)

HMAC-SHA-256(pinUvAuthToken, 32x0xFF || 0x0d || subCommand || subCommandParams)


Where 32 x 0xff = 32 zero bytes = ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff


We merge the arrayBuffers using _0x0D_ as the authenticator cmd, **authenticatorBioEnrollment**, and _0x01_ as the subCommand, **toggleAlwaysUV** . SubcommandParams not defined for 0x01 subCommand.

sessionPuat = 0125fecfd8bf3f679bd9ec221324baa7 // key 

pinUvAuthParam = HMAC-SHA-256(key = sessionPuat, message = message)
pinUvAuthParam = HMAC-SHA-256(key, 32 x 0xff || 0x0D || subcommand || subCommandParams)

message = 32 x 0xff || 0x0D || 0x01


pinUvAuthParam  = HMAC-SHA-256(sessionPuat, message)
		== 66e4e667922def036d0cc065fa6429f8d94910a7848a6853c01db49d50a4c48a
```


**IMPORTANT NOTE:**

For pinUvAuthParam, PinUvAuthProtocol 1 returns first 16 bytes of the HMAC output, whereas PinUvAuthProtocol 2 is returning the full 32 byte HMAC output (See #HMAC in PinProtocol 2)

**!! All examples below use exclusively PinUvAuthProtocol 2 !!**
<br/><br/>
## Enrolling a fingerprint - EnrollBegin (0x01)	EnrollCaptureNextSample (0x02)

- To enroll a fingerprint, platform calls EnrollBegin (0x01). Authenticator will flash, signalling user to press their fingerprint. Upon success, authenticator returns response including number of remainingSamples to capture. Platform then will continue prompting user to capture remaining samples. Authenticator subtracts 1 from remiaingSamples each response, until remainingSamples is 0.

> Note: This is a stateful command. During enrollBegin(0x03) and EnrollCaptureNextSample(0x03) authenticator must keep a track of which RP is following next by using some sort of state(in active transaction) counter to keep a track of enumerated RPs


```
puat = '0125fecfd8bf3f679bd9ec221324baa7'
modality: 0x01 // fingerprint
subCommand: 0x01 // enroll begin
subCommandParams = {
	0x03: 10000 // 10000ms timeout subCommandParam_bytes = CBOR encoding of subCommandParams ??????????? 
}
ENCODED: "a103192710" 

pinUvAuthParam = generateHMACSHA256(key=puat, msg)

msg = modality || subCommand || subCommandParam_bytes;	
msg = 0x01 || 0x01 || subCommandParamBytes ); // 0101a103192710

pinUvAuthParam = 'd16e35ea553d0c93a4a7cac7ef3801ce1ba386e38ad557fb29b63d0bee8be79c'
```

4. Platform generates **authenticatorBioEnrollment(0x09)** calling **enrollBegin(0x01)** request
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

5. On success the authenticator will return `CTAP_SUCCESS(0x00)` with a response struct containing `templateId(0x04)`, `lastEnrollSampleStatus(0x05)` and `remainingSamples(0x06)`.


e.g.,
```
response = {
	0x04    :    0x01 // arbitrary value for example
	0x05    :    0x00 // CTAP2_ENROLL_FEEDBACK_FP_GOOD -- good fingerprint capture
	0x06    :    0x01 // arbitrary value for example
}
ENCODED: a3040105000601
```

6. Platform sends authenticatorBioEnrollment cmd to continue enrollment, capturing remaining samples:

	1. Platform sends authenticatorBioEnrollment command with following parameters
	```
	modality: 0x01 // fingerprint
	subCommand: 0x02 // subCommand is now enrollCaptureNextSample
	subCommandParams = {
		0x01: 0x01 // taken from templatedId in response
		0x03: 10000 // 10000ms timeout subCommandParam_bytes = CBOR encoding of subCommandParams ??????????? 
	}
	subCommandParamBytes (encoded) = a2010103192710
	
	pinUvAuthParam = generateHMACSHA256(key=puat, msg) // puat=0125fecfd8bf3f679bd9ec221324baa7
		   msg = 0x01 || 0x02 || subCommandParamBytes
	pinUvAuthParam (encoded) = 0102a2010103192710 
	
	payload = { 0x01: 0x01, 0x02: 0x02, 0x03: subCommandParams, 0x04: 0x02, 0x05: pinUvAuthParam }
	encoded =
	```
	2. Authenticator response
	```
	response = {
		0x05: 0x00 // good fingerprint capture
		0x06: 0x00 // in step 5. the response contained 0x05: 0x01. As authenticator captured the next sample, remaining samples has gone from 0 -> 1
	}
	ENCODED: a205000600
	```


## Cancel Enrollment cancelCurrentEnrollment (0x03)


## Find All Fingerprints enumerateEnrollments (0x04)


## Rename Fingerprint With Alias setFriendlyName (0x05)


## Delete Fingerprint removeEnrollment (0x06)


## Get Info About Fingerprint Sensor getFingerprintSensorInfo (0x07)

GetFingerprintSensorInfo provides a mechanism to get information about the sensor such as the type of fingerprint (touch or swipe), the maximum number of good samples required for enrollment, and the size of a template alias name.

1. Platform REQ
```

```
2. Authenticator responds with struct containing `fingerprintKind` `maxCaptureSamplesRequiredForEnroll` `maxTemplateFriendlyName`
```
