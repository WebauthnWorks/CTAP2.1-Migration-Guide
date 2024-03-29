# PinUvAuthProtocol 0x06

The FIDO2 PinProtocol is a session authentication protocol based on the shared static secret overlay. The mains steps that happen during authentication:

- Key negotiation via ECDHA and static secret derivation.
- Session key derived from the shared secret.
- Generate session token for the command using the session key.

## PinProtocol 2 changes

- Encryption and MAC secrets are now different derived using KDF with different info field value. (See below)
- Encryption now uses random IVs (0x00{16} IV in PinProtocol1)
- Added permissions mode, so that you can obtain pinUvAuthToken for specific command


## Common

Request keys ENUM: 

```
pinUvAuthProtocol : 0x01 // Renamed from pinProtocol in CTAP2.1 spec
subCommand        : 0x02
keyAgreement      : 0x03
pinUvAuthParam    : 0x04 // Renamed from pinAuth in CTAP2.1 spec
newPinEnc         : 0x05
pinHashEnc        : 0x06
permissions       : 0x09 // New in CTAP2.1
permissionsRPID   : 0x0A // New in CTAP2.1
```

Response keys ENUM:

```
keyAgreement    : 0x01
pinUvAuthToken  : 0x02
pinRetries      : 0x03
powerCycleState : 0x04
uvRetries       : 0x05
```

Sub Commands ENUM: 

```
getPINRetries   : 0x01
getKeyAgreement : 0x02
setPIN          : 0x03
changePIN       : 0x04
getPINToken     : 0x05
getUVRetries    : 0x07 // New in CTAP2.1
getPinUvAuthTokenUsingUvWithPermissions  : 0x06 // New in CTAP2.1
getPinUvAuthTokenUsingPinWithPermissions : 0x09 // New in CTAP2.1
```

NEW: Permissions flags ENUM:

```
mc   : 0x01 // Make Credential
ga   : 0x02 // Get Assertion
cm   : 0x04 // Credential Management
be   : 0x08 // Biometric Enrollment
lbw  : 0x10 // Large Blob Write
acfg : 0x20 // Authenticator Config
```


Applicable GetInfo fields:

```
pinUvAuthProtocols 0x06 : Array<Int> - Supported pin protocols
minPINLength       0x0D : Int        - specifies current minimum pin length in Unicode points.
```

## Core

### Key Negotiation | Same for PinUvAuthProtocol 1 and 2

Platform sends ClientPin(0x06) with "getKeyAgreement" subcommand:

```
REQ: 06a201020202

CMD: 0x06
PAYLOAD: a201020202
DECODED:
{
    1: 2, // pinUvAuthProtocol - 2
    2: 2  // subCommand - getKeyAgreement(0x02)
}
```

Authenticator receives the request and generates authenticator session ECDH NIST P256 curve keypair - aKeypair

Example authenticator keypair
```
Authenticator Private Key: 90f0a83dba763dfa331061198fc90033a7adefe8fe63da224667b2c5ac0a52ba

Authenticator Public Key ANSI: 04d7404f738990a0af2bbc053d39f0a9c293898bf8e7c0fb9ab1ef4dd61bb70d37c09bed90721b78631be0e68f42997f7a1957b87a6537cfade9f0a68cd432e7f9
```

And send the public key back to the platform in response:

Authenticator response
```
RESP: 00a101a501020338182001215820d7404f738990a0af2bbc053d39f0a9c293898bf8e7c0fb9ab1ef4dd61bb70d37225820c09bed90721b78631be0e68f42997f7a1957b87a6537cfade9f0a68cd432e7f9

CODE: 0x00 - OK
PAYLOAD a101a501020338182001215820d7404f738990a0af2bbc053d39f0a9c293898bf8e7c0fb9ab1ef4dd61bb70d37225820c09bed90721b78631be0e68f42997f7a1957b87a6537cfade9f0a68cd432e7f9

DECODED:
{1: // keyAgreement
    {
        1: 2,   // KTY Key Type - EC2
        3: -25, // ALG - ECDH-ES+HKDF-256(-25)
        -1: 1,  // CRV - P256(1)
        -2: h'D7404F738990A0AF2BBC053D39F0A9C293898BF8E7C0FB9AB1EF4DD61BB70D37', // X
        -3: h'C09BED90721B78631BE0E68F42997F7A1957B87A6537CFADE9F0A68CD432E7F9'  // Y
    }
}
```

