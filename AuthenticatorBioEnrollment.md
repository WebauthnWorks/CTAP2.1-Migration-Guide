# AuthenticatorBioEnrollment (0x09)


  This command is used by the platform to provision/enumerate/delete bio enrollments in the authenticator. It is part of the authenticator API and is new to CTAP2.1. 
- Authenticator MUST? return true in getInfo.option.bioEnroll when platform makes authenticatorGetInfo request to authenticator (fact check this)
You will need pinUvAuthParam. Please make sure you are familiar with pin protocols. Read **Obtaining pinUvAuthParam** at https://github.com/WebAuthnWorks/CTAP2.1-Migration-Guide/blob/main/Protocol/PinProtocol/2.md

Do the exchange as specified in PinProtocol. We use the below value as a PinUvAuthToken test vector for authenticatorConfig in **[Example 1](example-1)**
```
puat = '0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad'
sessionPuat = puat.slice(0,32)
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa7
```

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

```
templateId		: 0x01 // required
templateFriendlyName	: 0x02 // optional
```

end

#### Enable Enterprise Attestation (0x01):
This 

#### Always Require User Verification (0x02):
This _**Example 2**_.

#### Setting a Minimum PIN Length (0x03):
This  **_Example 3_**.


## pinUvAuthParam
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

## Example 1 - Enrolling a fingerprint (0x01)

- Platform checks if authenticator supports BioEnrollment API,
- Platform gets pinUvAuthToken from the authenticator with the be permission.
- Platform sends authenticatorBioEnrollment command with payload to begin enrollment
- Authenticator on receiving such request performs procedures described in (4)
- Platform sends authenticatorBioEnrollment command with different payload to capture next sample, continuing enrollment in a loop till remaining samples to capture is zero or authenticator errors out with unrecoverable error or platform wants to cancel current enrollment:
- Authenticator on receiving such request performs following procedures.

Platform checks if authenticator supports BioEnrollment API, then sends authenticatorBioEnrollment(0x09) with enrollBegin(0x01) with supported modality, and check that response contains:
            (a) templateId(0x04) - byte string, at least one byte long
            (b) remainingSamples(0x06) - number, and above zero
            (c) lastEnrollSampleStatus(0x05) - number, a valid BE status code

1. Platform sends authenticatorGetInfo (0x04) CMD to authenticator 
2. Authenticator responds cborResponseStruct containing field (improve? remove?)
```
0x04: {
... // present options
'bioEnrollment': true
}
```

3. Platform checks that bioEnrollment is true
4. Platform generates pinUVAuthParam:
```
puat = '0125fecfd8bf3f679bd9ec221324baa7'
modality: 0x01 // fingerprint
subCommand: 0x01 // enroll begin
subCommandParams = {
	0x03: 10000 // subCommandParam_bytes = CBOR encoding of subCommandParams ??????????? 
}
ENCODED: "a103192710" 

pinUvAuthParam = generateHMACSHA256(key=puat, msg)

msg = mergeBuffers( UInt8Array[modality, subCommand], subCommandParam_bytes );	
msg = mergeBuffers( UInt8Array[0x01, 0x01], subCommandParamBytes ); // 0101a103192710

pinUvAuthParam = 'd16e35ea553d0c93a4a7cac7ef3801ce1ba386e38ad557fb29b63d0bee8be79c'
```

5. Platform generates request
```
payload = { modality, subCommand, subCommandParams, pinUvAuthProtocol=2, pinUvAuthParam }
payload = { 0x01: 0x01, 0x02: 0x01, 0x03: subCommandParams, 0x04: 0x02, 0x05: pinUvAuthParam }

PAYLOAD:
a50101020103a103192710040205784064313665333565613535336430633933613461376361633765663338303163653162613338366533386164353537666232396236336430626565386265373963

CMD: 0x09

REQUEST: 0x09a50101020103a103192710040205784064313665333565613535336430633933613461376361633765663338303163653162613338366533386164353537666232396236336430626565386265373963
```


