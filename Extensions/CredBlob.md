# Extension: CredBlob

CredBlob extension allows relying party to store opaque data with the credential, and retrieve with GetAssertion result. Authenticator that supports credBlob must support at least 32 bytes of storage.

## Feature detection

Authenticator advertises support for this extension by returning `credBlob` in the `GetInfo.extensions(0x02)`.

If authenticator supports `credBlob` it must return `GetInfo.maxCredBlobLength(0x0f)` field with the supported length, that is again, can not be less that 32.

## Set - MakeCredential

To set `credBlob` data the Platform will send MakeCredential with extensions map containing `credBlob` set to byte string containing the data specified by the relying party.

```
MakeCredential_Request
{
    1: 4886ec48727e...,    // clientDataHash
    2: {...},              // rp
    3: {...},              // user
    4: [...],              // pubKeyCredParams
    5: [...],              // excludeList
    6: {                   // extensions
        "credBlob": 57686f4e65656473436f6f6b6965733f
    },
    8: 51e971f093509e4..., // pinUvAuthParam
    9: 2                   // pinUvAuthProtocol
}
```

Authenticator will respond with MakeCredential result authData extensions map containing `credBlob` set to `true`

```
MakeCredential_Result
{
    1: "packed",             // fmt
    2: 4886ec48727eee8fc7... // authData
    3: {...},                // attStmt
}
```

The complete AuthData:

`4886ec48727eee8fc767c92aaafdb8ab54a3225d0f15a549d455ba80132ee92cc5000000011eccdc89946362bbd691ace84c3ac2c70040c4fe0b55f84024f73ac45cecb07df1b57f36b21b45ec42d354a4ddc81467ab88a6fef3feff3238757a352e90a12545c42eb7de439a4aaf055e34dbfa78781b39a5010203262001215820878e2a88b54c088461d0ac6fc5c8027707f4aa3d12e1a45c4ba0002232d665742258207de7e0b3b640e219f3e57dc24baede51680c012673a702875726e8861a692860a16863726564426c6f62f5`


Decoded

```
// clientDataHash
4886ec48727eee8fc767c92aaafdb8ab54a3225d0f15a549d455ba80132ee92c

// flags UP + UV + AT(attested data) + ED(extension data)
c5

// counter - 1
00000001

// AAGUID
1eccdc89946362bbd691ace84c3ac2c7

// credID Length - 64
0040

// credID
c4fe0b55f84024f73ac45cecb07df1b57f36b21b45ec42d354a4ddc81467ab88a6fef3feff3238757a352e90a12545c42eb7de439a4aaf055e34dbfa78781b39

// cose public key
a5010203262001215820878e2a88b54c088461d0ac6fc5c8027707f4aa3d12e1a45c4ba0002232d665742258207de7e0b3b640e219f3e57dc24baede51680c012673a702875726e8861a692860

// extension map - {"credBlob": true}
a16863726564426c6f62f5
```

## Get/GetAssertion

To retrieve `credBlob` platform will send request GetAssertion request with request extensions map containing `credBlob` key set to `true`.

```
GetAssertion_Request
{
    1: "webauthn.works",   // rpid
    1: 39fcc878ea01...,    // clientDataHash
    2: [...],              // allowList
    4: {                   // extensions
        "credBlob": true
    },
    6: 5544d1bf5f49563..., // pinUvAuthParam
    7: 2                   // pinUvAuthProtocol
}
```

The the authenticator will respond with GetAssertion authData extension map containing, `credBlob` set to the previously set byte array.

```
GetAssertion_Result
{
    1: {...},                // credential
    2: 824fc6f45e0d2ee82f... // authData
    3: 2a7eebb66f64b46f13... // signature
}
```

Complete AuthData


```
2e11fc029bdca4b577ab2b27c0219876176374373ca8e76f650faceb029afec58500000002a16863726564426c6f625057686f4e65656473436f6f6b6965733f

Decoded:

// clientDataHash
2e11fc029bdca4b577ab2b27c0219876176374373ca8e76f650faceb029afec5

// flags - UP + UV + ED
85

// counter - 2 
00000002

// extension containing credBlob - {"credBlob": 57686F4E65656473436F6F6B6965733F}
a16863726564426c6f625057686f4e65656473436f6f6b6965733f 
```
