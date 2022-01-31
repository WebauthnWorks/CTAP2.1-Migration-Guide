# Credential Management 0x0A

The credential management command provides platform an ability to manage discoverable(resident) credentials.

Using this API, the platform is able to:

 - Get information about existing credentials `GetCredsMetadata(0x01)`
 - Find available relying parties `EnumerateRPsBegin(0x02)/EnumerateRPsGetNextRP(0x03)`
 - Find all available credentials registered for the relying part `EnumerateCredentialsBegin(0x04)/EnumerateCredentialsGetNextCredential(0x05)`
 - Update credential information(name/displayName) `UpdateUserInformation(0x07)`
 - Delete credential `DeleteCredential(0x06)`


This command requires support of [PinUvAuthProtocol 2](../PinUvAuthnProtocol2.md).

The platform must obtain PinUvAuthToken with `cm(0x04)` permission flag.

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
getCredsMetadata                      : 0x01
enumerateRPsBegin                     : 0x02
enumerateRPsGetNextRP                 : 0x03
enumerateCredentialsBegin             : 0x04
enumerateCredentialsGetNextCredential : 0x05
deleteCredential                      : 0x06
updateUserInformation                 : 0x07
```

Sub Command Param Keys:

```
rpIDHash     : 0x01
credentialID : 0x02 // PublicKeyCredentialDescriptor
user         : 0x03 // PublicKeyCredentialUserEntity
```

Response keys ENUM:

```
existingResidentCredentialsCount : 0x01
maxPossibleRemainingResidentCredentialsCount : 0x02
rp               : 0x03
rpIDHash         : 0x04
totalRPs         : 0x05
user             : 0x06
credentialID     : 0x07
publicKey        : 0x08
totalCredentials : 0x09
credProtect      : 0x0A
largeBlobKey     : 0x0B
```

## Computing PinUvAuthParam

Credential management specifies the following structure to compute PinUvAuthParam:

```
message         = subCommand || subCommandParams

pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)
```

Example PinUvAuthParam for EnumerateCredentialsBegin(0x04)

```
subCommand       = 0x04 
subCommandParams = {
    0x01: a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce1947 // rpIDHash of example.com
} 
                 = a1015820a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce1947

message = 0x04 || a1015820a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce1947

pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)
                => 340ff740ea8c0daf48c5df402c65735315d296cc1982cd1fa184918bf7407c8b
```

## GetCredsMetadata (0x01)

To obtain information about amount of registered credential, platform send GetCredsMetadata(0x01) sub command:

```
PinUvAuthParam = HMAC-SHA-256(key = puat, message = 0x01)
               => 45676f3835f462adfa04b2066c5ec094c60dd480564326dfdcd4b2c36ac46025
```

**Generating request**
```
REQ: 0aa30101030204582045676f3835f462adfa04b2066c5ec094c40d480564326dfdcd4b2c36ac46025

CMD: 0x0A
PAYLOAD: a30101030204582045676f3835f462adfa04b2066c5ec094c60dd480564326dfdcd4b2c36ac46025

DECODED:
{
    1: 1, // SubCommand - GetCredsMetadata(0x01)
    3: 2, // PinUvAuthProtocol 2
    4: 45676f3835f462adfa04b2066c5ec094c60dd480564326dfdcd4b2c36ac46025 // PinUvAuthParam
}
```

On success the authenticator will return `CTAP_SUCCESS(0x00)` with a response struct containing `existingResidentCredentialsCount(0x01)` and `maxPossibleRemainingResidentCredentialsCount(0x02)`.

```
RESP: 00a20104021819

CODE: 0x00 - OK
PAYLOAD a20104021819

DECODED:
{
    1: 4, // existingResidentCredentialsCount
    2: 25 // maxPossibleRemainingResidentCredentialsCount
}
```

The platform then makes decisions to notify user that credentials memory is low and if user should perform some cleanup.

## Enumerating RPs - EnumerateRPsBegin(0x02) EnumerateRPsGetNextRP(0x03)

To obtain information about available RPs, platform initialises RP enumeration by calling `EnumerateRPsBegin(0x02)` and retrieving total available RPs from `totalRPs(0x05)`. Platform then will continue retrieving following RPs by repeatedly calling `EnumerateRPsGetNextRP(0x03)` and subtracting 1 from the `totalRPs` each request, until `totalRPs` is 0.

> Note: This is a stateful command. During `enumerateRPsGetNextRP(0x03)` and `EnumerateRPsGetNextRP(0x03)` authenticator must keep a track of which RP is following next by using some sort of state(in active transaction) counter to keep a track of enumerated RPs


### EnumerateRPsBegin(0x02)

EnumerateRPsBegin starts enumeration process for the relying parties. The platform generates request with SubCommand, PinUvAuthProtocol and PinUvAuthParam. SubCommandParams are not required.

```
PinUvAuthParam = HMAC-SHA-256(key = puat, message = 0x02)
               => fbfb518d519d9f4138b4997ea57492af7d1e2dbc9cd7d80d318980b1b9e9943b