Platform generates it's own session keypair, called pKeypair, and using ECDH algorithm derives session shared secret - sK

```
Platform Private Key: e5fccb46de79b04b62e930bbb72b274e30134545f08d54c72bba22407c825344

Platform Public Key ANSI: 04003a0bc15db601a238e891db21ca5f71eab98c8e05bf4e318daf471ea6e76dab3e303d19dcf2147db8d4ee0d932b0a0308ca55d69cf021c1c7e111d0cdc59efe

ECDH-ES256(pKeypair.private, aKeypair.public) -> ee33f2cb08a46d2ddcaf03214af14c31510fcaa4452b8e6e04710fbc66727bac - Session Shared Secret
```

Next we do key derivation
 
## Encryption/MAC keys

Next you need to derive encryption and HMAC keys.

For **PinUvAuthProtocol 1**, session encryption key eKs is the same as session MAC key mKs, as SHA-256 of the sK

```
sK  = ee33f2cb08a46d2ddcaf03214af14c31510fcaa4452b8e6e04710fbc66727bac
eKs = mKs = SHA256(sK) == 8f186f64182b86cfd8bbf73354772066ba46de6514149a7a79c050a3ce4f1ff9
```

For **PinUvAuthProtocol 2**, session encryption key eKS and mKS are derived from sK via HKDF-SHA-256 with a distinct info:

```
sK  = ee33f2cb08a46d2ddcaf03214af14c31510fcaa4452b8e6e04710fbc66727bac
eKs = HKDF-SHA-256(salt = 32 zero bytes, IKM = sK, L = 32, info = "CTAP2 AES key") == 6e1db0c8ebc48dad977a81d26ca7e5bfb0243bd1a2aba6c45adc7a00729b2027
mKs = HKDF-SHA-256(salt = 32 zero bytes, IKM = sK, L = 32, info = "CTAP2 HMAC key") == efe50001d97a7cf14096589e65147b108653683cc22aaf4d1f3bd91dc8e41cbd
```



## Encryption/Decryption - AES

For both protocol 1 and 2, ClientPIN uses AES256 in CBC mode, however there is a difference between IVs.

For **PinUvAuthProtocol 1** IV is 16 byte 0x00. The cipher text buffers are sent without any IV prefix.

```
eKs        = 8f186f64182b86cfd8bbf73354772066ba46de6514149a7a79c050a3ce4f1ff9
message    = 70617373776f72647341726542616400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
ciphertext = AES256_CBC_ENC(key = eKs, iv = 00000000000000000000000000000000, message = message)
    == b206fc6039c9817a045fd274c601f448900b37a9bd2a7fdfe02e60471f684d0d1be5318c1c1d29c02ac2333bb039b0f7e2f9cccb6b4faca97281f95a8805f782
```


Same for decryption

```
eKs        = 8f186f64182b86cfd8bbf73354772066ba46de6514149a7a79c050a3ce4f1ff9
ciphertext = b206fc6039c9817a045fd274c601f448900b37a9bd2a7fdfe02e60471f684d0d1be5318c1c1d29c02ac2333bb039b0f7e2f9cccb6b4faca97281f95a8805f782
message = AES256(key = eKs, iv = 00000000000000000000000000000000, ciphertext = ciphertext)
    == 70617373776f72647341726542616400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
```


For **PinUvAuthProtocol 2** IV is 16 byte long and RANDOMLY selected. The response is a concatenation of 16 byte IV and Ciphertext.

```
eKs           = 6e1db0c8ebc48dad977a81d26ca7e5bfb0243bd1a2aba6c45adc7a00729b2027
message       = 70617373776f72647341726542616400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
randomIV      = URandom(16) -> 3672a1bc264e2cef5f647aff51100b15
rawCiphertext = AES256_CBC_ENC(key = eKs, iv = randomIV, message = message)
    == c2cdbe642fe446d1e7eb6cf52ef5268b616d1d2720c5b0d05195d0ab3b07a248a2f99930060ad3ce58a89824928050a93aa453ca66a8feade0333721dbf1b966
finalCiphertext = 3672a1bc264e2cef5f647aff51100b15c2cdbe642fe446d1e7eb6cf52ef5268b616d1d2720c5b0d05195d0ab3b07a248a2f99930060ad3ce58a89824928050a93aa453ca66a8feade0333721dbf1b966
```

