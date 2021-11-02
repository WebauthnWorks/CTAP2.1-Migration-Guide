# LargeBlobs (0x0C)

LargeBlobs is an authenticator functionality to store large amount of information (1kb and more) in the device memory. It has a simple, byte array style access API with authentication for write operation.

The API always return CBOR BYTE ARRAY. This byte array is always followed by a first 16 bytes of the SHA256 hash of the full data byte array. So an empty LargeBlob storage will always will be 17 bytes long or `8076be8b528d0075f7aae98d6fa57a6d3c`

```
cborarray = 0x80

SHA256(cborarray) => 76be8b528d0075f7aae98d6fa57a6d3c83ae480a8469e668d7b0af968995ac71

LEFT(SHA256(cborarray), 16) => 76be8b528d0075f7aae98d6fa57a6d3c
```

## Setup

If authenticator supports LargeBlobs, the following fields must be specified in GetInfo:

- `GetInfo.maxSerializedLargeBlobArray(0x0B)` - MUST be specified. The device must support at least 1024 bytes of storage, so maxSerializedLargeBlobArray must be always set to 1024 or more.
- `GetInfo.options.largeBlobs` - MUST be set to true.

This command requires support of [PinUvAuthProtocol 2](../Protocol/PinProtocol/2.md).

The platform must obtain PinUvAuthToken with `lbw(0x10)` permission flag.

The specified PinUvAuthToken will be used in all future examples:

```
SessionPUAT = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad
```


## Common

Request keys ENUM: 

```
get               : 0x01 
set               : 0x02
offset            : 0x03 // Mandatory, must always be specified
length            : 0x04
pinUvAuthParam    : 0x05
pinUvAuthProtocol : 0x06
```

Response keys ENUM:

```
config : 0x01
```


LargeBlobs have two main operations: `get`, for getting data, and `set` for writing data.

## Getting data

Consider device storage byte array containing a string PasswordsAreBad, or `50617373776f726473417265426164`

```
  P     a     s     s     w     o     r      d    s     A     r     e     B     a     d
[0x50, 0x61, 0x73, 0x73, 0x77, 0x6f, 0x72, 0x64, 0x73, 0x41, 0x72, 0x65, 0x42, 0x61, 0x64] // 15 bytes.
```

To get the word `Passwords`(9 chars) platform sends LargeBlobs request with `get` set to 9, and offset is set to 0. No authentication is required for `get`.

```
REQ: 0ca201090300

CMD: 0x0c
PAYLOAD: a201090300

DECODED:
{
    1: 9, // Get - 9 bytes
    3: 0
}
```

The result will be presented in response `config(0x01)` field.

```
RESP: 00a1014950617373776f726473

CODE: 0x00 - OK
PAYLOAD a1014950617373776f726473

DECODED:
{
    1: 50617373776F726473 // Config(0x01) - Passwords
}
```

To get word `Bad`(3 chars, 12 offset), the platform would send a LargeBlob request with `get` set to 3, and `offset` set to 12

```
REQ: 0ca20103030C => {1: 3, 3: 12}

RESP: 00a10143426264 => {1: 426264}
```

The data is always followed by a 16 byte checksum(LEFT(16, SHA256)), so the device storage actually contains 31 bytes of information.

```
  P     a     s     s     w     o     r      d    s     A     r     e     B     a     d
[0x50, 0x61, 0x73, 0x73, 0x77, 0x6f, 0x72, 0x64, 0x73, 0x41, 0x72, 0x65, 0x42, 0x61, 0x64, // 15 bytes

// LEFT(16, SHA256("PasswordsaAreBad"))
0x75, 0x99, 0x35, 0x5a, 0xd5, 0xeb, 0x0e, 0x00, 0x44, 0x73, 0xa5, 0xc6, 0x6b, 0xba, 0xca, 0x8d] // 16 bytes
```

To read data together with checksum, then platform may send `get` set to 31, but the platform does not actually know how much data is written to the storage. What instead the platform will do is send `get` set to the `maxSerializedLargeBlobArray(0x0B)` from the `GetInfo` and device will only return available data.


## Writing data

Writing data to LargeBlob is always overwriting all data, so if the platform wishes to extend existing data, it will read existing data, then append new data to the end of it, and then write the full data again.

To write data, the platform will generate request containing `set`, `offset` and `length` fields.

The `set` field contains the new data, and is always postfixed with the checksum, the first 16 bytes of the SHA256 of the new data. This is done to ensure data integrity during the set.