```

**Generating request**
```
REQ: 0aa301020302045820fbfb518d519d9f4138b4997ea57492af7d1e2dbc9cd7d80d318980b1b9e9943b

CMD: 0x0A
PAYLOAD: a301020302045820fbfb518d519d9f4138b4997ea57492af7d1e2dbc9cd7d80d318980b1b9e9943b

DECODED:
{
    1: 2, // SubCommand - EnumerateRPsBegin(0x02)
    3: 2, // PinUvAuthProtocol 2
    4: fbfb518d519d9f4138b4997ea57492af7d1e2dbc9cd7d80d318980b1b9e9943b // PinUvAuthParam
}
```

On success the authenticator will return `CTAP_SUCCESS(0x00)` with a response struct containing `rp(0x03)`, `rpIDHash(0x04)` and `totalRPs(0x05)`. 

```
RESP: 00a303a26269646b6578616d706c652e636f6d646e616d656f546865204578616d706c6520496e63045820a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce19470504

CODE: 0x00 - OK
PAYLOAD a303a26269646b6578616d706c652e636f6d646e616d656f546865204578616d706c6520496e63045820a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce19470504

DECODED:
{
    3: { // rp - as defined in the WebAuthn API specification 
        "id": "example.com",
        "name": "The Example Inc"
    },
    4: a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce1947, // rpIDHash
    5: 4 // totalRPs
}
```

The platform takes the value of the `totalRPs` and subtracts 1. The remaining value (3 in current example) is the amount of times that platform must call `enumerateRPsGetNextRP(0x03)` in order to obtain all remaining RPs.

### EnumerateRPsGetNextRP(0x03)

EnumerateRPsGetNextRP is called by the platform to obtain next relying party entry. This command must return an error `CTAP2_ERR_NOT_ALLOWED(0x30)` if called prior to the `EnumerateRPsBegin(0x02)`

This command does not require SubCommandParams.

```
PinUvAuthParam = HMAC-SHA-256(key = puat, message = 0x03)
               => 9ff64687fc5faadb01a62c31acc1e06f03a4f96b60db53e8a0ad7a1c15cd4c4c
```

**Generating request**
```
REQ: 0aa3010303020458209ff64687fc5faadb01a62c31acc1e06f03a4f96b60db53e8a0ad7a1c15cd4c4c

CMD: 0x0A
PAYLOAD: a3010303020458209ff64687fc5faadb01a62c31acc1e06f03a4f96b60db53e8a0ad7a1c15cd4c4c

DECODED:
{
    1: 3, // SubCommand - EnumerateRPsGetNextRP(0x03)
    3: 2, // PinUvAuthProtocol 2
    4: 9ff64687fc5faadb01a62c31acc1e06f03a4f96b60db53e8a0ad7a1c15cd4c4c // PinUvAuthParam
}
```

On success the authenticator will return `CTAP_SUCCESS(0x00)` with a response struct containing `rp(0x03)` and `rpIDHash(0x04)`

```
RESP: 00a203a26269646e776562617574686e2e776f726b73646e616d65782150617373776f726473204572616469636174696f6e205370656369616c69737473045820bff672cd24e1015a9da6958f7beccffa1469ac33f82bec37d5cb66675b5840fb

CODE: 0x00 - OK
PAYLOAD a203a26269646e776562617574686e2e776f726b73646e616d65782150617373776f726473204572616469636174696f6e205370656369616c69737473045820bff672cd24e1015a9da6958f7beccffa1469ac33f82bec37d5cb66675b5840fb