And for decryption you slice off first 16 bytes of the cipher text to get IV and decrypt the rest:

```
eKs           = 6e1db0c8ebc48dad977a81d26ca7e5bfb0243bd1a2aba6c45adc7a00729b2027
ciphertext    = 3672a1bc264e2cef5f647aff51100b15c2cdbe642fe446d1e7eb6cf52ef5268b616d1d2720c5b0d05195d0ab3b07a248a2f99930060ad3ce58a89824928050a93aa453ca66a8feade0333721dbf1b966
iv            = ciphertext[0:16] -> 3672a1bc264e2cef5f647aff51100b15
rawCiphertext = ciphertext[16:] -> c2cdbe642fe446d1e7eb6cf52ef5268b616d1d2720c5b0d05195d0ab3b07a248a2f99930060ad3ce58a89824928050a93aa453ca66a8feade0333721dbf1b966

message = AES256_CBC_ENC(key = eKs, iv = iv, ciphertext = rawCiphertext)
    == 70617373776f72647341726542616400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
```


## HMAC

Both **PinUvAuthProtocol 1** and **PinUvAuthProtocol 2** are using the same HMAC-SHA-256 for computing message MAC. However **PinUvAuthProtocol 1** is returning first 16 bytes of the HMAC output, where **PinUvAuthProtocol 2** is returning the entire HMAC output

**PinUvAuthProtocol 1** MAC is first 16 bytes of the result


```
mKs    = 8f186f64182b86cfd8bbf73354772066ba46de6514149a7a79c050a3ce4f1ff9
message = b206fc6039c9817a045fd274c601f448900b37a9bd2a7fdfe02e60471f684d0d1be5318c1c1d29c02ac2333bb039b0f7e2f9cccb6b4faca97281f95a8805f782

rawMac  = HMAC-SHA-256(key = mKs, message = message)
    == 8d4bb4e328427041bfd1e3dd70b1fffdb4269c0a9d9b814936c18b85373aa877

mac = rawMac[0:16] -> 8d4bb4e328427041bfd1e3dd70b1fffd
```


**PinUvAuthProtocol 2** MAC is full 32 bytes of the result


```
mKs     = efe50001d97a7cf14096589e65147b108653683cc22aaf4d1f3bd91dc8e41cbd
message = 3672a1bc264e2cef5f647aff51100b15c2cdbe642fe446d1e7eb6cf52ef5268b616d1d2720c5b0d05195d0ab3b07a248a2f99930060ad3ce58a89824928050a93aa453ca66a8feade0333721dbf1b966

mac     = HMAC-SHA-256(key = mKs, message = message)
    == 6572fe937c1c04fb68b5a91408f9058246a6650013fe8dc023c5eec130f01a28
```

# **!!!!!All examples further along will use exclusively PinUvAuthProtocol 2 !!!!!**

**TEST VECTORS**

```
sessionEncryptionKey = eKs = 6e1db0c8ebc48dad977a81d26ca7e5bfb0243bd1a2aba6c45adc7a00729b2027

sessionHmacKey = mKs = efe50001d97a7cf14096589e65147b108653683cc22aaf4d1f3bd91dc8e41cbd

keyAgreement = {  // Response KeyAgreement - Platform Keypair 
    1: 2,   // KTY Key Type - EC2
    3: -25, // ALG - ECDH-ES+HKDF-256(-25)
    -1: 1,  // CRV - P256(1)
    -2: h'003A0BC15DB601A238E891DB21CA5F71EAB98C8E05BF4E318DAF471EA6E76DAB', // X
    -3: h'3E303D19DCF2147DB8D4EE0D932B0A0308CA55D69CF021C1C7E111D0CDC59EFE'  // Y
}
```


## Set PIN

