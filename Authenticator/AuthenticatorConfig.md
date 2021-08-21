# AuthenticatorConfig 0x06

The FIDO2 authenticatorConfig is a command used to configure various authenticator features using its subcommands. It is new to CTAP2.1. 

- Make sure you are familiar with pin protocols. Familiarise yourself with [obtaining pinUvAuthParam](../Protocol/PinProtocol/2.md) guide first.

Do the exchange as specified in PinProtocol. We use the below value as a PinUvAuthToken test vector for authenticatorConfig in **[Example 1](#example-1---enable-enterprise-attestation-0x01)**
```
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad
```

Request keys ENUM: 

```
subCommand               : 0x01 // required
subCommandParams         : 0x02 // optional
pinUvAuthProtocol        : 0x03 // optional
pinUvAuthParam           : 0x04 // optional
```
( Usage shown in examples ctrl+f authenticatorConfig )


## SubCommands:

Sub Commands ENUM:

```
enableEnterpriseAttestation    : 0x01
toggleAlwaysUv                 : 0x02
setMinPINLength                : 0x03
```

#### Enable Enterprise Attestation (0x01):
This *enableEnterpriseAttestation* subcommand is only implemented if the enterprise attestation feature is supported. No *subcommandParams* used. **[Example 1](#example-1---enable-enterprise-attestation-0x01)**.

#### Always Require User Verification (0x02):
This *toggleAlwaysUV* subcommand is only implemented if the Always Require User Verification feature is supported. No *subcommandParams* used. _**[Example 2](#example-2---toggling-always-uv-0x02)**_.

#### Setting a Minimum PIN Length (0x03):
This *setMinPINLength* subcommand is only implemented if _setMinPINLength_ option ID is supported
This command sets the minimum PIN length in unicode code points to be enforced by the authenticator while changing/setting up a ClientPIN. **[_Example 3_](#example-3---setting-a-minimum-pin-length-0x03)**.


## pinUvAuthParam
The result of calling 
```
HMAC-SHA-256(key, message)
HMAC-SHA-256(sessionPuat, 32 x 0xff || 0x0d || subCommand || subCommandParams)
```

Where 32 x 0xff = 32 zero bytes = ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff


We merge the arrayBuffers using _0x0d_ as the authenticator cmd, **authenticatorConfig**, and _0x01_ as the subCommand, **toggleAlwaysUV** . SubcommandParams not defined for 0x01 subCommand.

```
message = 32 x 0xff || 0x0d || 0x01

pinUvAuthParam  = HMAC-SHA-256(sessionPuat, message)
		== 66e4e667922def036d0cc065fa6429f8d94910a7848a6853c01db49d50a4c48a
```


**IMPORTANT NOTE:**

For pinUvAuthParam, PinUvAuthProtocol 1 returns first 16 bytes of the HMAC output, whereas PinUvAuthProtocol 2 is returning the full 32 byte HMAC output (See #HMAC in PinProtocol 2)

**!! All examples below use exclusively PinUvAuthProtocol 2 !!**

## Example 1 - Enable Enterprise Attestation (0x01)

Using the above *pinUvAuthParam* which was generated with *0x01 subCommand*, platform will generate a request to the authenticator to toggle always UV

**Generating pinUvAuthParam**
```
pinUvAuthParam = 66e4e667922def036d0cc065fa6429f8d94910a7848a6853c01db49d50a4c48a
```

**Receiving request**
```
REQ: 0x0da401010280030204784036366534653636373932326465663033366430636330363566613634323966386439343931306137383438613638353363303164623439643530613463343861
PAYLOAD: 0da401010280030204784036366534653636373932326465663033366430636330363566613634323966386439343931306137383438613638353363303164623439643530613463343861
authenticatorConfig = {
	0x01: 0x01,
	0x02: [], // not set
	0x03: 0x02,
    	0x04: 66e4e667922def036d0cc065fa6429f8d94910a7848a6853c01db49d50a4c48a
}
```

Upon receipt of request, authenticator will either:

a) If the enterprise attestation feature is disabled, then re-enable the enterprise attestation feature and authenticator return ```0x00 (CTAP2_OK)```.
- Upon re-enabling the enterprise attestation feature, the authenticator will return an ep (enterprise) option id with
the value of true in the _authenticatorGetInfo_ command response upon receipt of subsequent
_authenticatorGetInfo_ commands.

**OR**

b)  Else (implying the enterprise attestation feature is already enabled) take no action and authenticator return ```0x00```.


## Example 2 - Toggling always UV (0x02)

Platform request:

**Generating pinUvAuthParam**
```
pinUvAuthParam = HMAC-SHA-256(sessionPuat, 32 x 0xff || 0x0d || 0x02 ) // emphasis on subcommand=0x02
	       == bd303acf547f2fd391b8f39e719c2f14bd7e71f7b1c710f862832c74b9569d15
```

**Receiving request**
```     
REQ : 0x0da4010202f4030204784062643330336163663534376632666433393162386633396537313963326631346264376537316637623163373130663836323833326337346239353639643135
PAYLOAD: 0da4010202f4030204784062643330336163663534376632666433393162386633396537313963326631346264376537316637623163373130663836323833326337346239353639643135
DECODED:
{
	0x01: 0x02,
	0x03: 0x02,
    	0x04: bd303acf547f2fd391b8f39e719c2f14bd7e71f7b1c710f862832c74b9569d15
}

```

Upon receipt of request, authenticator will either:

a) If alwaysUv feature is disabled

- If the makeCredUvNotRqd option ID is present and true, then disable the makeCredUvNotRqd feature and set the makeCredUvNotRqd option ID to false or absent.
	
- Enable the alwaysUv feature and authenticator return ```0x00```.
	
**OR**
	
b) Else, implying alwaysUv feature is enabled

- If disabling feature supported,
	
	- Set the makeCredUvNotRqd option ID to its default.
		
	- Disable alwaysUv feature and authenticator return ```0x00``` 
		
-  OTHERWISE, disabling feature not supported, 
	
	-  authenticator return ```0x27 (CTAP2_ERR_OPERATION_DENIED)```

	

## Example 3 - Setting a Minimum PIN Length (0x03)

subCommandParams are defined as follows (note in example 1 and 2 there were no subcommandParams). All subcommandParams are optional.
```
newMinPINLength: 0x01
minPinLengthRPIDs: 0x02
forceChangePin: 0x03
```

**Define subCommandParams**
```
subCommandParams = { 0x01 : 16 } 
```
We have set newMinPINLength to an arbitrary number >= 4, in this example we have picked 16
Setting newMinPINLength value < 4 will result in authenticator response error ```0x37 CTAP2_ERR_PIN_POLICY_VIOLATION```.

**Generating pinUvAuthParam**
```
pinUvAuthParam = HMAC-SHA-256(sessionPuat, 32 x 0xff || 0x0d || 0x03 || subCommandParams) // emphasis on subcommand = 0x03
	       == 5996822955053057056c7a7572f3d84d60d1bea4e4c6d512146090a343d1a264
```

**Receiving request**
```
REQ: 0x0da4010202a10110030204784035393936383232393535303533303537303536633761373537326633643834643630643162656134653463366435313231343630393061333433643161323634
PAYLOAD: 0da4010202a10110030204784035393936383232393535303533303537303536633761373537326633643834643630643162656134653463366435313231343630393061333433643161323634
DECODED:
{
	0x01 : 0x03,
	0x02 : subCommandParams,
	0x03 : 0x02,
    	0x04 : 5996822955053057056c7a7572f3d84d60d1bea4e4c6d512146090a343d1a264
}

```

Authenticator will store newMinPINLength = 16 and authenticator return ```0x00```.
