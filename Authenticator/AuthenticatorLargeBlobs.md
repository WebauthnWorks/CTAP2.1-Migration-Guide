# AuthenticatorLargeBlobs (0x0C)

This command is **new** to CTAP 2.1. The credBlob extension allows for a small amount of additional, secret information to be stored with a credential. Useful items to store could include IDs, encrypted keys or certificates for example.

Authenticators need to allow at least 1024 bytes of storage.

- [Feature Detection](feature-detection)
- [Reading and writing serialised data](Reading-and-writing-serialised-data)
- [Large, per-credential blobs](Large-per-credential-blobs)
- [Reading per-credential large-blob data](Reading-per-credential-large-blob-data)
- [Writing per-credential large-blob data for a new credential](Writing-per-credential-large-blob-data-for-a-new-credential)
- [Updating per-credential large-blob data](Updating-per-credential-large-blob-data)
- [Garbage collection of large-blob data](Garbage-collection-of-large-blob-data)

## Feature Detection
The largeBlobs option ID in the [authenticatorGetInfo response](authenticatorGetInfo.md) defines feature support detection for this feature.

## Reading and writing serialised data
```


+--------------------------+------------------+-----------+-----------------------------------------------------------------------------------------------------------------------+
|      Parameter name      |    Data type     | Required? | Notes                                                                                                                 |
+--------------------------+------------------+-----------+-----------------------------------------------------------------------------------------------------------------------+
| get (0x01)               | Unsigned integer | Optional  | The number of bytes requested to read. MUST NOT be present if set is present.                                         |
| set (0x02)               | Byte String      | Optional  | A fragment to write. MUST NOT be present if get is present.                                                           |
| offset (0x03)            | Unsigned integer | Required  | The byte offset at which to read/write.                                                                               |
| length (0x04)            | Unsigned integer | Optional  | The total length of a write operation. Present if, and only if, set is present and offset is zero.                    |
| pinUvAuthParam (0x05)    | Byte String      | Optional  | authenticate(pinUvAuthToken, 32×0xff || h’0c00' || uint32LittleEndian(offset) || SHA-256(contents of set byte string, |
|                          |                  |           | SHA-256(contents of set byte string, i.e. not including an outer CBOR tag with major type 2))                         |
| pinUvAuthProtocol (0x06) | Unsigned integer | Optional  | PIN/UV protocol version chosen by the platform.                                                                       |
+--------------------------+------------------+-----------+-----------------------------------------------------------------------------------------------------------------------+
```

An authenticator performs the following actions upon receipt of this command:

 1. If offset is not present in the input map **OR** if neither get nor set are present in the input map **OR** if both get and set are present in the input map, return **CTAP1_ERR_INVALID_PARAMETER**.
 
 2. If **get** is present in the input map
   - If length present -> return **CTAP1_ERR_INVALID_PARAMETER**
   - If either of pinUvAuthParam or pinUvAuthProtocol are present -> **CTAP1_ERR_INVALID_PARAMETER**.
   - If the value of ```get``` > ```maxFragmentLength```, return **CTAP1_ERR_INVALID_LENGTH**
   - If value of ```offset``` > length of the stored serialized large-blob array, -> **CTAP1_ERR_INVALID_PARAMETER**

## Large per-credential blobs

## Reading per-credential large-blob data

## Writing per-credential large-blob data for a new credential

## Updating per-credential large-blob data

## Garbage collection of large-blob data

notes WIP
https://www.youtube.com/watch?v=g_eY7JXOc8U&t=715s

We define a large blob key. We will use a randomly generated value as the test vector. It can also be derived from a different key.

LargeBlobKey: 1ba29187dcf75af33734357f9a71670a0244d2b714216f423099a2dc8da0727c

We can store a large-blob map in a large-blob array. 
It must contain the ciphertext (0x01), nonce (0x02) and origSize (0x03) -> INPUT map (large blob map) to byte string for value of set (0x02)  ??

First we will define the data we want to store. This will be relying part specific nature. Here is an example

user = {
	id: 32x0xFF, // using 32 zero bytes as test vector
	icon: 'https://imgur.com/bond007,
	username: 'SuaveBond',
	displayName: 'James Bond'
}
ENC: a46269645820ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff6469636f6e781968747470733a2f2f696d6775722e636f6d2f626f6e6430303768757365726e616d65695375617665426f6e646b646973706c61794e616d656a4a616d657320426f6e64

We will get the cipher text:
```
CipherText = AEAD_AES_256_GCM(Nonce, Plaintext, Associated Data, Key)
```

We will use the blow nonce as a test vector
```
nonce = 6961d6e6f6e6365 // "iamnonce"
```
Plaintext will be encoded user. 
Next we want defined associated data.
```
user length (bytes) = 371
associatedData = 0x626c6f62 || uint64LittleEndian[size(originalData=user)] // ????
		= 0x626c6f62 || 371 ???

 = 00626c6f6240


key = largeBlobKey
    = 1ba29187dcf75af33734357f9a71670a0244d2b714216f423099a2dc8da0727c	


Now 
CipherText = 
AES(largeBlobKey, plain text, 
0ec89cdfb04059a2a234e166ef52e2bb00ff6d568399a9c4eff8c4766ba8cc5e9bf809b263dd5cccb4940cb61a3c39b5ea504e3ccb202e0145f9a7ee494b57c8ecdb6523d86c3bc85444d52ba4013127db46cf65296daf89715e2af3e2e12bb1625929aa5c8b2e925de2e591e9137991

https://www.youtube.com/watch?v=g_eY7JXOc8U&t=715s