1. User selects new PIN,e.g. 123456 or UTF8 encoded `313233343536`
2. The platform checks that give PIN is at least getInfo.minPINLength size or if missing, at least characters long.
3. The platform then takes the PIN, and pads it with 0x00 to get 64 byte buffer. The maximum PIN length is 63 bytes, and so the PIN must always be at least one byte padded.

`31323334353600000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000` - the padded pin buffer.


4. The platform then encrypts the pin using `eKs` as described in the encryption section.

**Generating newPinEnc**

```
pinPaddingBuffer = 31323334353600000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000

AES256-CBC-RANDOM-IV(eKs, pinPaddingBuffer) = 3672a1bc264e2cef5f647aff51100b15c2cdbe642fe446d1e7eb6cf52ef5268b616d1d2720c5b0d05195d0ab3b07a248a2f99930060ad3ce58a89824928050a93aa453ca66a8feade0333721dbf1b966
```

5. Platform then calculates the HMAC of the newPinEnc as described in the hmac section.

**Generating pinUvAuthParam**

```
HMAC-SHA-256(mKs, 3672a1bc264e2cef5f647aff51100b15c2cdbe642fe446d1e7eb6cf52ef5268b616d1d2720c5b0d05195d0ab3b07a248a2f99930060ad3ce58a89824928050a93aa453ca66a8feade0333721dbf1b966) = 6572fe937c1c04fb68b5a91408f9058246a6650013fe8dc023c5eec130f01a28
```

6. Platform then generates AuthenticatorClientPIN(0x06) command with pinUvAuthProtocol, subCommand, keyAgreement, pinEnc and pinUvAuthParam.

**The final pin set command**

```
REQ: 06a50102020303a501020338182001215820003a0bc15db601a238e891db21ca5f71eab98c8e05bf4e318daf471ea6e76dab2258203e303d19dcf2147db8d4ee0d932b0a0308ca55d69cf021c1c7e111d0cdc59efe0458206572fe937c1c04fb68b5a91408f9058246a6650013fe8dc023c5eec130f01a280558503672a1bc264e2cef5f647aff51100b15c2cdbe642fe446d1e7eb6cf52ef5268b616d1d2720c5b0d05195d0ab3b07a248a2f99930060ad3ce58a89824928050a93aa453ca66a8feade0333721dbf1b966

CMD: 0x06
PAYLOAD: a50102020303a501020338182001215820003a0bc15db601a238e891db21ca5f71eab98c8e05bf4e318daf471ea6e76dab2258203e303d19dcf2147db8d4ee0d932b0a0308ca55d69cf021c1c7e111d0cdc59efe0458206572fe937c1c04fb68b5a91408f9058246a6650013fe8dc023c5eec130f01a280558503672a1bc264e2cef5f647aff51100b15c2cdbe642fe446d1e7eb6cf52ef5268b616d1d2720c5b0d05195d0ab3b07a248a2f99930060ad3ce58a89824928050a93aa453ca66a8feade0333721dbf1b966

DECODED:
{
    1: 2, // pinUvAuthProtocol - 2
    2: 3, // subCommand - setPIN(0x03)
    3: {  // Response KeyAgreement - Platform Keypair 
        1: 2,   // KTY Key Type - EC2
        3: -25, // ALG - ECDH-ES+HKDF-256(-25)
        -1: 1,  // CRV - P256(1)
        -2: h'003A0BC15DB601A238E891DB21CA5F71EAB98C8E05BF4E318DAF471EA6E76DAB', // X
        -3: h'3E303D19DCF2147DB8D4EE0D932B0A0308CA55D69CF021C1C7E111D0CDC59EFE'  // Y
    },
    4: 6572fe937c1c04fb68b5a91408f9058246a6650013fe8dc023c5eec130f01a28 // pinUvAuthParam
    5: 3672a1bc264e2cef5f647aff51100b15c2cdbe642fe446d1e7eb6cf52ef5268b616d1d2720c5b0d05195d0ab3b07a248a2f99930060ad3ce58a89824928050a93aa453ca66a8feade0333721dbf1b966 // pinEnc
}
```

7. The authenticator then must return `CTAP_SUCCESS(0x00)`


## Change PIN

Changing pin is similar to setting new pin. They way it works:

