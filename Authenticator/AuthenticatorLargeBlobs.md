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

6.10.3 -- Encrypting and encoding the data to conform to large-blob map structure for storage in large-blob array. The platform can write to the authenticator using the below large-blob element while preserving the other elements, using the set subcommand.

# WIP -- probably wrong 
 
https://www.youtube.com/watch?v=g_eY7JXOc8U&t=715s

We define a large blob key. We will use a randomly generated value as the test vector. It can also be derived from a different key.

LargeBlobKey: 1ba29187dcf75af33734357f9a71670a0244d2b714216f423099a2dc8da0727c

We can store a large-blob map in a large-blob array. 
It must contain the ciphertext (0x01), nonce (0x02) and origSize (0x03) -> INPUT map (large blob map) to byte string for value of set (0x02)  ??

First we will define the data we want to store. This will be relying part specific nature. Here is an example

```
user = {
	id: 32x0xFF, // using 32 zero bytes as test vector
	icon: 'https://imgur.com/bond007,
	username: 'SuaveBond',
	displayName: 'James Bond'
}
ENC: a46269645820ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff6469636f6e781968747470733a2f2f696d6775722e636f6d2f626f6e6430303768757365726e616d65695375617665426f6e646b646973706c61794e616d656a4a616d657320426f6e64
```

We will get the cipher text:
```
CipherText = AEAD_AES_256_GCM using nonce, plaintext, associated data and key.
```

We will use the below nonce as a test vector (must be exactly 12 bytes long.)
```
nonce = 000000000000000000000000
```
Plaintext will be encoded user. 
Next we want defined associated data.
```
user length (bytes) = 371
associatedData = 0x626c6f62 || uint64LittleEndian[size(originalData=user)] // ????
		= 0x626c6f62 || 371 ???

 = 00626c6f62371 ?
```
Running the AES-256-GCM algorithm with encryption key, plaintext and nonce, outputting the ciphertext WITHOUT the authentication tag.
```
enc_key = largeBlobKey
    = 1ba29187dcf75af33734357f9a71670a0244d2b714216f423099a2dc8da0727c	
 AES-256-GCM(enc_key, plaintext, nonce)
	ciphertext = AAAAAAAAAAAAAAAAv7qfoUFmEh/kkaMMD6e2K/Z6UnMhsAKRNnld6RXxqLZ+fmp9ynyVnG2JyBhTKNTj8Sib5/087jWESkRYRYVxXy1bX4nF5hnSXNVgNjHO9ndRy6/VHvNOFBTh0kCbtGzOCXMvtUbOaAQWSifhKy6vR5L8HNNB+mJII7OUBK5+NqKPxEOJmcRrKqMrR/GjQaBtURsdPSJIpMb1YsMbT+q4ADDnOv3OejWr04rU7S5XSpTJxZDea+jWY3/Tl9rCZ/+eim0+31g/5ENOzKFzKXTjgs2PgpEeCyqElUmaN6G8TbU=
```

Prepending the associated data
```
S = associatedData + ciphertext 
= 00626c6f62371AAAAAAAAAAAAAAAAv7qfoUFmEh/kkaMMD6e2K/Z6UnMhsAKRNnld6RXxqLZ+fmp9ynyVnG2JyBhTKNTj8Sib5/087jWESkRYRYVxXy1bX4nF5hnSXNVgNjHO9ndRy6/VHvNOFBTh0kCbtGzOCXMvtUbOaAQWSifhKy6vR5L8HNNB+mJII7OUBK5+NqKPxEOJmcRrKqMrR/GjQaBtURsdPSJIpMb1YsMbT+q4ADDnOv3OejWr04rU7S5XSpTJxZDea+jWY3/Tl9rCZ/+eim0+31g/5ENOzKFzKXTjgs2PgpEeCyqElUmaN6G8TbU=
```

Getting the authentication tag of the string, S
```
T = HMAC-SHA-256(key, S)
 = 87b944c4b4d835dc5f16c46cc0f1e03bf14963d26f9d24426bb0bf2f283d5f8a
 ```
 
Concatenating the tag to the end of the ciphertext
```
ciphertext = S + T
  = 00626c6f62371AAAAAAAAAAAAAAAAv7qfoUFmEh/kkaMMD6e2K/Z6UnMhsAKRNnld6RXxqLZ+fmp9ynyVnG2JyBhTKNTj8Sib5/087jWESkRYRYVxXy1bX4nF5hnSXNVgNjHO9ndRy6/VHvNOFBTh0kCbtGzOCXMvtUbOaAQWSifhKy6vR5L8HNNB+mJII7OUBK5+NqKPxEOJmcRrKqMrR/GjQaBtURsdPSJIpMb1YsMbT+q4ADDnOv3OejWr04rU7S5XSpTJxZDea+jWY3/Tl9rCZ/+eim0+31g/5ENOzKFzKXTjgs2PgpEeCyqElUmaN6G8TbU=87b944c4b4d835dc5f16c46cc0f1e03bf14963d26f9d24426bb0bf2f283d5f8a
 ```
 
 Fulfilling the large-blob map structure, we have
 ```
 map = {
 	0x01: 00626c6f62371AAAAAAAAAAAAAAAAv7qfoUFmEh/kkaMMD6e2K/Z6UnMhsAKRNnld6RXxqLZ+fmp9ynyVnG2JyBhTKNTj8Sib5/087jWESkRYRYVxXy1bX4nF5hnSXNVgNjHO9ndRy6/VHvNOFBTh0kCbtGzOCXMvtUbOaAQWSifhKy6vR5L8HNNB+mJII7OUBK5+NqKPxEOJmcRrKqMrR/GjQaBtURsdPSJIpMb1YsMbT+q4ADDnOv3OejWr04rU7S5XSpTJxZDea+jWY3/Tl9rCZ/+eim0+31g/5ENOzKFzKXTjgs2PgpEeCyqElUmaN6G8TbU=87b944c4b4d835dc5f16c46cc0f1e03bf14963d26f9d24426bb0bf2f283d5f8a,
	0x02: 000000000000000000000000,
	0x03: 371
}

ENC = a3017901893030363236633666363233373141414141414141414141414141414141763771666f55466d45682f6b6b614d4d443665324b2f5a36556e4d6873414b524e6e6c6436525878714c5a2b666d7039796e79566e47324a794268544b4e546a38536962352f3038376a5745536b5259525956785879316258346e4635686e53584e56674e6a484f396e645279362f5648764e4f46425468306b436274477a4f43584d767455624f61415157536966684b79367652354c38484e4e422b6d4a4949374f55424b352b4e714b5078454f4a6d6352724b714d72522f476a516142745552736450534a49704d623159734d62542b71344144446e4f76334f656a577230347255375335585370544a785a4465612b6a5759332f546c3972435a2f2b65696d302b3331672f35454e4f7a4b467a4b58546a6773325067704565437971456c556d614e3647385462553d3837623934346334623464383335646335663136633436636330663165303362663134393633643236663964323434323662623062663266323833643566386102781830303030303030303030303030303030303030303030303003190173
```




