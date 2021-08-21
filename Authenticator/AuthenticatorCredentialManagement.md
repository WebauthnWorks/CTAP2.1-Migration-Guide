# AuthenticatorCredentialManagement (0x0A)

This command is used by the platform to manage discoverable credentials on the authenticator. It is part of the authenticator API and is **new** to CTAP2.1. 

Based on exchange as specified in PinProtocol, we use the below value as a PinUvAuthToken test vector for authenticatorCredentialManagement.
```
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad
```

See:

- [Example 1 - Getting Credentials Metadata](#example-1---getting-credentials-metadata)
- [Example 2 - Enumerating RPs](#example-2---enumerating-rps)
- [Example 3 - Enumerating Credentials for an RP](#example-3---enumerating-credentials-for-an-rp)
- [Example 4 - DeleteCredential](#example-4---deletecredential)

It takes the following input parameters:

```
+--------------------------+------------------+------------------------------------------------------------------+
|      Parameter name      |    Data type     |                            Definition                            |
+--------------------------+------------------+------------------------------------------------------------------+
| subCommand (0x01)        | Unsigned Integer | subCommand currently being requested                             |
| subCommandParams (0x02)  | CBOR Map         | Map of subCommands parameters.                                   |
| pinUvAuthProtocol (0x03) | Unsigned Integer | PIN/UV protocol version chosen by the platform.                  |
| pinUvAuthParam (0x04)    | Byte String      | First 16 bytes of HMAC-SHA-256 of contents using pinUvAuthToken. |
+--------------------------+------------------+------------------------------------------------------------------+
```

The subcommands for credential management is 
```
getCredsMetadata                         : 0x01
enumerateRPsBegin                        : 0x02
enumerateRPsGetNextRP                    : 0x03
enumerateCredentialsBegin                : 0x04
enumerateCredentialsGetNextCredential    : 0x05
deleteCredential                         : 0x06
updateUserInformation                    : 0x07
```

subCommandParam Fields:

```
+---------------------+-------------------------------+-----------------------+
|     Field name      |           Data type           |      Definition       |
+---------------------+-------------------------------+-----------------------+
| rpIDHash (0x01)     | Byte String                   | RP ID SHA-256 hash    |
| credentialID (0x02) | PublicKeyCredentialDescriptor | Credential Identifier |
| user (0x03)         | PublicKeyCredentialUserEntity | User Entity           |
+---------------------+-------------------------------+-----------------------+
```

On success, authenticator returns the following structure in its response:
```

+-----------------------------------------------------+-------------------------------+----------------------------------------------------------------------------------------------------------+
|                   Parameter name                    |           Data type           |                                                Definition                                                |
+-----------------------------------------------------+-------------------------------+----------------------------------------------------------------------------------------------------------+
| existingResidentCredentialsCount (0x01)             | Unsigned Integer              | Number of existing discoverable credentials present on the authenticator.                                |
| maxPossibleRemainingResidentCredentialsCount (0x02) | Unsigned Integer              | Number of maximum possible remaining discoverable credentials which can be created on the authenticator. |
| rp (0x03)                                           | PublicKeyCredentialRpEntity   | RP Information                                                                                           |
| rpIDHash (0x04)                                     | Byte String                   | RP ID SHA-256 hash                                                                                       |
| totalRPs (0x05)                                     | Unsigned Integer              | total number of RPs present on the authenticator                                                         |
| user (0x06)                                         | PublicKeyCredentialUserEntity | User Information                                                                                         |
| credentialID (0x07)                                 | PublicKeyCredentialDescriptor | PublicKeyCredentialDescriptor                                                                            |
| publicKey (0x08)                                    | COSE_Key                      | Public key of the credential.                                                                            |
| totalCredentials (0x09)                             | Unsigned Integer              | Total number of credentials present on the authenticator for the RP in question                          |
| credProtect (0x0A)                                  | Unsigned Integer              | Credential protection policy.                                                                            |
| largeBlobKey (0x0B)                                 | Byte string                   | Large blob encryption key.                                                                               |
+-----------------------------------------------------+-------------------------------+----------------------------------------------------------------------------------------------------------+

```

**!! All examples below use exclusively PinUvAuthProtocol 2 !!**

## Example 1 - Getting Credentials Metadata

1. Platform gets PinUvAuthToken from the authenticator with the cm, [Credential Management Permission](https://github.com/WebAuthnWorks/CTAP2.1-Migration-Guide/blob/main/Protocol/PinProtocol/2.md#get-pinuvauthtoken-with-permissions).
 
2. Platform sends authenticatorCredentialManagement command with following parameters:
```
message = subCommands = { 0x01: 0x01 }

pinUvAuthParam = generateHMACSHA256(key=sessionPuat, message)
               == 185690876fbc2637e93a4971089308e3f6381a36fcf74eb3860b13ace1a7a2c1
               
REQ = {
  0x01: 0x01, // getCredsMetadata 
  0x03: 2, //  pinUvAuthProtocol - 2
  0x04: 185690876fbc2637e93a4971089308e3f6381a36fcf74eb3860b13ace1a7a2c1
}

CMD: 0x0A
REQ: 0x0aa30101030204784031383536393038373666626332363337653933613439373130383933303865336636333831613336666366373465623338363062313361636531613761326331
```

3. Authenticator on receiving such request performs following procedures.
  - Validates parameters and pinUvAuthParam (see specs for full reference) -- if parameter is is invalid -- return [error status code](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#error-responses)
  - Authenticator returns authenticatorCredentialManagement response with following parameters:
```
RESP = {
  0x01: 8, // 8 is an arbitrary value for example saying that 8 discoverable credentials exist on the authenticator.
  0x02: 56 // 56 is an arbitrary value for example. Maximum number of possible remaining discoverable credentials that can be created on the authenticator. 
}

ENC: a20108021838
```
 
## Example 2 - Enumerating RPs


## Example 3 - Enumerating Credentials for an RP


## Example 4 - DeleteCredential

1. Platform gets pinUvAuthToken from the authenticator with the cm permission.
2. Platform sends authenticatorCredentialManagement command with following parameters:
```
subCommand: 0x06
subCommandParams: credentialId = {
 type : 'public-key'
 id: : 73a8ed4bebf8a76095e4d6855acf6cb8d702d15a9ebe25641ae2d097b4314f3700603030 // arbitrary example of credential ID 
}

pinUvAuthParam = generateHMACSHA256(key=puat, msg)
               = generateHMACSHA256(puat, mergeBuffers(puat || 0x06 || subCommandParams )
               = 7ff1ef5ae1dc084206401d009f3563f88568f71bd1e101263dc2101bf195a085
               
REQ = {
  0x01: 0x06, // getCredsMetadata 
  0x02: subCommandParams,
  0x03: 2, //  pinUvAuthProtocol - 2
  0x04: 7ff1ef5ae1dc084206401d009f3563f88568f71bd1e101263dc2101bf195a085
}

ENC: a4010602a102a2626964784837336138656434626562663861373630393565346436383535616366366362386437303264313561396562653235363431616532643039376234333134663337303036303330333064747970656a7075626c69632d6b6579030204784037666631656635616531646330383432303634303164303039663335363366383835363866373162643165313031323633646332313031626631393561303835

CMD: 0x0A
REQ: 0x0aa4010602a102a2626964784837336138656434626562663861373630393565346436383535616366366362386437303264313561396562653235363431616532643039376234333134663337303036303330333064747970656a7075626c69632d6b6579030204784037666631656635616531646330383432303634303164303039663335363366383835363866373162643165313031323633646332313031626631393561303835
```

3. Authenticator on receiving such request performs following procedures.
  - Validates parameters and pinUvAuthParam (see specs for full reference) -- if parameter is is invalid -- return [error status code](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#error-responses)
  - Authenticator deletes the credential and returns **0x00 CTAP2_OK**.
