# AuthenticatorConfig

This command is used to configure various authenticator features through the use of its subcommands. It is part of the authenticator API.

You will need pinUvAuthParam. Explained at # obtaining pinUvAuthParam Here are some **test vectors**
Do the exchange as specified in PinProtocol. We will use the below value as a pinUserAuthenticationToken test vector for authenticatorConfig
```
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad (11:53_
```
We use 4 parameteres to configure the authenticator:
subCommand (0x01) **Required**
subCommandParams(0x02) Optional
pinUvAuthProtocol (0x03) Optional
pinUvAuthParam (0x04) Optional

This is the input map that will be send to the authenticator 
```
authenticatorConfig: {
	subCommand: '', // choose from [0x01, 0x02, 0x03, 0xFF]
	subCommandParams: '',
	pinUvAuthProtocol: '',
	pinUvAuthParam: '', // described below
}
```

**The subCommands:**
```
**subCommand Name** : **subCommandNumber**
enableEnterpriseAttestation: 0x01
toggleAlwaysUv: 0x02
setMinPINLength: 0x03
```

Enable Enterprise Attestation (0x01)
This 'enableEnterpriseAttestation' subcommand is only implemented if the enterprise attestation feature is supported. No subcommandParams are used.
If enterprise attestation, re-enables attestation feature and returns CTAP2_OK. Else, no action occurs and CTAP2_OK returned.

Always Require User Verification (0x02)
This 'toggleAlwaysUV' subcommand is only implemented if the Always Require User Verification feature is supported. No subcommandParams are used.

Setting a Minimum PIN Length (0x03)
This setMinPINLength subcommand is only implemented if setMinPINLength option ID is supported
This command sets the minimum PIN length in unicode code points to be enforced by the authenticator while changing/setting up a ClientPIN.


# pinUvAuthParam
The result of calling authenticate(pinUvAuthToken, 32x0xff || 0x0d || uint8(subCommand) || subCommandParams)
```
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad // key 

// authenticate returns the results of computing HMAC-SHA-256 on key and message.
pinUvAuthParam = authenticate(key, message)
pinUvAuthParam = authenticate(key, 32x0xff || 0x0d || uint8(subcommand) || subCommandParams)

message = merge(32x0xFF, new UInt8Array([0x0d, 0x02]) // 0x0d = authenticator command, authenticatorConfig. 0x02 = subcommand. SubcommandParams not defined.

pinUvAuthParam = authenticate(sessionPuat, message)

ENCODDED:
pinUvAuthParam = 7c86aa4cebcdd0577df2e279b4799daf1362a94a0c674f77bc179dc2fc534967
```

**Note**
For pinUvAuthParam, PinUvAuthProtocol 1 returns first 16 bytes of the HMAC output, whereas PinUvAuthProtocol 2 is returning the entire HMAC output (See #HMAC in PinProtocol 2)

# Example 1 - Toggling always UV (0x02)
Using the above pinUvAuthParam which was generated with 0x02 subcommand, we will generate a request to the authenticator to toggle always UV

```
pinUvAuthParam = 7c86aa4cebcdd0577df2e279b4799daf1362a94a0c674f77bc179dc2fc534967

authenticatorConfig: {
	'0x01': '0x02',
	'0x03': '0x02',
    '0x04': '7c86aa4cebcdd0577df2e279b4799daf1362a94a0c674f77bc179dc2fc534967'
}
```

# IGNORE BELOW WIP

#Example 3 - Setting a Minimum PIN Length (0x03)
This setMinPINLength subcommand is only implemented if setMinPINLength option ID is supported
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
2. Authenticator performs following operations upon receiving the request
 1. If newMinPINLength is absent, then let newMinPINLength be present with the value of current minimum PIN length
 2. If minPinLengthRPIDs is present and authenticator does not support minPinLength extension -> return CTAP2_ERR_PIN_POLICY_VIOLATION
 3. 


	