DECODED:
{
    3: { // rp - as defined in the WebAuthn API specification 
        "id": "webauthn.works",
        "name": "Passwords Eradication Specialists"
    },
    4: bff672cd24e1015a9da6958f7beccffa1469ac33f82bec37d5cb66675b5840fb, // rpIDHash
}
```

The platform then repeats calling `EnumerateRPsGetNextRP(0x03)` until `totalRPs` counter is 0.



## Enumerating RPs - EnumerateCredentialsBegin(0x04) EnumerateCredentialsGetNextCredential(0x05)

To obtain information about available credentials for RP, platform initialises credential enumeration by calling `EnumerateCredentialsBegin(0x04)` and retrieving number of available credentials from `totalCredentials(0x09)`. Platform then will continue retrieving rest of credentials by repeatedly calling `EnumerateCredentialsGetNextCredential(0x05)` and subtracting 1 from the `totalCredentials` each request, until `totalCredentials` is 0.

> Note: This is a stateful command. During `EnumerateCredentialsBegin(0x03)` and `EnumerateCredentialsGetNextCredential(0x03)` authenticator must keep a track of which credentials are following next by using some sort of state(in active transaction) counter to keep a track of enumerated credentials.


### EnumerateCredentialsBegin(0x04)

EnumerateRPsBegin starts enumeration process for the relying parties. The platform generates request with SubCommand, SubCommandParams, PinUvAuthProtocol and PinUvAuthParam. SubCommandParams containing rpIDHash

```
subCommand       = 0x04 
subCommandParams = {
    1: a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce1947 // rpIDHash of example.com
} 
                 = a1015820a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce1947

message = 0x04 || a1015820a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce1947

pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)
                => 340ff740ea8c0daf48c5df402c65735315d296cc1982cd1fa184918bf7407c8b
```

**Generating request**
```
REQ: 0aa4010402a1015820bff672cd24e1015a9da6958f7beccffa1469ac33f82bec37d5cb66675b5840fb0302045820340ff740ea8c0daf48c5df402c65735315d296cc1982cd1fa184918bf7407c8b

CMD: 0x0A
PAYLOAD: a4010402a1015820bff672cd24e1015a9da6958f7beccffa1469ac33f82bec37d5cb66675b5840fb0302045820340ff740ea8c0daf48c5df402c65735315d296cc1982cd1fa184918bf7407c8b

DECODED:
{
    1: 4, // SubCommand - EnumerateCredentialsBegin(0x04)
    2: {
        1: a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce1947 // rpIDHash of example.com
    },
    3: 2, // PinUvAuthProtocol 2
    4: 340ff740ea8c0daf48c5df402c65735315d296cc1982cd1fa184918bf7407c8b // PinUvAuthParam
}
```

On success the authenticator will return `CTAP_SUCCESS(0x00)` with a response struct containing `user(0x06)`, `credentialID(0x07)`,`publicKey(0x08)`, `totalCredentials(0x09)`, `credProtect(0x0A)` and `largeBlobKey(0x0B)`. 

The `largeBlobKey(0x0B)` is only required when `largeBlobKey` extension was applied to the credential.

```
RESP: 00a606a362696450413caa9fc7679f2ef2b3147e54bb79c2646e616d65736a6f686e646f65406578616d706c652e636f6d6b646973706c61794e616d656c44722e204a6f686e20446f6507a26269645820d3e41ab4de7c536095c65f037afaadbf6db6b62dd42485c4389bfec59b3710dc64747970656a7075626c69632d6b657908a5010203262001215820e87625896ee4e46dc032766e8087962f36df9dfe8b567f3763015b1990a60e1422582027de612d66418bda1950581ebc5c8c1dad710cb14c22f8c97045f4612fb20c9109020a020b58206b370f92e0317f1a3900cf0ccc5de21c9296f19e4eccbe6cd768017b379b8499

CODE: 0x00 - OK
PAYLOAD a606a362696450413caa9fc7679f2ef2b3147e54bb79c2646e616d65736a6f686e646f65406578616d706c652e636f6d6b646973706c61794e616d656c44722e204a6f686e20446f6507a26269645820d3e41ab4de7c536095c65f037afaadbf6db6b62dd42485c4389bfec59b3710dc64747970656a7075626c69632d6b657908a5010203262001215820e87625896ee4e46dc032766e8087962f36df9dfe8b567f3763015b1990a60e1422582027de612d66418bda1950581ebc5c8c1dad710cb14c22f8c97045f4612fb20c9109020a020b58206b370f92e0317f1a3900cf0ccc5de21c9296f19e4eccbe6cd768017b379b8499

