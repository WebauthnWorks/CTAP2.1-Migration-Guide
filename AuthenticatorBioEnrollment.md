# AuthenticatorBioEnrollment (0x09)


This command is used by the platform to provision/enumerate/delete bio enrollments in the authenticator. It is part of the authenticator API and is new to CTAP2.1. 

Authenticator must return ```bioEnroll: true``` in options object field, when platform makes authenticatorGetInfo request to authenticator. See [authenticatorGetInfo](https://github.com/WebAuthnWorks/CTAP2.1-Migration-Guide/blob/stefan-auth/AuthenticatorGetInfo.md) section.
You will need pinUvAuthParam. Please make sure you are familiar with pin protocols. Read **Obtaining pinUvAuthParam** at https://github.com/WebAuthnWorks/CTAP2.1-Migration-Guide/blob/main/Protocol/PinProtocol/2.md


- [Platform Request](#platform-request)
	- [SubCommands](#subcommands)
- [Authenticator Response](#authenticator-response)
- [Obtaining pinUvAuthParam](#obtaining-pinuvauthparam)
- [Examples](#example-1---enrolling-a-fingerprint-0x01)

Do the exchange as specified in PinProtocol. We will use the below value as a PinUvAuthToken test vector for authenticatorConfig in **[Example 1](#example-1---enrolling-a-fingerprint-0x01)**
```
puat = '0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad'
sessionPuat = puat.slice(0,32)
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa7
```
<br/><br/>
## Platform Request

AuthenticatorBioEnrollment keys ENUM: 

```
// All fields are optional
modality   		: 0x01 
subCommand  		: 0x02 
subCommandParams       	: 0x03 
pinUvAuthProtocol       : 0x04 
pinUvAuthParam 		: 0x05 
getModality      	: 0x06 
```

The type of modalities supported are as under:
```
fingerprint   		: 0x01 // optional
```


### SubCommands:

Sub Commands ENUM:

```
enrollBegin			: 0x01
enrollCaptureNextSample		: 0x02
cancelCurrentEnrollment		: 0x03
enumerateEnrollments		: 0x04
setFriendlyName			: 0x05
removeEnrollment		: 0x06
getFingerprintSensorInfo	: 0x07
```

SubCommandParams Fields:
```
templateId 		: 0x01
templateFriendlyName	: 0x02
timeoutMilliseconds 	: 0x03
```


<br/><br/>
## Authenticator Response
```
// All response fields are optional
modality 				: 0x01
fingerprintKind 			: 0x02
maxCaptureSamplesRequiredForEnroll	: 0x03
templateId				: 0x04
lastEnrollSampleStatus			: 0x05
remainingSamples			: 0x06
templateInfos				: 0x07
maxTemplateFriendlyName			: 0x08
```
**modality:**
The user verification modality.

**fingerprintKind:**
Indicates the type of fingerprint sensor. For touch type sensor, its value is 1. For swipe type sensor its value is 2.

**maxCaptureSamplesRequiredForEnroll:**
Indicates the maximum good samples required for enrollment

**templateId:**
Template Identifier. 

**lastEnrollSampleStatus:**
Last enrollment sample status. Values between 0x00 and 0x0E.
See manual for full reference.

**remainingSamples:**
Number of more sample required for enrollment to complete

**templateInfos:**
Array of templateInfos. 
templateInfo:
```
templateId		: 0x01 // required
templateFriendlyName	: 0x02 // optional
```

**maxTemplateFriendlyName:**
Indicates the maximum number of bytes the authenticator will accept as a templateFriendlyName.



<br/><br/>
## Obtaining pinUvAuthParam
The result of calling 
```
HMAC-SHA-256(key, message)
HMAC-SHA-256(pinUvAuthToken, mergeBuffers(32x0xFF || 0x0d || uint8(subCommand) || subCommandParams))
```

Where 32x0xFF = 32 zero bytes = 0000000000000000000000000000000000000000000000000000000000000000


We merge the arrayBuffers using _0x0D_ as the authenticator cmd, **authenticatorConfig**, and _0x01_ as the subCommand, **toggleAlwaysUV** . SubcommandParams not defined for 0x01 subCommand.

```
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa7 // key 

pinUvAuthParam = HMAC-SHA-256(key = sessionPuat, message = message)
pinUvAuthParam = HMAC-SHA-256(key, mergeBuffers(32x0xff || 0x0d || uint8(subcommand) || subCommandParams)) //

message = mergeBuffers(32x0xFF, new UInt8Array([0x0d, 0x01]) 


pinUvAuthParam  = HMAC-SHA-256(sessionPuat, message)
		== 66e4e667922def036d0cc065fa6429f8d94910a7848a6853c01db49d50a4c48a
```


**IMPORTANT NOTE:**

For pinUvAuthParam, PinUvAuthProtocol 1 returns first 16 bytes of the HMAC output, whereas PinUvAuthProtocol 2 is returning the full 32 byte HMAC output (See #HMAC in PinProtocol 2)

**!! All examples below use exclusively PinUvAuthProtocol 2 !!**
<br/><br/>
## Example 1 - Enrolling a fingerprint (0x01)

- Platform checks if authenticator supports BioEnrollment API,

- Platform gets pinUvAuthToken from the authenticator with the be permission.

- Platform sends authenticatorBioEnrollment command with payload to begin enrollment

- Authenticator on receiving such request performs procedures described in step 5 below

- Platform sends authenticatorBioEnrollment command with different payload to capture next sample, continuing enrollment in a loop till remaining samples to capture is zero or authenticator errors out with unrecoverable error or platform wants to cancel current enrollment:

- Authenticator on receiving such request performs following procedures.
<br/><br/>
1. 
	1.  Platform checks if authenticator supports BioEnrollment API by sending **authenticatorGetInfo (0x04)** CMD. See [authenticatorGetInfo guide]
		- Authenticator responds with cborResponseStruct containing field ```bioEnrollment: true```. 

2. Platform gets pinUvAuthToken from the authenticator with the _**be**_ permission. TODO

3. Platform generates pinUVAuthParam:
```
puat = '0125fecfd8bf3f679bd9ec221324baa7'
modality: 0x01 // fingerprint
subCommand: 0x01 // enroll begin
subCommandParams = {
	0x03: 10000 // 10000ms timeout subCommandParam_bytes = CBOR encoding of subCommandParams ??????????? 
}
ENCODED: "a103192710" 

pinUvAuthParam = generateHMACSHA256(key=puat, msg)

msg = mergeBuffers( UInt8Array[modality, subCommand], subCommandParam_bytes );	
msg = mergeBuffers( UInt8Array[0x01, 0x01], subCommandParamBytes ); // 0101a103192710

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
5. Authenticator follows process:
	1. Authenticator cancels any unfinished ongoing enrollment.
	
	2. Authenticator generates templateId for new enrollment.

	3. Authenticator sends the command to the sensor to capture the sample.

	4. Authenticator returns authenticatorBioEnrollment response with following parameters

```
templateId 		: 0x04 // template identifier of the new template being enrolled.
lastEnrollSampleStatus	: 0x05 // Status of enrollment of last sample.
remainingSamples 	: 0x06 // Number of sample remaining to complete the enrollment.
```

e.g.,
```
response = {
	0x04	:	0x01 // arbitrary value for example
	0x05	:	0x00 // CTAP2_ENROLL_FEEDBACK_FP_GOOD -- good fingerprint capture
	0x06	:	0x01 // arbitrary value for example
}
ENCODED: a3040105000601
```

6. Platform sends authenticatorBioEnrollment command with following parameters to continue enrollment in a loop till remainingSamples is zero or authenticator errors out with unrecoverable error or platform wants to cancel current enrollment:

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
		   msg = mergeBuffers( UInt8Array[0x01, 0x02], subCommandParamBytes );
	pinUvAuthParam (encoded) = 0102a2010103192710 
	
	payload = { 0x01: 0x01, 0x02: 0x02, 0x03: subCommandParams, 0x04: 0x02, 0x05: pinUvAuthParam }
	encoded =
	```
	2. Authenticator on receiving such request performs following procedures.
		- Validates parameters and pinUvAuthParam (see specs for full reference) -- if parameter is is invalid -- return error status code
		- If fingerprint is already present on the sensor, authenticator waits for user to lift finger from the sensor.
		- Authenticator sends the command to the sensor to capture the sample.
		- Authenticator returns authenticatorBioEnrollment response with following parameters:
	```
	response = {
		0x05: 0x00 // good fingerprint capture
		0x06: 0x00 // in step 5. the response contained 0x05: 0x01. As authenticator captured the next sample, remaining samples has gone from 0 -> 1
	}
	ENCODED: a205000600
	```

