# AuthenticatorCredentialManagement (0x0A)

This command is used by the platform to manage discoverable credentials on the authenticator. It is part of the authenticator API and is **new** to CTAP2.1. 

Read through the pin token exchange specified in PinProtocol. We will use the below value as a PinUvAuthToken test vector for authenticatorCredentialManagement.
```
puat = '0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad'
sessionPuat = puat[0:32]
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa7
```

See:

- [Example 1 - Getting Credentials Metadata]
- [Example 2 - Enumerating RPs]
- [Example 3 - Enumerating Credentials for an RP]
- [Example 4 - DeleteCredential]
- 

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
getCredsMetadata				          		: 0x01
enumerateRPsBegin				          		: 0x02
enumerateRPsGetNextRP					        : 0x03
enumerateCredentialsBegin				      : 0x04
enumerateCredentialsGetNextCredential	: 0x05
deleteCredential					          	: 0x06
updateUserInformation				        	: 0x07
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