DECODED:
{
    6: { // user - as specified in WebAuthn spec
        id: 413caa9fc7679f2ef2b3147e54bb79c2,
        name: "johndoe@example.com",
        displayName: "Dr. John Doe"
    },
    7: { // credentialID - as specified in WebAuthn spec
        type: "public-key",
        id: d3e41ab4de7c536095c65f037afaadbf6db6b62dd42485c4389bfec59b3710dc
    },
    8: { // public key
        1: 2,
        3: -7,
        -1: 1,
        -2: e87625896ee4e46dc032766e8087962f36df9dfe8b567f3763015b1990a60e14,
        -3: 27de612d66418bda1950581ebc5c8c1dad710cb14c22f8c97045f4612fb20c91
    },
    9: 2, // totalCredentials
    10: 2, // credProtect
    11: 6b370f92e0317f1a3900cf0ccc5de21c9296f19e4eccbe6cd768017b379b8499 // largeBlobKey
}
```

The platform takes the value of the `totalCredentials` and subtracts 1. The remaining value (1 in current example) is the amount of times that platform must call `EnumerateCredentialsGetNextCredential(0x05)` in order to obtain all remaining RPs.

### EnumerateCredentialsGetNextCredential(0x05)

EnumerateCredentialsGetNextCredential is called by the platform to obtain next credential entry. This command must return an error `CTAP2_ERR_NOT_ALLOWED(0x30)` is called prior to the `EnumerateCredentialsBegin(0x04)`

This command does not require SubCommandParams.

```
PinUvAuthParam = HMAC-SHA-256(key = puat, message = 0x05)
               => 1efb5c760f1951a8d323244f87d35a5e0e122c5b55caf8a7613def8825c40bf8
```

**Generating request**
```
REQ: 0aa301030302045821efb5c760f1951a8d323244f87d35a5e0e122c5b55caf8a7613def8825c40bf8

CMD: 0x0A
PAYLOAD: a3010303020458201efb5c760f1951a8d323244f87d35a5e0e122c5b55caf8a7613def8825c40bf8

DECODED:
{
    1: 5, // SubCommand - EnumerateRPsGetEnumerateCredentialsGetNextCredentialNextRP(0x05)
    3: 2, // PinUvAuthProtocol 2
    4: 1efb5c760f1951a8d323244f87d35a5e0e122c5b55caf8a7613def8825c40bf8 // PinUvAuthParam
}
```

On success the authenticator will return `CTAP_SUCCESS(0x00)` with a response struct containing `user(0x06)`, `credentialID(0x07)`,`publicKey(0x08)`,`credProtect(0x0A)` and `largeBlobKey(0x0B)`. 

```
RESP: 00a406a3626964506b1657c061496a0c83af257942192699646e616d65706a616e65406578616d706c652e636f6d6b646973706c61794e616d65714d732e204a616e6520546f776e73656e6407a262696458209ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e64747970656a7075626c69632d6b657908a501020326200121582091d8bb2e3e36530b39f5702d98cf2c7718c600b3c0bcf41b8fc6358ea285b8ff2258205842b667a893d6878f2a0e008e376ab6db7a7adcc99818c06cb6d2a554b0b7e30a02

CODE: 0x00 - OK
PAYLOAD a406a3626964506b1657c061496a0c83af257942192699646e616d65706a616e65406578616d706c652e636f6d6b646973706c61794e616d65714d732e204a616e6520546f776e73656e6407a262696458209ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e64747970656a7075626c69632d6b657908a501020326200121582091d8bb2e3e36530b39f5702d98cf2c7718c600b3c0bcf41b8fc6358ea285b8ff2258205842b667a893d6878f2a0e008e376ab6db7a7adcc99818c06cb6d2a554b0b7e30a02