1. Platform collects current pin from the user: `123456` or UTF8 encoded `313233343536`, also known as `CurrentPin`
2. Platform collects new pin, `09876` or UTF8 encoded `3039383736`. also known as `NewPin`
3. Platform performs key exchange, and derives shared secret.
4. Platform hashes `CurrentPin` with SHA256, and then encrypts first 16 bytes of the result to get `pinHashEnc`:

```
eKs            = 6e1db0c8ebc48dad977a81d26ca7e5bfb0243bd1a2aba6c45adc7a00729b2027
currentPin     = 313233343536
currentPinHash = SHA256(currentPin) -> 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
leftCurrentPinHash = currentPinHash[0:16] -> 8d969eef6ecad3c29a3a629280e686cf

pinHashEnc     = AES256-CBC-RANDOM-IV(eKs, leftCurrentPinHash) -> 6af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e
```

5. Platform pads `NewPin` with 0x00 to get 64byte array, and encrypts it with the shared secret `eKs` to get `newPinEnc`:

**Generating newPinEnc**

```
newPin = 3039383736
newPinPadded = 30393837360000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000

newPinEnc = AES256-CBC-RANDOM-IV(eKs, newPinPadded) = 4b41b0af9beecf165e9a1f4abc852fe81441cf87ab46a5de55b752c5203498c14e8d0f52be4ec1ac731e1e1fb99c00bf0fc8a768b40a95a7b1f2dd907f3af35f2cc4f3a3aa00b3f5267623e624f33ebc
```

6. The platform then generates HMAC of the concatenation of the pinEnc and newPinEnc to get `pinUvAuthParam`:

**Generating pinUvAuthParam**

```
pinUvAuthInput = newPinEnc || pinHashEnc
 = 4b41b0af9beecf165e9a1f4abc852fe81441cf87ab46a5de55b752c5203498c14e8d0f52be4ec1ac731e1e1fb99c00bf0fc8a768b40a95a7b1f2dd907f3af35f2cc4f3a3aa00b3f5267623e624f33ebc || 6af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e

pinUvAuthParam = HMAC-SHA-256(mKs, pinUvAuthInput) = d6b8d7cf32879f9515ac310a3a99cf6ec025644e921745aba8a34be20faa17e7
```


7. Platform then generates AuthenticatorClientPIN(0x06) command with pinUvAuthProtocol, subCommand, keyAgreement, pinHashEnc, newPinEnc, and pinUvAuthParam.

**The final pin change command**

```
REQ: 06a60102020403a501020338182001215820003a0bc15db601a238e891db21ca5f71eab98c8e05bf4e318daf471ea6e76dab2258203e303d19dcf2147db8d4ee0d932b0a0308ca55d69cf021c1c7e111d0cdc59efe045820d6b8d7cf32879f9515ac310a3a99cf6ec025644e921745aba8a34be20faa17e70558504b41b0af9beecf165e9a1f4abc852fe81441cf87ab46a5de55b752c5203498c14e8d0f52be4ec1ac731e1e1fb99c00bf0fc8a768b40a95a7b1f2dd907f3af35f2cc4f3a3aa00b3f5267623e624f33ebc0658206af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e

CMD: 0x06
PAYLOAD: a60102020403a501020338182001215820003a0bc15db601a238e891db21ca5f71eab98c8e05bf4e318daf471ea6e76dab2258203e303d19dcf2147db8d4ee0d932b0a0308ca55d69cf021c1c7e111d0cdc59efe045820d6b8d7cf32879f9515ac310a3a99cf6ec025644e921745aba8a34be20faa17e70558504b41b0af9beecf165e9a1f4abc852fe81441cf87ab46a5de55b752c5203498c14e8d0f52be4ec1ac731e1e1fb99c00bf0fc8a768b40a95a7b1f2dd907f3af35f2cc4f3a3aa00b3f5267623e624f33ebc0658206af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e

DECODED:
{
    1: 2, // pinUvAuthProtocol - 2
    2: 4, // subCommand - changePin(0x04)
    3: {  // Response KeyAgreement - Platform Keypair 
        1: 2,   // KTY Key Type - EC2
        3: -25, // ALG - ECDH-ES+HKDF-256(-25)
        -1: 1,  // CRV - P256(1)
        -2: h'003A0BC15DB601A238E891DB21CA5F71EAB98C8E05BF4E318DAF471EA6E76DAB', // X
        -3: h'3E303D19DCF2147DB8D4EE0D932B0A0308CA55D69CF021C1C7E111D0CDC59EFE'  // Y
    },
    4: d6b8d7cf32879f9515ac310a3a99cf6ec025644e921745aba8a34be20faa17e7, // pinUvAuthParam
    5: 4b41b0af9beecf165e9a1f4abc852fe81441cf87ab46a5de55b752c5203498c14e8d0f52be4ec1ac731e1e1fb99c00bf0fc8a768b40a95a7b1f2dd907f3af35f2cc4f3a3aa00b3f5267623e624f33ebc, // newPinEnc
    6: 6af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e // pinHashEnc 
}
```

