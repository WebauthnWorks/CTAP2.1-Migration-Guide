# AuthenticatorConfig

This command is used to configure various authenticator features through the use of its subcommands. It is part of the authenticator API and is new to CTAP2.1. 

You will need pinUvAuthParam. Please make sure you are familiar with pin protocols. Read **Obtaining pinUvAuthParam** at https://github.com/WebAuthnWorks/CTAP2.1-Migration-Guide/blob/main/Protocol/PinProtocol/2.md

Here are some **test vectors**

Do the exchange as specified in PinProtocol. We will use the below value as a pinUserAuthenticationToken test vector for authenticatorConfig
```
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad
```

**We use 4 parameters to configure the authenticator:**
```
authenticatorConfig: {
	subCommand (0x01) **Required**
	subCommandParams(0x02) Optional
	pinUvAuthProtocol (0x03) Optional
	pinUvAuthParam (0x04) Optional
}
```

## The subCommands:

**subCommand Name** : **subCommand Number**
```
enableEnterpriseAttestation: 0x01
toggleAlwaysUv: 0x02
setMinPINLength: 0x03
```

#### Enable Enterprise Attestation (0x01):
This *enableEnterpriseAttestation* subcommand is only implemented if the enterprise attestation feature is supported. No *subcommandParams* are used.
If enterprise attestation, re-enables attestation feature and returns _CTAP2_OK_. Else, no action occurs and _CTAP2_OK_ returned.

#### Always Require User Verification (0x02):
This *toggleAlwaysUV* subcommand is only implemented if the Always Require User Verification feature is supported. No *subcommandParams* are used.

#### Setting a Minimum PIN Length (0x03):
This *setMinPINLength* subcommand is only implemented if _setMinPINLength_ option ID is supported
This command sets the minimum PIN length in unicode code points to be enforced by the authenticator while changing/setting up a ClientPIN.


## pinUvAuthParam
The result of calling 
```
authenticate(pinUvAuthToken, 32x0xff || 0x0d || uint8(subCommand) || subCommandParams)
```


Authenticate returns the results of computing HMAC-SHA-256 on key and message.

We merge the arrayBuffers using _0x0D_ as the authenticator cmd, **authenticatorConfig**, and _0x02_ as the subCommand, **toggleAlwaysUV** . SubcommandParams not defined for 0x02 subCommand.

```
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad // key 

pinUvAuthParam = authenticate(key, message)
pinUvAuthParam = authenticate(key, 32x0xff || 0x0d || uint8(subcommand) || subCommandParams)

message = merge(32x0xFF, new UInt8Array([0x0d, 0x02]) 

pinUvAuthParam = authenticate(sessionPuat, message)

ENCODED:
pinUvAuthParam = 7c86aa4cebcdd0577df2e279b4799daf1362a94a0c674f77bc179dc2fc534967
```

**Note**
For pinUvAuthParam, PinUvAuthProtocol 1 returns first 16 bytes of the HMAC output, whereas PinUvAuthProtocol 2 is returning the entire HMAC output (See #HMAC in PinProtocol 2)

## Example 1 - Toggling always UV (0x02)
Using the above *pinUvAuthParam* which was generated with *0x02 subCommand*, we will generate a request to the authenticator to toggle always UV

```
pinUvAuthParam = 7c86aa4cebcdd0577df2e279b4799daf1362a94a0c674f77bc179dc2fc534967

authenticatorConfig: {
	'0x01': '0x02',
	'0x03': '0x02',
    	'0x04': '7c86aa4cebcdd0577df2e279b4799daf1362a94a0c674f77bc179dc2fc534967'
}

ENCODED INPUT MAP:
a301643078303203643078303204784037633836616134636562636464303537376466326532373962343739396461663133363261393461306336373466373762633137396463326663353334393637
// temp = window.navigator.fido.fido2.cbor.JSONToCBORArrayBuffer(authenticatorConfig); hex.encode(temp)

Platform sends authenticatorConfig command + input map + options
ie. '0x0D' + 'a301643078303203643078303204784037633836616134636562636464303537376466326532373962343739396461663133363261393461306336373466373762633137396463326663353334393637'
= 0x0da301643078303203643078303204784037633836616134636562636464303537376466326532373962343739396461663133363261393461306336373466373762633137396463326663353334393637
```

# IGNORE BELOW WIP

# IGNORE:) Example 2 - Setting a Minimum PIN Length (0x03)
This *setMinPINLength* subcommand is only implemented if *setMinPINLength* option ID is supported
This command sets the minimum PIN length in unicode code points to be enforced by the authenticator while changing/setting up a ClientPIN.

1. Platform sends the following subCommandParams:
- newMinPINLength
- minPinLengthRPIDs
- forceChangePin

```
authenticatorConfig: {
	subCommand: '0x03',
	subCommandParams: {
		newMinPINLength: '4'
		minPinLengthRPIDs: ['rpID_1', 'rpID_2' ],
		forceChangePin: true
	}
```

# IGNORE:) Example 2 - Setting a Minimum PIN Length (0x03)
2. Authenticator performs following operations upon receiving the request
 1. If newMinPINLength is absent, then let newMinPINLength be present with the value of current minimum PIN length
 2. If minPinLengthRPIDs is present and authenticator does not support minPinLength extension -> return CTAP2_ERR_PIN_POLICY_VIOLATION
 3. 


	
