# authenticatorGetInfo (0x04)


Using this method, platforms can request that the authenticator report a list of its supported protocol versions and extensions, its AAGUID, and other aspects of its overall capabilities. Platforms should use this information to tailor their command parameters choices.


## What's stayed the same between CTAP2.0 and CTAP2.1?
The response for the following fields in authenticatorGetInfo are the same:



```
+-------------------------+------------------+-----------+
|        Member Name      |    Data type     | Required? |
+-------------------------+------------------+-----------+
| versions         0x01   | Array of strings | Required  |
| extensions       0x02   | Array of strings | Optional  |
| aaguid           0x03   | Byte String      | Required  |
| options          0x04   | Map              | Optional  |
| maxMsgSize       0x05   | Unsigned Integer | Optional  |
+-------------------------+------------------+-----------+
```

## What's changed between CTAP2.0 and CTAP2.1?

```pinProtocols (0x06)``` -> ```pinUvAuthProtocols (0x06)```.

Field pinUvAuthProtocols is an Array of Unsigned Integers of supported PIN/UV auth protocols in order of decreasing authenticator preference. 
MUST NOT contain duplicate values nor be empty if present.

## What's new in CTAP2.1?
All the below new response fields are optional.

```
+-----------------------------------------+------------------------------------------+
|               Member Name               |                Required?                 |
+-----------------------------------------+------------------------------------------+
| maxCredentialCountInList (0x07)         |   Unsigned Integer                       |
| maxCredentialIdLength (0x08)            |   Unsigned Integer                       |
| transports (0x09)                       |   Array of strings                       |
| algorithms (0x0A)                       |   Array of PublicKeyCredentialParameters |
| maxSerializedLargeBlobArray (0x0B)      |   Unsigned Integer                       |
| forcePINChange (0x0C)                   |   Boolean                                |
| minPINLength (0x0D)                     |   Unsigned Integer                       |
| firmwareVersion (0x0E)                  |   Unsigned Integer                       |
| maxCredBlobLength (0x0F)                |   Unsigned Integer                       |
| maxRPIDsForSetMinPINLength (0x10)       |   Unsigned Integer                       |
| preferredPlatformUvAttempts (0x11)      |   Unsigned Integer (CBOR major type 0)   |
| uvModality (0x12)                       |   Unsigned Integer (CBOR major type 0)   |
| certifications (0x13)                   |   Map                                    |
| remainingDiscoverableCredentials (0x14) |   Unsigned Integer                       |
| vendorPrototypeConfigCommands (0x15)    |   Array of Unsigned Integers             |
+-----------------------------------------+------------------------------------------+
```

## Options

All options are in the form key-value pairs with string IDs and boolean values. When an option is not present, the default is applied per table below. The following is a list of supported options:

## What's the same in options?
```
+-----------+---------------+
| Option ID |    Default    |
+-----------+---------------+
| plat      | false         |
| rk        | false         |
| clientPin | Not supported |
| up        | true          |
| uv        | Not supported |
+-----------+---------------+
```

## What's new in options?
```
+--------------------------------+---------------+
|           Option ID            |    Default    |
+--------------------------------+---------------+
| pinUvAuthToken                 | false         |
| noMcGaPermissionsWithClientPin | false         |
| largeBlobs                     | Not supported |
| ep                             | true          |
| bioEnroll                      | Not Supported |
| userVerificationMgmtPreview    | Not Supported |
| uvBioEnroll                    | false         |
| authnrCfg                      | Not supported |
| uvAcfgNot                      | supported     |
| credMgmtNot                    | supported     |
| credentialMgmtPreview          | Not supported |
| setMinPINLengthNot             | supported     |
| makeCredUvNotRqdNot            | supported     |
| alwaysUvNot                    | supported     |
+--------------------------------+---------------+
```

## Example of an authenticatorGetInfo response

