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

Upon receipt of request, authenticator will
1. If the authenticator does not support the subcommand being invoked, per subCommand's value (0x02), return
CTAP1_ERR_INVALID_PARAMETER.
2. If the authenticator is not protected by some form of user verification and alwaysUv optionId is present and true:
	i) Invoke subcommand
	ii) Return resulting status code as produced by subCommand (either _CTAP2_OK_ or _CTAP2_ERR_OPERATION_DENIED._)
	
## Example 2 - Enable Enterprise Attestation (0x01)
Platform:
```
pinUvAuthParam = authenticate(sessionPuat, merge(32x0xFF, new UInt8Array([0x0d, 0x01]) ) // note 0x02 now 0x01
authenticatorConfig: {
	'0x01': '0x01',
	'0x03': '0x02',
    	'0x04': '1879307eb5dd00ab4bac832e9174acbd2e981d04d45436811b533cc822c4a0fe'
}

ENCODED INPUT MAP:
a301643078303103643078303204784031383739333037656235646430306162346261633833326539313734616362643265393831643034643435343336383131623533336363383232633461306665

With authenticatorConfig cmd (0x0d):
0x0da301643078303103643078303204784031383739333037656235646430306162346261633833326539313734616362643265393831643034643435343336383131623533336363383232633461306665
```

Upon receipt of request, authenticator will either
a) If the enterprise attestation feature is disabled, then re-enable the enterprise attestation feature and return _CTAP2_OK_.
	->  Upon re-enabling the enterprise attestation feature, the authenticator will return an ep (enterprise) option id with
the value of true in the authenticatorGetInfo command response upon receipt of subsequent
authenticatorGetInfo commands. (todo add example)
b)  Else (implying the enterprise attestation feature is enabled) take no action and return _CTAP2_OK_.


## Example 3 - Setting a Minimum PIN Length (0x03)
subCommandParams are defined as follows (note in example 1 and 2 there were no subcommandParams). All subcommandParams are optional.
```
newMinPINLength: 0x01
minPinLengthRPIDs: 0x02
forceChangePin: 0x03
```

Platform:
```
pinUvAuthParam = authenticate(sessionPuat, merge(32x0xFF, new UInt8Array([0x0d, 0x03]) ) // note 0x02 now 0x03
authenticatorConfig: {
	'0x01': '0x03',
	'0x02' : { // note - error in doc ctrl+f 'Platform sends the following subCommandParams (0x03)'  -> 0x02
		'0x01': 16 // some random number...
	},
	'0x03': '0x02',
    	'0x04': 'f32f7a9217e19775813f1750631af806245a754781e6e322906fe735bfeaa060'
}

ENCODED INPUT MAP:
a401643078303302a1011003643078303204784066333266376139323137653139373735383133663137353036333161663830363234356137353437383165366533323239303666653733356266656161303630

With authenticatorConfig cmd (0x0d):
0x0da401643078303302a1011003643078303204784066333266376139323137653139373735383133663137353036333161663830363234356137353437383165366533323239303666653733356266656161303630
```