DECODED:
{
    6: { // user - as specified in WebAuthn spec
        id: 6b1657c061496a0c83af257942192699,
        name: "jane@example.com",
        displayName: "Ms. Jane Townsend"
    },
    7: { // credentialID - as specified in WebAuthn spec
        type: "public-key",
        id: 9ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e
    },
    8: { // public key
        1: 2,
        3: -7,
        -1: 1,
        -2: 91d8bb2e3e36530b39f5702d98cf2c7718c600b3c0bcf41b8fc6358ea285b8ff,
        -3: 5842b667a893d6878f2a0e008e376ab6db7a7adcc99818c06cb6d2a554b0b7e3
    },
    10: 2, // credProtect
}
```

The platform then repeats calling `EnumerateCredentialsGetNextCredential(0x05)` until `totalCredentials` counter is 0.

### UpdateUserInformation(0x07)

UpdateUserInformation provides a mechanism to update user name and displayName

Be aware that you can not change `user.id`(aka keyHandle).

To update credential platform sends UpdateUserInformation with SubCommandParams containing `user(0x06)` with updated `name` and `displayName` and original `id`, and `credentialID(0x07)`.

```
subCommand       = 0x07
subCommandParams = {
    6: { // user - as specified in WebAuthn spec
        id: 6b1657c061496a0c83af257942192699,
        name: "jane.doherty@example.com", // New email
        displayName: "Mrs. Jane Doherty" // New name
    },
    7: { // credentialID - as specified in WebAuthn spec
        type: "public-key",
        id: 9ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e
    }
} 
= a206a3626964506b1657c061496a0c83af257942192699646e616d65706a616e65406578616d706c652e636f6d6b646973706c61794e616d65714d732e204a616e6520546f776e73656e6407a262696458209ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e64747970656a7075626c69632d6b6579

message = 0x07 || subCommandParams

pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)
                => 2a6e5225e35576d8d70648275ce4b36fedc9445cdac6a748be742fab281a302a
```

**Generating request**
```
REQ: 0aa4010702a206a3626964506b1657c061496a0c83af257942192699646e616d65706a616e65406578616d706c652e636f6d6b646973706c61794e616d65714d732e204a616e6520546f776e73656e6407a262696458209ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e64747970656a7075626c69632d6b657903020458202a6e5225e35576d8d70648275ce4b36fedc9445cdac6a748be742fab281a302a

CMD: 0x0A
PAYLOAD: a4010702a206a3626964506b1657c061496a0c83af257942192699646e616d65706a616e65406578616d706c652e636f6d6b646973706c61794e616d65714d732e204a616e6520546f776e73656e6407a262696458209ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e64747970656a7075626c69632d6b657903020458202a6e5225e35576d8d70648275ce4b36fedc9445cdac6a748be742fab281a302a

DECODED:
{
    1: 7, // SubCommand - UpdateUserInformation(0x07)
    2: {
        6: { // user - as specified in WebAuthn spec
            id: 6b1657c061496a0c83af257942192699,
            name: "jane.doherty@example.com", // New email
            displayName: "Mrs. Jane Doherty" // New name
        },
        7: { // credentialID - as specified in WebAuthn spec
            type: "public-key",
            id: 9ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e
        }
    } 
    3: 2, // PinUvAuthProtocol 2
    4: 1efb5c760f1951a8d323244f87d35a5e0e122c5b55caf8a7613def8825c40bf8 // PinUvAuthParam
}
```

On success the authenticator will return `CTAP_SUCCESS(0x00)`.

### DeleteCredential(0x06)

DeleteCredential provides a mechanism to remove discoverable credential.

To remove the credential platform sends DeleteCredential with SubCommandParams containing `credentialID(0x07)`.

```
subCommand       = 0x06
subCommandParams = {
    7: { // credentialID - as specified in WebAuthn spec
        type: "public-key",
        id: 9ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e
    }
} 
= a107a262696458209ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e64747970656a7075626c69632d6b6579

message = 0x06 || subCommandParams

pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)
                => 63fa11d623e9a5a639ba98dd288ba770577e45ab7b058757addb088963494338
```

**Generating request**

```
REQ: 0aa4010602a107a262696458209ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e64747970656a7075626c69632d6b6579030204582063fa11d623e9a5a639ba98dd288ba770577e45ab7b058757addb088963494338

CMD: 0x0A
PAYLOAD: a4010602a107a262696458209ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e64747970656a7075626c69632d6b6579030204582063fa11d623e9a5a639ba98dd288ba770577e45ab7b058757addb088963494338

DECODED:
{
    1: 6, // SubCommand - DeleteCredential(0x06)
    2: {
        7: { // credentialID - as specified in WebAuthn spec
            type: "public-key",
            id: 9ae59f309fae32d4d0f999eec320380b8d4045f924673643336a8aa74b2e0c8e
        }
    } 
    3: 2, // PinUvAuthProtocol 2
    4: 63fa11d623e9a5a639ba98dd288ba770577e45ab7b058757addb088963494338 // PinUvAuthParam
}
```

On success the authenticator will return `CTAP_SUCCESS(0x00)`.