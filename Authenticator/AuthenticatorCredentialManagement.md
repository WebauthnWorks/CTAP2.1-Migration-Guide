# AuthenticatorCredentialManagement (0x0A)

This command is used by the platform to manage discoverable credentials on the authenticator. It is part of the authenticator API and is **new** to CTAP2.1. 

Do the exchange as specified in PinProtocol. We will use the below value as a PinUvAuthToken test vector for authenticatorCredentialManagement.
```
puat = '0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad'
sessionPuat = puat[0:32]
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa7
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
getCredsMetadata                       : 0x01
enumerateRPsBegin                      : 0x02
enumerateRPsGetNextRP                  : 0x03
enumerateCredentialsBegin              : 0x04
enumerateCredentialsGetNextCredential  : 0x05
deleteCredential                       : 0x06
updateUserInformation                  : 0x07
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

1. Platform gets PinUvAuthToken from the authenticator with the cm (Credential Management) permission
 
2. Platform sends authenticatorCredentialManagement command with following parameters:
```
pinUvAuthParam = generateHMACSHA256(key=puat, msg)
               = generateHMACSHA256(puat, subCommands={0x01: 0x01} )
               = 185690876fbc2637e93a4971089308e3f6381a36fcf74eb3860b13ace1a7a2c1
               
REQ = {
  0x01: 0x01, // getCredsMetadata 
  0x03: 2,
  0x04: 185690876fbc2637e93a4971089308e3f6381a36fcf74eb3860b13ace1a7a2c1
}

CMD: 0x0A
REQ: 0x0aa30101030204784031383536393038373666626332363337653933613439373130383933303865336636333831613336666366373465623338363062313361636531613761326331
```

3. Authenticator on receiving such request performs following procedures.
  - Validates parameters and pinUvAuthParam (see specs for full reference) -- if parameter is is invalid -- return error status code
  - Authenticator returns authenticatorCredentialManagement response with following parameters:
```
RESP = {
  0x01: 8, // 8 is an abritrary value for example saying that 8 discoverable credentials exist on the authenticator.
  0x02: 56 // 56 is an abritrary value for example. Maximum number of possible remaining discoverable credentials that can be created on the authenticator. 
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
 type = 'public-key'
 id = credentialId
 transports = ["usb"]
};
pinUvAuthParam = generateHMACSHA256(key=puat, msg)
               = generateHMACSHA256(puat, mergeBuffers(puat || 0x06 || subCommandParams )
               = 
               
REQ = {
  0x01: 0x06, // getCredsMetadata 
  0x02: subCommandParams,
  0x03: 0x02
  0x04: 
}

CMD: 0x0A
REQ: 
```