The `length` is set to the length of the new data, and does not include checksum length. The `length` must be excluded if `offset` is larger than 0 (See [#Set Chaining](#set-chaining))

The `offset` field is mandatory. For single `set` request the offset must be 0. For write operations larger than the maximum command size the device can handle [#Set Chaining](#set-chaining).


LargeBlobs require authentication for `set` operations. To compute PinUvAuthParam platform uses the following format:

```
message         = 0xff{32} || 0x0c00 || Uint32LE(offset) || SHA256(data)

pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)
```

Where 0xff{32} is 32 byte set 0xff:

`ffffffffffffffffffffffffffffffff`

Example PinUvAuthParam for setting LargeBlobs storage to `NewSecretData`

```
SessionPUAT     = 0125fecfd8bf3f679bd9ec221324baa74f3cade0314b4fba8029500a320612ad


DataBytes       = "NewSecretData" => 4e657753656372657444617461
DataHash        = SHA256(4e657753656372657444617461) => e4ca763c39666bf79cd48edc453ea50b0aeb41b7ef367da6071b87a99638d55d

pinUvAuthParam  = HMAC-SHA-256(key = puat, message = 0xff{32} || 0x0c00 || 0x00000000 || DataHash)
                => f4c3f6ed54d4cf0f9c3df0eff10351475562bde63724a7ffbbf0012ddf90f8f6
```

**Generating request**
```
REQ: 0ca502581d4e657753656372657444617461e4ca763c39666bf79cd48edc453ea50b0300040d055820f4c3f6ed54d4cf0f9c3df0eff10351475562bde63724a7ffbbf0012ddf90f8f60602

CMD: 0x0c
PAYLOAD: a502581d4e657753656372657444617461e4ca763c39666bf79cd48edc453ea50b0300040d055820f4c3f6ed54d4cf0f9c3df0eff10351475562bde63724a7ffbbf0012ddf90f8f60602

DECODED:
{
    // "NewSecretData" + LEFT(16, SHA256("NewSecretData"))
    2: 4e657753656372657444617461e4ca763c39666bf79cd48edc453ea50b
    3: 0,  // Offset
    4: 13, // Length
    // PinUvAuthParam
    5: f4c3f6ed54d4cf0f9c3df0eff10351475562bde63724a7ffbbf0012ddf90f8f6,
    6: 2   // PinUvAuthProtocol
}
```

On success the authenticator shall return `CTAP_SUCCESS(0x00)`.

## Set Chaining

The LargeBlobs defines new constant `maxFragmentLength` as `GetInfo.maxMsgSize(0x04)` subtract 64. So if device supports maxMsgSize of 256 bytes, then `maxFragmentLength` is 256 - 64 => 192 bytes. So if platform wishes to write more than 192 bytes of data to the LargeBlob, it must chain the `set` operations.

The chaining is achieved by a repeating `set` operation with updated offset, until the total written data is equivalent to the original length.

Consider this:

 - `maxFragmentLength` - 192 bytes
 - new data - 500 bytes (In this example, repeating `0xab`)
 - total frames - ceil(500/192) => 3 or 192, 192 and 116 bytes.


The first command in the chain will define `length` of the entire data and have `offset` of 0, while only supplying 192 bytes of the actual data.

```
REQ: 0ca50258d0ababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababb64e8a3c7c180de347213a12aa10b53b0300041901f40558200c53993354454c606e56361744cd5ac40ed30b305140e7dccef0ff6a213fb7140602

CMD: 0x0c
PAYLOAD: a50258d0ababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababb64e8a3c7c180de347213a12aa10b53b0300041901f40558200c53993354454c606e56361744cd5ac40ed30b305140e7dccef0ff6a213fb7140602

DECODED:
{
    2: abababababababab...b64e8a3c7c180de347213a12aa10b53b // 192 bytes of 0xab followed by LEFT(16, SHA256(0xab{192}))
    3: 0,                  // Offset
    4: 500,                // Length
    5: 0c53993354454c606e56361744cd5ac40ed30b305140e7dccef0ff6a213fb714,
    6: 2                   // PinUvAuthProtocol
}
```

On success the authenticator shall return `CTAP_SUCCESS(0x00)`.

The second command will have `offset` of 192 and excludes `length`

```
REQ: 0ca40258d0ababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababb64e8a3c7c180de347213a12aa10b53b03000558209b1fa7399e6a0c8c3bcfc2036fd792b1752bac39bbed0e12364c262b5f4359d10602

CMD: 0x0c
PAYLOAD: a40258d0ababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababb64e8a3c7c180de347213a12aa10b53b03000558209b1fa7399e6a0c8c3bcfc2036fd792b1752bac39bbed0e12364c262b5f4359d10602

DECODED:
{
    2: abababababababab...b64e8a3c7c180de347213a12aa10b53b // 192 bytes of 0xab followed by LEFT(16, SHA256(0xab{192}))
    3: 192,                  // Offset
    5: 9b1fa7399e6a0c8c3bcfc2036fd792b1752bac39bbed0e12364c262b5f4359d1,
    6: 2                   // PinUvAuthProtocol
}
```

On success the authenticator shall return `CTAP_SUCCESS(0x00)`.

And the last command will be same as previous, with the `offset` of 384, and the remaining bytes.

```
REQ: 0ca4025884abababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababab1c1b826675d96a0f5f71b430ebd967440300055820a5e6cd7ff66d59288a21f0affd2b3e9114a54ddafa48a608f3def725da26a9fd0602

CMD: 0x0c
PAYLOAD: a4025884abababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababab1c1b826675d96a0f5f71b430ebd967440300055820a5e6cd7ff66d59288a21f0affd2b3e9114a54ddafa48a608f3def725da26a9fd0602

DECODED:
{
    2: abababababababab...1c1b826675d96a0f5f71b430ebd96744 // 116 bytes of 0xab followed by LEFT(16, SHA256(0xab{116}))
    3: 384,                  // Offset
    5: a5e6cd7ff66d59288a21f0affd2b3e9114a54ddafa48a608f3def725da26a9fd,
    6: 2                   // PinUvAuthProtocol
}
```

On success the authenticator shall return `CTAP_SUCCESS(0x00)`.
