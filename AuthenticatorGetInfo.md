# authenticatorGetInfo (0x04)

TODO : ADD examples

Using this method, platforms can request that the authenticator report a list of its supported protocol versions and extensions, its AAGUID, and other aspects of its overall capabilities. Platforms should use this information to tailor their command parameters choices.


## What's stayed the same between CTAP2.0 and CTAP2.1?
The response for the following fields in authenticatorGetInfo are the same:



```
Member Name		Data type		Required?
===========================================================

versions (0x01)		Array of strings	Required
extensions (0x02)	Array of strings	Optional
aaguid (0x03)		Byte String		Required
options (0x04)		Map			Optional
maxMsgSize (0x05)	Unsigned Integer	Optional
```

## What's changed between CTAP2.0 and CTAP2.1?

```pinProtocols (0x06)``` -> ```pinUvAuthProtocols (0x06)```.

Field pinUvAuthProtocols is an Array of Unsigned Integers of supported PIN/UV auth protocols in order of decreasing authenticator preference. 
MUST NOT contain duplicate values nor be empty if present.

## What's new in CTAP2.1?
All the below new response fields are optional.

```
Member Name		                  Data type		
=========================================================

maxCredentialCountInList (0x07)		  Unsigned Integer
maxCredentialIdLength (0x08)		  Unsigned Integer
transports (0x09)			  Array of strings
algorithms (0x0A)			  Array of PublicKeyCredentialParameters
maxSerializedLargeBlobArray (0x0B)	  Unsigned Integer
forcePINChange (0x0C)			  Boolean
minPINLength (0x0D)			  Unsigned Integer
firmwareVersion (0x0E)			  Unsigned Integer
maxCredBlobLength (0x0F)		  Unsigned Integer
maxRPIDsForSetMinPINLength (0x10)	  Unsigned Integer
preferredPlatformUvAttempts (0x11)	  Unsigned Integer. (CBOR major type 0)
uvModality (0x12)			  Unsigned Integer. (CBOR major type 0)
certifications (0x13)			  Map
remainingDiscoverableCredentials (0x14)	  Unsigned Integer
vendorPrototypeConfigCommands (0x15)	  Array of Unsigned Integers
```

## Options

All options are in the form key-value pairs with string IDs and boolean values. When an option is not present, the default is applied per table below. The following is a list of supported options:

## What's the same in options?
```
plat			false
rk			false
clientPin		Not supported
up			true
uv			Not supported
```

## What's new in options?
```
pinUvAuthToken			false
noMcGaPermissionsWithClientPin	false
largeBlobs			Not supported
ep				true
bioEnroll			Not Supported
userVerificationMgmtPreview	Not Supported
uvBioEnroll			false
authnrCfg			Not supported
uvAcfgNot			supported
credMgmtNot			supported
credentialMgmtPreview		Not supported
setMinPINLengthNot 		supported
makeCredUvNotRqdNot 		supported
alwaysUvNot 			supported
```