8. The authenticator then must return `CTAP_SUCCESS(0x00)`

## Get PinUvAuthToken - PUAT (Default. MakeCredential and GetAssertion permission)

1. Platform collects user PIN: `123456` or UTF8 encoded `313233343536`, also known as `CurrentPin`
2. Platform performs key exchange, and derives shared secret.
3. Platform hashes `CurrentPin` with SHA256, and then encrypts first 16 bytes of the result to get `pinHashEnc`:

```
eKs            = 6e1db0c8ebc48dad977a81d26ca7e5bfb0243bd1a2aba6c45adc7a00729b2027
currentPin     = 313233343536
currentPinHash = SHA256(currentPin) -> 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
leftCurrentPinHash = currentPinHash[0:16] -> 8d969eef6ecad3c29a3a629280e686cf

pinHashEnc     = AES256-CBC-RANDOM-IV(eKs, leftCurrentPinHash) -> 6af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e
```

4: Platform then generates AuthenticatorClientPIN(0x06) command with pinUvAuthProtocol, subCommand, keyAgreement and pinHashEnc

```
REQ: 06a40102020503a501020338182001215820003a0bc15db601a238e891db21ca5f71eab98c8e05bf4e318daf471ea6e76dab2258203e303d19dcf2147db8d4ee0d932b0a0308ca55d69cf021c1c7e111d0cdc59efe0658206af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e

CMD: 0x06
PAYLOAD: a40102020503a501020338182001215820003a0bc15db601a238e891db21ca5f71eab98c8e05bf4e318daf471ea6e76dab2258203e303d19dcf2147db8d4ee0d932b0a0308ca55d69cf021c1c7e111d0cdc59efe0658206af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e

DECODED:
{
    1: 2, // pinUvAuthProtocol - 2
    2: 5, // subCommand - getPinToken(0x05)
    3: {  // Response KeyAgreement - Platform Keypair 
        1: 2,   // KTY Key Type - EC2
        3: -25, // ALG - ECDH-ES+HKDF-256(-25)
        -1: 1,  // CRV - P256(1)
        -2: h'003A0BC15DB601A238E891DB21CA5F71EAB98C8E05BF4E318DAF471EA6E76DAB', // X
        -3: h'3E303D19DCF2147DB8D4EE0D932B0A0308CA55D69CF021C1C7E111D0CDC59EFE'  // Y
    },
    6: 6af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e // pinHashEnc 
}
```

5. The authenticator processes the response, decrypts encrypted hash of the pin, and validates it against the pin stored in it's db.

6. The authenticator then generates random session access token, and encrypts it using session encryption key `eKs`:

```
sessionPuat = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad
encryptedPuat = AES256-CBC-RANDOM-IV(eKs, sessionPuat) -> 387d215a2ff31e31b0520c7ea5ed2e07777861b34f890b385746cc8278d7bee0626c2007ca2fb6f9c89ca60725f97cb6

AuthenticatorResponse = 00a1025830387d215a2ff31e31b0520c7ea5ed2e07777861b34f890b385746cc8278d7bee0626c2007ca2fb6f9c89ca60725f97cb6

CODE: 0x00 // SUCESS
PAYLOAD: a1025830387d215a2ff31e31b0520c7ea5ed2e07777861b34f890b385746cc8278d7bee0626c2007ca2fb6f9c89ca60725f97cb6

DECODED:
{
    2: 387d215a2ff31e31b0520c7ea5ed2e07777861b34f890b385746cc8278d7bee0626c2007ca2fb6f9c89ca60725f97cb6 //pinUvAuthToken
}
```

