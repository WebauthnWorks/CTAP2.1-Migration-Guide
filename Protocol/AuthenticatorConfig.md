# AuthenticatorConfig 0x0D

This command is used to configure various authenticator features using its subcommands.

The supported configuration options as now are:

- EnableEnterpriseAttestation(0x01) - enables Enterprise Attestation
- ToggleAlwaysUv(0x02) - "Always UV" is a features that forces authenticator to fail any authenticated request unless explicit user verification, via pin or biometrics is done.
- SetMinPINLength(0x03) - sets minimum pin length for the authenticator

This command requires support of [PinUvAuthProtocol 2](../PinUvAuthnProtocol2.md).

The platform must obtain PinUvAuthToken with `acfg(0x20)` permission flag.

The specified PinUvAuthToken will be used in all future examples:

```
SessionPUAT = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad
```

## Common

Request keys ENUM: 

```
subCommand        : 0x01
subCommandParams  : 0x02
pinUvAuthProtocol : 0x03
pinUvAuthParam    : 0x04
```

Sub Commands ENUM:

```
enableEnterpriseAttestation : 0x01
toggleAlwaysUv              : 0x02
setMinPINLength             : 0x03
```

Response only contains status code.


## Computing PinUvAuthParam

Authenticator config specifies the following structure to compute PinUvAuthParam:

```
message         = 0xff{32} || 0x0d || subCommand || subCommandParams

pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)
```

Where 0xff{32} is 32 byte set 0xff:

`ffffffffffffffffffffffffffffffff`

Example PinUvAuthParam for ToggleAlwaysUv(0x02)

```
pinUvAuthParam  = HMAC-SHA-256(key = puat, message = 0xff{32} || 0x0d || 0x02)
                => 40d0d64f5030fa46d8e27c1bb358d5eb7b0da88fd4955b83ed19335bb35d886c
```

## Enable Enterprise Attestation (0x01)

To enable enterprise attestation platform sends AuthenticatorConfig with EnableEnterpriseAttestation(0x01) sub command:

```
PinUvAuthParam = HMAC-SHA-256(key = puat, message = 0xff{32} || 0x0d || 0x01)
               => 4e724bc8af8776d0e92a3961c28bf9980b7b3962c610ba7abf07c74a0972b074
```

**Generating request**
```
REQ: 0da3010103020458204e724bc8af8776d0e92a3961c28bf9980b7b3962c610ba7abf07c74a0972b074

CMD: 0x0d
PAYLOAD: a3010103020458204e724bc8af8776d0e92a3961c28bf9980b7b3962c610ba7abf07c74a0972b074

DECODED:
{
    1: 1, // SubCommand - EnableEnterpriseAttestation(0x01)
    3: 2, // PinUvAuthProtocol 2
    4: 4e724bc8af8776d0e92a3961c28bf9980b7b3962c610ba7abf07c74a0972b074 // PinUvAuthParam
}
```

On success the authenticator will return `CTAP_SUCCESS(0x00)`

To check if enterprise attestation was activated, you can check authenticator GetInfo options flag `ep`

## Toggling always UV (0x02)

To enable alwaysUv, platform sends AuthenticatorConfig with ToggleAlwaysUv(0x02) sub command:

```
pinUvAuthParam  = HMAC-SHA-256(key = puat, message = 0xff{32} || 0x0d || 0x02)
                => 40d0d64f5030fa46d8e27c1bb358d5eb7b0da88fd4955b83ed19335bb35d886c
```

**Generating request**
```
REQ: 0da30101030204582040d0d64f5030fa46d8e27c1bb358d5eb7b0da88fd4955b83ed19335bb35d886c

CMD: 0x0d
PAYLOAD: a30101030204582040d0d64f5030fa46d8e27c1bb358d5eb7b0da88fd4955b83ed19335bb35d886c

DECODED:
{
    1: 2, // SubCommand - ToggleAlwaysUv(0x02)
    3: 2, // PinUvAuthProtocol 2  
    4: 40d0d64f5030fa46d8e27c1bb358d5eb7b0da88fd4955b83ed19335bb35d886c // PinUvAuthParam
}
```

On success the authenticator will return `CTAP_SUCCESS(0x00)`

To check if alwaysUv is enabled, you can check authenticator GetInfo options flag `alwaysUv`
    

## Minimum PIN Length (0x03)

SetMinPINLength(0x03) is a command that is use to setting minimum pin length for the future change of the pin.

To check if this subcommand is supported, see GetInfo options `setMinPINLength` option is present.

There are following available sub command options

```
newMinPINLength  : 0x01
minPinLengthRPIDs: 0x02
forceChangePin   : 0x03
```

- `newMinPINLength` is used to specify the new pin length requirements that will be enforced next time user will be changing password.
- `minPinLengthRPIDs` is used together with `MinPinLength` extension. The `MinPinLength` extension is used to allow some relying parties to obtain information regarding the current minimum pin length. Authenticator config `minPinLengthRPIDs` parameter specifies which relying parties are allowed to obtain such information.
- `forceChangePin` forces user to change their pin code next transaction and returns `CTAP2_ERR_PIN_NOT_SET(0x35)`.

These sub command options may be combined together or used separately. All sub command options are optional.


**Generating PinUvAuthParam**

To generate PinUvAuthParam, platform first must generate sub command CBOR map, and encode it buffer. Then following previously defined structure generate message buffer and HMAC it with the corresponding PUAT

```
SubCommandParams = {
    // New pin length is 6
    1: 6,
    
    // These two RPIDs will have access to the MinPinLength extension
    2: ["example.com", "enterprise.com"],

    // Will force user to change pin next transaction
    3: true
}

SubCommandBytes = a3010602826b6578616d706c652e636f6d6e656e74657270726973652e636f6d03f5

Message = 0xff{32} || 0x0d || 0x03 || a3010602826b6578616d706c652e636f6d6e656e74657270726973652e636f6d03f5

PinUvAuthParam = HMAC-SHA-256(key = puat, message = Message)
               => 450acdb5fe618880d1774295f2723cb5d0ecc0ddbd7e35a4b651b501fd1f8f65
```

**Generating request**
```
REQ: 0da4010102a3010602826b6578616d706c652e636f6d6e656e74657270726973652e636f6d03f50302045820450acdb5fe618880d1774295f2723cb5d0ecc0ddbd7e35a4b651b501fd1f8f65

CMD: 0x0d
PAYLOAD: a4010102a3010602826b6578616d706c652e636f6d6e656e74657270726973652e636f6d03f50302045820450acdb5fe618880d1774295f2723cb5d0ecc0ddbd7e35a4b651b501fd1f8f65

DECODED:
{
    1 : 3, // 
    2 : {
        1: 6,
        2: ["example.com", "enterprise.com"],
        3: true
    },
    3 : 2,
    4 : 450acdb5fe618880d1774295f2723cb5d0ecc0ddbd7e35a4b651b501fd1f8f65
}
```

On success the authenticator shall return `CTAP_SUCCESS(0x00)`