```
{
  "legalHeader": "https://fidoalliance.org/metadata/metadata-statement-legal-header/",
  "aaguid": "a3f7a9e8-fe77-11eb-9a03-0242ac130003",
  "description": "YubiKey Series 5 with NFC",
  "authenticatorVersion": 50200,
  "protocolFamily": "fido2",
  "schema": 3,
  "upv": [
    {
      "major": 1,
      "minor": 0
    }
  ],
  "authenticationAlgorithms": [
    "secp256r1_ecdsa_sha256_raw",
    "ed25519_eddsa_sha512_raw"
  ],
  "publicKeyAlgAndEncodings": [
    "cose"
  ],
  "attestationTypes": [
    "basic_full"
  ],
  "userVerificationDetails": [
    [
      {
        "userVerificationMethod": "presence_internal"
      },
      {
        "userVerificationMethod": "none"
      },
      {
        "userVerificationMethod": "passcode_internal",
        "caDesc": {
          "base": 64,
          "minLength": 4,
          "maxRetries": 8,
          "blockSlowdown": 0
        }
      }
    ]
  ],
  "keyProtection": [
    "hardware",
    "secure_element"
  ],
  "matcherProtection": [
    "on_chip"
  ],
  "cryptoStrength": 128,
  "attachmentHint": [
    "external",
    "wired",
    "wireless",
    "nfc"
  ],
  "tcDisplay": [],
  "attestationRootCertificates": [
    "MIIDHjCCAgagAwIBAgIEG0BT9zANBgkqhkiG9w0BAQsFADAuMSwwKgYDVQQDEyNZdWJpY28gVTJGIFJvb3QgQ0EgU2VyaWFsIDQ1NzIwMDYzMTAgFw0xNDA4MDEwMDAwMDBaGA8yMDUwMDkwNDAwMDAwMFowLjEsMCoGA1UEAxMjWXViaWNvIFUyRiBSb290IENBIFNlcmlhbCA0NTcyMDA2MzEwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC/jwYuhBVlqaiYWEMsrWFisgJ+PtM91eSrpI4TK7U53mwCIawSDHy8vUmk5N2KAj9abvT9NP5SMS1hQi3usxoYGonXQgfO6ZXyUA9a+KAkqdFnBnlyugSeCOep8EdZFfsaRFtMjkwz5Gcz2Py4vIYvCdMHPtwaz0bVuzneueIEz6TnQjE63Rdt2zbwnebwTG5ZybeWSwbzy+BJ34ZHcUhPAY89yJQXuE0IzMZFcEBbPNRbWECRKgjq//qT9nmDOFVlSRCt2wiqPSzluwn+v+suQEBsUjTGMEd25tKXXTkNW21wIWbxeSyUoTXwLvGS6xlwQSgNpk2qXYwf8iXg7VWZAgMBAAGjQjBAMB0GA1UdDgQWBBQgIvz0bNGJhjgpToksyKpP9xv9oDAPBgNVHRMECDAGAQH/AgEAMA4GA1UdDwEB/wQEAwIBBjANBgkqhkiG9w0BAQsFAAOCAQEAjvjuOMDSa+JXFCLyBKsycXtBVZsJ4Ue3LbaEsPY4MYN/hIQ5ZM5p7EjfcnMG4CtYkNsfNHc0AhBLdq45rnT87q/6O3vUEtNMafbhU6kthX7Y+9XFN9NpmYxr+ekVY5xOxi8h9JDIgoMP4VB1uS0aunL1IGqrNooL9mmFnL2kLVVee6/VR6C5+KSTCMCWppMuJIZII2v9o4dkoZ8Y7QRjQlLfYzd3qGtKbw7xaF1UsG/5xUb/Btwb2X2g4InpiB/yt/3CpQXpiWX/K4mBvUKiGn05ZsqeY1gx4g0xLBqcU9psmyPzK+Vsgw2jeRQ5JlKDyqE0hebfC1tvFu0CCrJFcw=="
  ],
  "icon": "data:image/png;base64,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",
  "authenticatorGetInfo": {
    "versions": [
      "U2F_V2",
      "FIDO_2_0",
      "FIDO_2_1_PRE"
    ],
    "extensions": [
      "credProtect",
      "hmac-secret"
    ],
    "aaguid": "2fc0579f811347eab116bb5a8db9202a",
    "options": {
      "plat": false,
      "rk": true,
      "clientPin": false,
      "up": true,
      "credentialMgmtPreview": true
    },
    "maxMsgSize": 1200,
    "pinUvAuthProtocols": [
      2,
      1
    ],
    "maxCredentialCountInList": 8,
    "maxCredentialIdLength": 128,
    "transports": [
      "nfc",
      "usb"
    ],
    "algorithms": [
      {
        "type": "public-key",
        "alg": -7
      },
      {
        "type": "public-key",
        "alg": -8
      }
    ],
    "minPINLength": 4,
    "firmwareVersion": 328706
  }
}
```

Note: You can also use the MDS3 Generator Tool to convert or build your Metadata statements.
**TODO -- insert link**