7. The platform then decrypts pinUvAuthToken to obtain session token


## Get PinUvAuthToken with Permissions

`PinUvAuthProtocol 2` introduced permissions. Permission are bit flats that specify what kind of actions can specified PinUvAuthToken be used for. For example when platform would like to use PinUvAuthToken for `BiometricEnroll` it needs to obtain PinUvAuthToken with permission flag BE(0x08) set to 1.

Available permissions

```
mc   : 0x01 // Make Credential
ga   : 0x02 // Get Assertion
cm   : 0x04 // Credential Management
be   : 0x08 // Biometric Enrollment
lbw  : 0x10 // Large Blob Write
acfg : 0x20 // Authenticator Config
```

1. Platform collects user PIN: `123456` or UTF8 encoded `313233343536`, also known as `CurrentPin`
2. Platform performs key exchange, and derives shared secret.
3. Platform selects the necessary permissions, in current example biometricEnrollment `be(0x08)` and largeBlob `lbw(0x10)` to get permissions value to `0x18`
3. Platform hashes `CurrentPin` with SHA256, and then encrypts first 16 bytes of the result to get `pinHashEnc`:

```
eKs            = 6e1db0c8ebc48dad977a81d26ca7e5bfb0243bd1a2aba6c45adc7a00729b2027
currentPin     = 313233343536
currentPinHash = SHA256(currentPin) -> 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
leftCurrentPinHash = currentPinHash[0:16] -> 8d969eef6ecad3c29a3a629280e686cf

pinHashEnc     = AES256-CBC-RANDOM-IV(eKs, leftCurrentPinHash) -> 6af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e
```

4: Platform then generates AuthenticatorClientPIN(0x06) command with pinUvAuthProtocol, subCommand, keyAgreement, pinHashEnc, and permissions:

subCommand set to `getPinUvAuthTokenUsingPinWithPermissions(0x09)`

```
REQ: 06a50102020903a501020338182001215820003a0bc15db601a238e891db21ca5f71eab98c8e05bf4e318daf471ea6e76dab2258203e303d19dcf2147db8d4ee0d932b0a0308ca55d69cf021c1c7e111d0cdc59efe0658206af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e091818

CMD: 0x06
PAYLOAD: a50102020903a501020338182001215820003a0bc15db601a238e891db21ca5f71eab98c8e05bf4e318daf471ea6e76dab2258203e303d19dcf2147db8d4ee0d932b0a0308ca55d69cf021c1c7e111d0cdc59efe0658206af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e091818

DECODED:
{
    1: 2, // pinUvAuthProtocol - 2
    2: 9, // subCommand - getPinUvAuthTokenUsingPinWithPermissions(0x09)
    3: {  // Response KeyAgreement - Platform Keypair 
        1: 2,   // KTY Key Type - EC2
        3: -25, // ALG - ECDH-ES+HKDF-256(-25)
        -1: 1,  // CRV - P256(1)
        -2: h'003A0BC15DB601A238E891DB21CA5F71EAB98C8E05BF4E318DAF471EA6E76DAB', // X
        -3: h'3E303D19DCF2147DB8D4EE0D932B0A0308CA55D69CF021C1C7E111D0CDC59EFE'  // Y
    },
    6: 6af8b75c50296805d34fc6326956ca20708f38d17a782ffdd1a537d221eecb3e, // pinHashEnc 
    9: 0x18 // biometricEnrollment `be(0x08)` and largeBlobs `lbw(0x10)`
}
```

The the result PinUVAuthToken will be usable with the `BiometricEnrollment` and `LargeBlobs`


### Other

#### Retries

In PinUvAuthProtocol 2 retries are no longer fixed to the 8. Authenticators now may now set retries value to 8 OR lower. Platform may get the max retries by calling ClientPin with GetRetries(0x01) subcommand


### Handling errors

All FIDO2 authenticators must handle requests accordingly. They must do strict type checking, and ensure that all fields are conform to the specs. Here is the diagram that describes error handling in PinUvAuthProtocol:



